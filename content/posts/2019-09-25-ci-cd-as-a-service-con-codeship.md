---
title: CI/CD as a service con Codeship
author: Raibaz
type: post
date: 2019-09-25T07:03:55+00:00
url: /2019/09/ci-cd-as-a-service-con-codeship/
categories:
  - Nerd

---
Non so per quale buffo motivo, ma la continuous integration e il suo diretto successore, il continuous deployment, sono sempre stati un po&#8217; la mia passione: già nel mio primissimo lavoro, ormai più di dieci anni fa, avevo installato un cron sulla macchina Windows che usavamo come server di staging per fare le nightly build con ant (madonna a scriverlo sembra veramente la preistoria), e in praticamente ogni lavoro successivo, in un modo o nell&#8217;altro, sono finito a essere quello che si occupava di gestire i processi di rilascio.

Negli anni, quindi, ho imparato a conoscere [Jenkins][1] un po&#8217; come le mie tasche e ho pure contribuito una [feature][2] abbastanza di successo al plugin che aggiorna lo stato delle build su Bitbucket.

Sono pure finito nei credits di un [libro fighissimo][3] che parla di DevOps, sul quale scriverò più diffusamente in futuro.

Nel mio nuovo lavoro, però, c&#8217;è un sacco di roba da fare, tra cui anche mettere in piedi l&#8217;infrastruttura di CI/CD, ma installare un Jenkins da zero sembrava un po&#8217; overkill, visto anche che i tempi sono cambiati e anziché avere delle macchine &#8220;vere e proprie&#8221; dove hostiamo le cose usiamo estensivamente Heroku e in piccola parte AWS, per cui sarebbe stato davvero troppo sbattimento mettere in piedi una macchina e un&#8217;istanza Jenkins, soprattutto adesso che la priorità era avere qualche sorta di CI/CD prima possibile.

Esclusa quindi l&#8217;opzione &#8220;faccio tutto da me e mi gestisco il Jenkins da solo&#8221;, che magari riprenderò in considerazione in futuro, le tre opzioni che ho preso in considerazione sono state:

  * Le [pipelines][4] di Bitbucket
  * [Heroku CI][5]
  * [Codeship][6]<figure class="wp-block-image">

<img src="https://raibaz.it/wp-content/uploads/2019/09/Schermata-2019-09-24-alle-22.45.12-1024x456.png" alt="" class="wp-image-211" srcset="https://www.raibaz.it/wp-content/uploads/2019/09/Schermata-2019-09-24-alle-22.45.12-1024x456.png 1024w, https://www.raibaz.it/wp-content/uploads/2019/09/Schermata-2019-09-24-alle-22.45.12-300x134.png 300w, https://www.raibaz.it/wp-content/uploads/2019/09/Schermata-2019-09-24-alle-22.45.12-768x342.png 768w" sizes="(max-width: 1024px) 100vw, 1024px" /> <figcaption>Solo io ci vedo un diss nemmeno troppo nascosto a CircleCI?</figcaption></figure> 

I criteri con cui ho scelto cosa usare sono stati abbastanza semplici:

  * Portabilità a piattaforme diverse: se possibile, preferisco non legarmi mani e piedi a un VCS o a un provider di infrastruttura
  * Notifiche via mail e su Slack configurabili per le buil
  * API per ottenere lo stato dell&#8217;ultima build di master: prima o poi una delle cose che dovrò fare è mettere in piedi delle dashboard per vedere metriche, e lo stato dell&#8217;ultima build è una delle cose che voglio visualizzare
  * Costo: ok che non pago di tasca mia, but still.

Inoltre, attualmente almeno una delle applicazioni che devo buildare è un monolite abbastanza articolato che ha dentro varie cose diverse tutte assieme, per cui ho bisogno anche di un po&#8217; di flessibilità nelle cose che posso fare.

Le pipeline di Bitbucket, _ça va sans dire_, funzionano solo su Bitbucket, e sono un po&#8217; limitate sia in termini di notifiche che possono mandare che come facilità di mettere in piedi build articolate, mentre Heroku CI richiede per forza di usare GitHub, per cui avrei dovuto migrare tutti i progetti e tutto il team da Bitbucket e, ovviamente, se un giorno volessimo spostarci da Heroku avremmo una pigna in più da gestire.

