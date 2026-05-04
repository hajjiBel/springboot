# 🔑 LAB 7 — JWT & API Stateless

**Prérequis** : Labs 5 & 6

---

## 🎯 Objectifs pédagogiques

À la fin de ce lab, vous serez capable de :

- Comprendre et générer des **JSON Web Tokens (JWT)**
- Implémenter un système **Access Token + Refresh Token**
- Créer un **filtre JWT** personnalisé dans Spring Security
- Révoquer des tokens via une **blacklist Redis**
- Configurer le frontend **Vue.js** pour gérer les tokens automatiquement
- Comprendre pourquoi JWT est idéal pour les **microservices**

---

## 💡 Concept fondamental : JWT vs Sessions

### Sessions (Labs 5-6) — Stateful

```
SERVEUR stocke : SESSION_ID=abc123 → {userId:1, role:"ADMIN"}
CLIENT stocke  : Cookie JSESSIONID=abc123

Problème : si vous avez 3 serveurs, chaque serveur a ses propres sessions
→ l'utilisateur peut être connecté sur le serveur A mais pas B ou C
```

### JWT (Lab 7) — Stateless

```
SERVEUR ne stocke RIEN
CLIENT stocke : le token JWT complet (dans localStorage)

Le token contient TOUTES les informations :
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhbGljZSIsInJvbGUiOiJVU0VSIn0.xxxxx
     Header                    Payload                        Signature

Avantage : n'importe quel serveur peut vérifier n'importe quel token
→ parfait pour les microservices et le scaling horizontal
```

---

## 🗺️ Anatomie d'un JWT

```
eyJhbGciOiJIUzI1NiJ9  .  eyJzdWIiOiJhbGljZSIsInJvbGUiOiJVU0VSIiwiaWF0IjoxNzA5MDAwMDAwLCJleHAiOjE3MDkwMDM2MDB9  .  SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
        │                                           │                                                                                  │
   ┌────┴────┐                              ┌───────┴──────┐                                                                   ┌───────┴──────┐
   │ HEADER  │                              │   PAYLOAD    │                                                                   │  SIGNATURE   │
   │─────────│                              │──────────────│                                                                   │──────────────│
   │ alg:    │                              │ sub: "alice" │                                                                   │ HMAC-SHA256( │
   │ HS256   │                              │ role: "USER" │                                                                   │  header +    │
   │ typ:    │                              │ iat: 1709... │                                                                   │  payload +   │
   │ JWT     │                              │ exp: 1709... │                                                                   │  secret_key) │
   └─────────┘                              └──────────────┘                                                                   └──────────────┘
   Base64 encodé                            Base64 encodé                                                                      Non modifiable
   (lisible)                                (lisible, pas chiffré !)                                                           sans la clé
```

> ⚠️ **Important** : Le payload JWT est **encodé** (Base64), pas **chiffré**. Ne jamais mettre de données sensibles (mot de passe, carte bancaire) dans un JWT.

---

## 🔄 Schéma du flux JWT complet

```
        CLIENT (Vue.js)                    SERVEUR (Spring Boot)
              │                                      │
              │──── POST /api/auth/login ────────────▶│
              │     {username, password}              │  Vérification BCrypt
              │                                      │  Génération tokens
              │◀── {accessToken, refreshToken} ───────│
              │                                      │
    localStorage.setItem('accessToken', ...)         │
    localStorage.setItem('refreshToken', ...)        │
              │                                      │
              │──── GET /api/produits ───────────────▶│
              │     Authorization: Bearer ACCESS_TOKEN│  Filtre JWT :
              │                                      │  1. Extraire token
              │                                      │  2. Vérifier signature
              │                                      │  3. Vérifier expiration
              │                                      │  4. Vérifier blacklist
              │◀── 200 OK + données ─────────────────│
              │                                      │
              │  (30 min après... token expiré)      │
              │                                      │
              │──── GET /api/produits ───────────────▶│
              │     Authorization: Bearer EXPIRED_TOKEN
              │◀── 401 Unauthorized ─────────────────│
              │                                      │
              │──── POST /api/auth/refresh ──────────▶│
              │     {refreshToken}                   │  Vérifier refresh token
              │                                      │  Générer nouveau access token
              │◀── {newAccessToken} ─────────────────│
              │                                      │
              │──── GET /api/produits ───────────────▶│
              │     Authorization: Bearer NEW_TOKEN  │  ✅ Autorisé
              │◀── 200 OK + données ─────────────────│
```

