# TP1 – Spring Boot MVC (Niveau débutant) — Guide complet pas à pas

**Durée estimée** : 6-8 heures  
**Niveau** : L2/L3 Informatique  
**Langue** : Français  

---

## 🎯 Objectifs pédagogiques

À la fin de ce TP, l'étudiant sera capable de :

1. ✅ Créer un projet Spring Boot avec les dépendances appropriées.
2. ✅ Comprendre la structure et l'architecture d'une application Spring Boot.
3. ✅ Créer un contrôleur MVC et mapper des URLs.
4. ✅ Passer des données du contrôleur à la vue via le modèle.
5. ✅ Utiliser Thymeleaf pour générer des vues dynamiques.
6. ✅ Intégrer un fichier CSS et structurer les ressources statiques.
7. ✅ Construire une application MVC complète : gestion d'une liste de personnes (CRUD simplifié).
8. ✅ Écrire des tests unitaires pour valider le comportement de la couche MVC.

---

## 📋 Pré-requis

### Avant de commencer, vérifiez

- [ ] Java 17 ou 21 LTS est installé → `java -version` en terminal
- [ ] Un IDE est installé : **STS 4** (recommandé) ou IntelliJ IDEA Community / VS Code
- [ ] Accès à Internet (pour télécharger les dépendances Maven et accéder à start.spring.io)
- [ ] Connaissances de base en Java : classes, méthodes, packages, collections (List, ArrayList)

### Durée du pré-requis : 10-15 minutes

---

## 📚 Concepts clés à retenir

### Spring Boot

**Qu'est-ce que Spring Boot ?**

> Spring Boot est une extension du framework Spring qui simplifie le démarrage et le développement d'applications Java en supprimant la complexité de la configuration XML.

**Avantages de Spring Boot**

- ✅ Configuration automatique (autoconfiguration)
- ✅ Serveur web intégré (Tomcat)
- ✅ Déploiement simplifié
- ✅ Dépendances gérées automatiquement (versions compatibles)

**Références officielles**

- https://spring.io/projects/spring-boot
- https://start.spring.io

---

### Architecture MVC (Model – View – Controller)

**Schéma conceptuel**

```
┌─────────────────────────────────────────────────────────┐
│                     Navigateur web                       │
│                  (Utilisateur final)                     │
└──────────────────────┬──────────────────────────────────┘
                       │
                   HTTP Request (GET, POST)
                       ↓
┌──────────────────────────────────────────────────────────┐
│                   CONTROLLER                             │
│          (Reçoit les requêtes HTTP)                      │
│   - @GetMapping, @PostMapping                           │
│   - Traitement logique métier                           │
│   - Prépare les données pour la vue                     │
└──────────────┬──────────────────────┬───────────────────┘
               │                      │
               ↓                      ↓
        ┌─────────────┐         ┌──────────────┐
        │  MODEL      │         │  VIEW        │
        │ (Données)   │         │ (Thymeleaf)  │
        │  - Classes  │         │ - Templates  │
        │  - Objects  │         │ - HTML + CSS │
        │  - Collections         │              │
        └─────────────┘         └──────────────┘
                       ↑                 │
                       │                 │
                       └────────────────┘
                     HTML rendu au navigateur
```

**Flux MVC dans ce TP**

1. L'utilisateur accède à une URL (ex : `/home?framework=Spring`)
2. Le **Controller** reçoit la requête
3. Le **Controller** prépare les données (Model)
4. Le **Controller** sélectionne une **View** (template Thymeleaf)
5. La **View** rend le HTML en utilisant les données du Model
6. La réponse HTML est envoyée au navigateur

---

## 🚀 Partie 1 : Création du projet Spring Boot

### Étape 1.1 : Accéder à Spring Initializr

**Objectif** : générer un squelette de projet Spring Boot avec les dépendances nécessaires.

**Procédure pas à pas**

1. Ouvrir le navigateur et aller sur : https://start.spring.io

2. Vous verrez une interface qui ressemble à ceci :

```
┌─────────────────────────────────────────────────────┐
│  Spring Initializr – Generate a Spring Boot project │
│                                                     │
│  Project : Maven Project    ▼                       │
│  Language : Java            ▼                       │
│  Spring Boot : 3.x.x        ▼                       │
│                                                     │
│  Group            : [com.example_________]          │
│  Artifact         : [springboot-mvc-workshop]       │
│  Name             : [SpringBoot MVC Workshop]       │
│  Description      : [My Spring Boot application]   │
│  Package name     : [com.example.springbootmvc]    │
│  Packaging        : Jar     ▼                       │
│  Java             : 17      ▼                       │
│                                                     │
│  Dependencies : [ + Add Dependencies ]              │
│                                                     │
│                    [ GENERATE ]                     │
└─────────────────────────────────────────────────────┘
```

**À remplir**

| Champ | Valeur |
|-------|--------|
| Project | Maven Project |
| Language | Java |
| Spring Boot | 3.3.x ou 3.4.x (dernière version) |
| Group | `com.example` |
| Artifact | `springboot-mvc-workshop` |
| Name | `SpringBoot MVC Workshop` |
| Description | `TP MVC avec Thymeleaf` |
| Package name | `com.example.springbootmvc` |
| Packaging | Jar |
| Java | 17 ou 21 |

### Étape 1.2 : Ajouter les dépendances

**Important** : avant de générer, cliquer sur **Add Dependencies** et sélectionner :

1. **Spring Web**
   - Fournit les annotations `@Controller`, `@GetMapping`, etc.
   - Inclut Tomcat (serveur web)

2. **Thymeleaf**
   - Moteur de templates pour générer du HTML dynamique
   - Alternative à JSP (plus moderne et lisible)

3. **Spring Boot DevTools** (optionnel mais recommandé)
   - Redémarre automatiquement l'application après un changement de code
   - Améliore la productivité pendant le développement

**Après ajout des dépendances, le formulaire montre**

```
Dependencies
  [X] Spring Web
  [X] Thymeleaf
  [X] Spring Boot DevTools
```

### Étape 1.3 : Générer et télécharger

1. Cliquer sur **GENERATE** (en bas à droite).
2. Un fichier `.zip` est téléchargé : `springboot-mvc-workshop.zip`
3. Extraire le fichier dans un dossier de travail : `C:\Users\...\workspace\springboot-mvc-workshop` (Windows) ou équivalent (Mac/Linux).

**Résultat attendu après extraction**

```
springboot-mvc-workshop/
├── src/
│   ├── main/
│   │   ├── java/com/example/springbootmvc/
│   │   │   └── SpringbootMvcWorkshopApplication.java
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── static/           ← Fichiers CSS, JS, images
│   │       └── templates/        ← Fichiers HTML (Thymeleaf)
│   └── test/
│       └── java/...
├── pom.xml                        ← Configuration Maven
├── mvnw (Windows : mvnw.cmd)      ← Wrapper Maven
└── README.md
```

