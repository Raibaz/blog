---
title: La poesia dei sudoku
author: Raibaz
type: post
date: 2018-11-21T08:10:45+00:00
url: /2018/11/la-poesia-dei-sudoku/
categories:
  - Cose di cultura

---
Sono un appassionato di sudoku da un sacco di tempo e nonostante un sacco di gente li detesti (anche se ci sono degli appassionati insospettabili, tipo che di fianco alla tazza del cesso di casa dei miei genitori c&#8217;è sempre un giornalino coi sudoku e una matita), al punto che qualche anno fa in un&#8217;estate ricca di tempo libero e di voglia di imparare cose nuove ho scritto [un coso in Ruby][1] che li risolve.

Non sono neanche lontanamente un professionista, mi sono stufato di approfondirne il funzionamento ben prima di arrivare a capire tutto il funzionamento di [questo algoritmo][2] elegantissimo di Peter Norvig che sostiene di essere in grado di risolvere qualunque sudoku del mondo, ma di recente mi sono interrogato (sì, di recente mi interrogo un sacco, e non è sempre un bene) su cosa sia che effettivamente trovo così attraente in una cazzo di griglia con dei numeri da completare.<figure class="wp-block-image">

<img src="https://raibaz.it/wp-content/uploads/2018/11/Schermata-2018-11-20-alle-21.23.10.png" alt="" class="wp-image-114" srcset="https://www.raibaz.it/wp-content/uploads/2018/11/Schermata-2018-11-20-alle-21.23.10.png 866w, https://www.raibaz.it/wp-content/uploads/2018/11/Schermata-2018-11-20-alle-21.23.10-150x150.png 150w, https://www.raibaz.it/wp-content/uploads/2018/11/Schermata-2018-11-20-alle-21.23.10-300x298.png 300w, https://www.raibaz.it/wp-content/uploads/2018/11/Schermata-2018-11-20-alle-21.23.10-768x763.png 768w" sizes="(max-width: 866px) 100vw, 866px" /> </figure> 

C&#8217;è sicuramente un pochino di quell&#8217;OCD insito in qualunque nerd che si rispetti, nel fascino di estrarre dell&#8217;ordine da una situazione inizialmente criptica, come insita in qualunque nerd che si rispetti c&#8217;è della fascinazione per l&#8217;enigmistica, ma a modo suo credo che il sudoku sia anche, in diversi modi, una discreta metafora della vita.

Nei sudoku particolarmente difficili, ad esempio, capita spesso e volentieri di riempire una casella apparentemente slegata da tutte le altre, che sembrerebbe non dare nessun vantaggio rilevante e rendersi conto solo dopo che ne ha sbloccate alcune che sembrava non c&#8217;entrassero niente: altre volte capita davvero che riempire una casella non dia alcun vantaggio rilevante, ma non per questo non la si riempie, perché è comunque una casella in meno da riempire per arrivare a ottantuno, e ogni passo in avanti, anche minimo, conta.

In anni e anni di esperienza da &#8220;sudokista&#8221;, poi, ho imparato che esistono dozzine di tecniche diverse per approcciare il problema: puoi scegliere di partire dai numeri che compaiono tante volte e vedere se per caso riesci a metterli tutti e nove per poter non pensarci più, oppure puoi partire dai numeri che compaiono una volta sola e cercare di riempire la riga, la colonna o il quadrante che compaiono, sapendo che siccome hanno un numero &#8220;singolo&#8221; dovrebbe essere, almeno in teoria, più facile riempirli; puoi concentrarti sulle righe o le colonne con poche caselle vuote e cercare di completarle, oppure puoi prendere in considerazione quelle con tanti spazi vuoti ma dai quali puoi escludere dei numeri, e ovviamente quando pensi di essere bloccato la cosa migliore da fare è provare un&#8217;altra strada.

Nella mia conoscenza piuttosto naïf del funzionamento dei sudoku, inoltre, c&#8217;è una certa pigrizia che mi impedisce di proseguire con i sudoku in cui dovrei infilarmi in un albero di scelte col rischio di finire in un cul de sac, risalire fino alla scelta iniziale e prendere l&#8217;altra strada; in quel caso, invero piuttosto frequente nei sudoku molto difficili, banalmente sfanculo la griglia e ne inizio una nuova, quindi ho imparato anche a evitare di restare troppo incastrato su un problema: se risolverlo richiede un investimento di tempo e scelte che potrebbe rivelarsi fallimentare, passo ad altro.

Non so dire se questa filosofia di approccio ai sudoku sia quella giusta anche nella vita di tutti i giorni, anzi, probabilmente non lo è, ma sticazzi, è vero che i sudoku sono la metafora della vita ma le metafore spesso sono incomplete o parziali.

In ogni caso, sarà perché è un&#8217;invenzione dei giapponesi che sono maestri in questo stile, o perché sono un asino e trovo della poesia dove non c&#8217;è, ma credo davvero che nel prendere una griglia di numeri semivuota e ricavarne un insieme ordinato e coerente secondo le proprie, basilari, regole interne, sia una cosa estremamente poetica ed elegante.

 [1]: https://github.com/Raibaz/rdoku
 [2]: http://norvig.com/sudoku.html