---

## 🔄 Schéma du cycle de vie des tokens

```
ACCESS TOKEN                      REFRESH TOKEN
────────────                      ─────────────
Durée : 30 minutes                Durée : 7 jours
Stockage : localStorage           Stockage : localStorage (ou httpOnly cookie)
Usage : chaque requête API        Usage : uniquement pour obtenir un nouveau access token
Révocation : blacklist Redis      Révocation : blacklist Redis

Flux :
┌─────────────┐  expire   ┌─────────────────┐  expire   ┌────────────┐
│ Access Token│ ────────▶ │  Refresh Token   │ ────────▶ │  Logout    │
│  (30 min)   │           │   (7 jours)      │           │  forcé     │
└─────────────┘           └─────────────────┘           └────────────┘
       │                          │
       │ invalide                 │ valide
       ▼                          ▼
  401 Unauthorized         Nouveau Access Token
```

---

## 📁 Nouveaux fichiers à créer

```
src/main/java/tn/iset/produits/
├── security/
│   ├── SecurityConfig.java           ← MODIFIÉ (Sessions → JWT)
│   ├── JwtService.java               ← NOUVEAU
│   └── JwtAuthenticationFilter.java  ← NOUVEAU
├── controllers/
│   └── TokenController.java          ← NOUVEAU (login JWT)
└── dto/
    ├── TokenResponse.java            ← NOUVEAU
    └── RefreshTokenRequest.java      ← NOUVEAU
```

---

## 🔧 Étape 1 — Dépendances Maven

```xml
<!-- JWT library (jjwt) -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.12.3</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.12.3</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.12.3</version>
    <scope>runtime</scope>
</dependency>

<!-- Redis pour la blacklist des tokens révoqués -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

---

## 🔧 Étape 2 — Configuration application.properties

```properties
# ===== JWT =====
# Clé secrète en Base64 (256 bits minimum pour HS256)
# Générer avec : openssl rand -base64 64
jwt.secret=dGhpcyBpcyBhIHZlcnkgbG9uZyBzZWNyZXQga2V5IGZvciBqd3QgdGhhdCBpcyBzZWN1cmU=
jwt.expiration=1800000          # 30 minutes en millisecondes
jwt.refresh.expiration=604800000 # 7 jours en millisecondes

