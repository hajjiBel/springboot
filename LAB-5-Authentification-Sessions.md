# 🔐 LAB 5 — Authentification Simple & Sessions HTTP

**Prérequis** : Labs 1-4

---

## 🎯 Objectifs pédagogiques

À la fin de ce lab, vous serez capable de :

- Créer une entité `Utilisateur` avec rôles
- Hasher les mots de passe avec **BCrypt**
- Configurer **Spring Security** basique
- Gérer des **sessions HTTP** (login / logout)
- Protéger des routes par rôle (`ADMIN`, `USER`)

---

## 💡 Concept fondamental : Qu'est-ce qu'une session HTTP ?

HTTP est un protocole **sans mémoire** : chaque requête est indépendante. Le serveur ne sait pas naturellement qui vous êtes d'une requête à l'autre.

La **session** résout ce problème : après le login, le serveur crée un identifiant unique (JSESSIONID) stocké dans un cookie côté navigateur. À chaque requête suivante, le navigateur envoie ce cookie et le serveur retrouve votre identité.

```
SANS SESSION                    AVEC SESSION
─────────────                   ────────────
Requête 1 : "Bonjour ?"         Login → serveur crée SESSION_ID=abc123
Serveur : "Qui êtes-vous ?"     
                                Requête 2 : Cookie: JSESSIONID=abc123
Requête 2 : "C'est moi !"       Serveur : "Ah, c'est alice, ADMIN !"
Serveur : "Qui êtes-vous ?"
```

---

## 🗺️ Schéma d'architecture global

```
┌──────────────────────────────────────────────────────────────┐
│                        NAVIGATEUR                            │
│                                                              │
│   Vue.js          ────── HTTP ──────→    Spring Boot         │
│   Login Form      Cookie: JSESSIONID    Port 8080            │
└──────────────────────────────────────────────────────────────┘

                    ┌─────────────────────────────┐
                    │      SPRING SECURITY         │
                    │  (Filtre toutes les requêtes)│
                    └──────────────┬──────────────┘
                                   │
                    ┌──────────────▼──────────────┐
                    │  /api/auth/**  → LIBRE       │
                    │  /api/admin/** → ADMIN only  │
                    │  /api/**       → USER+        │
                    └──────────────┬──────────────┘
                                   │
          ┌────────────────────────┼──────────────────────┐
          │                        │                       │
┌─────────▼──────┐      ┌──────────▼──────┐   ┌──────────▼──────┐
│ AuthController  │      │UtilisateurService│   │UtilisateurRepo  │
│ POST /register  │      │ register()       │   │findByUsername() │
│ POST /login     │      │ login()          │   │existsByUsername │
│ POST /logout    │      │ changePassword() │   └─────────────────┘
│ GET  /profile   │      └─────────────────┘
└─────────────────┘
```

---

## 🔄 Schéma du flux d'authentification

```
INSCRIPTION (register)
──────────────────────
Client          AuthController      UtilisateurService      Base de données
  │                  │                      │                      │
  │─ POST /register ─▶                      │                      │
  │  {username, email, password}            │                      │
  │                  │─ register() ────────▶│                      │
  │                  │                      │─ existsByUsername? ──▶│
  │                  │                      │◀─ false ─────────────│
  │                  │                      │─ BCrypt.encode(pwd)  │
  │                  │                      │─ save(utilisateur) ──▶│
  │                  │                      │◀─ OK ────────────────│
  │◀─ 201 Created ───│                      │                      │

CONNEXION (login)
──────────────────
Client          AuthController      UtilisateurService      HttpSession
  │                  │                      │                   │
  │─ POST /login ───▶│                      │                   │
  │  {username, pwd} │─ login() ───────────▶│                   │
  │                  │                      │─ findByUsername   │
  │                  │                      │─ BCrypt.matches() │
  │                  │◀─ Utilisateur ───────│                   │
  │                  │─ session.setAttribute("userId") ────────▶│
  │◀─ 200 + {username, role} ─────────────────────────────────  │

ACCÈS RESSOURCE
───────────────
Client          Spring Security     Controller      Session
  │                  │                  │              │
  │─ GET /produits ─▶│                  │              │
  │  Cookie: JSESSIONID=abc123          │              │
  │                  │─ session valide? ──────────────▶│
  │                  │◀─ userId=1, role=USER ──────────│
  │                  │─ autorisé ───────▶│              │
  │◀─ 200 + données ─│                  │              │
```

---

## 📁 Structure des fichiers à créer

