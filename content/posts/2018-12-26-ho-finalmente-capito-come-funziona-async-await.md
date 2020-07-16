---
title: Ho finalmente capito come funziona async/await
author: Raibaz
type: post
date: 2018-12-26T11:09:12+00:00
url: /2018/12/ho-finalmente-capito-come-funziona-async-await/
categories:
  - Nerd

---
Per lavoro ormai non scrivo più tanto codice, e quel poco che scrivo non è (fortunatamente, sfortunatamente, dipende dai gusti) Javascript, il linguaggio in più rapida evoluzione degli ultimi anni.

Sono stato senza scrivere Java per lunghi periodi, sono stato per lunghi e felicissimi periodi senza scrivere PHP e quando li ho ripresi in mano erano esattamente come li avevo lasciati, mentre Javascript è cambiato radicalmente pure nel corso degli anni in cui lo scrivevo tutti i giorni, figurarsi adesso che ho perso un po&#8217; il contatto.

Cerco di tenermi aggiornato, ma scrivere il codice è, come tutti i lavori pratici, qualcosa con cui si perde la mano se non lo si fa tutti i giorni: caso ha voluto che avessi un progettino da fare nel (poco) tempo libero, però, e un po&#8217; perché comunque so come funziona e un po&#8217; per fare quello che in gergo si chiama &#8220;affilare l&#8217;ascia&#8221; ho deciso di farlo, per l&#8217;appunto, in Javascript.

Tra l&#8217;altro, è una cosa che deve fare diverse richieste HTTP in sequenza usando di volta in volta le risposte ricevute come parametri per le richieste successive, che è un po&#8217; un grande classico delle applicazioni node e un pain point piuttosto famoso in Javascript.

All&#8217;inizio, come era inevitabile che fosse, l&#8217;ho scritto come lo sapevo scrivere, per cui sono finito ben presto nel più classico dei [callback hell][1] e, poco più tardi, nel più recente ma altrettanto classico [promise hell][2]:

<pre class="wp-block-code"><code lang="javascript" class="language-javascript">axios.get(url)
    .then((result) => {
        // Fai qualcosa col risultato della get
        axios.post(postUrl, result.data)
            .then((postResult) => {
                //Fai qualcosa col risultato della post
                axios.get(anotherGetUrl)
                    .then((getResult) => {
                        //Fai qualcos'altro
                    })
                    .catch((error) => console.log(error))
            }).catch((error) => console.log(error))
    }).catch((error) => console.log(error));</code></pre>

Ora ok, sicuramente il codice lì sopra è pienissimo di errori da n00b, ma maledetto tutto quanto quelle tre catch lì messe a freccia mi fanno salire il sangue al cervello e mi fanno venir voglia di piangere: non esiste al mondo che riesca a dormire la notte sapendo di aver scritto codice del genere.

Dato che si parla di affilare l&#8217;ascia, quindi, e che comunque il problema in questione sembra essere piuttosto comune, è stato relativamente semplice trovare una soluzione nei costrutti (ormai nemmeno troppo) nuovi del linguaggio, che nelle sue versioni più recenti ha aggiunto dello [zucchero sintattico][3] volto appunto a risolvere questo tipo di problemi.

Il caso in questione, quindi, nel Javascript del 2018 quasi 2019, si scrive molto meglio così:

<pre class="wp-block-code"><code lang="javascript" class="language-javascript">const faiCoseHttp = async () => {
    try {
        let getResult = await axios.get(url);
        //Fai cose con getResult
        let postResult = await axios.post(postUrl, getResult.data);
        //Fai cose con postResult
        let anotherGetResult = await axios.get(anotherGetUrl);
    } catch (error) {
        console.log(error);
    }
}</code></pre>

Meno righe, meno boilerplate, flusso più chiaro, codice più leggibile: la figata di async/await è che ti permette di distinguere tra codice asincrono e codice parallelo: callback e promises sono meccanismi più potenti, in un certo senso, perché permettono di scrivere codice che faccia più cose contemporaneamente (anche se, in un linguaggio single-threaded, il parallelismo è sempre illusorio), ma quando hai delle cose da fare in sequenza, seppure asincrona, è facile scrivere del codice agghiacciante come quello del mio primo esempio.

Usare async e await, invece, permette di scrivere del codice che fa cose sì asincrone, ma comunque in sequenza, aspettando di volta in volta che le operazioni asincrone finiscano.

Ok, è syntactic sugar e forse avrei potuto scrivere chiaro, conciso e leggibile anche usando correttamente le promises, ma chi me lo fa fare, se posso farlo molto più in fretta e semplicemente con un costrutto più recente? E soprattutto, se è un buon pretesto per imparare decentemente come funziona un costrutto recente che non avevo ancora avuto occasione di usare?

 [1]: http://callbackhell.com/
 [2]: https://medium.com/@pyrolistical/how-to-get-out-of-promise-hell-8c20e0ab0513
 [3]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function