# ===== Redis =====
spring.data.redis.host=localhost
spring.data.redis.port=6379
```

> **Pourquoi une clé longue ?** La sécurité du JWT repose entièrement sur le secret. Une clé courte ou prévisible rend le JWT falsifiable.

---

## 🔧 Étape 3 — JwtService

```java
@Service
@Slf4j
@RequiredArgsConstructor
public class JwtService {

    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.expiration}")
    private long jwtExpiration;

    @Value("${jwt.refresh.expiration}")
    private long refreshExpiration;

    private final RedisTemplate<String, String> redisTemplate;

    // ─── GÉNÉRATION ──────────────────────────────────────────────

    public String generateAccessToken(Utilisateur utilisateur) {
        return buildToken(utilisateur, jwtExpiration);
    }

    public String generateRefreshToken(Utilisateur utilisateur) {
        return buildToken(utilisateur, refreshExpiration);
    }

    private String buildToken(Utilisateur utilisateur, long expiration) {
        return Jwts.builder()
            .subject(utilisateur.getUsername())          // ← "sub" claim
            .claim("userId", utilisateur.getIdUser())   // ← claim custom
            .claim("role", utilisateur.getRole().name())// ← claim custom
            .issuedAt(new Date())                        // ← "iat" : émis le
            .expiration(new Date(                        // ← "exp" : expire le
                System.currentTimeMillis() + expiration))
            .signWith(getSigningKey())                   // ← signature HMAC-SHA256
            .compact();
    }

    // ─── EXTRACTION ──────────────────────────────────────────────

    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }

    private Claims extractAllClaims(String token) {
        try {
            return Jwts.parser()
                .verifyWith((SecretKey) getSigningKey())
                .build()
                .parseSignedClaims(token)
                .getPayload();
        } catch (ExpiredJwtException e) {
            throw new RuntimeException("Token expiré");
        } catch (JwtException e) {
            throw new RuntimeException("Token invalide ou falsifié");
        }
    }

    // ─── VALIDATION ──────────────────────────────────────────────

    public boolean isTokenValid(String token, String username) {
        // 1. Vérifier la blacklist Redis
        if (isTokenBlacklisted(token)) {
            log.warn("Token blacklisté pour {}", username);
            return false;
        }
        // 2. Vérifier le username et l'expiration
        try {
            String tokenUsername = extractUsername(token);
            return tokenUsername.equals(username) && !isTokenExpired(token);
        } catch (Exception e) {
            return false;
        }
    }

    private boolean isTokenExpired(String token) {
        return extractClaim(token, Claims::getExpiration).before(new Date());
    }

    // ─── BLACKLIST (RÉVOCATION) ───────────────────────────────────

    public void blacklistToken(String token) {
        try {
            // Calculer le TTL restant avant expiration naturelle
            Date expiration = extractClaim(token, Claims::getExpiration);
            long ttl = expiration.getTime() - System.currentTimeMillis();

            if (ttl > 0) {
                // Stocker dans Redis avec expiration automatique
                redisTemplate.opsForValue().set(
                    "blacklist:" + token,
                    "revoked",
                    Duration.ofMillis(ttl)
                );
                log.info("Token blacklisté, TTL restant : {} ms", ttl);
            }
        } catch (Exception e) {
            log.error("Erreur blacklist token : {}", e.getMessage());
        }
    }

    private boolean isTokenBlacklisted(String token) {
        return Boolean.TRUE.equals(redisTemplate.hasKey("blacklist:" + token));
    }

    // ─── CLÉ DE SIGNATURE ────────────────────────────────────────

    private Key getSigningKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secret);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}
```

---

## 🔧 Étape 4 — JwtAuthenticationFilter

Ce filtre est le **cœur** du système JWT. Il s'exécute à **chaque requête** HTTP, avant d'atteindre le controller.

```java
@Component
@Slf4j
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain) throws ServletException, IOException {

        // 1. Extraire le token du header Authorization
        String token = extractTokenFromRequest(request);

        if (token != null) {
            try {
                // 2. Extraire le username du token
                String username = jwtService.extractUsername(token);

                // 3. Vérifier que l'utilisateur n'est pas déjà authentifié
                if (username != null &&
                    SecurityContextHolder.getContext().getAuthentication() == null) {

                    // 4. Charger l'utilisateur depuis la base
                    UserDetails userDetails =
                        userDetailsService.loadUserByUsername(username);

                    // 5. Valider le token (signature + expiration + blacklist)
                    if (jwtService.isTokenValid(token, userDetails.getUsername())) {

                        // 6. Créer l'objet Authentication
                        UsernamePasswordAuthenticationToken authentication =
                            new UsernamePasswordAuthenticationToken(
                                userDetails,
                                null,
                                userDetails.getAuthorities()  // ← rôles et permissions
                            );
                        authentication.setDetails(
                            new WebAuthenticationDetailsSource()
                                .buildDetails(request));

                        // 7. Placer dans le SecurityContext
                        // → Spring Security considère l'utilisateur comme connecté
                        SecurityContextHolder.getContext()
                            .setAuthentication(authentication);

                        log.debug("✅ Authentifié via JWT : {}", username);
                    }
                }
            } catch (Exception e) {
                log.error("❌ Erreur JWT : {}", e.getMessage());
                // On ne bloque pas : le filtre continue, Spring Security gérera le 401
            }
        }

        // 8. Passer la requête au filtre suivant (et au controller)
        filterChain.doFilter(request, response);
    }

    private String extractTokenFromRequest(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        // Format attendu : "Bearer eyJhbGci..."
        if (bearerToken != null && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);  // ← enlever "Bearer "
        }
        return null;
    }
}
```

### Schéma du filtre

```
Requête HTTP arrive
        │
        ▼