---

## 💻 Partie 2 : Import du projet dans l'IDE et découverte

### Étape 2.1 : Importer le projet dans STS 4

**Si vous utilisez STS 4**

1. Lancer STS 4.
2. Aller dans `File → Open Projects from File System...`
3. Sélectionner le dossier `springboot-mvc-workshop`.
4. Cliquer **Finish**.
5. Attendre que Maven télécharge les dépendances (quelques minutes).

**Vous verrez dans l'onglet "Console"**

```
[INFO] Scanning for projects...
[INFO] Downloading org.springframework.boot:spring-boot:3.3.0
[INFO] Downloaded org.springframework.boot:spring-boot:3.3.0 (29 KB)
...
[INFO] BUILD SUCCESS
```

**Si vous utilisez IntelliJ IDEA**

1. Aller dans `File → Open...`
2. Sélectionner le dossier du projet.
3. Attendre que les dépendances se téléchargent (Maven).

**Si vous utilisez VS Code**

1. Ouvrir le dossier avec `File → Open Folder...`
2. Installer les extensions :
   - "Extension Pack for Java" (Microsoft)
   - "Spring Boot Extension Pack" (Pivotal)
3. Attendre l'indexation.

### Étape 2.2 : Explorer la structure du projet

**Ouvrir et examiner les fichiers clés**

1. **`SpringbootMvcWorkshopApplication.java`** (classe principale)

```java
package com.example.springbootmvc;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringbootMvcWorkshopApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootMvcWorkshopApplication.class, args);
    }
}
```

> **L'annotation `@SpringBootApplication`** : elle combine trois annotations importantes :
> - `@Configuration` : cette classe contient des configurations Spring
> - `@ComponentScan` : autorise le scan des composants (@Controller, @Service, etc.)
> - `@EnableAutoConfiguration` : active l'autoconfiguration de Spring Boot

2. **`pom.xml`** (fichier Maven)

```xml
<project ...>
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.0</version>
    </parent>
    <groupId>com.example</groupId>
    <artifactId>springboot-mvc-workshop</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

> **Qu'est-ce que Maven (pom.xml) ?**
> - Maven gère les dépendances du projet (comme npm pour Node.js ou composer pour PHP).
> - Le fichier `pom.xml` déclare quelles "briques" (JAR) votre application utilise.
> - Maven télécharge automatiquement ces briques et les rend disponibles au projet.

3. **`application.properties`** (fichier de configuration)

```properties
# Vide par défaut, on peut y ajouter des configurations
# server.port=8080
# spring.application.name=springboot-mvc-workshop
```

> **À quoi sert `application.properties` ?**
> - Centralise les configurations de l'application (port du serveur, base de données, logs, etc.).
> - Les valeurs peuvent être injectées dans les beans Spring via `@Value`.

### Étape 2.3 : Premiers repères

**À mémoriser**

- `src/main/java/` → code source Java
- `src/main/resources/static/` → fichiers statiques (CSS, JS, images, HTML statiques)
- `src/main/resources/templates/` → vues Thymeleaf (HTML dynamique)
- `src/main/resources/application.properties` → configuration de l'app
- `src/test/java/` → tests unitaires

**Questions de compréhension**

1. Pourquoi y a-t-il deux emplacements pour les fichiers HTML (static vs templates) ?
   - **Réponse** : `static` = HTML statiques servis directement. `templates` = HTML traités par Thymeleaf (injection de données dynamiques).

2. À quoi sert le dossier `test` ?
   - **Réponse** : contient les classes de test (JUnit) pour valider que le code fonctionne correctement.

---

## 🏃 Partie 3 : Lancer l'application et voir la première erreur

### Étape 3.1 : Démarrer l'application

**Avec STS 4**

1. Clic droit sur le projet → `Run As → Spring Boot App`
2. Attendre que le serveur démarre (10-15 secondes)

**Avec IntelliJ IDEA**

1. Clic droit sur la classe `SpringbootMvcWorkshopApplication` → `Run 'SpringbootMvcWorkshopApplication.main()'`

**Avec VS Code (terminal)**

```bash
cd C:\Users\...\springboot-mvc-workshop
./mvnw spring-boot:run
```

### Étape 3.2 : Vérifier le démarrage

**Dans la console (console de l'IDE), vous devriez voir**

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_|\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 
 :: Spring Boot ::        (v3.3.0)

2026-02-13 07:50:00.123  INFO 1234 --- [  main] c.e.s.SpringbootMvcWorkshopApplication  : Starting SpringbootMvcWorkshopApplication
2026-02-13 07:50:02.456  INFO 1234 --- [  main] o.s.b.w.e.t.TomcatWebServer              : Tomcat started on port(s): 8080 (http)
2026-02-13 07:50:02.789  INFO 1234 --- [  main] c.e.s.SpringbootMvcWorkshopApplication  : Started SpringbootMvcWorkshopApplication in 2.345 seconds (JVM running for 3.456)
```

✅ **Excellent** ! Le serveur écoute sur `http://localhost:8080`

### Étape 3.3 : Accéder à l'application

1. Ouvrir un navigateur web (Chrome, Firefox, Edge, Safari).
2. Aller à : `http://localhost:8080`

**Vous verrez cette page d'erreur (c'est normal !)**

```
Whitelabel Error Page

This application has no explicit mapping for /error, 
so you are seeing this as a fallback.

There was an unexpected error (type=Not Found, status=404).

No message available
```

**Pourquoi cette erreur ?**

- ✅ Le serveur fonctionne correctement.
- ❌ Aucun contrôleur n'a mappé l'URL `/` (accueil).
- ❌ Aucune vue n'a été créée.

> **C'est attendu et normal à ce stade du TP !** On va créer notre premier contrôleur à la prochaine étape.

**Questions de compréhension**

1. Qu'est-ce qu'une erreur HTTP 404 ?
   - **Réponse** : "Not Found" - la ressource demandée n'existe pas sur le serveur.

2. Pourquoi l'application démarre correctement alors qu'on obtient une erreur au navigateur ?
   - **Réponse** : Démarrer l'application ≠ avoir du contenu à servir. Tomcat tourne, mais pas de routes définies.

---

## 🎨 Partie 4 : Créer le premier contrôleur

### Étape 4.1 : Créer le package des contrôleurs

**Objectif** : organiser le code dans des packages logiques.

**Procédure**

1. Dans l'IDE, clic droit sur `src/main/java/com/example/springbootmvc/` → `New → Package`
2. Nommer le package : `com.example.springbootmvc.controllers`
3. Cliquer **Finish**.

**Résultat**

