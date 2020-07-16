---
title: Autenticazione via token JWT con Angular e Spring Boot
author: Raibaz
type: post
date: 2019-10-08T13:49:48+00:00
url: /2019/10/autenticazione-via-token-jwt-con-angular-e-spring-boot/
categories:
  - Nerd

---
L&#8217;applicazione su cui lavoro è un monolite col frontend e il backend nello stesso progetto, fatti rispettivamente con Angular 8 e Spring Boot; l&#8217;autenticazione, quindi, è gestita interamente dal backend tramite cookie.

Quando un utente accede tramite Google Sign-in, Spring Boot gli setta un cookie col JSESSIONID e uno col token XSRF, che poi verranno passati anche per ogni richiesta ajax del frontend.

Ora, per una serie di motivi abbiamo deciso di spostare il frontend in un&#8217;applicazione separata e lasciare il backend a esporre solo un&#8217;API REST: sembra facile, ma nel momento in cui le due applicazioni sono su due domini diversi il backend non può più settare dei cookie ad uso del frontend.

Serve un&#8217;altra soluzione.

Il flusso di login da implementare, quindi, è una cosa così:<figure class="wp-block-image">

<img src="https://raibaz.it/wp-content/uploads/2019/10/Schermata-2019-10-08-alle-12.43.52-878x1024.png" alt="" class="wp-image-225" srcset="https://www.raibaz.it/wp-content/uploads/2019/10/Schermata-2019-10-08-alle-12.43.52-878x1024.png 878w, https://www.raibaz.it/wp-content/uploads/2019/10/Schermata-2019-10-08-alle-12.43.52-257x300.png 257w, https://www.raibaz.it/wp-content/uploads/2019/10/Schermata-2019-10-08-alle-12.43.52-768x896.png 768w, https://www.raibaz.it/wp-content/uploads/2019/10/Schermata-2019-10-08-alle-12.43.52.png 890w" sizes="(max-width: 878px) 100vw, 878px" /> </figure> 

Il frontend effettua l&#8217;autenticazione con Google Sign-in e passa i dati ricevuti da Google a un endpoint del backend, che li valida sia nuovamente con l&#8217;autenticazione di Google che col proprio DB, per verificare che l&#8217;utente che sta accedendo sia effettivamente un utente valido per entrambi.

Fatto questo, l&#8217;endpoint restituisce al frontend un token JWT, che quest&#8217;ultimo userà per autenticare tutte le richieste successive.

Come si fa tutto ciò? È relativamente facile, ma ci sono un paio di _gotcha_ a cui stare attenti che rischiano di rendere tutto piuttosto fastidioso, dato che quando c&#8217;è qualcosa che non va il massimo che potete sperare di ottenere è un poco chiarificatore HTTP 403.

Dato che ci sono diversi pezzi coinvolti e fare tutto in un muraglione di testo diventerebbe troppo noioso e impossibile da seguire, ho deciso di dividere il post in una serie di puntate, anche per creare più _suspence_, per cui per oggi basta così: nella prossima puntata vedremo la parte di autenticazione con Google nel frontend.

Avanti allora: [Parte 1 &#8211; Google login con Angular][1].

 [1]: https://raibaz.it/2019/10/autenticazione-via-token-jwt-con-angular-e-spring-boot-parte-1-google-login-con-angular/