┌─────────────────────────────────┐
│  JwtAuthenticationFilter        │
│                                 │
│  1. Header Authorization ?      │
│     Non → passer au suivant     │
│     Oui → continuer             │
│                                 │
│  2. Extraire username du token  │
│                                 │
│  3. Token valide ?              │
│     - Signature OK ?            │
│     - Non expiré ?              │
│     - Pas blacklisté ?          │
│                                 │
│  4. Si OK → SecurityContext     │
│     setAuthentication(user)     │
└─────────────┬───────────────────┘
              │
              ▼
        Controller
```

---

## 🔧 Étape 5 — SecurityConfig avec JWT (stateless)

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtAuthFilter;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // ── CSRF désactivé : JWT est stateless, pas besoin ──
            .csrf(csrf -> csrf.disable())

            // ── CORS configuré pour Vue.js ───────────────────────
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))

            // ── Règles d'accès ────────────────────────────────────
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()  // login, register, refresh
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )

            // ── Ajouter le filtre JWT avant UsernamePasswordAuthentication ──
            .addFilterBefore(
                jwtAuthFilter,
                UsernamePasswordAuthenticationFilter.class
            )

            // ── STATELESS : pas de session côté serveur ──────────
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            );

        return http.build();
    }

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowedOrigins(List.of("http://localhost:5173"));
        config.setAllowedMethods(List.of("GET","POST","PUT","DELETE","OPTIONS"));
        config.setAllowedHeaders(List.of("*"));
        config.setAllowCredentials(true);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        return source;
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public AuthenticationManager authenticationManager(
            UserDetailsService userDetailsService,
            PasswordEncoder passwordEncoder) {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder);
        return new ProviderManager(provider);
    }
}
```

---

## 🔧 Étape 6 — TokenController (login JWT)

```java
@RestController
@RequestMapping("/api/auth")
@RequiredArgsConstructor
@Slf4j
public class TokenController {

    private final UtilisateurService utilisateurService;
    private final JwtService jwtService;

    // ─── LOGIN → retourne ACCESS + REFRESH token ─────────────────
    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody @Valid LoginRequest request) {
        try {
            Utilisateur utilisateur = utilisateurService.login(
                request.username(), request.password());

            String accessToken  = jwtService.generateAccessToken(utilisateur);
            String refreshToken = jwtService.generateRefreshToken(utilisateur);

            log.info("✅ Login JWT : {}", utilisateur.getUsername());

            return ResponseEntity.ok(new TokenResponse(
                accessToken,
                refreshToken,
                "Bearer",
                utilisateur.getIdUser(),
                utilisateur.getRole().name()
            ));
        } catch (RuntimeException e) {
            return ResponseEntity.status(401)
                .body(Map.of("error", "Identifiants incorrects"));
        }
    }

    // ─── REFRESH → renouveler l'access token ─────────────────────
    @PostMapping("/refresh")
    public ResponseEntity<?> refresh(
            @RequestBody @Valid RefreshTokenRequest request) {
        try {
            String username = jwtService.extractUsername(request.refreshToken());

            if (!jwtService.isTokenValid(request.refreshToken(), username)) {
                return ResponseEntity.status(401)
                    .body(Map.of("error", "Refresh token invalide ou expiré"));
            }

            Utilisateur utilisateur = utilisateurService.findByUsername(username);
            String newAccessToken = jwtService.generateAccessToken(utilisateur);

            log.info("🔄 Token rafraîchi pour : {}", username);

            return ResponseEntity.ok(new TokenResponse(
                newAccessToken,
                request.refreshToken(),  // ← même refresh token (pas de rotation ici)
                "Bearer",
                utilisateur.getIdUser(),
                utilisateur.getRole().name()
            ));
        } catch (RuntimeException e) {
            return ResponseEntity.status(401)
                .body(Map.of("error", "Refresh échoué : " + e.getMessage()));
        }
    }

    // ─── LOGOUT → blacklister le token ───────────────────────────
    @PostMapping("/logout")
    public ResponseEntity<Map<String, String>> logout(
            @RequestHeader("Authorization") String authHeader) {
        String token = authHeader.substring(7); // enlever "Bearer "
        jwtService.blacklistToken(token);
        log.info("✅ Token blacklisté (logout)");
        return ResponseEntity.ok(Map.of("message", "Déconnecté avec succès"));
    }
}
```

