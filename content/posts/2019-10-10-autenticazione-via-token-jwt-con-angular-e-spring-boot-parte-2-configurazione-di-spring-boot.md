---
title: 'Autenticazione via token JWT con Angular e Spring Boot: parte 2, configurazione di Spring Boot'
author: Raibaz
type: post
date: 2019-10-10T07:50:47+00:00
url: /2019/10/autenticazione-via-token-jwt-con-angular-e-spring-boot-parte-2-configurazione-di-spring-boot/
categories:
  - Nerd

---
_(Nota: questo è il terzo post di una serie che inizia [qui][1] e prosegue [qui][2])_

Nella puntata precedente abbiamo visto come far fare il social login all&#8217;applicazione frontend, ora è il momento di configurare l&#8217;applicazione backend perché non permetta ad utenti non autenticati di accedere all&#8217;API.

Premessa: la mia applicazione backend è scritta in Kotlin, per cui i pezzi di codice che incollerò potranno essere leggermente diversi da quelli della maggior parte dei tutorial che si trovano in rete, che sono scritti in Java, ma confido che si riescano a leggere lo stesso.

La prima cosa da fare per configurare un&#8217;applicazione Spring Boot per avere l&#8217;autenticazione è, appunto, avere una classe di configurazione per l&#8217;autenticazione: per farlo, è sufficiente annotare una classe con `@Configuration`, `@EnableWebSecurity` e `@EnableGlobalMethodSecurity(prePostEnabled = true)`.

La classe in questione, che nella mia applicazione si chiama `JwtSecurityConfiguration`, estende `WebSecurityConfigurerAdapter`, e ne fa l&#8217;override di alcuni metodi, il più importante dei quali è `configure(http: HttpSecurity)`, all&#8217;interno del quale dichiarativamente si può impostare la configurazione dell&#8217;autenticazione HTTP.

Di seguito la mia configurazione:

<pre class="wp-block-code"><code lang="kotlin" class="language-kotlin">val aep = AuthenticationEntryPoint { request, response, authException ->
    if (authException is InsufficientAuthenticationException && request.requestURI.contains("/api/v1")) {
        response.status = 401
        response.writer.write(authException.message)
        response.writer.flush()
        response.writer.close()
    } else {
        response.sendRedirect("/")
    }
}

http
.authorizeRequests()
.antMatchers("/api/v1/authenticate", "/", "/app/**/*").permitAll()
.anyRequest().fullyAuthenticated()
.and()
.logout().logoutRequestMatcher(AntPathRequestMatcher("/logout"))
.logoutSuccessUrl("/")
.and()
.cors()
.and()
.csrf().disable()
.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
.and()
.exceptionHandling()
.authenticationEntryPoint(aep)
.and()
.addFilterBefore(JwtLoginFilter(jwtTokenService), UsernamePasswordAuthenticationFilter::class.java)</code></pre>

La dichiarazione è piuttosto lunga e articolata, quindi vediamola un pezzo per volta:

Innanzitutto diciamo che gli URL che corrispondono a `/api/v1/authenticate`, `/` e alla cartella degli asset statici sono accessibili anche dagli utenti non autenticati, mentre tutti gli altri richiedono che l&#8217;utente sia fully authenticated.

Poi definiamo qual è l&#8217;URL che slogga gli utenti (banalmente, `/logout`) e dove redirigerli dopo che si sono sloggati, abilitiamo CORS e disabilitiamo CSRF e stabiliamo come gestire le sessioni (in maniera stateless, perché il token JWT tanto verrà passato con ogni richiesta).

Poi ancora definiamo come gestire le eccezioni di autenticazione, che vedete in alto nel mio blocco di codice: se si tratta di una `AuthenticationException` lanciata mentre si faceva una richiesta all&#8217;API, allora il metodo API deve ritornare 401, altrimenti il browser viene rediretto all&#8217;home page.

Infine, mettiamo un filtro nella catena di filtri che viene eseguita prima di ogni richiesta per verificare che ci sia l&#8217;header `Authorization` con il token JWT.

Il filtro in questione è fatto così:

