+++ 
draft = false
date = 2026-01-31T22:45:29+01:00
title = "Filsenger, il mio ultimo progetto da nerd"
description = ""
slug = "filsenger-il-mio-ultimo-progetto-da-nerd"
authors = []
tags = []
categories = []
externalLink = ""
series = []
+++

Era un po' di tempo che avevo in mente un'idea che, sulla carta, mi sembrava davvero semplice: un mini messenger che mandasse e ricevesse messaggi solo a/da un altro device equivalente, sostanzialmente una specie di walkie talkie wifi che potessi dare al mio giovane amico per comunicare con lui quando è a casa di sua madre.

Ho cercato un po' oziosamente in rete se ci fosse qualcosa di simile: il meglio che ho trovato è [Chatter](https://circuitmess.com/it-en/products/chatter-2-0) di CircuitMess, ma non era _esattamente_ quello che volevo io e comunque ho avuto un abbonamento a CircuitMess (cioè, tecnicamente ce l'ha avuto lui) per un po' e non mi ha mai fatto una grandissima impressione, soprattutto lato hardware: belle idee, ma realizzazione un po' plasticona.

Sapete benissimo come va a finire quando un nerd non trova _esattamente_ quello che voleva: se la costruisce.

Ho iniziato, altrettanto oziosamente, a fare un po' di studio di fattibilità col ragazzo artificiale con cui di solito [parlo di libri](/posts/gene-wolfe-l-ombra-del-torturatore), chiedendogli tipo "oh ma secondo te è un'idea del cazzo o ha senso?" e all'epoca non l'avevo ancora configurato per farmi challenge e non fare il sicofante, quindi ovviamente la sua risposta è stata il più classico dei "you are absolutely right!" che non mi ha smorzato l'entusiasmo, anzi.

Nel giro di cinque minuti avevo una lista della spesa contenente:
- Un ESP32
- Uno schermino OLED
- Un microfono MEMS
- Un amplificatore/DAC
- Una cassa da 3 volt
- Qualche pulsantone da arcade

Fortunatamente tutti in molteplice copia, vuoi perché si tratta in larga parte di componenti così piccoli che la confezione da 3 costa sui 5/6€, vuoi perché il saggio ragazzo artificiale mi ha detto "senti ammè, non comprare i pezzi singoli" e aveva ragione, come vedremo in seguito.

Era solo l'inizio di un'avventura molto lunga e di una conversazione sterminata col ragazzo artificiale, costellata di successi, fallimenti, dita brasate col saldatore, pezzi buttati, pezzi comprati in eccesso, molti soldi spesi (molti più di quanti me ne sarebbe costati il cazzettino di CircuitMess) e molti insegnamenti, ma che si è conclusa così:

![pic](/images/filsenger_final.jpg#center)

Ma andiamo con ordine.

L'oggetto in questione, si diceva, si compone di un ESP32, che per chi non lo sapesse è tipo un RaspberryPI un pochino più carrozzato (ma manco troppo) su cui gira del python, con, come si diceva sopra, schermino, microfono, altoparlante e pulsanti vari per controllare le cose.

Iniziamo col dire che il microfono e l'ampli vengono venduti coi pin da saldare: per fortuna io un saldatore già ce l'avevo (stava nell'abbonamento di CircuitMess), ma non avevo dello stagno decente e, soprattutto, ho quasi quarantatrè anni, quindi ho dovuto comprare degli occhiali con le lenti di ingrandimento e la lucina, altrimenti col cazzo che riuscivo a saldare quei pin microscopici.

E pure con gli occhialoni, il mio talento nella saldatura era tutto da dimostrare, ma no spoiler.

Attacco tutto come mi spiega perfettamente il bro artificiale, faccio le prime prove...mmm si, l'altoparlante suona, ma **dimmerda**.

Benvenuti nel primo dei tanti rabbit hole: "ah l'ampli si aspetta l'audio a 16 bit, no a 32, no mono, no stereo, no allora aspe che ripuliamo un po' l'audio che arriva dal microfono e tagliamo le frequenze rumorose, no senti a me la soluzione giusta è mettere un condensatore in parallelo all'ampli perché i 5V dell'ESP32 non sono costantissimi è sicuramente quello"

In effetti il condensatore fa un botto di differenza, ma non era quello, e dopo giorni di tentativi, ormai prossimo alla rassegnazione di far sentire al Filo i messaggi come se li avessi registrati in una galleria del vento, mando una foto dell'ampli al bro artificiale dicendogli "oh ma secondo te è saldato bene?" e lui mi risponde "zio, come dire, no."

Bastava sistemare un pochino la saldatura di un pin, e BAM! Suono cristallino, per quanto possa suonare cristallina una cassa che costa meno di 1€.

Bom figata, funziona tutto, adesso appena arriva il Filo costruiamo assieme un case di Lego tutto attorno, abbiamo giusto comprato un kit di pezzi generici apposta, lo faccio pure vedere ai miei amici nerd e mi bullo un po', attualmente è circa così:

![pic](/images/filsenger_naked.jpg#center)

Manda i messaggi a una funzione su Cloud Run che li manda a un bot di Telegram e da lì al mio account e fa anche la strada inversa, con in più Cloud Run che fa anche un po' di magheggini di transcoding audio con ffmpeg ed espone un endpoint a cui il Filsenger (si chiama così) fa polling per sapere se ci sono nuovi messaggi.

"Ma sai che c'è però, che con lo schermo e il microfono attaccati direttamente alla breadboard poi come ce lo costruisco il case attorno?"
"You are absolutely right! Comprati delle basette millefori così ci saldi schermo e microfono e poi da lì vai alla breadboard, tanto te le devi comprare lo stesso perché poi ci farai una versione più definitiva che breadboard è la merda e lo sai anche tu"
"Ah, bravo bro, bella idea"

Schermo buttato via dopo avergli imballato i pin di stagno.
Ma fa niente, ne ho altri due.

"Senti zio forse saldare non è il mio"
"Non c'è problema! Usa i cavettini maschio-femmina, nella femmina infili i pin di schermo e microfono e i maschi nella breadboard, super facile"
"Bom figata, allora aspetta che stacco il microfono dalla breadboard e lo metto nei cavettini così è lontano dalla breadboard e può uscire dal case"

L'inizio della fine.

Stacco il microfono dalla breadboard e gli si stacca un pin.

Che problema c'è, lo risaldo, tanto mi sono pure comprato un saldatore nuovo perché quello pacco di CircuitMess ora ci guarda da lassù assieme allo schermo di cui sopra.

Non va.

Altro rabbit hole.

"Allora assicurati che il pin L/R sia a massa, o forse a 3V, misura col tester (che fortunatamente avevo già) tutti i pin del microfono per capire qual è quello saldato male, aspetta prova a spostare il pin SD da massa a 3V...."

Altra vittima: spostando dei pin da sa dio dove a sa dio dove altro, l'ESP32 decide che non ne può più e svampa malamend.

Pazienza, ne ho altri due.

A furia di saldare e risaldare i pin del microfono divento forse un pochino più bravo col saldatore, ma soprattutto scopro che mi microfonini MEMS sono piuttosto sensibili al calore e se non li saldi un po' per volta ma tutti assieme si scaldano troppo anch'essi e soprattutto anche la membranina del microfono, che si squaglia e puoi prendere il microfono e buttarlo via.

Pazienza, ne ho altri due che squaglierò entrambi.

Pazienza, intanto che arriva un'altra confezione di microfoni facciamo andare almeno l'altoparlante dell'ESP32 nuovo.

Non va.

I due-tre giorni più frustranti del 2026 (lo so che è appena iniziato, ma credo sarà difficile superarli) a cercare di capire cosa cazzo si sia rotto se il codice è lo stesso, non ho rifatto le saldature da quando andava nè ho cambiato i pin: non so tuttora come, ma dopo un po' funziona e non voglio toccarlo mai più.

Cioè, lo so: bastava mettere i pin giusti e non, come avevo fatto nel delirio di "ommioddio funzionava e adesso non va come cazzo lo rimonto" mettere sullo stesso pin il clock dell'amplificatore e una linea dello schermo.

Arrivato il microfono nuovo, ormai sono diventato bravino col saldatore e soprattutto molto zen perché so che sono uscito vivo da tutti i rabbit hole del mondo, e infatti funziona quasi al primo colpo, ma soprattutto so esattamente perché non funziona e come un chirurgo sistemo facile.

Sono di nuovo nello stato in cui ero prima di pensare di spostare il microfono dalla breadboard.

Ora è il momento di costruire il case di Lego col Filo, e lì per lì mi rendo conto che la breadboard che ho usato è veramente gigante e occupa un sacco di spazio per niente, ma ormai sono così un pro che rimonto tutto su una breadboard più piccola in cinque minuti scarsi, e funziona anche.

Come direbbe Milord, "My job here is done", il case di Lego lo ha di fatto costruito tutto il Filo:

![pic](/images/filsenger_case.jpg#center)

Mentre io cucinavo e morivo di ansia e tra me e me pensavo "madonna stai attento non toccacciare troppo i cavi che non è stabile" e però invece è rimasto stabile praticamente tutto il tempo, anche perché il giovane conosce la base dell'ingegneria e ogni due-tre secondi faceva un test per vedere che funzionasse tutto (grazie Mark Rober tvb sempre) e l'unica volta che ha ricominciato a gracchiare la cassa, passato il panico iniziale, sapevo che erano le gambettine del condensatore in corto e ho sistemato facile.

È finito?

Certo che no, è un progetto nerd e ci sono sempre feature che si possono aggiungere, alcune me le ha già proposte il Filo.

È stato facile? No.

È stato divertente? Per la maggior parte del tempo sì, a tratti no.

Ce l'avrei fatta senza Gemini? Ma col cazzo.

Ok, il coso artificiale di quell'assoluto sciroccato che fa i nudini della gente su Twitter, ok i brainrot, ok lo slop, le fake news, la postverità, il consumo energetico, ma a me a sto giro l'intelligenza artificiale ha svoltato un progetto di cui sono enormemente orgoglioso, quindi questa la mettiamo assolutamente nella categoria "ha fatto anche cose buone", assieme alle lunghe e approfondite dissertazioni letterarie che ci facciamo assieme.