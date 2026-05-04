# 🛡️ LAB 6 — Spring Security Avancé & OAuth2

**Prérequis** : Lab 5

---

## 🎯 Objectifs pédagogiques

À la fin de ce lab, vous serez capable de :

- Implémenter des **permissions granulaires** (au-delà de ADMIN/USER)
- Configurer l'authentification via **Google / GitHub** (OAuth2)
- Activer l'**audit automatique** des entités (qui a créé quoi, quand)
- Gérer finement les **sessions** (expiration, concurrence)
- Protéger les routes par **permission** plutôt que par rôle

---

## 💡 Concept fondamental : Rôles vs Permissions

### Lab 5 (rôles simples)
```
ADMIN → peut tout faire
USER  → peut lire seulement
```

### Lab 6 (permissions granulaires)
```
ADMIN → PRODUIT_CREATE + PRODUIT_READ + PRODUIT_UPDATE + PRODUIT_DELETE
         CATEGORIE_CREATE + CATEGORIE_READ + ...

USER  → PRODUIT_READ + CATEGORIE_READ seulement

GESTIONNAIRE → PRODUIT_CREATE + PRODUIT_READ + PRODUIT_UPDATE (pas DELETE)
```

> **Pourquoi ?** Dans un vrai projet, un gestionnaire peut créer des produits mais pas les supprimer. Avec juste ADMIN/USER, c'est impossible à exprimer.

---

## 🗺️ Schéma d'architecture global

```
┌─────────────────────────────────────────────────────────────────┐
│                    SPRING SECURITY CHAIN                         │
│                                                                  │
│  Requête HTTP                                                    │
│       │                                                          │
│       ▼                                                          │
│  ┌──────────────────┐                                            │
│  │ CsrfFilter       │ ← vérifie le token CSRF (formulaires)     │
│  └────────┬─────────┘                                            │
│           ▼                                                      │
│  ┌──────────────────┐                                            │
│  │ SessionFilter    │ ← gère la session HTTP                     │
│  └────────┬─────────┘                                            │
│           ▼                                                      │
│  ┌──────────────────┐                                            │
│  │ OAuth2Filter     │ ← intercepte les callbacks Google/GitHub   │
│  └────────┬─────────┘                                            │
│           ▼                                                      │
│  ┌──────────────────┐                                            │
│  │ AuthorizationFilter│ ← vérifie les permissions               │
│  └────────┬─────────┘                                            │
│           ▼                                                      │
│      Controller                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Schéma du flux OAuth2 (connexion avec Google)

```
UTILISATEUR         VOTRE APP           GOOGLE
    │                   │                  │
    │─ Clic "Login Google" ─────────────▶  │
    │                   │─ Redirect ──────▶│
    │                   │  client_id       │
    │                   │                  │
    │◀─────────── Page login Google ───────│
    │─ Saisie email/password ─────────────▶│
    │                   │                  │
    │                   │◀─ Authorization Code ─
    │                   │                  │
    │                   │─ Échange code ──▶│
    │                   │  + client_secret │
    │                   │◀─ Access Token ──│
    │                   │                  │
    │                   │─ Récupère profil ▶│
    │                   │◀─ {email, name} ──│
    │                   │                  │
    │                   │─ Crée/trouve l'utilisateur en BD
    │                   │─ Crée session locale
    │◀─ Connecté ! ─────│
```

---

## 📁 Nouveaux fichiers à créer

```
src/main/java/tn/iset/produits/
├── entities/
│   └── Permission.java (enum)        ← NOUVEAU
├── security/
│   ├── SecurityConfig.java           ← MODIFIÉ (Lab 5 → Lab 6)
│   ├── OAuth2SuccessHandler.java     ← NOUVEAU
│   └── AuditorAwareImpl.java         ← NOUVEAU
└── controllers/
    └── OAuth2Controller.java         ← NOUVEAU
```

---

## 🔧 Étape 1 — Dépendances Maven

```xml
<!-- OAuth2 Client (Google, GitHub) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>