```
src/main/java/
└── com/example/springbootmvc/
    ├── SpringbootMvcWorkshopApplication.java
    └── controllers/              ← Nouveau package
```

### Étape 4.2 : Créer la classe FirstController

**Procédure**

1. Clic droit sur le package `controllers` → `New → Class`
2. Nommer la classe : `FirstController`
3. Cliquer **Finish**.

**Contenu initial (à remplir)**

```java
package com.example.springbootmvc.controllers;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class FirstController {

    @GetMapping("/")
    @ResponseBody
    public String home() {
        return "Hello, Spring Boot MVC !";
    }
}
```

### Étape 4.3 : Comprendre les annotations

| Annotation | Rôle |
|-----------|------|
| `@Controller` | Marque la classe comme un contrôleur Spring MVC (composant géré par Spring) |
| `@GetMapping("/")` | Mappe les requêtes HTTP GET sur l'URL `/` à la méthode `home()` |
| `@ResponseBody` | Indique que la valeur retournée par `home()` doit être écrite directement dans le corps de la réponse HTTP (pas une vue Thymeleaf) |

**Schéma du flux**

```
Navigateur                      Serveur
─────────────────────────────────────────

GET http://localhost:8080/
      │
      │  Requête HTTP GET sur "/"
      ├──────────────────────────→  Spring cherche @GetMapping("/")
                                    ↓
                                    Trouve FirstController.home()
                                    ↓
                                    Exécute home()
                                    ↓
                                    Retourne "Hello, Spring Boot MVC !"
                                    ↓
      ←──────────────────────────────┘
      │
   Réponse HTML simple
   affichée au navigateur
```

### Étape 4.4 : Tester le contrôleur

1. **Sauvegarder** le fichier `FirstController.java` (Ctrl+S ou Cmd+S).
2. **Grâce à DevTools**, l'application redémarre automatiquement.
3. **Dans la console**, vous verrez :

```
2026-02-13 07:55:10.123  INFO 1234 --- [  main] c.e.s.SpringbootMvcWorkshopApplication  : Started SpringbootMvcWorkshopApplication in 1.234 seconds
```

4. **Rafraîchir le navigateur** (F5 ou Cmd+R) sur `http://localhost:8080`

**Résultat attendu au navigateur**

```
Hello, Spring Boot MVC !
```

✅ **Succès !** Votre premier contrôleur fonctionne !

### Exercice 4.1 (à faire immédiatement)

**Modifiez le code de `home()`**

1. Retournez un texte en HTML lisible :

```java
@GetMapping("/")
@ResponseBody
public String home() {
    return "<h1>Bienvenue sur mon premier contrôleur Spring !</h1>" +
           "<p>Ceci est une réponse depuis FirstController</p>";
}
```

2. Sauvegardez et rafraîchissez le navigateur.
3. Observez que le navigateur rend maintenant du HTML formaté.

**Questions de compréhension**

1. Quelle est la différence entre une réponse `@ResponseBody` et une réponse qui retourne une vue Thymeleaf ?
   - **Réponse** : `@ResponseBody` retourne directement du texte/HTML brut. Une vue Thymeleaf retourne le nom logique d'un template, que Spring résout ensuite.

2. Si je veux afficher 100 lignes de HTML, est-ce que c'est une bonne pratique de retourner une longue chaîne depuis `@ResponseBody` ?
   - **Réponse** : Non ! C'est pour ça qu'on utilise Thymeleaf et des fichiers `.html` séparés. C'est plus lisible et maintenable.

---

## 📄 Partie 5 : Créer une vue Thymeleaf

### Étape 5.1 : Créer la structure de dossiers pour les vues

**Procédure (via l'IDE ou l'explorateur de fichiers)**

1. Aller dans `src/main/resources/templates/`
2. Créer un dossier `pages`

**Résultat**

```
src/main/resources/
├── templates/
│   └── pages/                  ← Nouveau dossier
└── static/
```

### Étape 5.2 : Créer le fichier home.html

**Procédure**

1. Clic droit sur le dossier `templates/pages/` → `New → File`
2. Nommer le fichier : `home.html`
3. Cliquer **Finish**.

**Contenu du fichier home.html**

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Accueil Spring Boot MVC</title>
</head>
<body>
    <h1>Ma première vue Thymeleaf</h1>
    <p>Ceci est une vue Thymeleaf rendue par Spring Boot MVC</p>
</body>
</html>
```

### Étape 5.3 : Modifier le contrôleur pour retourner la vue

**Ancien code (à remplacer)**

```java
@GetMapping("/")
@ResponseBody
public String home() {
    return "<h1>Bienvenue sur mon premier contrôleur Spring !</h1>";
}
```

**Nouveau code**

```java
@GetMapping("/")
public String home() {
    return "pages/home";  // Spring cherchera templates/pages/home.html
}
```

**Points clés**

- ❌ On enlève l'annotation `@ResponseBody`
- ✅ On retourne le nom logique de la vue : `"pages/home"` (sans l'extension `.html`)
- ✅ Spring résout automatiquement ce nom vers `src/main/resources/templates/pages/home.html`

### Étape 5.4 : Tester la vue

1. **Sauvegarder** le fichier contrôleur.
2. **Attendre** que DevTools redémarre l'app (message dans la console).
3. **Rafraîchir** le navigateur sur `http://localhost:8080`

**Résultat attendu**

```
Ma première vue Thymeleaf

Ceci est une vue Thymeleaf rendue par Spring Boot MVC
```

✅ **Succès !** Votre première vue Thymeleaf fonctionne !

### Schéma du flux (avec vue Thymeleaf)

```
Navigateur                          Serveur
─────────────────────────────────────────────────

GET http://localhost:8080/
      │
      │  Requête HTTP GET
      ├────────────────────────────→  Spring trouve @GetMapping("/")
                                      ↓
                                      Exécute home() → retourne "pages/home"
                                      ↓
                                      Spring et Thymeleaf résolvent
                                      "pages/home" → templates/pages/home.html
                                      ↓
                                      Charge le fichier HTML
                                      ↓
      ←────────────────────────────────┘
      │
   HTML complet
   affiché au navigateur
```

### Questions de compréhension

1. Pourquoi écrit-on `"pages/home"` et non `"pages/home.html"` ?
   - **Réponse** : Thymeleaf ajoute automatiquement `.html` et cherche dans `templates/`. C'est une convention pour simplifier le code.

2. Où se trouve physiquement le fichier `home.html` sur le disque ?
   - **Réponse** : Dans le projet : `src/main/resources/templates/pages/home.html`

3. Que se passe-t-il si on renomme le fichier en `home2.html` et qu'on rafraîchit le navigateur ?
   - **Réponse** : On obtient une erreur 404 ou "Template not found" car Spring ne trouve pas `pages/home.html`.

---