---

## 🔧 Étape 7 — DTOs

```java
// TokenResponse.java
public record TokenResponse(
    String accessToken,
    String refreshToken,
    String tokenType,      // toujours "Bearer"
    Long userId,
    String role
) {}

// RefreshTokenRequest.java
public record RefreshTokenRequest(
    @NotBlank String refreshToken
) {}
```

---

## 🔧 Étape 8 — Frontend Vue.js avec gestion automatique des tokens

```javascript
// src/api/authApi.js
import axios from 'axios'

const API_URL = 'http://localhost:8080/api'

// Instance axios avec token automatique
const authApi = axios.create({ baseURL: API_URL })

// ── Intercepteur REQUEST : ajouter le token à chaque requête ──────────
authApi.interceptors.request.use(config => {
    const token = localStorage.getItem('accessToken')
    if (token) {
        config.headers.Authorization = `Bearer ${token}`
    }
    return config
})

// ── Intercepteur RESPONSE : refresh automatique si 401 ───────────────
authApi.interceptors.response.use(
    response => response,   // ← réponse OK : rien à faire
    async error => {
        const originalRequest = error.config

        // Si 401 et pas déjà retesté
        if (error.response?.status === 401 && !originalRequest._retry) {
            originalRequest._retry = true

            try {
                const refreshToken = localStorage.getItem('refreshToken')
                const response = await axios.post(`${API_URL}/auth/refresh`, {
                    refreshToken
                })

                const { accessToken, refreshToken: newRefresh } = response.data
                localStorage.setItem('accessToken', accessToken)
                if (newRefresh) localStorage.setItem('refreshToken', newRefresh)

                // Réessayer la requête originale avec le nouveau token
                originalRequest.headers.Authorization = `Bearer ${accessToken}`
                return authApi(originalRequest)

            } catch (refreshError) {
                // Refresh échoué → déconnexion forcée
                localStorage.removeItem('accessToken')
                localStorage.removeItem('refreshToken')
                window.location.href = '/login'
                return Promise.reject(refreshError)
            }
        }

        return Promise.reject(error)
    }
)

// ── Service auth ─────────────────────────────────────────────────────
export const authService = {

    login: async (username, password) => {
        const response = await axios.post(`${API_URL}/auth/login`, {
            username, password })
        const { accessToken, refreshToken, userId, role } = response.data

        localStorage.setItem('accessToken',  accessToken)
        localStorage.setItem('refreshToken', refreshToken)
        localStorage.setItem('userId',       userId)
        localStorage.setItem('role',         role)

        return response.data
    },

    logout: async () => {
        const token = localStorage.getItem('accessToken')
        if (token) {
            await authApi.post('/auth/logout', null, {
                headers: { Authorization: `Bearer ${token}` }
            })
        }
        localStorage.removeItem('accessToken')
        localStorage.removeItem('refreshToken')
        localStorage.removeItem('userId')
        localStorage.removeItem('role')
    },

    isAuthenticated: () => !!localStorage.getItem('accessToken'),
    getRole: () => localStorage.getItem('role'),
    getUserId: () => localStorage.getItem('userId')
}

export default authApi
```

---

## 🔧 Étape 9 — Guards Vue Router