Codeship, invece, è agnostico rispetto all&#8217;hosting del codice e dove deploya, anche perché fa un uso estensivo e molto interessante dei container Docker (un&#8217;altra cosa che mi sono portato dietro in molti posti dove ho lavorato, [anche importanti][7]), non solo come artefatto creato al termine delle build, che quindi si può deployare un po&#8217; dappertutto, ma anche come ambiente di build di per sé: l&#8217;equivalente del Jenkinsfile, più o meno, è infatti costituito da tre elementi:

  * Uno o più Dockerfile che descrivono i container dove gireranno le build
  * Uno yml che descrive eventuali altri servizi necessari per la build, tipo un db di integrazione o un RabbitMQ; è fondamentalmente un docker-compose, e infatti se Codeship non trova il suo file cerca proprio un docker-compose
  * Un altro yml che descrive gli step della build e in quale servizio devono essere eseguiti

(Nota: tutto quello che scrivo a proposito di Codeship si riferisce a Codeship pro: c&#8217;è anche un piano free che è anche molto più semplice da configurare, essendo un filo più limitato, ma ha un limite di 100 build al mese che nel mio caso era assolutamente impraticabile)

Insomma, nel giro di mezza giornata ho messo in piedi, da zero, il processo di CI/CD per tre applicazioni anche piuttosto diverse tra loro; nel mio caso il Dockerfile è praticamente un&#8217;immagine di base con un OpenJDK 11 e un paio di cose installate aggiuntive e non mi servono altri servizi, per ora, per cui l&#8217;unico file davvero interessante è quello degli step, di cui di seguito vi mostro un esempio:

<pre class="wp-block-code"><code class="">- type: parallel
  steps:
    - name: frontend
      service: [REDACTED]
      command: ./gradlew frontend:build --console=plain
    - name: test
      service: [REDACTED]
      command: ./gradlew check test -x frontend:buildClientLocal --console=plain
- name: deploy
  service: heroku-deployment
  command: codeship_heroku_deploy -f /deploy -N [REDACTED] -u [REDACTED] -c true
  tag: master</code></pre>

In pratica, non fa niente di complicato, se non:

  * Esegui in parallelo la build del frontend e i test del resto dell&#8217;applicazione
  * Quando hai finito, deploya su heroku: tra l&#8217;altro, `-u` definisce l&#8217;url da pingare una volta finito il deployment per assicurarsi che sia andato tutto ok, che è una feature molto carina che arriva praticamente gratis; con Jenkins o nella build di Gradle sarebbe stato nettamente più noioso da fare

E questo è quello &#8220;difficile&#8221; dei tre file.

Ma non è tutto: se hai mai installato una pipeline di CI/CD, sai che il vero sbattimento iniziale è dover committare e pushare ogni modifica, aspettare che buildi e vedere se funziona, e all&#8217;inizio va da sé che di modifiche e assestamenti ce ne siano tanti, per cui sarebbe bello avere un loop di feedback più veloce possibile.

Avendo le build che girano in dei container Docker, Codeship può permettersi di darti `jet`, che è la vera figata del tutto: si installa facile facile con <code class="">&lt;code>brew cask install codeship/taps/je</code>t</code> e, con un rapido colpo di `jet steps`, esegue esattamente quello che verrà poi eseguito su Codeship, in locale, senza bisogno di spammare il mondo di commit e aspettare per ogni modifica; fai andare tutto in locale, poi pushi, e Codeship fa lo stesso processo di build che sai che funziona.

Ok, non è il massimo per le ventole del mio Mac che nell&#8217;ultimo paio di giorni hanno lavorato come non mai, ma per la mia sanità mentale è veramente la manna dal cielo.

Morale, per 49$ al mese, che è una cifra tutto sommato accettabile (soprattutto se non devo pagarli io :)) hai un&#8217;infrastruttura di CI/CD molto molto ben fatta, e a zero sbatti di manutenzione, dato che è &#8220;as a service&#8221;; tra l&#8217;altro, ho avuto un mezzo sbattimento iniziale e il supporto mi ha risposto veramente in fretta (e poi era colpa del firewall aziendale che non mi faceva vedere le build), quindi ecco, tutto molto bene finora.

 [1]: https://jenkins.io/
 [2]: https://github.com/jenkinsci/bitbucket-build-status-notifier-plugin/pull/37
 [3]: https://www.amazon.it/gp/product/8850334508?tag=raibaz-21
 [4]: https://bitbucket.org/product/features/pipelines
 [5]: https://devcenter.heroku.com/articles/heroku-ci
 [6]: https://codeship.com/
 [7]: https://github.com/googleads/google-ads-php/pull/37