## 🔄 Partie 6 : Passer des données du contrôleur à la vue (Model)

### Étape 6.1 : Utiliser @RequestParam

**Objectif** : récupérer un paramètre de l'URL et le passer à la vue.

**Exemple d'URL**

```
http://localhost:8080/home?framework=Spring
```

Le paramètre `framework` a la valeur `Spring`.

### Étape 6.2 : Modifier le contrôleur

**Code à écrire**

```java
package com.example.springbootmvc.controllers;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class FirstController {

    @GetMapping("/")
    public String home(
        @RequestParam(required = false, defaultValue = "Spring Boot") String framework,
        Model model
    ) {
        // Ajouter la variable au modèle
        model.addAttribute("myframework", framework);
        
        // Retourner la vue
        return "pages/home";
    }
}
```

**Explication ligne par ligne**

| Code | Explication |
|------|-------------|
| `@RequestParam(...)` | Récupère un paramètre de l'URL |
| `required = false` | Le paramètre est optionnel (pas d'erreur s'il manque) |
| `defaultValue = "Spring Boot"` | Valeur par défaut si le paramètre manque |
| `String framework` | Le paramètre est stocké dans la variable `framework` |
| `Model model` | Objet spring pour transmettre des données à la vue |
| `model.addAttribute("myframework", framework)` | Ajoute au modèle : clé = `myframework`, valeur = contenu de `framework` |
| `return "pages/home"` | Retourne la vue (qui a accès à `myframework` via le model) |

### Étape 6.3 : Modifier la vue home.html

**Nouveau contenu**

```html
<!DOCTYPE html>
<html lang="fr" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Accueil Spring Boot MVC</title>
</head>
<body>
    <h1>Ma première vue Thymeleaf</h1>
    <p>Ceci est une vue Thymeleaf rendue par Spring Boot MVC</p>
    
    <!-- Affiche la valeur de myframework -->
    <h2 th:text="'Votre framework préféré est : ' + ${myframework}"></h2>
</body>
</html>
```

**Points clés**

- `xmlns:th="http://www.thymeleaf.org"` : déclare le namespace Thymeleaf (obligatoire pour utiliser `th:*`)
- `th:text="..."` : l'attribut Thymeleaf qui remplace le contenu du texte
- `${myframework}` : syntaxe pour accéder à une variable du model (ici, `myframework`)
- `'...' + ${myframework}` : concaténation de texte avec la variable

### Étape 6.4 : Tester

1. **Sauvegarder** les fichiers (contrôleur et vue).
2. **Rafraîchir** le navigateur sur `http://localhost:8080/`

**Résultat (URL sans paramètre)**

```
Ma première vue Thymeleaf

Ceci est une vue Thymeleaf rendue par Spring Boot MVC

Votre framework préféré est : Spring Boot
```

3. Essayer avec un paramètre : `http://localhost:8080/?framework=Java`

**Résultat (avec paramètre)**

```
...
Votre framework préféré est : Java
```

✅ **Succès !** Les données passent du contrôleur à la vue !

### Schéma du flux (avec Model)

```
Navigateur                          Serveur (Java)
─────────────────────────────────────────────────────

GET http://localhost:8080/?framework=Java
      │
      │  Requête avec paramètre
      ├────────────────────────────→  Spring intercepte
                                      ↓
                                      @RequestParam extrait framework = "Java"
                                      ↓
                                      home(framework, model) exécutée
                                      ↓
                                      model.addAttribute("myframework", "Java")
                                      ↓
                                      Thymeleaf traite home.html
                                      ↓
                                      ${myframework} → "Java"
                                      ↓
      ←────────────────────────────────┘
      │
   HTML avec "Java" intégré
```

### Exercice 6.1 (à faire)

**Ajouter un deuxième paramètre optionnel**

1. Modifiez le contrôleur pour accepter aussi un paramètre `version` (ex : `v3.3.0`).
2. Modifiez la vue pour afficher : `"Votre framework préféré est Spring avec la version 3.3.0"` (en utilisant la variable `version`).

**Indices**

- Ajouter un nouveau paramètre `@RequestParam` dans la signature de `home()`
- Ajouter `model.addAttribute("myversion", version)`
- Modifier le `th:text` pour afficher aussi `${myversion}`

### Questions de compréhension

1. Que se passe-t-il si j'accède à `http://localhost:8080/?framework=Quarkus` et que le paramètre n'existe pas dans le contrôleur avec `required = true` ?
   - **Réponse** : Spring génère une erreur `MissingServletRequestParameterException` (400 Bad Request).

2. Quelle est la différence entre `Model` et `ModelMap` ?
   - **Réponse** : Très minime en pratique. `Model` est une interface, `ModelMap` est une implémentation. Dans ce TP, on utilise `Model` (plus moderne).

3. Dans la vue, pourquoi utilise-t-on `${myframework}` et non simplement `myframework` ?
   - **Réponse** : `${}` est la notation Thymeleaf pour accéder aux variables du model. C'est la syntaxe requise.

---

## 🎨 Partie 7 : Ajouter un fichier CSS et styliser la vue

### Étape 7.1 : Créer le fichier CSS

**Procédure**

1. Dans `src/main/resources/static/`, créer un dossier `css`
2. Créer un fichier `style.css`

**Résultat**

```
src/main/resources/static/
└── css/
    └── style.css               ← Nouveau fichier
```

### Étape 7.2 : Ajouter du contenu CSS

**Contenu de style.css**

```css
/* Styles généraux */
body {
    font-family: Arial, sans-serif;
    margin: 20px;
    background-color: #f5f5f5;
}

h1 {
    color: #2c3e50;
    text-align: center;
    border-bottom: 3px solid #3498db;
    padding-bottom: 10px;
}

h2 {
    color: #27ae60;
    margin-top: 20px;
}

p {
    color: #34495e;
    line-height: 1.6;
}
```

### Étape 7.3 : Lier le CSS à la vue Thymeleaf

**Modifier home.html**

```html
<!DOCTYPE html>
<html lang="fr" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Accueil Spring Boot MVC</title>
    
    <!-- Lien vers le fichier CSS -->
    <link rel="stylesheet" type="text/css" th:href="@{/css/style.css}">
</head>
<body>
    <h1>Ma première vue Thymeleaf</h1>
    <p>Ceci est une vue Thymeleaf rendue par Spring Boot MVC</p>
    
    <!-- Affiche la valeur de myframework -->
    <h2 th:text="'Votre framework préféré est : ' + ${myframework}"></h2>
</body>
</html>
```

**Point clé**

- `th:href="@{/css/style.css}"` : la notation Thymeleaf `@{}` génère automatiquement l'URL correcte
- `/css/style.css` : Spring cherche dans `src/main/resources/static/css/style.css`

### Étape 7.4 : Tester le CSS