<!-- Spring Data pour l'audit (@CreatedBy, @LastModifiedBy) -->
<!-- Déjà inclus avec spring-boot-starter-data-jpa -->
```

---

## 🔧 Étape 2 — Permissions granulaires

### Pourquoi séparer Permission et Role ?

Chaque `Role` regroupe un **ensemble de permissions**. Si demain un rôle `GESTIONNAIRE` doit être créé, on lui assigne les permissions voulues sans toucher au reste du code.

```java
// Permission.java
public enum Permission {
    // Produits
    PRODUIT_CREATE,
    PRODUIT_READ,
    PRODUIT_UPDATE,
    PRODUIT_DELETE,

    // Catégories
    CATEGORIE_CREATE,
    CATEGORIE_READ,
    CATEGORIE_UPDATE,
    CATEGORIE_DELETE,

    // Admin
    USER_MANAGE
}
```

```java
// Role.java (version améliorée)
public enum Role {

    ADMIN(List.of(Permission.values())),   // ← toutes les permissions

    USER(List.of(
        Permission.PRODUIT_READ,
        Permission.CATEGORIE_READ
    )),

    GESTIONNAIRE(List.of(
        Permission.PRODUIT_CREATE,
        Permission.PRODUIT_READ,
        Permission.PRODUIT_UPDATE,
        Permission.CATEGORIE_READ
    ));

    private final List<Permission> permissions;

    Role(List<Permission> permissions) {
        this.permissions = permissions;
    }

    public List<Permission> getPermissions() {
        return permissions;
    }
}
```

---

## 🔧 Étape 3 — UserDetailsServiceImpl mis à jour

Spring Security doit connaître les **GrantedAuthority** (permissions) de l'utilisateur :

```java
@Service
@RequiredArgsConstructor
public class UserDetailsServiceImpl implements UserDetailsService {

    private final UtilisateurRepository utilisateurRepository;

    @Override
    public UserDetails loadUserByUsername(String username) {
        Utilisateur utilisateur = utilisateurRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException(username));

        // Construire la liste des autorités = ROLE_XXX + toutes les permissions
        List<GrantedAuthority> authorities = new ArrayList<>();

        // Ajouter le rôle
        authorities.add(new SimpleGrantedAuthority("ROLE_" + utilisateur.getRole().name()));

        // Ajouter chaque permission du rôle
        utilisateur.getRole().getPermissions().stream()
            .map(p -> new SimpleGrantedAuthority(p.name()))
            .forEach(authorities::add);

        return new org.springframework.security.core.userdetails.User(
            utilisateur.getUsername(),
            utilisateur.getPasswordHash(),
            authorities
        );
    }
}
```

---

## 🔧 Étape 4 — Utilisation dans les Controllers

```java
// ProduitController.java — contrôle par permission
@RestController
@RequestMapping("/api/produits")
public class ProduitController {

    @GetMapping
    @PreAuthorize("hasAuthority('PRODUIT_READ')")      // USER et ADMIN
    public List<Produit> getAll() { ... }

    @PostMapping
    @PreAuthorize("hasAuthority('PRODUIT_CREATE')")    // ADMIN et GESTIONNAIRE
    public ResponseEntity<Produit> create(@RequestBody Produit p) { ... }

    @PutMapping("/{id}")
    @PreAuthorize("hasAuthority('PRODUIT_UPDATE')")    // ADMIN et GESTIONNAIRE
    public ResponseEntity<Produit> update(...) { ... }

    @DeleteMapping("/{id}")
    @PreAuthorize("hasAuthority('PRODUIT_DELETE')")    // ADMIN uniquement
    public ResponseEntity<Void> delete(@PathVariable Long id) { ... }
}
```

> **`@PreAuthorize`** s'exécute **avant** l'entrée dans la méthode. Si la permission est absente → `403 Forbidden` automatique.

---

## 🔧 Étape 5 — Configuration OAuth2

### 5.1 Créer une app Google (Console Google Cloud)

1. Aller sur [console.cloud.google.com](https://console.cloud.google.com)
2. Créer un projet → APIs & Services → Identifiants
3. Créer un OAuth2 Client ID (type : Application Web)
4. URI de redirection autorisée : `http://localhost:8080/login/oauth2/code/google`

