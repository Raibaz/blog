---
title: Quick win per rendere più veloci le build di Gradle
author: Raibaz
type: post
date: 2019-10-29T09:02:06+00:00
url: /2019/10/quick-win-per-rendere-piu-veloci-le-build-di-gradle/
categories:
  - Nerd

---
Gradle ha ormai soppiantato Maven praticamente del tutto ed è diventato lo standard _de facto_ per le build in tutto l&#8217;ecosistema Java e JVM-based: siccome i processi di build sono in assoluto uno dei miei argomenti preferiti, se non si fosse capito, ho passato una buona quantità di tempo a giocarci e ho scoperto un paio di trick interessanti che permettono di rendere più veloci le build.

Build più veloci = tempo di round-trip tra quando scrivete il codice e quando lo eseguite più rapido = vita migliore, vi ricordo, quindi qualunque investimento fatto per rendere più veloce anche solo di qualche secondo una cosa che fate dozzine di volte al giorno è assolutamente degno di esser fatto.

**Caching degli artefatti**

Una build di Gradle, in realtà, è costituita da una serie di step e da una serie di moduli, ma è relativamente raro che sia necessario ribuildare veramente tutto: se il vostro progetto è composto da più sottoprogetti, al contrario, è facile che ne modifichiate uno per volta, e che non sia necessario ricompilare e rieseguire i test sugli altri.

Anche se questa cosa sembra assolutamente intuitiva, non è attiva di default su Gradle, ma va accesa esplicitamente aggiungendo a un file `gradle.properties` una riga che dica `org.gradle.caching=true`.

Così facendo, Gradle ricompilerà solo i moduli che vanno effettivamente ricompilati, prendendo dalla cache quelli che non sono stati modificati: soprattutto in ambiente locale, in cui ribuildate a ogni micromodifica, il vantaggio è evidente e consistente.

**Build in parallelo**

In questo caso, i benefici si notano maggiormente sui progetti composti da più moduli, ma di nuovo: per default, Gradle esegue le build in sequenza e in ordine alfabetico, a meno che un modulo non dipenda da un altro; nel mondo moderno, però, i computer hanno svariati core, ed è bene usarli tutti assieme, per cui, nello stesso `gradle.properties` di prima si può aggiungere una riga che dica `org.gradle.parallel=true` e BAM! per magia la compilazione, i test e quant&#8217;altro verranno eseguiti in parallelo, rendendo potenzialmente ancora più veloci le vostre build.

Occhio, però, a quel &#8220;potenzialmente&#8221;: il parallelismo è un grande potere, e da esso, come sapete, derivano grandi responsabilità, e non è detto che accenderlo risulti necessariamente in un vantaggio.

Innanzitutto, su macchine non carrozzatissime come quelle che spesso offrono i servizi di CI/CD as a service tipo quello di cui raccontavo [qui][1], è possibile che cercare di eseguire più task contemporaneamente con risorse limitate generi solo un sacco di _context switching_, col risultato che le build finiscono per essere più lente quando eseguono cose in parallelo che quando lo fanno in sequenza, ma non è tutto.

Il parallelismo, infatti, è una di quelle cose in grado di introdurre i famigerati _heisenbug_, quei bug che si verificano solo randomicamente e sono praticamente impossibili da riprodurre in maniera deterministica, soprattutto se non siete stati attenti a gestire eventuali dipendenze tra i moduli del vostro progetto.

Se, per esempio, uno dei vostri moduli ha bisogno dell&#8217;output della build di un altro modulo ma non avete specificato che la sua build `dependsOn` l&#8217;altra, rischiate di trovarvi le build rotte randomicamente: per fortuna, Gradle sa di questa possibilità e mette a disposizione un tool molto bello per debuggare questi casi.

Lanciando la build col parametro `--scan`, infatti, Gradle creerà una fantastica pagina ricca di informazioni tipo queste:<figure class="wp-block-image">

<img src="https://www.raibaz.it/wp-content/uploads/2019/10/Schermata-2019-10-28-alle-14.56.35-1024x531.png" alt="" class="wp-image-268" srcset="https://www.raibaz.it/wp-content/uploads/2019/10/Schermata-2019-10-28-alle-14.56.35-1024x531.png 1024w, https://www.raibaz.it/wp-content/uploads/2019/10/Schermata-2019-10-28-alle-14.56.35-300x155.png 300w, https://www.raibaz.it/wp-content/uploads/2019/10/Schermata-2019-10-28-alle-14.56.35-768x398.png 768w" sizes="(max-width: 1024px) 100vw, 1024px" /> <figcaption>Sì, ho chiaramente un problema con frontend:buildClientLocal.</figcaption></figure> 

Assolutamente indispensabile per capire quanto tempo richiede ciascun task, in che ordine vengono eseguiti e quali si possono parallelizzare.

Insomma, è vero che Gradle è ormai uno standard _de facto_ e che le impostazioni di default sono più che valide per la maggioranza dei casi, ma è vero anche che ci sono alcuni quick wins che si possono mettere in pratica per ottenere vantaggi anche piuttosto sostanziosi.

 [1]: https://www.raibaz.it/2019/09/ci-cd-as-a-service-con-codeship/