1. **Sauvegarder** les fichiers.
2. **Rafraîchir** le navigateur.

**Résultat**

La page affiche maintenant les styles CSS :
- Titre centré en bleu
- Texte en gris
- Arrière-plan gris clair

✅ **Succès !** Le CSS est bien appliqué !

### Questions de compréhension

1. Pourquoi utilise-t-on `th:href="@{/css/style.css}"` au lieu de `href="/css/style.css"` ?
   - **Réponse** : C'est une bonne pratique Thymeleaf. La notation `@{}` gère les contextes d'application (utile si l'app n'est pas à la racine du serveur).

2. Où doit-on mettre les images et les fichiers JavaScript ?
   - **Réponse** : Aussi dans `src/main/resources/static/` dans des sous-dossiers respectifs : `images/` et `js/`.

3. Si je mets un CSS dans `templates/` au lieu de `static/`, ça fonctionne ?
   - **Réponse** : Non. `templates/` est réservé aux fichiers traités par Thymeleaf. Les ressources statiques doivent être dans `static/`.

---

## 👥 Partie 8 : Construire une application MVC complète (Gestion de personnes)

### Vue d'ensemble

À cette étape, on va créer une application pour gérer une liste de personnes avec :

- Une **classe entité** `Person` (firstName, lastName)
- Une **classe formulaire** `PersonForm` pour les données POST
- Un **contrôleur** `PersonController` avec plusieurs actions
- Trois **vues** :
  1. `index.html` : page d'accueil
  2. `personList.html` : affichage de la liste
  3. `addPerson.html` : formulaire d'ajout

### Étape 8.1 : Créer le package entities

**Procédure**

1. Clic droit sur `src/main/java/com/example/springbootmvc/` → `New → Package`
2. Nommer : `com.example.springbootmvc.entities`

### Étape 8.2 : Créer la classe Person

**Procédure**

1. Clic droit sur le package `entities` → `New → Class`
2. Nommer : `Person`

**Contenu**

```java
package com.example.springbootmvc.entities;

public class Person {
    
    private String firstName;
    private String lastName;
    
    // Constructeur vide (obligatoire pour Spring)
    public Person() {
    }
    
    // Constructeur avec paramètres
    public Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
    
    // Getters et setters
    public String getFirstName() {
        return firstName;
    }
    
    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }
    
    public String getLastName() {
        return lastName;
    }
    
    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
    
    // toString (utile pour déboguer)
    @Override
    public String toString() {
        return "Person{" +
                "firstName='" + firstName + '\'' +
                ", lastName='" + lastName + '\'' +
                '}';
    }
}
```

### Étape 8.3 : Créer le package forms

**Procédure**

1. Clic droit sur `src/main/java/com/example/springbootmvc/` → `New → Package`
2. Nommer : `com.example.springbootmvc.forms`

### Étape 8.4 : Créer la classe PersonForm

**Procédure**

1. Clic droit sur le package `forms` → `New → Class`
2. Nommer : `PersonForm`

**Contenu**

```java
package com.example.springbootmvc.forms;

public class PersonForm {
    
    private String firstName;
    private String lastName;
    
    // Getters et setters
    public String getFirstName() {
        return firstName;
    }
    
    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }
    
    public String getLastName() {
        return lastName;
    }
    
    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
}
```

### Étape 8.5 : Mettre à jour application.properties

**Contenu à ajouter dans `application.properties`**

```properties
# Configuration serveur
server.port=8080

# Messages personnalisés
welcome.message=Bienvenue sur la gestion des personnes !
error.message=Le prénom et le nom de famille sont obligatoires.
```

### Étape 8.6 : Créer PersonController

**Procédure**

1. Clic droit sur le package `controllers` → `New → Class`
2. Nommer : `PersonController`

**Contenu**

```java
package com.example.springbootmvc.controllers;

import java.util.ArrayList;
import java.util.List;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import com.example.springbootmvc.entities.Person;
import com.example.springbootmvc.forms.PersonForm;

@Controller
@RequestMapping("/person")
public class PersonController {
    
    // Liste statique de personnes (en mémoire, pour ce TP)
    private static List<Person> persons = new ArrayList<>();
    
    // Initialiser avec quelques données
    static {
        persons.add(new Person("Albert", "Einstein"));
        persons.add(new Person("Marie", "Curie"));
        persons.add(new Person("Stephen", "Hawking"));
    }
    
    // Injecter les messages depuis application.properties
    @Value("${welcome.message}")
    private String welcomeMessage;
    
    @Value("${error.message}")
    private String errorMessage;
    
    // Action 1 : Afficher la page d'accueil
    @GetMapping("/index")
    public String index(Model model) {
        model.addAttribute("message", welcomeMessage);
        return "pages/person/index";
    }
    
    // Action 2 : Afficher la liste des personnes
    @GetMapping("/list")
    public String personList(Model model) {
        model.addAttribute("persons", persons);
        return "pages/person/personList";
    }
    
    // Action 3 : Afficher le formulaire d'ajout (GET)
    @GetMapping("/add")
    public String showAddPersonPage(Model model) {
        PersonForm personForm = new PersonForm();
        model.addAttribute("personForm", personForm);
        return "pages/person/addPerson";
    }
    
    // Action 4 : Traiter l'ajout d'une personne (POST)
    @PostMapping("/add")
    public String savePerson(
        Model model,
        @ModelAttribute("personForm") PersonForm personForm
    ) {
        String firstName = personForm.getFirstName();
        String lastName = personForm.getLastName();
        
        // Validation simple
        if (firstName != null && firstName.trim().length() > 0 &&
            lastName != null && lastName.trim().length() > 0) {
            
            // Créer une nouvelle personne et l'ajouter à la liste
            Person newPerson = new Person(firstName, lastName);
            persons.add(newPerson);
            
            // Rediriger vers la liste
            return "redirect:/person/list";
        }
        
        // En cas d'erreur, afficher le message d'erreur
        model.addAttribute("errorMessage", errorMessage);
        return "pages/person/addPerson";
    }
}
```

**Explication des annotations utilisées**

| Annotation | Rôle |
|-----------|------|
| `@Controller` | Classe contrôleur |
| `@RequestMapping("/person")` | Préfixe pour toutes les routes : `/person/index`, `/person/list`, etc. |
| `@GetMapping("/list")` | Route complète : `GET /person/list` |
| `@PostMapping("/add")` | Route complète : `POST /person/add` |
| `@ModelAttribute("personForm")` | Lie le formulaire POST à un objet `PersonForm` |
| `@Value("${...}")` | Injecte une valeur depuis `application.properties` |

### Étape 8.7 : Créer les vues

**Structure à créer**

```
src/main/resources/templates/
└── pages/
    └── person/
        ├── index.html
        ├── personList.html
        └── addPerson.html
```

