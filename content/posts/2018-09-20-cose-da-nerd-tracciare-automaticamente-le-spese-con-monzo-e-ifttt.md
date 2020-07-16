---
title: 'Cose da nerd: tracciare automaticamente le spese con Monzo e IFTTT'
author: Raibaz
type: post
date: 2018-09-20T07:30:51+00:00
url: /2018/09/cose-da-nerd-tracciare-automaticamente-le-spese-con-monzo-e-ifttt/
categories:
  - Londinesità
  - Nerd
tags:
  - Google Sheets
  - Monzo
  - Soldi

---
Insomma che come si diceva, quissù tutto solo ho una buona quantità di tempo libero, e la fase di adattamento alla vita in questa nazione include, tra le altre cose, la necessità di tracciare in modo dettagliato quanto spendo, per capire, a tendere, quanta birra posso bere, quanto spesso posso mangiare fuori, e quante cazzate posso comprarmi per impiegare il tempo suddetto.

Ora, cosa fa uno sviluppatore quando vuole sapere qualcosa in dettaglio? Mette in piedi un sistema per raccogliere dati automaticamente, _ça va sans dire._

Fortunatamente, sembra che il bisogno di tracciare dettagliatamente le spese sia condiviso, almeno da questo lato della manica, visto che è uno dei selling point principali di [Monzo][1], una startup che ha come claim un moderatissimo e per niente pretenzioso &#8220;The bank of the future&#8221;.

Su suggerimento della validissima gente di [WEBdeLDN][2] apro un account in tempo zero dall&#8217;app, e dopo un paio di giorni ho già la mia nuova Mastercard color corallo:

<div class="wp-block-image">
  <figure class="aligncenter"><img src="https://raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-17-alle-22.09.37.png" alt="" class="wp-image-33" srcset="https://www.raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-17-alle-22.09.37.png 656w, https://www.raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-17-alle-22.09.37-300x192.png 300w" sizes="(max-width: 656px) 100vw, 656px" /></figure>
</div>

L&#8217;app fa già un ottimo lavoro per noi nerd del tracciamento delle spese, mandandoti una push notification per ogni addebito (che non è sempre istantaneo, perché quei maledetti di TFL, l&#8217;azienda dei trasporti di Londra, ti addebitano un po&#8217; quando vogliono) e dividendo automagicamente le spese in trasporti, cibo, generi di prima necessità, etc., ma ovviamente non è abbastanza.

Si dà il caso che avessi già, ovviamente, un documento su Google Sheets con un sacco di formule e grafici che teneva traccia di tutte le spese tipo luce-gas-ADSL-macchina-cibo-blablabla, frutto di anni di esperienza, aggiustamenti e abitudine a tracciare a mano ogni spesa rilevante, al quale ormai mi sono affezionato e del quale faccio fatica a fare a meno: e se ci fosse un modo di salvare le spese che Monzo traccia già nello stesso documento, magari senza lo sbattimento di doverle scrivere a manina ogni volta?

[IFTTT][3] to the rescue!

La banca del futuro, infatti, presta fede al proprio claim offrendo una serie sufficientemente ampia di integrazioni col magico servizio di cui ogni nerd appassionato di automazione non può fare a meno:

<div class="wp-block-image">
  <figure class="aligncenter"><img src="https://raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-17-alle-22.18.39.png" alt="" class="wp-image-34" srcset="https://www.raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-17-alle-22.18.39.png 808w, https://www.raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-17-alle-22.18.39-235x300.png 235w, https://www.raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-17-alle-22.18.39-768x981.png 768w, https://www.raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-17-alle-22.18.39-802x1024.png 802w" sizes="(max-width: 808px) 100vw, 808px" /></figure>
</div>

&#8220;Ok, quindi posso loggare tutte le spese che faccio con la mia Mastercard color corallo su un Google sheet, fatto, giusto?&#8221;

Ni.<figure class="wp-block-image">

<img src="https://raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-17-alle-22.20.26.png" alt="" class="wp-image-35" srcset="https://www.raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-17-alle-22.20.26.png 1310w, https://www.raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-17-alle-22.20.26-300x102.png 300w, https://www.raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-17-alle-22.20.26-768x261.png 768w, https://www.raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-17-alle-22.20.26-1024x349.png 1024w" sizes="(max-width: 1310px) 100vw, 1310px" /> </figure> 

I più attenti di voi non faranno fatica a notare che il formato di questa tabella è un cesso a pedali:

  * La data è in un formato impossibile da gestire
  * Le spese sono mischiate anzichè essere raggruppate in qualche modo, chessò, per data, per settimana, per mese, per categoria, per salcazzo
  * Quel £ prima degli importi rende impossibile fare le somme

Insomma, ci vuole un po&#8217; di Google Sheet Kung Fu.

<p style="font-family: monospace">
  =IFERROR(DATEVALUE(LEFT(Sheet1!A2,FIND(&#8221; at&#8221;,Sheet1!A2))), &#8220;01/01/1970&#8221;)
</p>

&#8220;Prendi la parte prima di &#8220;at&#8221; nella cella con data e ora e interpretala, appunto, come una data, usando il primo gennaio del 1970 come default se c&#8217;è qualche errore&#8221; &#8211;> &#8220;9/10/2018&#8221;, ecco, iniziamo a ragionare.

<p style="font-family: monospace">
  =SUBSTITUTE(Sheet1!E2, &#8220;£&#8221;, &#8220;&#8221;) + 0
</p>

&#8220;Togli quel cazzo di £ dalla cella con l&#8217;importo e sommaci zero, giusto per chiarire che è un valore numerico&#8221;

Ecco quindi che ci troviamo con una tabella come si deve, con gli importi sommabili e raggruppabili per data, e con un paio di colpi di [SUMIFS][4], voila! Abbiamo la somma di quanto ho speso di metropolitana nel mese di settembre, pronta per essere inserita nel foglio riassuntivo che ho imparato ad apprezzare.

&#8220;Aspetta, ma nel foglio riassuntivo devi copiarcela a mano? Monzo scrive nel suo Google Sheet, non nel tuo.&#8221; 

Ma certo che no, suvvia.

<p style="font-family: monospace">
  =IMPORTRANGE(&#8220;<url del google sheet dove scrive monzo e dove hai messo tutte le formule di cui sopra>&#8221;, &#8220;<cella dove hai salvato la somma delle spese di metro per il mese che ti interessa>&#8221;)
</p>

Ecco fatto: il foglio delle spese a cui sei abituato si aggiorna automagicamente ogni volta che usi la carta di Monzo per pagare la metro.

Problema risolto, tempo libero impiegato, cose imparate.

 [1]: https://monzo.com/
 [2]: http://webdeldn.rocks
 [3]: https://ifttt.com/
 [4]: https://support.google.com/docs/answer/3238496?hl=en