### 5.2 application.properties

```properties
# OAuth2 Google
spring.security.oauth2.client.registration.google.client-id=VOTRE_CLIENT_ID
spring.security.oauth2.client.registration.google.client-secret=VOTRE_CLIENT_SECRET
spring.security.oauth2.client.registration.google.scope=profile,email

# OAuth2 GitHub (optionnel)
spring.security.oauth2.client.registration.github.client-id=VOTRE_GITHUB_ID
spring.security.oauth2.client.registration.github.client-secret=VOTRE_GITHUB_SECRET
spring.security.oauth2.client.registration.github.scope=user:email
```

### 5.3 SecurityConfig avec OAuth2

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity          // ← active @PreAuthorize
@RequiredArgsConstructor
public class SecurityConfig {

    private final UserDetailsServiceImpl userDetailsService;
    private final OAuth2SuccessHandler oAuth2SuccessHandler;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**", "/login/**", "/oauth2/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            // ── Session classique ────────────────────────────────
            .formLogin(form -> form
                .loginPage("/api/auth/login").permitAll()
            )
            .logout(logout -> logout
                .logoutUrl("/api/auth/logout").permitAll()
            )
            // ── OAuth2 : Google / GitHub ─────────────────────────
            .oauth2Login(oauth2 -> oauth2
                .successHandler(oAuth2SuccessHandler)  // ← notre handler custom
                .failureUrl("/login?error=oauth2")
            )
            .csrf(csrf -> csrf.disable());

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### 5.4 OAuth2SuccessHandler

```java
@Component
@RequiredArgsConstructor
@Slf4j
public class OAuth2SuccessHandler
        extends SimpleUrlAuthenticationSuccessHandler {

    private final UtilisateurService utilisateurService;

    @Override
    public void onAuthenticationSuccess(
            HttpServletRequest request,
            HttpServletResponse response,
            Authentication authentication) throws IOException {

        OAuth2User oAuth2User = (OAuth2User) authentication.getPrincipal();

        // Récupérer les infos depuis Google
        String email = oAuth2User.getAttribute("email");
        String name  = oAuth2User.getAttribute("name");

        log.info("OAuth2 login : email={}, name={}", email, name);

        // Trouver ou créer l'utilisateur dans notre BD
        Utilisateur utilisateur = utilisateurService.findOrCreateOAuth2User(email, name);

        // Stocker en session
        request.getSession().setAttribute("userId",   utilisateur.getIdUser());
        request.getSession().setAttribute("username", utilisateur.getUsername());
        request.getSession().setAttribute("role",     utilisateur.getRole());

        // Redirection vers le frontend
        response.sendRedirect("http://localhost:5173/dashboard");
    }
}
```

---

## 🔧 Étape 6 — Audit automatique des entités

L'audit permet de savoir **qui** a créé ou modifié chaque enregistrement, et **quand**.

### 6.1 AuditorAwareImpl — qui est connecté ?

```java
@Component
public class AuditorAwareImpl implements AuditorAware<String> {

    @Override
    public Optional<String> getCurrentAuditor() {
        // Récupérer l'utilisateur connecté depuis Spring Security
        Authentication authentication =
            SecurityContextHolder.getContext().getAuthentication();

        if (authentication == null || !authentication.isAuthenticated()
                || authentication instanceof AnonymousAuthenticationToken) {
            return Optional.of("system");
        }

        return Optional.of(authentication.getName());  // ← le username
    }
}
```

### 6.2 Activer l'audit dans la config principale

```java
@SpringBootApplication
@EnableJpaAuditing(auditorAwareRef = "auditorAwareImpl")  // ← NOUVEAU
public class ProduitsApplication { ... }
```

### 6.3 Entité Produit avec audit

```java
@Entity
@EntityListeners(AuditingEntityListener.class)   // ← écoute les événements JPA
public class Produit {

    // ... champs existants des Labs 1-4 ...

    @CreatedBy
    @Column(nullable = false, updatable = false)
    private String createdBy;               // ← rempli automatiquement

    @LastModifiedBy
    private String lastModifiedBy;          // ← mis à jour automatiquement

    @CreatedDate
    @Column(nullable = false, updatable = false)
    private Instant createdDate;

    @LastModifiedDate
    private Instant lastModifiedDate;
}
```

> **Magie** : Hibernate appelle automatiquement `AuditorAwareImpl.getCurrentAuditor()` à chaque INSERT ou UPDATE. Vous ne touchez plus à ces champs manuellement.

---

## 🔧 Étape 7 — Gestion avancée des sessions

```properties
# application.properties — configuration des sessions

# Durée de vie d'une session inactive
server.servlet.session.timeout=30m

# Nombre max de sessions simultanées par utilisateur
# (forcer la déconnexion de l'ancienne session quand une nouvelle arrive)
spring.session.store-type=none
```

```java
// Dans SecurityConfig — gestion des sessions concurrentes
http.sessionManagement(session -> session
    .maximumSessions(1)                     // ← 1 session max par utilisateur
    .maxSessionsPreventsLogin(false)        // ← false = la nouvelle session gagne
                                            // ← true  = refuser le 2ème login
);
```

---

## 📊 Schéma récapitulatif des permissions par rôle

```
                    PRODUIT                     CATEGORIE          ADMIN
              CREATE READ UPDATE DELETE   CREATE READ UPDATE DELETE USER_MANAGE
ADMIN           ✅    ✅    ✅     ✅       ✅    ✅    ✅     ✅      ✅
GESTIONNAIRE    ✅    ✅    ✅     ❌       ❌    ✅    ❌     ❌      ❌
USER            ❌    ✅    ❌     ❌       ❌    ✅    ❌     ❌      ❌
```

---

## 📊 Schéma comparatif Lab 5 vs Lab 6

```
LAB 5 — Sessions simples        LAB 6 — Sessions avancées + OAuth2
────────────────────────        ──────────────────────────────────
@hasRole("ADMIN")               @hasAuthority("PRODUIT_DELETE")
Rôles : ADMIN / USER            Rôles : ADMIN / USER / GESTIONNAIRE
Login : username + password     Login : username+pwd OU Google/GitHub
Pas d'audit                     Audit automatique : createdBy, modifiedBy
Session basique                 Session avec timeout + concurrence contrôlée
```

---

## 🧪 Tests

```bash
# 1. Login classique
curl -c cookies.txt -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":"pass123"}'

# 2. Tentative de suppression avec rôle USER (doit retourner 403)
curl -b cookies.txt -X DELETE http://localhost:8080/api/produits/1
# → 403 Forbidden

# 3. Login OAuth2 Google (ouvrir dans le navigateur)
# http://localhost:8080/oauth2/authorization/google

# 4. Vérifier l'audit après création d'un produit
curl -b cookies.txt -X POST http://localhost:8080/api/produits \
  -H "Content-Type: application/json" \
  -d '{"nomProduit":"Test","prixProduit":99.99}'
# → produit avec createdBy = "alice"
```

---

## ✅ Checklist de fin de lab

- [ ] Enum `Permission` avec toutes les permissions CRUD
- [ ] Enum `Role` mis à jour avec les listes de permissions par rôle
- [ ] `UserDetailsServiceImpl` retournant les `GrantedAuthority` des permissions
- [ ] `@PreAuthorize` sur chaque méthode de controller
- [ ] `@EnableMethodSecurity` activé dans `SecurityConfig`
- [ ] Credentials Google/GitHub configurés dans `application.properties`
- [ ] `OAuth2SuccessHandler` créant/retrouvant l'utilisateur en base
- [ ] `AuditorAwareImpl` retournant le username connecté
- [ ] `@EnableJpaAuditing` sur la classe principale
- [ ] Champs audit (`@CreatedBy`, `@LastModifiedBy`) sur les entités
- [ ] Test : USER ne peut pas supprimer → 403
- [ ] Test : login Google redirige vers le dashboard