```
src/main/java/tn/iset/produits/
├── entities/
│   ├── Utilisateur.java          ← NOUVEAU
│   └── Role.java (enum)          ← NOUVEAU
├── repositories/
│   └── UtilisateurRepository.java ← NOUVEAU
├── services/
│   ├── UtilisateurService.java   ← NOUVEAU
│   └── UserDetailsServiceImpl.java ← NOUVEAU
├── controllers/
│   └── AuthController.java       ← NOUVEAU
├── dto/
│   ├── RegisterRequest.java      ← NOUVEAU
│   ├── LoginRequest.java         ← NOUVEAU
│   └── LoginResponse.java        ← NOUVEAU
└── config/
    └── SecurityConfig.java       ← NOUVEAU
```

---

## 🔧 Étape 1 — Dépendances Maven

Ajouter dans `pom.xml` :

```xml
<!-- Spring Security -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

> **Pourquoi ?** Spring Security est le framework officiel de Spring pour gérer l'authentification et l'autorisation. Sans lui, toutes les routes sont publiques.

---

## 🔧 Étape 2 — L'entité Utilisateur

### Pourquoi une entité séparée ?

On ne stocke **jamais** le mot de passe en clair. On stocke un hash BCrypt irréversible. L'entité `Utilisateur` représente un compte dans la base de données.

```java
@Entity
@Table(name = "utilisateur")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Utilisateur {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long idUser;

    @NotBlank
    @Column(unique = true)           // ← username doit être unique
    private String username;

    @NotBlank
    @Email
    @Column(unique = true)           // ← email doit être unique
    private String email;

    @NotBlank
    private String passwordHash;     // ← JAMAIS le mot de passe en clair !

    @Enumerated(EnumType.STRING)     // ← stocke "ADMIN" ou "USER" (pas 0/1)
    private Role role;

    @CreationTimestamp               // ← Hibernate remplit automatiquement
    private LocalDateTime dateCreation;

    @UpdateTimestamp
    private LocalDateTime dateModification;
}
```

```java
public enum Role {
    ADMIN,   // accès total
    USER     // accès limité
}
```

> **`@Enumerated(EnumType.STRING)`** : stocke le nom du rôle en texte ("ADMIN") plutôt que son index (0), ce qui est plus lisible et résistant aux modifications futures de l'enum.

---

## 🔧 Étape 3 — Le Repository

```java
@Repository
public interface UtilisateurRepository extends JpaRepository<Utilisateur, Long> {

    Optional<Utilisateur> findByUsername(String username);
    Optional<Utilisateur> findByEmail(String email);
    boolean existsByUsername(String username);
    boolean existsByEmail(String email);
}
```

---

## 🔧 Étape 4 — Le Service d'authentification

### Concept BCrypt

BCrypt est un algorithme de **hachage à sens unique**. Il est impossible de retrouver le mot de passe original depuis le hash.

```
Mot de passe :  "monPassword123"
Hash BCrypt  :  "$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy"

Même mot de passe → hash DIFFÉRENT à chaque fois (sel aléatoire)
Vérification : BCrypt.matches("monPassword123", hash) → true
```

```java
@Service
@Slf4j
@RequiredArgsConstructor
public class UtilisateurService {

    private final UtilisateurRepository utilisateurRepository;
    private final PasswordEncoder passwordEncoder;  // injecté depuis SecurityConfig

    // ─── INSCRIPTION ─────────────────────────────────────────────
    public Utilisateur register(String username, String email, String password) {

        // 1. Vérifier les doublons
        if (utilisateurRepository.existsByUsername(username))
            throw new IllegalArgumentException("Username déjà utilisé");
        if (utilisateurRepository.existsByEmail(email))
            throw new IllegalArgumentException("Email déjà utilisé");

        // 2. Hasher le mot de passe (JAMAIS stocker en clair !)
        String hashedPassword = passwordEncoder.encode(password);

        // 3. Créer et sauvegarder l'utilisateur
        Utilisateur utilisateur = Utilisateur.builder()
            .username(username)
            .email(email)
            .passwordHash(hashedPassword)
            .role(Role.USER)    // ← par défaut USER
            .build();

        Utilisateur saved = utilisateurRepository.save(utilisateur);
        log.info("✅ Nouvel utilisateur enregistré : {}", username);
        return saved;
    }