**Créer le dossier `pages/person` (via l'IDE ou l'explorateur)**

### Étape 8.8 : Créer index.html

**Contenu**

```html
<!DOCTYPE html>
<html lang="fr" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Accueil - Gestion des personnes</title>
    <link rel="stylesheet" type="text/css" th:href="@{/css/style.css}">
</head>
<body>
    <h1>Gestion des personnes</h1>
    <h2 th:text="${message}"></h2>
    
    <ul>
        <li><a th:href="@{/person/list}">Afficher la liste des personnes</a></li>
        <li><a th:href="@{/person/add}">Ajouter une personne</a></li>
    </ul>
</body>
</html>
```

### Étape 8.9 : Créer personList.html

**Contenu**

```html
<!DOCTYPE html>
<html lang="fr" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Liste des personnes</title>
    <link rel="stylesheet" type="text/css" th:href="@{/css/style.css}">
</head>
<body>
    <h1>Liste des personnes</h1>
    
    <table border="1">
        <tr>
            <th>Prénom</th>
            <th>Nom</th>
        </tr>
        <!-- Boucle Thymeleaf : th:each -->
        <tr th:each="person : ${persons}">
            <td th:text="${person.firstName}"></td>
            <td th:text="${person.lastName}"></td>
        </tr>
    </table>
    
    <br/>
    <a th:href="@{/person/add}">Ajouter une personne</a>
    <br/>
    <a th:href="@{/person/index}">Retour à l'accueil</a>
</body>
</html>
```

**Explication de la boucle**

```html
<tr th:each="person : ${persons}">
```

- `th:each="person : ${persons}"` : pour chaque objet `person` dans la liste `persons`
- À chaque itération, `person` est accessible dans le contexte de la boucle
- Ainsi, `${person.firstName}` accède à la propriété `firstName` de chaque personne

### Étape 8.10 : Créer addPerson.html

**Contenu**

```html
<!DOCTYPE html>
<html lang="fr" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ajouter une personne</title>
    <link rel="stylesheet" type="text/css" th:href="@{/css/style.css}">
</head>
<body>
    <h1>Ajouter une nouvelle personne</h1>
    
    <!-- Formulaire avec th:object et th:field -->
    <form th:action="@{/person/add}" th:object="${personForm}" method="POST">
        <label for="firstName">Prénom :</label>
        <input type="text" id="firstName" th:field="*{firstName}" required/>
        <br/>
        
        <label for="lastName">Nom de famille :</label>
        <input type="text" id="lastName" th:field="*{lastName}" required/>
        <br/>
        
        <input type="submit" value="Créer"/>
    </form>
    
    <!-- Afficher le message d'erreur s'il existe -->
    <div th:if="${errorMessage != null}" style="color: red; margin-top: 15px;">
        <strong>Erreur :</strong> <span th:text="${errorMessage}"></span>
    </div>
    
    <br/>
    <a th:href="@{/person/list}">Retour à la liste</a>
</body>
</html>
```

**Explication des attributs Thymeleaf pour formulaires**

| Attribut | Rôle |
|----------|------|
| `th:action="@{/person/add}"` | URL cible du formulaire (POST) |
| `th:object="${personForm}"` | Lie le formulaire à l'objet `personForm` |
| `th:field="*{firstName}"` | Crée un `<input>` pour la propriété `firstName` |
| `th:if="${errorMessage != null}"` | Affiche le `<div>` si `errorMessage` n'est pas null |
| `th:text="${errorMessage}"` | Affiche le contenu de `errorMessage` |

### Étape 8.11 : Tester l'application complète

1. **Sauvegarder** tous les fichiers.
2. **Attendre** que DevTools redémarre l'app.
3. **Accéder** à : `http://localhost:8080/person/index`

**Flux de test**

| Étape | URL | Action | Résultat attendu |
|-------|-----|--------|------------------|
| 1 | `http://localhost:8080/person/index` | Affichage page d'accueil | "Bienvenue sur la gestion des personnes !" |
| 2 | Cliquer "Afficher la liste" | Naviguer vers `/person/list` | Liste avec Albert Einstein, Marie Curie, Stephen Hawking |
| 3 | Cliquer "Ajouter une personne" | Naviguer vers `/person/add` | Formulaire vide |
| 4 | Remplir "Ada" et "Lovelace", cliquer Créer | POST vers `/person/add` | Redirection vers la liste avec Ada Lovelace ajoutée |
| 5 | Soumettre le formulaire vide | POST sans remplir | Message d'erreur : "Le prénom et le nom sont obligatoires." |

✅ **Succès !** L'application MVC complète fonctionne !

### Schéma du flux complet

```
Utilisateur navigue

1. http://localhost:8080/person/index
   ↓
   GET /person/index
   ↓
   PersonController.index() → ajoute message au Model
   ↓
   Retourne "pages/person/index"
   ↓
   Thymeleaf rend index.html
   ↓
   Page d'accueil affichée

2. Clic sur "Afficher la liste"
   ↓
   GET /person/list
   ↓
   PersonController.personList() → ajoute personnes au Model
   ↓
   Retourne "pages/person/personList"
   ↓
   Thymeleaf rend personList.html (th:each pour chaque personne)
   ↓
   Liste affichée

3. Clic sur "Ajouter une personne"
   ↓
   GET /person/add
   ↓
   PersonController.showAddPersonPage() → crée PersonForm vide
   ↓
   Retourne "pages/person/addPerson"
   ↓
   Formulaire vide affiché

4. Remplir le formulaire et soumettre
   ↓
   POST /person/add avec firstName et lastName
   ↓
   PersonController.savePerson() traite les données
   ↓
   Si valide : nouvelle Person créée et ajoutée à la liste
   ↓
   Redirection (redirect:/person/list)
   ↓
   GET /person/list lancé automatiquement
   ↓
   Liste mise à jour affichée
```

### Exercice 8.1 (à faire)

**Ajouter les actions DELETE et EDIT**

1. **Supprimer une personne**
   - Ajouter un lien "Supprimer" dans la table de `personList.html`
   - Créer une action `@GetMapping("/delete/{index}")` dans le contrôleur
   - Supprimer la personne à l'index donné

2. **Modifier une personne**
   - Ajouter un lien "Modifier" dans la table
   - Créer une vue `editPerson.html` similaire à `addPerson.html`
   - Créer deux actions : GET pour afficher le formulaire pré-rempli, POST pour traiter l'édition

### Questions de compréhension

1. Pourquoi utilise-t-on une liste statique `persons` et non une base de données ?
   - **Réponse** : Pour simplifier ce premier TP. Les données disparaissent au redémarrage, c'est normal à ce stade.

2. Que signifie `redirect:/person/list` dans le contrôleur ?
   - **Réponse** : C'est une redirection HTTP. Après ajouter une personne, on renvoie le navigateur vers `/person/list` pour voir la liste mise à jour.

