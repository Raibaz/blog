---
title: Unit testing delle rotte di Apache Camel
author: Raibaz
type: post
date: 2019-11-19T08:35:00+00:00
url: /2019/11/unit-testing-delle-rotte-di-apache-camel/
categories:
  - Nerd

---
Supponiamo che abbiate un&#8217;applicazione che usa Camel per prendere dei dati da una sorgente, trasformarli in qualche modo e mandarli da qualche altra parte, tipo una cosa del genere:

<div style="height: 250px; position:relative; margin-bottom: 50px;" class="wp-block-simple-code-block-ace">
  <pre class="wp-block-simple-code-block-ace" style="position:absolute;top:0;right:0;bottom:0;left:0" data-mode="kotlin" data-theme="solarized_dark" data-fontsize="14" data-lines="Infinity" data-showlines="true" data-copy="false"> class MyRouteBuilder : RouteBuilder() {
 
 override fun configure() {
     from("ftp://qualchecosa")
        .log("Connected to FTP")
        .onException(SupplierLockedException::class.java)
            .backOffMultiplier(supplierLockedRetryBackoff)
            .redeliveryDelay(supplierLockedRetryDelay)
            .maximumRedeliveries(supplierLockedRetryMaxRetries)
        .end()
        .onException(Exception::class.java)
            .bean("exceptionHandlerBean")
        .end()
        .setProperty("qualchecosa", "qualchecosaltro")
        .bean("supplierLockBean", "lock")
        .choice()
            .`when`(exchangeProperty("zip"))
            .unmarshal()
            .zipFile()
            .log("Unzip complete")
        .end()
        .bean("s3UploaderBean")
        .bean("qualchecosaBean")
        .bean("supplierLockBean", "unlock")
        .log("Conversion complete")
        .log("Upload DataLake")
        .bean("s3DataLakeUploaderBean")
        .choice()
            .`when`(writeCsv)
                .multicast()
                    .to("direct:sendToFtp")
                    .to(rabbitMqEndpoint)
                    .to("micrometer:counter:qualchecosa")
            .endChoice()
            .otherwise()
                .to(rabbitMqEndpoint)
                .to("micrometer:counter:qualchecosa")
        .end()
    }
}</pre>
</div>

_Nota: i miei esempi sono in Kotlin, ma tradurli in Java è banale_.

Senza stare ad arrovellarsi troppo su quello che succede, è una roba che legge un file da un FTP, lo passa attraverso una serie di bean ognuno dei quali fa qualcosa, tipo caricare dei dati su S3, o trasformarli in qualche modo, o lockare/unlockare qualcosa, in qualche occasione fa anche delle scelte, quindi ha della logica interna, e poi manda tutto in degli altri posti.

Mettiamo caso che siate dei bravi sviluppatori, e che quindi vogliate scrivere degli unit test per essere sicuri che i bean vengano eseguiti nell&#8217;ordine giusto, con i parametri giusti e che tra sei mesi, quando cambiate una virgola da qualche parte, non introduciate delle regressioni per cui i file non vengono più mandati come dovrebbero.

Come si testa questa roba? Camel, che è un progetto Apache e quindi vuole bene ai bravi sviluppatori, ha [un modulo apposta][1] per testare le cose.

Tutto inizia facendo estendere alla classe dei vostri test la classe `CamelTestSupport`: questo farà sì che eseguendo i test contenuti al suo interno Camel crei il suo contesto e lo esegua, con i tweak opportuni legati al fatto che siamo in ambiente di test.

Per dire che vogliamo testare le rotte del routebuilder in questione, si fa l&#8217;override di `createRouteBuilder`: 

<div style="height: 250px; position:relative; margin-bottom: 50px;" class="wp-block-simple-code-block-ace">
  <pre class="wp-block-simple-code-block-ace" style="position:absolute;top:0;right:0;bottom:0;left:0" data-mode="kotlin" data-theme="solarized_dark" data-fontsize="14" data-lines="Infinity" data-showlines="true" data-copy="false">override fun createRouteBuilder(): RoutesBuilder {
    return MyRouteBuilder(conf)
}</pre>
</div>

Ed ecco il primo problema: è uno unit test, che realisticamente voglio eseguire anche nel mio ambiente di CI, per cui non ho a disposizione, nè voglio averli, un server FTP da cui leggere o una coda RabbitMQ su cui scrivere: come faccio a dire al mio test di leggere e scrivere da altre parti?

Camel-test fa anche questo, con `adviceWith` che permette di sovrascrivere alcune proprietà di una rotta con altre più utili al contesto di un test.

Per farlo, è sufficiente dire al test che si vuole usarlo e, appunto, chiamare `adviceWith` sulla rotta da modificare:

<div style="height: 250px; position:relative; margin-bottom: 50px;" class="wp-block-simple-code-block-ace">
  <pre class="wp-block-simple-code-block-ace" style="position:absolute;top:0;right:0;bottom:0;left:0" data-mode="kotlin" data-theme="solarized_dark" data-fontsize="14" data-lines="Infinity" data-showlines="true" data-copy="false">@EndpointInject(uri = "mock:testEndpoint")
private var mockEndpoint = MockEndpoint()


override fun isUseAdviceWith(): Boolean { return true }

@Before
fun setup() {
    context.routeDefinitions[0].adviceWith(context, object : AdviceWithRouteBuilder() {
        override fun configure() {
            replaceFromWith("direct:test")
            weaveByToString&lt;RouteDefinition>(".*rabbitmq.*").replace().to("mock:testEndpoint")
        }
    })
    context.start()
}</pre>
</div>

Qui, oltre a dire che il test usa `adviceWith`, stiamo sostituendo il from della rotta con un endpoint sul quale potremo scrivere dati a nostro piacimento per guidare il comportamento della rotta stessa, e già che ci siamo sostituiamo la destinazione RabbitMQ con un `MockEndpoint`, sul quale poi potremo fare delle asserzioni per verificare che tutto avvenga come ci aspettiamo.

Nota: `AdviceWithRouteBuilder` ha anche un metodo `interceptSendToEndpoint` che serve a intercettare i dati mandati agli endpoint e farci cose utili per i test, ma faceva un po&#8217; a pugni con le mie rotte RabbitMQ per cui ho deciso di prendere l&#8217;endpoint e sostituirlo del tutto, tanto per lo scopo del mio test non fa differenza.

Se eseguite un test, anche vuoto, a questo punto, ottenete comunque un errore, perché quando Camel prova a prendere i bean dal contesto JNDI non trova niente. Il pezzo mancante del puzzle, quindi, è questo:

<div style="height: 250px; position:relative; margin-bottom: 50px;" class="wp-block-simple-code-block-ace">
  <pre class="wp-block-simple-code-block-ace" style="position:absolute;top:0;right:0;bottom:0;left:0" data-mode="kotlin" data-theme="solarized_dark" data-fontsize="14" data-lines="Infinity" data-showlines="true" data-copy="false">override fun createJndiContext(): Context {
    val context = JndiContext()
    context.bind("supplierLockBean", mockSupplierLockBean)
    context.bind("s3UploaderBean", mockS3UploaderBean)
    context.bind("qualchecosaBean", mockQualchecosaBean)
    context.bind("s3DataLakeUploaderBean", mockS3DataLakeUploaderBean)
    context.bind("writeCsvBean", mockWriteCsvBean)
    context.bind("exceptionHandlerBean", mockExceptionHandlerBean)

    return context
}</pre>
</div>

Che crea un contesto JNDI e ci binda dentro una serie di oggetti mockati (io uso [MockK][2], ma potete usare Mockito o il framework di mocking che più vi piace) che nel test poi andremo a pilotare e ispezionare alla bisogna.

Ora è tutto pronto, possiamo sbizzarrirci a testare cose, tipo ad esempio:

<div style="height: 250px; position:relative; margin-bottom: 50px;" class="wp-block-simple-code-block-ace">
  <pre class="wp-block-simple-code-block-ace" style="position:absolute;top:0;right:0;bottom:0;left:0" data-mode="kotlin" data-theme="solarized_dark" data-fontsize="14" data-lines="Infinity" data-showlines="true" data-copy="false">@Test
fun `It sends to the final endpoint`() {
    mockEndpoint.expectedMessageCount(1)
    template.sendBody("direct:test", "test body")
    mockEndpoint.assertIsSatisfied()
}</pre>
</div>

Ecco svelato a cosa serviva il `MockEndpoint` di prima: possiamo verificare che la rotta arrivi in fondo dichiarando che ci aspettiamo che arrivi un messaggio se mandiamo un messaggio al from.

Oppure possiamo fare buon uso dei mock che abbiamo iniettato nel contesto JNDI, verificando che lock e unlock vengano eseguite in sequenza:

<div style="height: 250px; position:relative; margin-bottom: 50px;" class="wp-block-simple-code-block-ace">
  <pre class="wp-block-simple-code-block-ace" style="position:absolute;top:0;right:0;bottom:0;left:0" data-mode="kotlin" data-theme="solarized_dark" data-fontsize="14" data-lines="Infinity" data-showlines="true" data-copy="false">@Test
fun `It locks and unlocks the supplier`() {
    template.sendBody("direct:test", "test body")
    verifySequence {
        mockSupplierLockBean.lock(any())
        mockSupplierLockBean.unlock(any())
    }
}</pre>
</div>

Insomma, avete capito, a questo punto l&#8217;infrastruttura per testare la rotta è a posto e sta a voi (o a me) sbizzarrirsi a testare tutto quello che succede.

 [1]: https://camel.apache.org/manual/latest/testing.html
 [2]: https://mockk.io/