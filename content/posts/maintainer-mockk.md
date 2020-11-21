+++ 
draft = false
date = 2020-11-23T09:06:08+01:00
title = "Come sono diventato maintainer di un progetto open source grosso"
description = ""
slug = "come-sono-diventato-maintainer" 
tags = []
categories = []
externalLink = ""
series = []
+++

Come tanti altri che fanno il mio mestiere, sono un grande appassionato di open source.

È inutile che stia a spiegare il valore infinito che la comunità open source ha donato al mondo del software, e di conseguenza, al mondo, nel corso degli anni, e per spiegare che valore abbia in termini di formazione ci ho persino fatto il primo episodio della [serie di panel](https://www.raibaz.it/posts/meetup-codemotion/) che ho moderato questo autunno.

Negli anni, ho rilasciato su GitHub [alcune cose che ho scritto](https://github.com/Raibaz), a tempo perso, e contribuito a qualche progetto, principalmente per fare quello che in gergo si chiama "scratch my own itch": quando trovavo qualcosa che non andava o che si poteva migliorare in un progetto che usavo, mandavo una pull request.

Quello che mi ha sempre stupito, e che stupisce spesso tutti quelli che mandano pull request a progetti open source, grossi e piccoli che siano, è la gentilezza con cui è sempre stato accolto il mio contributo: da progetti di nicchia scritti da una persona sola nel tempo libero a tool molto più "established" con team di decine di persone dietro, ogni volta che ho, di fatto, detto a qualcuno di loro "ho trovato questo problema, e anche la soluzione, già che c'ero" la reazione è stata una delle sfumature possibili di "grazie, sei il re, ti vogliamo bene!"

Fin qui, però, è storia generica e comune a moltissime persone del mio mestiere.

Ora, la mia storia invece vuole che a un certo punto, nel mio lavoro attuale, mi imbatta in un [framework di mocking](https://mockk.io) per Kotlin piuttosto bello e ben scritto: il mocking, in particolare, è un tema che mi piace molto, da quando l'ho imparato su [uno dei libri fondamentali](https://www.amazon.it/Growing-Object-Oriented-Software-Guided-Tests/dp/0321503627/ref=sr_1_1?__mk_it_IT=%C3%85M%C3%85%C5%BD%C3%95%C3%91&crid=1RUS7TR6M9UWU&dchild=1&keywords=growing+object-oriented+software%2C+guided+by+tests&qid=1605993267&sprefix=growing+ob%2Caps%2C171&sr=8-1&tag=raibaz-21) della storia della computer science, e questo framework è così ben scritto che a un certo punto il mio amico Fabio mi invita per un talk all'Xpug Milano e decido di parlarne:

{{<youtube 8UtNasobzUI >}}

E, ovviamente, nell'usarlo estensivamente mi imbatto in alcune piccole pecche, o mancanze di cose che mi farebbero comodo, per risolvere le quali mando prontamente PR.

Già che ci sono, dò una mano anche a quelli che si imbattono in qualche problema e aprono issue, spiegando loro come usare alcune feature non immediatissime.

Il caso vuole, però, che il maintainer principale del progetto, nonché quello che ne ha scritto un buon 90%, abbia una serie di cazzi personali che gli impediscono di dedicargli il tempo che meriterebbe, per cui c'è un po' di polemica, ogni tanto, per via del fatto che un progetto così utile e, in fondo, così ben fatto, necessiterebbe di attenzione, manutenzione ed evoluzioni più frequenti e attente di quanto non abbia in realtà: non è in fondo tantissimo un mio problema, perché per l'uso che ne faccio io va tutto sommato bene così, ma cerco comunque di dare il po' di supporto che posso.

Fino a quando, un giorno.

Oleksiy, il maintainer di cui sopra, mi manda una mail, senza che ci fossimo mai parlati direttamente se non sugli issue di Github, per dirmi, fondamentalmente "oh, io non ce la faccio a stare dietro al progetto adesso, se vuoi ti dò gli accessi e lo mantieni tu".

Ora, fondamentalmente è un'inculata: è un progetto usato da un sacco di gente, che in questo preciso momento ha centotrentaquattro issue aperti e una comunità di utenti molto vasta e un po' scoglionata dalla scarsa attività degli ultimi tempi, per cui prendersene carico, ovviamente aggratis, non promette niente di buono se non un sacco di tempo da impiegarci e poche (if any) gratificazioni a breve termine.

Quindi, se siete lettori attenti, sapete già cosa gli ho risposto: da una dozzina di giorni sono, di fatto, il maintainer principale del framework standard _de facto_ di mocking per Kotlin.

Ora, se per caso siete arrivati a leggere fino qua e avete del tempo libero che volete devolvere a una comunità, quella open source, che tanto ha dato a tutti noi, e a un personaggio, me stesso medesimo, altrettanto caro a tutti noi e a me in particolare, e magari conoscete un po' di Kotlin, oppure no e volete impararlo, oppure no ma vi va di darmi una mano, a me un co-maintainer farebbe davvero comodo :)