3. Quelle est la différence entre `GET /person/add` et `POST /person/add` ?
   - **Réponse** : GET affiche le formulaire vide. POST traite les données soumises.

4. Qu'est-ce que `@ModelAttribute("personForm")` fait exactement ?
   - **Réponse** : Spring crée automatiquement un objet `PersonForm` à partir des paramètres POST du formulaire (automatiquement récupérés grâce à `th:field`).

---

## 🧪 Partie 9 : Tests unitaires avec JUnit (optionnel)

### Étape 9.1 : Pourquoi tester ?

**Objectifs des tests**

- ✅ Valider que le contrôleur se comporte correctement
- ✅ Vérifier que la vue est retournée correctement
- ✅ Tester sans démarrer tout le serveur (tests rapides)
- ✅ Documenter le comportement attendu

### Étape 9.2 : Créer une classe de test

**Procédure**

1. Clic droit sur `src/test/java/com/example/springbootmvc/controllers/` → `New → Class`
2. Nommer : `FirstControllerTest`

**Contenu**

```java
package com.example.springbootmvc.controllers;

import static org.hamcrest.Matchers.containsString;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.web.servlet.MockMvc;

@WebMvcTest(FirstController.class)
public class FirstControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    public void testHomeReturnsStatus200() throws Exception {
        mockMvc.perform(get("/"))
            .andExpect(status().isOk());
    }
    
    @Test
    public void testHomeContainsFramework() throws Exception {
        mockMvc.perform(get("/?framework=Spring"))
            .andExpect(status().isOk())
            .andExpect(content().string(containsString("Spring")));
    }
    
    @Test
    public void testHomeDefaultFramework() throws Exception {
        mockMvc.perform(get("/"))
            .andExpect(status().isOk())
            .andExpect(content().string(containsString("Spring Boot")));
    }
}
```

**Explication des annotations et méthodes**

| Élément | Rôle |
|---------|------|
| `@WebMvcTest(FirstController.class)` | Teste uniquement le contrôleur MVC, sans charger toute l'appli |
| `@Autowired private MockMvc mockMvc` | Objet qui simule des requêtes HTTP |
| `@Test` | Marque une méthode comme test |
| `mockMvc.perform(get("/"))` | Simule une requête HTTP GET sur `/` |
| `.andExpect(status().isOk())` | Vérifie que le statut HTTP est 200 (OK) |
| `.andExpect(content().string(containsString("...")))` | Vérifie que la réponse contient le texte donné |

### Étape 9.3 : Exécuter les tests

**Procédure (STS)**

1. Clic droit sur la classe `FirstControllerTest` → `Run As → JUnit Test`
2. Dans l'onglet "JUnit" en bas, vous devez voir :

```
✓ testHomeReturnsStatus200
✓ testHomeContainsFramework
✓ testHomeDefaultFramework

Tests passed: 3 / 3
```

✅ **Tous les tests passent !**

### Étape 9.4 : Créer un test qui échoue volontairement

**Ajouter cette méthode à FirstControllerTest**

```java
@Test
public void testHomeContainsSymfony() throws Exception {
    mockMvc.perform(get("/"))
        .andExpect(status().isOk())
        .andExpect(content().string(containsString("Symfony")));
}
```

**Exécuter le test**

- Ce test échoue car la page ne contient pas "Symfony"
- L'onglet JUnit affiche une croix rouge (✗)
- Le message d'erreur indique ce qui manquait

**Purpose** : montrer que les tests vérifient réellement le comportement.

### Questions de compréhension

1. Qu'est-ce que `MockMvc` ?
   - **Réponse** : C'est un objet qui simule des requêtes HTTP sans vraiment démarrer le serveur Tomcat. Cela rend les tests rapides.

