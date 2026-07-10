+++ 
draft = false
date = 2026-07-10T18:58:58+02:00
title = "Guarda mamma, sono un vibe coder"
description = ""
slug = "guarda-mamma-sono-un-vibe-coder"
authors = []
tags = []
categories = []
externalLink = ""
series = []
featured_image = "/images/dj_king.png"
images = ["/images/dj_king.png"]
+++

Questa cosa dell'intelligenza artificiale e del vibe coding è un'assoluta croce e delizia per noi nerds con la scimmia dei side project: se una volta le idee per i side project restavano nel cassetto per settimane, mesi, anni, a volte per sempre, prima che ci venisse la voglia di affrontare la sindrome del foglio bianco, o che avessimo tempo libero a sufficienza per studiare la tecnologia coinvolta, adesso tutti questi problemi si risolvono con un prompt.

Quest'inverno ho deciso che avrei [imparato l'hardware](/posts/filsenger-il-mio-ultimo-progetto-da-nerd), mentre stavolta invece avevo proprio un problema reale da risolvere, per cui ho fatto quello che in gergo si chiama _scratch your own itch_.

Il problema in questione non è affatto comune e me lo porto dietro da anni, da quando ero un giovane dj famoso: quotidianamente entro in possesso di qualche decina di file di musica nuova, vuoi attraverso canali promozionali, vuoi attraverso la condivisione da parte di amici dj, produttori e musici assortiti, vuoi attraverso altri canali, e non ho fisicamente il tempo di ascoltare tutto con l'attenzione che meriterebbe, col risultato che la musica da scremare si accumula più velocemente di quanto io riesca a scremarla.

Nel momento in cui ho iniziato a pensare a questo side project, verso fine giugno 2026, ero arrivato a scremare la musica uscita a settembre 2025, per capirci: serviva un approccio ingegneristico alla questione.

Ho esposto il problema al ragazzo artificiale con cui abitualmente converso di libri, e in qualche iterazione siamo arrivati a una soluzione, composta da uno script in python e una web application molto basica.

Lo script si prende in pasto un po' di release, sotto forma di file zip o di cartelle contenenti file audio, e per ogni traccia crea una preview da un minuto e mezzo facendo un'analisi elaborata ma non troppo di quale potrebbe essere la parte più rilevante.

La web application è fatta così:

![pic](/images/dj_king.png#center)

Ha la lista di preview in ordine casuale, che posso scegliere se ascoltare "alla cieca" senza sapere di cosa si tratti oppure vedendo titolo e autore, e un po' di tasti con cui posso decidere cosa farne:

- Tieni questa traccia
- Butta via questa traccia
- Tieni questa traccia e metti il rating a cinque stelle nel file in modo che poi lo veda su rekordbox e sulla console quando sto suonando
- Butta nel cesso tutta la release di questa traccia che ne ho già sentite alcune e non fa per me
- Non lo so ci penso poi

Una volta presa una decisione per tutte le tracce, se tutte le tracce di una release sono state buttate via la release intera viene cancellata e non ne sapremo mai più niente, altrimenti resta a disposizione per un ascolto più attento e una successiva archiviazione nel complicatissimo sistema di organizzazione con cui archivio da più di vent'anni i file che uso per suonare.

Finora sono soddisfattissimo di questo tool super basico che risolve un problema che avevo solo io in un modo costruito su misura sulle mie esigenze grazie al ragazzo artificiale, a pochissime conoscenze tecniche da parte mia ma a una conoscenza molto approfondita del dominio applicativo: nel giro di pochi giorni sono già arrivato a scremare fino a fine ottobre, e vedo la luce alla fine del tunnel, anche se è ancora molto lontana.

Il prossimo passo sarà cercare di riordinare meglio quei vent'anni di file archiviati, ma ci penserò poi.
