---
title: Automatizzare le mail noiose con Google Apps Script
author: Raibaz
type: post
date: 2018-09-21T07:19:04+00:00
url: /2018/09/automatizzare-le-mail-noiose-con-google-apps-script/
categories:
  - Nerd

---
<figure class="wp-block-image"><img src="https://raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-19-alle-23.16.40-1.png" alt="" class="wp-image-58" srcset="https://www.raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-19-alle-23.16.40-1.png 2146w, https://www.raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-19-alle-23.16.40-1-300x36.png 300w, https://www.raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-19-alle-23.16.40-1-768x93.png 768w, https://www.raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-19-alle-23.16.40-1-1024x124.png 1024w" sizes="(max-width: 2146px) 100vw, 2146px" /></figure> 

Premessa doverosa: questo post è leggermente aziendalista, perché parla in toni entusiastici di un prodotto dell&#8217;azienda per cui lavoro. Tuttavia, disclaimer: è _veramente_ una figata, lo conoscevo un pochino (e lo conosco ancora poco) ma ci si possono fare un sacco di cose meravigliose.

Allora, la situazione è questa: anche se mi sono trasferito da poco all&#8217;estero, ho ancora una partita IVA italiana e la avrò ancora per un po&#8217;, oltre ad avere una casa in Italia, per cui mi capita spesso di ricevere fatture per spese, bollette e quant&#8217;altro che devo girare alla mia commercialista per poter detrarre dalle tasse le spese suddette.

In passato, questa cosa la gestivo stampando le fatture e portandole a mano alla commercialista una volta a trimestre: tutti d&#8217;accordo, non è neanche lontanamente la soluzione ottimale, ma funzionava, e in fondo passare una volta a trimestre dalla commercialista (o spedirci mia madre quando non avevo tempo di farlo io, lo ammetto) era una discreta occasione di socialità.

Tuttavia, prendere un aereo una volta a trimestre per andare a portare della carta alla commercialista non mi pare una soluzione fattibile: quando prendo un aereo per tornare a Milano ho tonnellate di cose migliori da fare, senza contare che lo faccio praticamente sempre nei weekend in cui logicamente la commercialista ha anch&#8217;essa di meglio da fare.

Soluzione: &#8220;cara commercialista, è un problema per te se le fatture delle spese te le mando per email come già facevo per quelle che emettevo io verso i miei clienti?&#8221; Ovviamente no, ma come ogni soluzione che si rispetti questa crea degli altri problemi.

Mi pare scortese, infatti, spammare la povera commercialista ogni volta che mi arriva una fattura, per cui l&#8217;accordo è che le manderò uno zip con un po&#8217; di fatture una volta ogni tanto: ma come faccio a ricordarmi quali fatture le ho già mandato e quali no? E non è uno sbatti allucinante scaricarle tutte, fare lo zip, riallegarlo a una mail nuova e mandarla?

Certo che lo è, ma io sono un nerd.

E i nerd odiano fare a mano le operazioni semplici e ripetitive.

Si dà il caso che il mio attuale datore di lavoro abbia un prodotto, [Google Apps Script][1], che serve giustappunto a soddisfare questa necessità di qualunque nerd che si rispetti: tra le tante cose che fa, infatti, permette di scrivere pezzettini di codice che interagiscano con diversi altri prodotti Google, principalmente GMail e Google Drive, che sono quelli che interessano a me, ma anche Calendar, Hangouts, Maps e Translate tra gli altri.

Ora, quello che devo fare io non è altro che quanto segue:

  * Prendi tutte le mail con la label &#8220;fatture&#8221; (ok, questo lo faccio a mano, ma sticazzi)
  * Crea una cartella nuova su Google Drive e salvaci tutti gli allegati delle mail prese in precedenza
  * Fai uno zip della cartella e mandalo per email alla commercialista, mettendomi in bcc che non si sa mai
  * Cancella la cartella
  * Togli la label &#8220;fatture&#8221; a tutte le mail e aggiungi la label &#8220;fatture-inviate&#8221;<figure class="wp-block-image">

<img src="https://raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-19-alle-23.17.50.png" alt="" class="wp-image-59" srcset="https://www.raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-19-alle-23.17.50.png 994w, https://www.raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-19-alle-23.17.50-300x129.png 300w, https://www.raibaz.it/wp-content/uploads/2018/09/Schermata-2018-09-19-alle-23.17.50-768x331.png 768w" sizes="(max-width: 994px) 100vw, 994px" /> <figcaption>Una piccola parte dello script (si fa per dire, è praticamente metà del lavoro)  
</figcaption></figure> 

Sono stato un po&#8217; prolisso e ho cercato di essere ordinato, per cui tutto quanto sopra ammonta a **ben** trentasette righe di script: in pratica ci sto mettendo di più a raccontare quello che ho fatto che a farlo.

Oltretutto, bonus: GAS è praticamente Javascript, quindi se hai mai scritto del codice in vita tua è facile che tu non faccia troppa fatica ad adattartici, e le API per interagire coi servizi Google sono estremamente semplici e ben documentate: io avevo scritto uno script solo in vita mia, mille anni fa, per cui ero praticamente vergine dell&#8217;ambiente di sviluppo, ma le trentasette righe in questione le ho scritte, testate e debuggate in poco più di un&#8217;ora.

Problema risolto, cose imparate, gioia nel cuore.

Il problema nuovo, perché le soluzioni spesso creano problemi nuovi, è trovare altre cose che posso automatizzare o semplificare usando GAS, visto che adesso mi ci sono affezionato e voglio usarlo per la qualunque.

Idee?

 [1]: https://developers.google.com/apps-script/