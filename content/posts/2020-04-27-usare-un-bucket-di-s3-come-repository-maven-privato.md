---
title: Usare un bucket di S3 come repository Maven privato
author: Raibaz
type: post
date: 2020-04-27T14:00:59+00:00
url: /2020/04/usare-un-bucket-di-s3-come-repository-maven-privato/
categories:
  - Nerd

---
Un problema tipico nella fase in cui si cerca di spacchettare un monolite in un ecosistema di microservizi specializzati è la gestione delle parti di codice comuni: nel mio caso, ad esempio, mi sono ritrovato ad avere cose come la configurazione di [Datadog][1] (sempre solo cuori per Datadog ❤️), o quella dell&#8217;SMTP per inviare le mail, copiate e incollate in una decina di progetti diversi.

Non credo ci sia bisogno che stia a dilungarmi sul motivo per cui questo è il male: penso basti dire che alla seconda volta che ho dovuto perdere un pomeriggio per riportare una modifica su ogni progetto ho deciso di trovare una soluzione più pratica.

Iniziamo col dire che non è un problema solo mio e che la mia situazione non è necessariamente la migliore per tutti i casi: diverse organizzazioni, più grandi e più piccole della mia, hanno affrontato la cosa in modi diversi, dal [monorepo][2] di Google all&#8217;installazione (e manutenzione) in-house di applicazioni non banali come [Artifactory][3] o [Nexus][4].

La mia soluzione, invece, è molto più semplice e terra terra, ma fa assolutamente al caso mio: ok, voglio estrarre tutte le classi comuni a tutte le applicazioni in una libreria da includere come dipendenza, ma dove la metto?

Una opzione che avevo considerato era dipendere direttamente da un repository Bitbucket, ma l&#8217;ho scartata perché mi pareva complicato (col senno di poi, non lo è) da gestire dentro un build.gradle e per dipendere un po&#8217; meno da Bitbucket, che ha incident un po&#8217; troppo spesso per i miei gusti.

La scelta definitiva, quindi, come avrete intuito dal titolo, è stata usare un bucket S3 come repository, e devo dire che è stato sorprendentemente facile.

La parte legata strettamente a S3 è piuttosto straightforward: si tratta, fondamentalmente, di creare un bucket nuovo, ovviamente privato, e due coppie di chiavi, una con permessi di lettura che sarà condivisa tra tutti gli sviluppatori e tutte le applicazioni dell&#8217;ecosistema, e una con permessi di scrittura che sarà usata solo dalla (dalle) libreria comune in questione per pubblicarsi sul repository.

Poi, siccome non ci piace tenere le credenziali salvate nei repo, dai build.gradle le recupereremo così:

<div style="height: 250px; position:relative; margin-bottom: 50px;" class="wp-block-simple-code-block-ace">
  <pre class="wp-block-simple-code-block-ace" style="position:absolute;top:0;right:0;bottom:0;left:0" data-mode="php" data-theme="monokai" data-fontsize="14" data-lines="Infinity" data-showlines="true" data-copy="false">allprojects {
    awsAccessKeyId = System.getenv("aws_access_key_id") ?: findProperty("aws_access_key_id") ?: ""
    awsSecretAccessKey = System.getenv("aws_secret_access_key") ?: findProperty("aws_secret_access_key") ?: ""
}</pre>
</div>

Questo, oltre a essere meglio che avere le credenziali hardcoded nel build.gradle, ha anche il vantaggio che permette di mettere la coppia chiave-secret nel gradle.properties globale e dimenticarsene forever and ever, senza dover copiarle in ogni progetto che è un po&#8217; il razionale di tutto ciò.

Fatto questo, dobbiamo dire a gradle che esiste il nostro repository, e gradle, fortunatamente, supporta proprio i bucket S3, tra le altre cose, per cui sarà sufficiente questo:

<div style="height: 250px; position:relative; margin-bottom: 50px;" class="wp-block-simple-code-block-ace">
  <pre class="wp-block-simple-code-block-ace" style="position:absolute;top:0;right:0;bottom:0;left:0" data-mode="php" data-theme="monokai" data-fontsize="14" data-lines="Infinity" data-showlines="true" data-copy="false">repositories {
        mavenCentral()

        maven {
            url = "s3://&lt;il nome del vostro bucket>"
            credentials(AwsCredentials) {
                accessKey = awsAccessKeyId
                secretKey = awsSecretAccessKey
            }
        }
    }</pre>
</div>