```javascript
// src/router/index.js
import { authService } from '../api/authApi.js'

const routes = [
    { path: '/login',     component: LoginView,     meta: { public: true } },
    { path: '/',          component: Dashboard,      meta: { requiresAuth: true } },
    { path: '/produits',  component: ProductList,    meta: { requiresAuth: true } },
    { path: '/admin',     component: AdminPanel,     meta: { requiresAuth: true, role: 'ADMIN' } }
]

router.beforeEach((to, from, next) => {
    if (to.meta.public) {
        next()  // ← route publique, pas de vérification
    } else if (!authService.isAuthenticated()) {
        next('/login')  // ← non connecté → login
    } else if (to.meta.role && authService.getRole() !== to.meta.role) {
        next('/')   // ← rôle insuffisant → accueil
    } else {
        next()  // ← OK
    }
})
```

---

## 📊 Schéma Redis — Blacklist des tokens

```
Redis (base de données clé-valeur en mémoire)
─────────────────────────────────────────────

Clé                                    Valeur   TTL (auto-expiration)
──────────────────────────────────────────────────────────────────────
"blacklist:eyJhbGci...token1..."       "revoked"  28 min (expire avec le token)
"blacklist:eyJhbGci...token2..."       "revoked"  5 jours

Avantage :
- Redis supprime automatiquement les clés expirées
- Pas besoin de nettoyer manuellement la blacklist
- Ultra-rapide (stockage mémoire)
```

---

## 📊 Tableau comparatif final

| Aspect | Lab 5 (Sessions) | Lab 6 (OAuth2) | Lab 7 (JWT) |
|--------|-----------------|----------------|-------------|
| **Stockage serveur** | Session en mémoire | Session en mémoire | Rien (stateless) |
| **Scalabilité** | ❌ Limitée | ⚠️ Moyenne | ✅ Excellente |
| **Microservices** | ❌ | ❌ | ✅ |
| **Mobile** | ⚠️ Difficile | ⚠️ Difficile | ✅ Idéal |
| **Logout immédiat** | ✅ | ✅ | ⚠️ Via Redis |
| **Complexité** | Faible | Moyenne | Élevée |
| **Recommandé pour** | Prototypage | Web app | API / Mobile |

---

## 🧪 Tests complets

```bash
# 1. Login → récupérer les tokens
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":"pass123"}'
# → {"accessToken":"eyJ...","refreshToken":"eyJ...","role":"USER"}

# 2. Utiliser l'access token
curl http://localhost:8080/api/produits \
  -H "Authorization: Bearer eyJ..."
# → [liste des produits]

# 3. Rafraîchir le token
curl -X POST http://localhost:8080/api/auth/refresh \
  -H "Content-Type: application/json" \
  -d '{"refreshToken":"eyJ..."}'
# → {"accessToken":"eyJ_NOUVEAU..."}

# 4. Logout → blacklister le token
curl -X POST http://localhost:8080/api/auth/logout \
  -H "Authorization: Bearer eyJ..."
# → {"message": "Déconnecté avec succès"}

# 5. Tentative avec token blacklisté (doit échouer)
curl http://localhost:8080/api/produits \
  -H "Authorization: Bearer eyJ..."
# → 401 Unauthorized
```

---

## ✅ Checklist de fin de lab

- [ ] Dépendances jjwt et Redis ajoutées dans `pom.xml`
- [ ] `jwt.secret`, `jwt.expiration`, `jwt.refresh.expiration` dans `application.properties`
- [ ] `JwtService` avec génération, extraction, validation et blacklist
- [ ] `JwtAuthenticationFilter` placé avant `UsernamePasswordAuthenticationFilter`
- [ ] `SecurityConfig` avec `SessionCreationPolicy.STATELESS`
- [ ] `TokenController` avec `/login`, `/refresh`, `/logout`
- [ ] `TokenResponse` et `RefreshTokenRequest` DTOs
- [ ] Frontend : intercepteurs Axios (ajout token + refresh automatique)
- [ ] Vue Router guards (`meta.requiresAuth`, `meta.role`)
- [ ] Redis démarré sur `localhost:6379`
- [ ] Test : token blacklisté → 401 après logout
- [ ] Test : refresh automatique quand access token expiré