2. Pourquoi tester au niveau du contrôleur et pas du navigateur ?
   - **Réponse** : Les tests au niveau du contrôleur sont plus rapides, plus fiables et plus faciles à automatiser. Les tests navigateur viennent après (tests d'intégration/e2e).

3. Que teste exactement `containsString("Spring")` ?
   - **Réponse** : Il vérifie que le texte "Spring" apparaît quelque part dans le contenu HTML retourné par le contrôleur.

---

## 🚀 Partie 10 : Déploiement (optionnel)

### Remarque

Le déploiement nécessite une plateforme de cloud. Ce TP se concentre sur le développement local. Pour le déploiement, consultez :

- Heroku : https://www.heroku.com (gratuit avec limitations)
- Azure App Service : https://azure.microsoft.com/services/app-service/
- Railway : https://railway.app

Les étapes générales sont similaires à celles documentées dans le PDF original.

---

## 📝 Résumé et points clés à retenir

| Concept | Explication |
|---------|-------------|
| **Spring Boot** | Framework qui simplifie la création d'applications web Java |
| **MVC** | Architecture Modèle-Vue-Contrôleur pour les applications web |
| **@Controller** | Annotation pour marquer une classe comme contrôleur |
| **@GetMapping** | Mappe une requête HTTP GET à une méthode |
| **@PostMapping** | Mappe une requête HTTP POST à une méthode |
| **Model** | Objet pour transmettre des données du contrôleur à la vue |
| **Thymeleaf** | Moteur de templates pour générer du HTML dynamique |
| **th:text** | Attribut Thymeleaf pour injecter du texte dynamique |
| **th:each** | Attribut Thymeleaf pour itérer sur une collection |
| **th:if** | Attribut Thymeleaf pour affichage conditionnel |
| **@RequestParam** | Annotation pour récupérer les paramètres de l'URL |
| **@ModelAttribute** | Annotation pour lier les données de formulaire à un objet |
| **DevTools** | Permet le rechargement automatique après changement de code |
| **MockMvc** | Outil pour tester les contrôleurs sans démarrer le serveur |

---

## 🎓 Travail à rendre

### ✅ Partie obligatoire (minimum)

1. ✅ Créer l'application jusqu'à l'Étape 8 (Application MVC complète Person)
2. ✅ Tester chaque action (index, list, add) dans le navigateur
3. ✅ Fournir un document montrant les captures d'écran de chaque page

### ✨ Partie optionnelle (bonus)

1. ✨ Implémenter les actions DELETE et EDIT (Exercice 8.1)
2. ✨ Ajouter des tests unitaires (Partie 9)
3. ✨ Déployer l'application sur une plateforme cloud (Heroku, Azure, etc.)
4. ✨ Améliorer le CSS pour une meilleure présentation
5. ✨ Ajouter un second contrôleur pour gérer une autre entité (ex : Article)

---

## 🤔 Questions fréquemment posées (FAQ)

### Q1 : Quelle est la différence entre `@ResponseBody` et retourner une vue ?

**Réponse**

- `@ResponseBody` : retourne directement le texte/JSON (pas de template). Utile pour les API REST.
- Retourner un nom de vue : Spring cherche le fichier template correspondant et le rend dynamique avec Thymeleaf.

**Exemple**

```java
// Option 1 : @ResponseBody
@GetMapping("/api/hello")
@ResponseBody
public String hello() {
    return "Hello"; // Texte brut au navigateur
}

// Option 2 : Vue Thymeleaf
@GetMapping("/hello")
public String hello(Model model) {
    return "hello"; // Spring cherche templates/hello.html
}
```

---

### Q2 : Comment passer plusieurs variables à une vue ?

**Réponse**

Utiliser `model.addAttribute()` plusieurs fois

```java
model.addAttribute("name", "Alice");
model.addAttribute("age", 25);
model.addAttribute("city", "Paris");
return "profile";
```

Puis dans la vue

```html
<p>Name: <span th:text="${name}"></span></p>
<p>Age: <span th:text="${age}"></span></p>
<p>City: <span th:text="${city}"></span></p>
```

---

### Q3 : Qu'est-ce qu'une "redirection" (redirect:) ?

**Réponse**

Une redirection envoie le navigateur vers une nouvelle URL via HTTP 302.

```java
@PostMapping("/form")
public String submitForm(@ModelAttribute PersonForm form) {
    // Traiter les données
    
    // Redirection vers la liste (au lieu de rendre une vue)
    return "redirect:/list";
}
```

Le navigateur reçoit une réponse "allez à `/list`" et exécute une nouvelle requête GET automatiquement.

**Utilité** : éviter que l'utilisateur ne rafraîchisse la page et ne resoumette le formulaire par accident.

---

### Q4 : Comment afficher une liste en HTML avec Thymeleaf ?

**Réponse**

Utiliser `th:each`

```java
// Dans le contrôleur
List<String> fruits = Arrays.asList("Pomme", "Banane", "Orange");
model.addAttribute("fruits", fruits);
```

```html
<!-- Dans la vue -->
<ul>
    <li th:each="fruit : ${fruits}" th:text="${fruit}"></li>
</ul>

<!-- Résultat -->
<!-- <li>Pomme</li> -->
<!-- <li>Banane</li> -->
<!-- <li>Orange</li> -->
```

---

### Q5 : Comment afficher une valeur conditionnellement avec Thymeleaf ?

**Réponse**

Utiliser `th:if` et/ou `th:unless`

```html
<!-- Afficher si errorMessage existe -->
<div th:if="${errorMessage != null}">
    <strong>Erreur :</strong> <span th:text="${errorMessage}"></span>
</div>

<!-- Afficher si isAdmin est true -->
<div th:if="${isAdmin == true}">
    Vous êtes administrateur
</div>

<!-- Afficher si user est vide -->
<div th:unless="${user != null}">
    Aucun utilisateur connecté
</div>
```

---

### Q6 : Qu'est-ce que `@Value` et comment l'utiliser ?

**Réponse**

`@Value` injecte des valeurs depuis `application.properties` dans un bean Spring.

**Dans application.properties**

```properties
app.title=Ma Super App
app.version=1.0.0
app.admin.email=admin@example.com
```

**Dans le contrôleur**

```java
@Controller
public class HomeController {
    
    @Value("${app.title}")
    private String title;
    
    @Value("${app.version}")
    private String version;
    
    @GetMapping("/")
    public String home(Model model) {
        model.addAttribute("title", title);
        model.addAttribute("version", version);
        return "home";
    }
}
```

**Dans la vue**

```html
<h1 th:text="${title}"></h1>
<p>Version: <span th:text="${version}"></span></p>
```

---

### Q7 : Quelle est la différence entre GET et POST ?

**Réponse**

| Aspect | GET | POST |
|--------|-----|------|
| **Usage** | Récupérer des données | Soumettre des données |
| **Paramètres** | Visibles dans l'URL (`?param=value`) | Dans le corps de la requête (invisible) |
| **Sécurité** | Moins sûr (params visibles) | Plus sûr (params cachés) |
| **Exemple** | `GET /search?q=spring` | `POST /form` avec données du formulaire |
| **Annotation** | `@GetMapping` | `@PostMapping` |

---

### Q8 : Comment déboguer une application Spring Boot ?

**Réponse**

1. **Console** : lire les messages d'erreur
2. **Logs** : ajouter des `System.out.println()` ou utiliser un logger
3. **Debugger** : placer des breakpoints et exécuter en mode debug

**Exemple avec logs**

```java
System.out.println("DEBUG: firstName = " + firstName);
System.out.println("DEBUG: Liste de personnes = " + persons);
```

---

### Q9 : Pourquoi DevTools redémarre automatiquement l'app ?

**Réponse**

DevTools surveille les fichiers du projet. Quand vous sauvegardez un fichier, DevTools redémarre le serveur Tomcat automatiquement. Cela améliore la productivité pendant le développement.

---

### Q10 : Comment accéder au projet depuis une autre machine en réseau local ?

**Réponse**

1. Trouver l'adresse IP de votre machine : `ipconfig` (Windows) ou `ifconfig` (Mac/Linux)
2. Exemple : `192.168.1.100`
3. Depuis une autre machine : `http://192.168.1.100:8080`

⚠️ **Attention** : cela marche seulement si les machines sont sur le même réseau local.

---

## 📚 Ressources complémentaires

- **Documentation officielle Spring Boot** : https://spring.io/projects/spring-boot
- **Documentation Thymeleaf** : https://www.thymeleaf.org/
- **Tutoriels Spring Boot** : https://spring.io/guides
- **Maven Repository** : https://mvnrepository.com/
- **Stack Overflow** : recherchez "spring boot" + votre question

---

## 🏁 Conclusion

Vous avez terminé ce TP complet sur **Spring Boot MVC** !

### Ce que vous avez appris

✅ Créer une application Spring Boot from scratch  
✅ Utiliser les annotations MVC (@Controller, @GetMapping, etc.)  
✅ Passer des données entre contrôleur et vue avec Model  
✅ Utiliser Thymeleaf pour générer du HTML dynamique  
✅ Gérer des formulaires HTML avec th:object et th:field  
✅ Intégrer CSS et ressources statiques  
✅ Construire une application MVC complète (CRUD simplifié)  
✅ Tester avec JUnit et MockMvc  

### Prochaines étapes possibles

🚀 Ajouter une **base de données** (JPA/Hibernate)  
🚀 Implémenter **Spring Security** (authentification/autorisation)  
🚀 Créer une **API REST** (JSON au lieu d'HTML)  
🚀 **Déployer** sur le cloud  
🚀 Ajouter des **tests d'intégration**  

---

**Bonne chance et happy coding !** 🎉