Ovviamente [mavenCentral][5] lo teniamo come prioritario, ma in seconda battuta diciamo a gradle che le cose che non trova lì, come la nostra libreria comune, le cerca su quest&#8217;altro nuovo repository.

A questo punto possiamo includere la nostra nuova libreria come dipendenza e iniziare a spostarci quello che vogliamo, comprese anche delle altre dipendenze: nel mio caso, tutti i progetti dipendono da `io.micrometer:micrometer-registry-datadog` e non volevo dover copiare la dipendenza in ognuno, per cui ho tagliato la riga e l&#8217;ho incollata nel build.gradle della mia libreria.

Facile? Sì. Funziona? No.

Non funziona perché lo scope che si usa di solito per le dipendenze in Gradle, cioè `compile` (o `implementation` se volete essere compliant con Gradle 7 e non prendervi un warning), non rende la dipendenza transitiva, cioè non fa sì che venga esportata anche come dipendenza per i progetti che da questa dipendono.

Lo scope giusto, e questo è un &#8220;gotcha&#8221; che mi ha richiesto un paio d&#8217;ore di mal di testa, è `api`: se specifico la dipendenza così, tutte le applicazioni che dipendono dalla mia libreria &#8220;vedono&#8221; anche `io.micrometer:micrometer-registry-datadog`.

&#8220;Sì ok&#8221;, mi direte ora, &#8220;ma come la pubblichi la libreria su S3?&#8221;

Anche questa è una cosa sorprendentemente facile: c&#8217;è un plugin di Gradle endorsed, che si chiama per l&#8217;appunto [maven-publish][6], che fa esattamente questa cosa, aggiungendo ai comandi possibili un pratico `./gradlew publish` che fa esattamente quello che vi aspettereste.

Per agevolare cicli di sviluppo veloci, inoltre, ha anche un fantastico `./gradlew publishToMavenLocal` che, a patto che abbiate messo `mavenLocal()` tra i repositories in cui cercare le dipendenze, vi risparmia il giro &#8220;pubblica su S3 &#8211; scarica da S3&#8221; a ogni minima modifica della libreria, utile quando siete nella fase di transizione in cui la libreria cambia di frequente.

E a proposito di cose che cambiano di frequente, pensate forse funzioni tutto così facile? Ovviamente no, perché Gradle, che è furbo, non aggiorna tutte le dipendenze a ogni build, neanche se sono marcate come SNAPSHOT, ma le tiene in cache per un po&#8217;, dove &#8220;un po&#8217;&#8221; è per fortuna sufficientemente configurabile.

Come si fa? Si fa così: innanzitutto la dipendenza va di chiarata come `changing`, cioè

<div style="height: 250px; position:relative; margin-bottom: 50px;" class="wp-block-simple-code-block-ace">
  <pre class="wp-block-simple-code-block-ace" style="position:absolute;top:0;right:0;bottom:0;left:0" data-mode="php" data-theme="monokai" data-fontsize="14" data-lines="Infinity" data-showlines="true" data-copy="false">api("it.tuaazienda:tualibreria:1.0-SNAPSHOT") { changing = true }</pre>
</div>

E poi bisogna dire a Gradle come gestire le dipendenze changing, per semplicità sempre dentro `allprojects`:

<div style="height: 250px; position:relative; margin-bottom: 50px;" class="wp-block-simple-code-block-ace">
  <pre class="wp-block-simple-code-block-ace" style="position:absolute;top:0;right:0;bottom:0;left:0" data-mode="php" data-theme="monokai" data-fontsize="14" data-lines="Infinity" data-showlines="true" data-copy="false">allprojects {
    configurations.all {
        resolutionStrategy.cacheChangingModulesFor 0, 'seconds'
    }
}</pre>
</div>

Fatto questo, a ogni build (o a ogni reimport di build.gradle, se usate IntelliJ) avrete l&#8217;ultima versione della libreria pubblicata su S3, e ovviamente anche i sorgenti, con la possibilità quindi di mettere breakpoints all&#8217;interno del codice della libreria.

Et voila, la prossima volta che devo cambiare l&#8217;indirizzo da cui mandiamo le mail transazionali potrò cambiarlo in un posto solo anziché ottantasette.

 [1]: https://www.datadoghq.com/
 [2]: https://research.google/pubs/pub45424/
 [3]: https://jfrog.com/artifactory/
 [4]: https://www.sonatype.com/product-nexus-repository
 [5]: https://search.maven.org/
 [6]: https://docs.gradle.org/current/userguide/publishing_maven.html