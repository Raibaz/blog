---
title: 'Autenticazione via token JWT con Angular e Spring Boot: parte 1, Google login con Angular'
author: Raibaz
type: post
date: 2019-10-09T07:50:24+00:00
url: /2019/10/autenticazione-via-token-jwt-con-angular-e-spring-boot-parte-1-google-login-con-angular/
categories:
  - Nerd

---
[Nella puntata precedente][1] abbiamo descritto il flusso che abbiamo in mente di implementare per separare il frontend e il backend dell&#8217;applicazione su cui lavoro e renderli indipendenti.

Il primo step è far fare l&#8217;autenticazione via Google login all&#8217;applicazione frontend anziché averla incastrata nel backend.

Per farlo, ci viene in aiuto [angularx-social-login][2], un modulo per Angular 4+ che permette di gestire proprio il login con i provider social, come Facebook e, appunto, Google.

La logica di login la incapsuliamo in un service, che esporrà all&#8217;esterno:

  * I metodi `login`e `logout`, abbastanza autoesplicativi
  * Un metodo `getAuthorizationToken`, che ritorna il token JWT da usare per l&#8217;autenticazione di tutte le chiamate all&#8217;API
  * Un metodo `getLoginState`, che ritorna un `Observable`a cui registrarsi per reagire ai cambiamenti di stato del login dell&#8217;utente 

Il motivio per cui ci serve `getLoginState` è che i token JWT non sono eterni, ma a un certo punto scadranno e le richieste all&#8217;API ritorneranno 401, per cui sarà necessario gestire questo caso refreshando il token o sloggando l&#8217;utente.

angularx-social-login stesso, a sua volta, espone una proprietà `authState` observable, a cui registrarsi per reagire ai cambiamenti di stato del login social: quello che faremo, quindi, è questo:

<pre class="wp-block-code"><code lang="typescript" class="language-typescript">    this.authService.authState.subscribe((user) => {
      if (user !== null) {
        this.loginToBackend(user.email, user.idToken).subscribe((loggedIn) => {
          this._loginState.next(loggedIn);
        });
      } else {
        this._loginState.next(false);
      }
    });</code></pre>

Quando cambia lo stato di login social, se l&#8217;utente è loggato (`user !== null`) passiamo token ed email al backend e usiamo la risposta per pubblicare un aggiornamento dello stato del login dell&#8217;applicazione, altrimenti pubblichiamo un aggiornamento di stato dicendo all&#8217;applicazione che l&#8217;utente non è loggato.

Il login al backend altro non è che una POST a `/authenticate` con email e idToken di Google, che ritorna un token JWT; il service lo salverà da qualche parte e lo ritornerà col metodo `getAuthorizationToken`.

Nel mio caso, il token viene salvato nel `sessionStorage`; è possibile anche salvarlo in un cookie, e la scelta ha delle implicazioni di sicurezza e implementative non banali che esulano da questa serie di post.

Nella [prossima puntata][3], in compenso, inizieremo a vedere come configurare Spring Boot per gestire l&#8217;autenticazione e in seguito vedremo come è implementato il metodo `/authenticate`.

 [1]: https://raibaz.it/2019/10/autenticazione-via-token-jwt-con-angular-e-spring-boot/
 [2]: https://github.com/abacritt/angularx-social-login
 [3]: https://raibaz.it/2019/10/autenticazione-via-token-jwt-con-angular-e-spring-boot-parte-2-configurazione-di-spring-boot/