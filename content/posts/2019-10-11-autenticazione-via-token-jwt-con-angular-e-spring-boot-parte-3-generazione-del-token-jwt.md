---
title: 'Autenticazione via token JWT con Angular e Spring Boot: parte 3, generazione del token JWT'
author: Raibaz
type: post
date: 2019-10-11T07:51:01+00:00
url: /2019/10/autenticazione-via-token-jwt-con-angular-e-spring-boot-parte-3-generazione-del-token-jwt/
categories:
  - Nerd

---
_(Questo è il quarto post di una serie che inizia [qui][1] e prosegue [qui][2] e [qui][3])_

Nelle puntate precedenti:

  * Abbiamo descritto come funziona il flusso dell&#8217;autenticazione
  * Abbiamo visto l&#8217;implementazione del Google login nell&#8217;applicazione Angular
  * Abbiamo visto come configurare Spring Boot per gestire l&#8217;autenticazione

Ci resta solo da vedere come è effettivamente implementato l&#8217;endpoint `/authenticate`, che valida l&#8217;utente e gli restituisce un token JWT.

Il servizio invocato dal controller è fatto così:

<pre class="wp-block-code"><code lang="kotlin" class="language-kotlin">@Service
class JwtAuthenticationService(
    private val userService: UserService,
    private val googleIdTokenVerifierService: GoogleIdTokenVerifierService,
    private val jwtTokenService: JwtTokenService
) {

    fun authenticate(request: JwtAuthenticationRequest): JwtAuthenticationResponse {
        userService.findByEmail(request.username)
            ?: throw UsernameNotFoundException("User with email $request.username not found.")

        googleIdTokenVerifierService.verifyToken(request.googleToken)

        val jwtToken = jwtTokenService.generateToken(request.username)
        return JwtAuthenticationResponse(jwtToken, "")
    }
}</code></pre>

Non fa niente di complicato, in effetti, fa solo tre cose:

  * Valida lo username sul DB chiedendo allo `userService` se esiste effettivamente un utente con quello username
  * Valida il token ritornato al frontend dall&#8217;autenticazione di Google
  * Genera un token JWT e lo restituisce

La validazione su DB è banale, per cui confido che non serva dilungarcisi ulteriormente; anche quella del token di Google non è particolarmente complicata, grazie alla dipendenza da `com.google.api-client:google-api-client` che mette a disposizione la classe `GoogleIdTokenVerifier`.

<pre class="wp-block-code"><code lang="kotlin" class="language-kotlin">@Service
class GoogleIdTokenVerifierService {

    @Value("\${security.oauth2.client.clientId}")
    val clientId: String = ""

    fun verifyToken(token: String): GoogleIdToken {
        val verifier = GoogleIdTokenVerifier
                .Builder(NetHttpTransport(), JacksonFactory.getDefaultInstance())
                .setAudience(listOf(clientId))
                .build()

        return verifier.verify(token) ?: throw IllegalArgumentException()
    }
}</code></pre>

A questo punto, ci resta solo da generare il token JWT:

<pre class="wp-block-code"><code lang="kotlin" class="language-kotlin">fun generateToken(username: String): String {
    return Jwts.builder()
        .setClaims(mutableMapOf())
        .setSubject(username)
        .setIssuedAt(Date())
        .setExpiration(Date(System.currentTimeMillis() + (jwtValidity * 10000)))
        .signWith(SignatureAlgorithm.HS256, jwtSecret)
        .compact()
}</code></pre>

E il gioco è fatto: come abbiamo visto nelle puntate precedenti, il frontend passerà il token così generato e appena ottenuto nell&#8217;header `Authorization` di ogni richiesta al backend, permettendo quindi di autenticarsi.

La parte più critica in tutto questo cinema, l&#8217;avrete intuito, è la configurazione dell&#8217;autenticazione di Spring Boot, che ha una serie di _gotcha_ e momenti _AHA!_ niente affatto banali che mi hanno fatto sbattere la testa ripetutamente contro il muro e mi hanno spinto a scrivere questa serie di post, nella speranza di aiutare i poveri sventurati che si troveranno nella mia stessa situazione.

Come si suol dire, il modo migliore per imparare qualcosa è spiegarlo a qualcun&#8217;altro.

 [1]: https://raibaz.it/2019/10/autenticazione-via-token-jwt-con-angular-e-spring-boot/
 [2]: https://raibaz.it/2019/10/autenticazione-via-token-jwt-con-angular-e-spring-boot-parte-1-google-login-con-angular/
 [3]: https://raibaz.it/2019/10/autenticazione-via-token-jwt-con-angular-e-spring-boot-parte-2-configurazione-di-spring-boot/