    // ─── CONNEXION ───────────────────────────────────────────────
    public Utilisateur login(String username, String password) {

        // 1. Chercher l'utilisateur
        Utilisateur utilisateur = utilisateurRepository.findByUsername(username)
            .orElseThrow(() -> new RuntimeException("Utilisateur non trouvé"));

        // 2. Vérifier le mot de passe avec BCrypt
        if (!passwordEncoder.matches(password, utilisateur.getPasswordHash()))
            throw new RuntimeException("Mot de passe incorrect");

        log.info("✅ Connexion réussie : {}", username);
        return utilisateur;
    }

    // ─── CHANGEMENT MOT DE PASSE ─────────────────────────────────
    public void changePassword(Long userId, String oldPassword, String newPassword) {
        Utilisateur utilisateur = utilisateurRepository.findById(userId)
            .orElseThrow(() -> new RuntimeException("Utilisateur non trouvé"));

        if (!passwordEncoder.matches(oldPassword, utilisateur.getPasswordHash()))
            throw new RuntimeException("Ancien mot de passe incorrect");

        utilisateur.setPasswordHash(passwordEncoder.encode(newPassword));
        utilisateurRepository.save(utilisateur);
        log.info("✅ Mot de passe modifié pour : {}", utilisateur.getUsername());
    }
}
```

---

## 🔧 Étape 5 — UserDetailsServiceImpl

Spring Security a besoin d'un pont entre votre entité `Utilisateur` et son propre système. Ce pont est `UserDetailsService`.

```java
@Service
@RequiredArgsConstructor
public class UserDetailsServiceImpl implements UserDetailsService {

    private final UtilisateurRepository utilisateurRepository;

    @Override
    public UserDetails loadUserByUsername(String username)
            throws UsernameNotFoundException {
        Utilisateur utilisateur = utilisateurRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException(
                "Utilisateur non trouvé : " + username));

        // Convertir notre Utilisateur en UserDetails Spring Security
        return User.builder()
            .username(utilisateur.getUsername())
            .password(utilisateur.getPasswordHash())
            .roles(utilisateur.getRole().toString())  // "ADMIN" ou "USER"
            .build();
    }
}
```

> **Rôle de cette classe** : Spring Security ne connaît pas votre entité `Utilisateur`. `UserDetailsService` est le contrat qu'il utilise pour charger un utilisateur depuis n'importe quelle source (base de données, LDAP, fichier...).

---

## 🔧 Étape 6 — Configuration Spring Security

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    private final UserDetailsServiceImpl userDetailsService;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // ── Règles d'accès aux URLs ──────────────────────────
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()      // login, register : libres
                .requestMatchers("/api/admin/**").hasRole("ADMIN") // admin uniquement
                .anyRequest().authenticated()                     // tout le reste : connecté
            )
            // ── Formulaire de login ──────────────────────────────
            .formLogin(form -> form
                .loginPage("/api/auth/login")
                .permitAll()
            )
            // ── Déconnexion ──────────────────────────────────────
            .logout(logout -> logout
                .logoutUrl("/api/auth/logout")
                .permitAll()
            )
            // ── CSRF désactivé pour REST API ─────────────────────
            .csrf(csrf -> csrf.disable());

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();   // force = 10 par défaut
    }
}
```

### Pourquoi désactiver CSRF ?

CSRF (Cross-Site Request Forgery) est une protection pour les applications web classiques avec formulaires HTML. Pour une API REST consommée par Vue.js avec des tokens, CSRF n'est pas nécessaire et compliquerait les appels.

---

## 🔧 Étape 7 — DTOs (objets de transfert)

Les DTOs évitent d'exposer directement l'entité dans l'API.

```java
// RegisterRequest.java
public record RegisterRequest(
    @NotBlank String username,
    @Email @NotBlank String email,
    @Size(min = 6) String password
) {}

// LoginRequest.java
public record LoginRequest(
    @NotBlank String username,
    @NotBlank String password
) {}

// LoginResponse.java
public record LoginResponse(
    Long userId,
    String username,
    String role
) {}
```

---

## 🔧 Étape 8 — Le Contrôleur

