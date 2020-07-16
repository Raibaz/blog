---
title: Aggiornamenti automatici delle applicazioni Angular con service worker
author: Raibaz
type: post
date: 2019-10-23T07:38:36+00:00
url: /2019/10/aggiornamenti-automatici-delle-applicazioni-angular-con-service-worker/
categories:
  - Nerd
tags:
  - angular
  - service worker
  - web

---
[Nel lungo post precedente][1] abbiamo visto come fare per spacchettare in due un monolite e rendere autonoma la single page application, sviluppata con Angular 8, che costituisce il frontend del progetto su cui lavoro.

Rendere del tutto indipendente l&#8217;applicazione frontend da quella backend, però, ha un effetto collaterale piuttosto interessante: una volta che hai una single page application servita come un set di file HTML/CSS/JS statici, nulla vieta agli utenti di aprire l&#8217;applicazione una volta nella vita, tenere aperto il tab del browser per sempre e non aggiornarla mai.

Noi però sull&#8217;applicazione continuiamo a lavorare, a sistemare bug e ad aggiungere feature, e ho già assistito troppe volte a una conversazione di questo tipo:

&#8220;Non funziona&#8221;  
&#8220;Refresha&#8221;  
&#8220;Ah ok, adesso va&#8221;

Sarebbe bello, quindi, che il browser sapesse quando è stata rilasciata una versione nuova dell&#8217;applicazione e si aggiornasse.

La mia prima soluzione era una cosa assolutamente involuta, con un endpoint dell&#8217;applicazione frontend che espone la variabile di ambiente di Heroku con lo SHA-1 dell&#8217;ultimo commit, e l&#8217;applicazione stessa che forza un `window.location.reload()` quando il valore ritornato è diverso da quello che ha salvato nel `sessionStorage`, ma poi mi sono chiesto se non ci fosse qualcosa di più semplice, dato che non mi pare esattamente un problema solo mio.

La risposta, infatti, è che esiste un modo praticamente gratis per farlo, con una tecnologia che piace ai giovani e che è estremamente semplice da usare: i service worker.

Angular, nello specifico, ha un [service worker suo][2] che fa _esattamente_ questo: tiene in cache una lista di file e asset statici che gli diciamo noi in un file di configurazione ed espone un servizio che triggera degli eventi quando questi vengono aggiornati.

Inizia tutto così: `ng add @angular/pwa`.

Così facendo, la nostra applicazione diventa magicamente una [PWA][3] che installa un service worker sui browser degli utenti; è tutto magicamente già configurato e justworka, serve solo dare un&#8217;occhiata alla lista dei file cachati e autoaggiornati nel file `ngsw-config.json` e verificare che ci siano tutti i file che compongono l&#8217;applicazione (tipicamente `/*.js` e `/*.css` sono sufficienti)

Ok, ora ho un service worker, cosa ci faccio?

Posso registrare un servizio che polli il server per aggiornamenti ai file che il service worker tiene in cache, e quando c&#8217;è un aggiornamento forzare un refresh del browser, per l&#8217;appunto:

<div style="height: 250px; position:relative; margin-bottom: 50px;" class="wp-block-simple-code-block-ace">
  <div style="position:absolute;top:-20px;right:0px;cursor:pointer" class="copy-simple-code-block">
    <svg aria-hidden="true" role="img" focusable="false" class="dashicon dashicons-admin-page" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewbox="0 0 20 20"><path d="M6 15V2h10v13H6zm-1 1h8v2H3V5h2v11z"></path></svg>
  </div>
  
  <pre class="wp-block-simple-code-block-ace" style="position:absolute;top:0;right:0;bottom:0;left:0" data-mode="typescript" data-theme="ambiance" data-fontsize="14" data-lines="Infinity" data-showlines="true" data-copy="true">@Injectable()
export class AutoUpdaterService {

  constructor(appRef: ApplicationRef, updates: SwUpdate) {
    console.log('Subscribing to application updates for autorefresh...');
    updates.available.subscribe(event => {
        console.log('Application was updated, need to refresh!');
        updates.activateUpdate().then(() => document.location.reload());
    });

    console.log('Configuring application update polling');
    const appIsStable$ = appRef.isStable.pipe(first(isStable => isStable === true));
    const everyMinute$ = interval(60 * 1000);
    const everyMinuteOnceAppIsStable$ = concat(appIsStable$, everyMinute$);

    everyMinuteOnceAppIsStable$.subscribe(() => {
      updates.checkForUpdate();
    });
  }
}</pre>
</div>

Ci sono un paio di gotcha non banali: per esempio, cos&#8217;è quell&#8217;`appIsStable$`?

I service worker hanno un lifecycle che si articola in diverse fasi, e l&#8217;applicazione è definita &#8220;stable&#8221; solo una volta che il service worker è installato e running; prima di questo momento, fare polling per aggiornamenti sarebbe dannoso, perché si rischia di finire in un loop in cui l&#8217;applicazione sembra aggiornata e la riaggiorno prima che abbia finito di installarsi.

Così facendo, quindi, ogni sessanta secondi l&#8217;applicazione chiede al service worker se ci sono aggiornamenti nei file che tiene in cache, e in caso ce ne siano forza un refresh del browser: ora, io posso fare questa cosa perché la mia applicazione è fondamentalmente stateless e refresharla &#8220;sotto il culo&#8221; agli utenti non è un grosso problema, ma nulla vi vieta di mostrare un avviso che dice &#8220;ci sono aggiornamenti, per favore refresha&#8221;.

Insomma, non c&#8217;è davvero bisogno di mettere in piedi cose complicate con gli SHA-1 dei commit: basta usare la tecnologia del web del 2019.

 [1]: https://www.raibaz.it/2019/10/autenticazione-via-token-jwt-con-angular-e-spring-boot/
 [2]: https://angular.io/guide/service-worker-communications
 [3]: https://developers.google.com/web/progressive-web-apps