<pre class="wp-block-code"><code lang="kotlin" class="language-kotlin">class JwtLoginFilter(private val jwtTokenService: JwtTokenService) : OncePerRequestFilter() {

    private val headerName = "Authorization"
    private val headerPrefix = "Bearer "

    override fun doFilterInternal(
        request: HttpServletRequest,
        response: HttpServletResponse,
        filterChain: FilterChain
    ) {
        val authorizationHeader = request.getHeader(headerName)

        if (authorizationHeader != null && authorizationHeader.startsWith(headerPrefix)) {

            val token = authorizationHeader.substring(headerPrefix.length)

            if (jwtTokenService.isValidToken(token)) {

                val username = jwtTokenService.getUsernameFromToken(token)

                val usernamePasswordAuthenticationToken = UsernamePasswordAuthenticationToken(username, "")
                usernamePasswordAuthenticationToken.details = WebAuthenticationDetailsSource().buildDetails(request)
                SecurityContextHolder.getContext().authentication = usernamePasswordAuthenticationToken
            }
        }

        filterChain.doFilter(request, response)
    }
}</code></pre>

Dove il _gotcha_ è che, per dire a Spring Boot che l&#8217;utente è autenticato, dobbiamo settare l&#8217;`authentication` nel `SecurityContextHolder`, mettendoci esattamente quello che si aspetta.

Tutto pronto quindi, funziona tutto? No, ovviamente, fioccano i 403.

Cosa manca?

Spring Boot è molto pignolo, e dopo essersi sincerato che l&#8217;utente che sta eseguendo una richiesta autenticata è, effettivamente, autenticato cerca di ottenerne i dettagli dal servizio configurato per i dettagli degli utenti&#8230;che noi però non abbiamo ancora configurato.

Nella classe di configurazione, infatti, dobbiamo fare l&#8217;override di un altro metodo:

<pre class="wp-block-code"><code lang="kotlin" class="language-kotlin">@Throws(Exception::class)
override fun configure(auth: AuthenticationManagerBuilder?) {
    super.configure(auth)
    auth?.userDetailsService(jwtUserDetailsService)?.passwordEncoder(passwordEncoder())
}

@Bean
fun passwordEncoder(): PasswordEncoder {
    // We are not relying on user passwords, so we can use this password encoder.
    return NoOpPasswordEncoder.getInstance()
}</code></pre>

Così facendo, diciamo a Spring Boot di usare il nostro servizio, `jwtUserDetailsService`, per recuperare i dettagli degli utenti, usando un password encoder che non fa l&#8217;encoding delle password: è una pratica assolutamente non sicura, ma nel nostro caso non usiamo le password degli utenti perché ci fidiamo di Google per dire se un utente è effettivamente valido o no, quindi a posto così.

Il `JwtUserDetailsService`, infine, è piuttosto semplice:

<pre class="wp-block-code"><code lang="kotlin" class="language-kotlin">@Service
class JwtUserDetailsService(private val userService: UserService) : UserDetailsService {

    override fun loadUserByUsername(username: String?): UserDetails {
        val user = userService.findByEmail(username!!) ?: throw UsernameNotFoundException(username)

        return org.springframework.security.core.userdetails.User(user.email, user.password, mutableListOf())
    }
}</code></pre>

Cerca un utente su DB per username, che è quello che abbiamo messo in precedenza nel `SecurityContext`, e ritorna un oggetto `User` corrispondente.

Per stavolta direi che può bastare così: nella [prossima][3], e ultima, puntata, vedremo l&#8217;implementazione dell&#8217;endpoint di autenticazione e la generazione del token JWT.

 [1]: https://raibaz.it/2019/10/autenticazione-via-token-jwt-con-angular-e-spring-boot/
 [2]: https://raibaz.it/2019/10/autenticazione-via-token-jwt-con-angular-e-spring-boot-parte-1-google-login-con-angular/
 [3]: https://www.raibaz.it/2019/10/autenticazione-via-token-jwt-con-angular-e-spring-boot-parte-3-generazione-del-token-jwt/