```java
@RestController
@RequestMapping("/api/auth")
@RequiredArgsConstructor
public class AuthController {

    private final UtilisateurService utilisateurService;
    private final HttpSession session;

    // ─── INSCRIPTION ─────────────────────────────────────────────
    @PostMapping("/register")
    public ResponseEntity<Map<String, String>> register(
            @RequestBody @Valid RegisterRequest request) {
        try {
            utilisateurService.register(
                request.username(), request.email(), request.password());
            return ResponseEntity.status(201)
                .body(Map.of("message", "Compte créé avec succès"));
        } catch (IllegalArgumentException e) {
            return ResponseEntity.badRequest()
                .body(Map.of("error", e.getMessage()));
        }
    }

    // ─── CONNEXION ───────────────────────────────────────────────
    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody @Valid LoginRequest request) {
        try {
            Utilisateur utilisateur = utilisateurService.login(
                request.username(), request.password());

            // Stocker les infos en session
            session.setAttribute("userId",   utilisateur.getIdUser());
            session.setAttribute("username", utilisateur.getUsername());
            session.setAttribute("role",     utilisateur.getRole());

            return ResponseEntity.ok(new LoginResponse(
                utilisateur.getIdUser(),
                utilisateur.getUsername(),
                utilisateur.getRole().toString()
            ));
        } catch (RuntimeException e) {
            return ResponseEntity.status(401)
                .body(Map.of("error", "Identifiants incorrects"));
        }
    }

    // ─── DÉCONNEXION ─────────────────────────────────────────────
    @PostMapping("/logout")
    public ResponseEntity<Map<String, String>> logout() {
        session.invalidate();    // ← détruit la session côté serveur
        return ResponseEntity.ok(Map.of("message", "Déconnecté avec succès"));
    }

    // ─── PROFIL ──────────────────────────────────────────────────
    @GetMapping("/profile")
    public ResponseEntity<?> getProfile() {
        Long userId = (Long) session.getAttribute("userId");
        if (userId == null)
            return ResponseEntity.status(401)
                .body(Map.of("error", "Non connecté"));

        Utilisateur u = utilisateurService.findById(userId);
        return ResponseEntity.ok(new LoginResponse(
            u.getIdUser(), u.getUsername(), u.getRole().toString()));
    }
}
```

---

## 🧪 Tests avec curl

```bash
# 1. Inscription
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","email":"alice@test.com","password":"pass123"}'
# → {"message": "Compte créé avec succès"}

# 2. Connexion (récupérer le cookie JSESSIONID)
curl -c cookies.txt -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":"pass123"}'
# → {"userId":1,"username":"alice","role":"USER"}

# 3. Accès ressource sécurisée (avec le cookie)
curl -b cookies.txt http://localhost:8080/api/produits
# → [liste des produits]

# 4. Déconnexion
curl -b cookies.txt -X POST http://localhost:8080/api/auth/logout
# → {"message": "Déconnecté avec succès"}

# 5. Tentative après logout (refusé)
curl -b cookies.txt http://localhost:8080/api/produits
# → 401 Unauthorized
```

---

## ⚠️ Points importants à retenir

| Concept | Ce qu'il faut savoir |
|---|---|
| **BCrypt** | Hachage à sens unique. On ne peut pas retrouver le mot de passe original |
| **Session** | Stockée côté serveur. JSESSIONID = cookie envoyé au navigateur |
| **`@PreAuthorize`** | Contrôle l'accès au niveau méthode : `@PreAuthorize("hasRole('ADMIN')")` |
| **`PasswordEncoder`** | Bean obligatoire dans SecurityConfig, injecté dans le service |
| **CSRF désactivé** | Correct pour API REST, jamais pour applications Thymeleaf classiques |

---

## 📊 Schéma récapitulatif des rôles

```
ADMIN                               USER
─────                               ────
GET    /api/produits      ✅         GET    /api/produits      ✅
POST   /api/produits      ✅         POST   /api/produits      ❌
PUT    /api/produits/{id} ✅         PUT    /api/produits/{id} ❌
DELETE /api/produits/{id} ✅         DELETE /api/produits/{id} ❌
GET    /api/admin/**      ✅         GET    /api/admin/**      ❌
POST   /api/auth/login    ✅         POST   /api/auth/login    ✅
```

---

## ✅ Checklist de fin de lab

- [ ] Entité `Utilisateur` et enum `Role` créés
- [ ] `UtilisateurRepository` avec `findByUsername` et `existsByUsername`
- [ ] `UtilisateurService` avec `register()`, `login()`, `changePassword()`
- [ ] `UserDetailsServiceImpl` implémentant `UserDetailsService`
- [ ] `SecurityConfig` avec les règles d'accès et `BCryptPasswordEncoder`
- [ ] `AuthController` avec `/register`, `/login`, `/logout`, `/profile`
- [ ] Test des 4 scénarios curl : register → login → accès → logout
- [ ] Vérifier que `/api/produits` retourne 401 sans être connecté
