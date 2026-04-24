# Lab 4 - Spring Boot MVC : Interface Web avec Thymeleaf


---

## Table des Matières

1. [Objectifs Pédagogiques](#objectifs-pédagogiques)
2. [Prérequis](#prérequis)
3. [Concepts Fondamentaux](#concepts-fondamentaux)
4. [Architecture MVC](#architecture-mvc)
5. [Partie 1 : Configurer Thymeleaf](#partie-1--configurer-thymeleaf)
6. [Partie 2 : Créer les Vues](#partie-2--créer-les-vues)
7. [Partie 3 : Contrôleur Web](#partie-3--contrôleur-web)
8. [Partie 4 : Formulaires](#partie-4--formulaires)
9. [Partie 5 : Validation Côté Serveur](#partie-5--validation-côté-serveur)
10. [Partie 6 : Affichage des Données](#partie-6--affichage-des-données)
11. [Partie 7 : CSS et Design](#partie-7--css-et-design)
12. [Partie 8 : Sécurité et Bonnes Pratiques](#partie-8--sécurité-et-bonnes-pratiques)
13. [Pièges Courants](#pièges-courants)
14. [Glossaire](#glossaire)
15. [Exercices d'Évaluation](#exercices-dévaluation)
16. [Évaluation Finale](#évaluation-finale)

---

## Objectifs Pédagogiques

À l'issue de ce lab, vous serez capable de :

✅ Configurer **Thymeleaf** comme moteur de template  
✅ Créer des **vues HTML** avec Thymeleaf  
✅ Utiliser les **contrôleurs web** (@Controller)  
✅ Passer des données au template avec le **Model**  
✅ Créer et traiter des **formulaires**  
✅ **Valider les données** côté serveur  
✅ Afficher les **erreurs de validation**  
✅ Appliquer du **CSS et du design** responsive  
✅ Implémenter la **pagination** et le **tri**  
✅ Gérer les **sessions utilisateur**  

---

## Prérequis

- ✅ Lab 1, 2 et 3 complétés
- ✅ Comprendre l'architecture MVC
- ✅ Maîtriser HTML et CSS basique
- ✅ Connaître Spring Web

---

## Concepts Fondamentaux

### ⭐ Thymeleaf

**Thymeleaf** = Moteur de template Java pour générer des vues HTML dynamiques

**Caractéristiques** :
- **Natural Templates** : Fichiers HTML valides
- **Spring Integration** : Accès direct aux objets Spring
- **Expression Language** : Syntaxe simple et claire
- **Sécurité** : Protection contre les injections

**Exemple** :
```html
<!-- Afficher une variable -->
<h1 th:text="${product.name}">Nom du produit</h1>

<!-- Boucle -->
<tr th:each="product : ${products}">
    <td th:text="${product.name}">iPhone</td>
</tr>

<!-- Condition -->
<div th:if="${!products.isEmpty()}">
    <p>Il y a des produits</p>
</div>
```

### 🌐 @Controller vs @RestController

| Aspect | @Controller | @RestController |
|--------|-------------|-----------------|
| **Retour** | Vue HTML | JSON/XML |
| **Template** | Thymeleaf | Non |
| **@ResponseBody** | Non, vue retournée | Oui, automatique |
| **Utilisation** | Interface Web | API |

### 📝 Model et ModelAndView

**Model** : Conteneur pour passer des données au template

```java
@GetMapping
public String list(Model model) {
    // Ajouter des données
    model.addAttribute("products", produitService.getAll());
    model.addAttribute("title", "Liste des produits");
    
    // Retourner le nom du template
    return "produits/list";  // Cherche templates/produits/list.html
}
```

### 🎯 Cycle de Requête Web

```
1. Utilisateur demande /produits
        ↓
2. Spring mappe à la méthode getList()
        ↓
3. Récupérer les données avec Service
        ↓
4. Ajouter au Model
        ↓
5. Thymeleaf génère le HTML
        ↓
6. Navigateur affiche la page
```

---

## Architecture MVC

```
┌─────────────────────────────────────────┐
│      Navigateur (Vue HTML)              │
│   ← Thymeleaf génère le HTML            │
└─────────────────────────────────────────┘
                    ↑
         HTTP Response (HTML)
                    ↓
┌─────────────────────────────────────────┐
│     @Controller (Contrôleur Web)        │
│  - Reçoit les requêtes                  │
│  - Appelle le Service                   │
│  - Ajoute au Model                      │
│  - Retourne la vue (nom du template)    │
└─────────────────────────────────────────┘
                    ↑
         HTTP Request (GET/POST)
                    ↓
┌─────────────────────────────────────────┐
│      Service (Logique Métier)           │
│  - Validation                           │
│  - Orchestration                        │
└─────────────────────────────────────────┘
                    ↑
         Appels Repository
                    ↓
┌─────────────────────────────────────────┐
│     Repository (Accès Données)          │
│  - Requêtes BD                          │
└─────────────────────────────────────────┘
```

---

## Partie 1 : Configurer Thymeleaf

### Étape 1.1 : Ajouter les Dépendances

Modifiez `pom.xml` :

```xml
<!-- Ajouter ces dépendances -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>

<!-- Optionnel : Bootstrap pour le design -->
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>bootstrap</artifactId>
    <version>5.3.0</version>
</dependency>
```

### Étape 1.2 : Configuration dans application.properties

Modifiez `application.properties` :

```properties
# Thymeleaf
spring.thymeleaf.mode=HTML
spring.thymeleaf.encoding=UTF-8
spring.thymeleaf.cache=false  # Désactiver le cache en développement

# Chemin des templates
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
```

### Étape 1.3 : Structure des Répertoires

Créez cette structure :

```
src/main/resources/
├── templates/
│   ├── layout.html              (Template de base)
│   ├── index.html               (Page d'accueil)
│   ├── produits/
│   │   ├── list.html            (Liste des produits)
│   │   ├── detail.html          (Détail d'un produit)
│   │   ├── create.html          (Formulaire création)
│   │   └── edit.html            (Formulaire modification)
│   └── errors/
│       └── 404.html             (Page d'erreur)
├── static/
│   ├── css/
│   │   └── style.css
│   ├── js/
│   │   └── script.js
│   └── images/
└── application.properties
```

---

## Partie 2 : Créer les Vues

### Étape 2.1 : Template de Base (Layout)

Créez `src/main/resources/templates/layout.html` :

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title th:text="${title}">Gestion des Produits</title>
    
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" 
          rel="stylesheet">
    
    <!-- CSS personnalisé -->
    <link href="#" th:href="@{/css/style.css}" rel="stylesheet">
</head>
<body>
    <!-- Navigation -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" th:href="@{/}">Gestion Produits</a>
            
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" 
                    data-bs-target="#navbarNav">
                <span class="navbar-toggler-icon"></span>
            </button>
            
            <div class="collapse navbar-collapse" id="navbarNav">
                <ul class="navbar-nav ms-auto">
                    <li class="nav-item">
                        <a class="nav-link" th:href="@{/produits}">Produits</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" th:href="@{/categories}">Catégories</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" th:href="@{/produits/create}">Nouveau</a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>
    
    <!-- Contenu principal -->
    <main class="container my-4">
        <div th:insert="~{this :: content}"></div>
    </main>
    
    <!-- Footer -->
    <footer class="bg-dark text-white text-center py-3 mt-5">
        <p>&copy; 2024 Gestion Produits - Tous droits réservés</p>
    </footer>
    
    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

### Étape 2.2 : Page d'Accueil

Créez `src/main/resources/templates/index.html` :

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Accueil</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" 
          rel="stylesheet">
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" th:href="@{/}">Gestion Produits</a>
            <div class="collapse navbar-collapse">
                <ul class="navbar-nav ms-auto">
                    <li class="nav-item">
                        <a class="nav-link" th:href="@{/produits}">Produits</a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>
    
    <main class="container my-4">
        <div class="row">
            <div class="col-md-12">
                <h1>Bienvenue dans la Gestion des Produits</h1>
                <p class="lead">Gérez facilement vos produits et catégories</p>
                
                <div class="row mt-4">
                    <div class="col-md-6">
                        <div class="card">
                            <div class="card-body">
                                <h5 class="card-title">Produits</h5>
                                <p class="card-text">Gérez votre catalogue de produits</p>
                                <a th:href="@{/produits}" class="btn btn-primary">
                                    Voir les produits
                                </a>
                            </div>
                        </div>
                    </div>
                    
                    <div class="col-md-6">
                        <div class="card">
                            <div class="card-body">
                                <h5 class="card-title">Catégories</h5>
                                <p class="card-text">Organisez vos produits par catégories</p>
                                <a th:href="@{/categories}" class="btn btn-primary">
                                    Voir les catégories
                                </a>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </main>
    
    <footer class="bg-dark text-white text-center py-3 mt-5">
        <p>&copy; 2024 Gestion Produits</p>
    </footer>
    
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

### Étape 2.3 : Liste des Produits

Créez `src/main/resources/templates/produits/list.html` :

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Liste des Produits</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" 
          rel="stylesheet">
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" th:href="@{/}">Gestion Produits</a>
            <div class="collapse navbar-collapse">
                <ul class="navbar-nav ms-auto">
                    <li class="nav-item">
                        <a class="nav-link" th:href="@{/produits}">Produits</a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>
    
    <main class="container my-4">
        <div class="row mb-4">
            <div class="col-md-8">
                <h1>Produits</h1>
            </div>
            <div class="col-md-4 text-end">
                <a th:href="@{/produits/create}" class="btn btn-success">
                    + Ajouter un produit
                </a>
            </div>
        </div>
        
        <!-- Barre de recherche -->
        <div class="row mb-4">
            <div class="col-md-12">
                <form th:action="@{/produits}" method="GET" class="row g-3">
                    <div class="col-md-6">
                        <input type="text" class="form-control" name="search" 
                               placeholder="Rechercher un produit..." 
                               th:value="${search}">
                    </div>
                    <div class="col-md-6">
                        <button type="submit" class="btn btn-primary">Rechercher</button>
                    </div>
                </form>
            </div>
        </div>
        
        <!-- Messages d'alerte -->
        <div th:if="${message}" class="alert alert-success alert-dismissible fade show">
            <span th:text="${message}"></span>
            <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
        </div>
        
        <div th:if="${error}" class="alert alert-danger alert-dismissible fade show">
            <span th:text="${error}"></span>
            <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
        </div>
        
        <!-- Tableau des produits -->
        <div class="table-responsive">
            <table class="table table-striped table-hover">
                <thead class="table-dark">
                    <tr>
                        <th>ID</th>
                        <th>Nom</th>
                        <th>Catégorie</th>
                        <th>Prix</th>
                        <th>Date</th>
                        <th>Actions</th>
                    </tr>
                </thead>
                <tbody>
                    <!-- Boucle sur les produits -->
                    <tr th:each="product : ${products}">
                        <td th:text="${product.idProduit}">1</td>
                        <td th:text="${product.nomProduit}">iPhone 15</td>
                        <td th:text="${product.categorie != null ? product.categorie.nomCat : 'N/A'}">
                            Électronique
                        </td>
                        <td th:text="${#numbers.formatDecimal(product.prixProduit, 1, 2)} + ' DT'">
                            999.00 DT
                        </td>
                        <td th:text="${#dates.format(product.dateCreation, 'dd/MM/yyyy')}">
                            05/03/2024
                        </td>
                        <td>
                            <a th:href="@{/produits/{id}(id=${product.idProduit})}" 
                               class="btn btn-sm btn-info">Voir</a>
                            <a th:href="@{/produits/{id}/edit(id=${product.idProduit})}" 
                               class="btn btn-sm btn-warning">Éditer</a>
                            <a th:href="@{/produits/{id}/delete(id=${product.idProduit})}" 
                               class="btn btn-sm btn-danger" 
                               onclick="return confirm('Êtes-vous sûr ?')">Supprimer</a>
                        </td>
                    </tr>
                    
                    <!-- Si pas de produits -->
                    <tr th:if="${#lists.isEmpty(products)}">
                        <td colspan="6" class="text-center text-muted">
                            Aucun produit trouvé
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </main>
    
    <footer class="bg-dark text-white text-center py-3 mt-5">
        <p>&copy; 2024 Gestion Produits</p>
    </footer>
    
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

---

## Partie 3 : Contrôleur Web

### Étape 3.1 : Créer le Contrôleur Web

Créez `src/main/java/tn/iset/produits/controllers/ProduitWebController.java` :

```java
package tn.iset.produits.controllers;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;
import tn.iset.produits.entities.Produit;
import tn.iset.produits.services.ProduitService;

import jakarta.validation.Valid;
import java.util.List;

/**
 * Contrôleur Web pour les produits
 * 
 * @Controller : Retourne une vue (HTML) via Thymeleaf
 * Différent de @RestController qui retourne JSON
 */
@Controller
@RequestMapping("/produits")
@Slf4j
@RequiredArgsConstructor
public class ProduitWebController {
    
    private final ProduitService produitService;
    
    // ========== AFFICHAGE DES LISTES ==========
    
    /**
     * GET /produits
     * Afficher la liste des produits
     */
    @GetMapping
    public String list(
            @RequestParam(required = false) String search,
            Model model) {
        
        log.info("GET /produits - search={}", search);
        
        List<Produit> produits;
        
        if (search != null && !search.isBlank()) {
            produits = produitService.findByNomProduitContains(search);
        } else {
            produits = produitService.getAllProduits();
        }
        
        model.addAttribute("products", produits);
        model.addAttribute("search", search);
        model.addAttribute("title", "Produits");
        
        return "produits/list";  // Cherche templates/produits/list.html
    }
    
    /**
     * GET /produits/{id}
     * Afficher le détail d'un produit
     */
    @GetMapping("/{id}")
    public String detail(@PathVariable Long id, Model model) {
        log.info("GET /produits/{} - Détail du produit", id);
        
        try {
            Produit produit = produitService.getProduitById(id);
            model.addAttribute("product", produit);
            return "produits/detail";
        } catch (RuntimeException e) {
            return "redirect:/produits";
        }
    }
    
    // ========== CRÉATION ==========
    
    /**
     * GET /produits/create
     * Afficher le formulaire de création
     */
    @GetMapping("/create")
    public String createForm(Model model) {
        log.info("GET /produits/create - Formulaire de création");
        
        model.addAttribute("product", new Produit());
        model.addAttribute("title", "Créer un produit");
        
        return "produits/create";
    }
    
    /**
     * POST /produits
     * Traiter la création d'un produit
     */
    @PostMapping
    public String create(
            @Valid @ModelAttribute("product") Produit produit,
            Model model,
            RedirectAttributes redirectAttributes) {
        
        log.info("POST /produits - Créer le produit : {}", produit.getNomProduit());
        
        try {
            Produit created = produitService.saveProduit(produit);
            redirectAttributes.addFlashAttribute("message", 
                "Produit créé avec succès !");
            return "redirect:/produits";
        } catch (IllegalArgumentException e) {
            model.addAttribute("error", e.getMessage());
            return "produits/create";
        }
    }
    
    // ========== MODIFICATION ==========
    
    /**
     * GET /produits/{id}/edit
     * Afficher le formulaire d'édition
     */
    @GetMapping("/{id}/edit")
    public String editForm(@PathVariable Long id, Model model) {
        log.info("GET /produits/{}/edit - Formulaire d'édition", id);
        
        try {
            Produit produit = produitService.getProduitById(id);
            model.addAttribute("product", produit);
            model.addAttribute("title", "Éditer le produit");
            return "produits/edit";
        } catch (RuntimeException e) {
            return "redirect:/produits";
        }
    }
    
    /**
     * PUT /produits/{id}
     * Traiter la modification
     */
    @PostMapping("/{id}")
    public String update(
            @PathVariable Long id,
            @Valid @ModelAttribute("product") Produit produit,
            Model model,
            RedirectAttributes redirectAttributes) {
        
        log.info("POST /produits/{} - Modifier le produit", id);
        
        try {
            produitService.updateProduit(id, produit);
            redirectAttributes.addFlashAttribute("message", 
                "Produit modifié avec succès !");
            return "redirect:/produits";
        } catch (RuntimeException e) {
            model.addAttribute("error", "Produit non trouvé");
            return "produits/edit";
        }
    }
    
    // ========== SUPPRESSION ==========
    
    /**
     * GET /produits/{id}/delete
     * Supprimer un produit
     */
    @GetMapping("/{id}/delete")
    public String delete(
            @PathVariable Long id,
            RedirectAttributes redirectAttributes) {
        
        log.info("GET /produits/{}/delete - Supprimer", id);
        
        try {
            produitService.deleteProduit(id);
            redirectAttributes.addFlashAttribute("message", 
                "Produit supprimé avec succès !");
        } catch (RuntimeException e) {
            redirectAttributes.addFlashAttribute("error", 
                "Erreur lors de la suppression");
        }
        
        return "redirect:/produits";
    }
    
    // ========== GESTION DES ERREURS ==========
    
    @ExceptionHandler(Exception.class)
    public String handleException(Exception e, Model model) {
        log.error("Erreur : ", e);
        model.addAttribute("error", "Une erreur inattendue s'est produite");
        return "error";
    }
}
```

---

## Partie 4 : Formulaires

### Étape 4.1 : Formulaire de Création

Créez `src/main/resources/templates/produits/create.html` :

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Créer un produit</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" 
          rel="stylesheet">
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" th:href="@{/}">Gestion Produits</a>
        </div>
    </nav>
    
    <main class="container my-4">
        <div class="row">
            <div class="col-md-8 offset-md-2">
                <h1>Créer un produit</h1>
                
                <!-- Messages d'erreur -->
                <div th:if="${error}" class="alert alert-danger">
                    <span th:text="${error}"></span>
                </div>
                
                <!-- Formulaire -->
                <form th:action="@{/produits}" th:object="${product}" 
                      method="POST" class="mt-4">
                    
                    <div class="mb-3">
                        <label for="nomProduit" class="form-label">Nom du produit *</label>
                        <input type="text" class="form-control" id="nomProduit" 
                               th:field="*{nomProduit}" required>
                        <!-- Afficher les erreurs de validation -->
                        <small class="text-danger" 
                               th:if="${#fields.hasErrors('nomProduit')}" 
                               th:errors="*{nomProduit}">Erreur nom</small>
                    </div>
                    
                    <div class="mb-3">
                        <label for="prixProduit" class="form-label">Prix *</label>
                        <input type="number" class="form-control" id="prixProduit" 
                               th:field="*{prixProduit}" step="0.01" required>
                        <small class="text-danger" 
                               th:if="${#fields.hasErrors('prixProduit')}" 
                               th:errors="*{prixProduit}">Erreur prix</small>
                    </div>
                    
                    <div class="mb-3">
                        <button type="submit" class="btn btn-primary">Créer</button>
                        <a th:href="@{/produits}" class="btn btn-secondary">Annuler</a>
                    </div>
                </form>
            </div>
        </div>
    </main>
    
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

---

## Partie 5 : Validation Côté Serveur

### Étape 5.1 : Ajouter les Validations

Les validations se font via :
- **Annotations JPA** (@NotBlank, @Size, @Positive)
- **@Valid** dans le contrôleur
- **BindingResult** pour les erreurs

```java
@PostMapping
public String create(
        @Valid @ModelAttribute("product") Produit produit,
        BindingResult bindingResult,  // Contient les erreurs
        Model model) {
    
    // Vérifier s'il y a des erreurs
    if (bindingResult.hasErrors()) {
        return "produits/create";  // Retourner au formulaire
    }
    
    // Si pas d'erreurs, créer le produit
    produitService.saveProduit(produit);
    return "redirect:/produits";
}
```

### Étape 5.2 : Afficher les Erreurs dans le Template

```html
<!-- Pour chaque champ -->
<div class="mb-3" th:classappend="${#fields.hasErrors('nomProduit') ? 'has-error' : ''}">
    <label for="nomProduit">Nom *</label>
    <input type="text" th:field="*{nomProduit}" class="form-control">
    
    <!-- Afficher les erreurs -->
    <div th:if="${#fields.hasErrors('nomProduit')}" class="alert alert-danger mt-2">
        <span th:each="error : ${#fields.errors('nomProduit')}" 
              th:text="${error}">Erreur</span>
    </div>
</div>
```

---

## Partie 6 : Affichage des Données

### Étape 6.1 : Expressions Thymeleaf

**Afficher une variable** :
```html
<h1 th:text="${title}">Titre par défaut</h1>
```

**Formatage** :
```html
<!-- Nombre décimal -->
<p th:text="${#numbers.formatDecimal(price, 1, 2)}">999.00</p>

<!-- Date -->
<p th:text="${#dates.format(date, 'dd/MM/yyyy')}">05/03/2024</p>

<!-- Texte majuscule/minuscule -->
<p th:text="${#strings.toUpperCase(name)}">IPHONE</p>
```

**Conditions** :
```html
<!-- If/Else -->
<div th:if="${products.isEmpty()}">
    <p>Aucun produit</p>
</div>
<div th:unless="${products.isEmpty()}">
    <p>Il y a des produits</p>
</div>

<!-- Switch -->
<div th:switch="${category}">
    <span th:case="'Électronique'">Électronique</span>
    <span th:case="'Vêtements'">Vêtements</span>
    <span th:case="*">Autre</span>
</div>
```

**Boucles** :
```html
<ul>
    <li th:each="product : ${products}" 
        th:text="${product.name} + ' - ' + ${product.price}">
        iPhone
    </li>
</ul>

<!-- Avec index -->
<tr th:each="product, iter : ${products}">
    <td th:text="${iter.index + 1}">1</td>
    <td th:text="${iter.count}">1</td>
    <td th:text="${iter.size}">10</td>
</tr>
```

---

## Partie 7 : CSS et Design

### Étape 7.1 : Fichier CSS Personnalisé

Créez `src/main/resources/static/css/style.css` :

```css
/* Couleurs personnalisées */
:root {
    --primary-color: #007bff;
    --success-color: #28a745;
    --danger-color: #dc3545;
    --light-color: #f8f9fa;
}

/* Body */
body {
    font-family: 'Segoe UI', sans-serif;
    background-color: #f5f5f5;
}

/* Navbar */
.navbar {
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

/* Conteneur principal */
main {
    min-height: calc(100vh - 200px);
}

/* Tables */
table {
    background-color: white;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

table thead {
    background-color: #343a40;
}

/* Boutons */
.btn {
    border-radius: 4px;
    font-weight: 500;
}

.btn-sm {
    padding: 0.25rem 0.5rem;
    font-size: 0.875rem;
}

/* Formulaires */
.form-control {
    border: 1px solid #ddd;
    border-radius: 4px;
}

.form-control:focus {
    border-color: var(--primary-color);
    box-shadow: 0 0 0 0.2rem rgba(0,123,255,0.25);
}

/* Cards */
.card {
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    border: none;
    border-radius: 8px;
}

/* Footer */
footer {
    margin-top: auto;
    border-top: 1px solid #ddd;
}

/* Responsive */
@media (max-width: 768px) {
    .table-responsive {
        font-size: 0.875rem;
    }
    
    .btn-sm {
        padding: 0.2rem 0.4rem;
        font-size: 0.75rem;
    }
}
```

---

## Partie 8 : Sécurité et Bonnes Pratiques

### Étape 8.1 : Prévention XSS

**❌ Dangereux** :
```html
<!-- Si data contient du JavaScript malveillant -->
<div th:utext="${data}"></div>  <!-- Exécute le JS -->
```

**✅ Sûr** :
```html
<!-- Échappe le contenu -->
<div th:text="${data}"></div>  <!-- Affiche en texte -->
```

### Étape 8.2 : CSRF Protection

Ajoutez le token CSRF dans les formulaires :

```html
<form method="POST" th:action="@{/produits}">
    <!-- Token CSRF automatique avec Thymeleaf -->
    <input type="hidden" th:name="${_csrf.parameterName}" 
           th:value="${_csrf.token}">
    
    <!-- Champs du formulaire -->
</form>
```

### Étape 8.3 : Bonnes Pratiques

**1. Utiliser @ModelAttribute au lieu de @RequestBody pour les vues** :
```java
// ✅ Correct pour les vues
public String save(@ModelAttribute Produit produit) {}

// ❌ Pour les vues, utilisez @ModelAttribute
public String save(@RequestBody Produit produit) {}
```

**2. Redirect after POST** :
```java
// ✅ Correct
@PostMapping
public String save(Produit produit) {
    service.save(produit);
    return "redirect:/produits";  // Évite la double soumission
}
```

**3. Utiliser les classes CSS Thymeleaf** :
```html
<!-- Ajouter/retirer des classes selon une condition -->
<div th:classappend="${isActive ? 'active' : ''}">
    Contenu
</div>
```

---

## Pièges Courants

### ⚠️ Piège 1️⃣ : Chemin des templates

**❌ Erreur** :
```
Resource not found at path /template/list.html
```

**✅ Solution** :
- Templates dans `src/main/resources/templates/`
- Retourner `"produits/list"` (sans .html)

---

### ⚠️ Piège 2️⃣ : th:field dans les formulaires

**❌ Erreur** :
```html
<input type="text" name="nomProduit" value="${product.nomProduit}">
```

**✅ Correction** :
```html
<input type="text" th:field="*{nomProduit}">
```

---

### ⚠️ Piège 3️⃣ : Valeurs nulles

**❌ Erreur** :
```html
<p th:text="${product.categorie.nomCat}"></p>
<!-- NullPointerException si categorie est null -->
```

**✅ Correction** :
```html
<p th:text="${product.categorie != null ? product.categorie.nomCat : 'N/A'}"></p>
```

---

## Glossaire

**Thymeleaf** : Moteur de template Java pour générer du HTML dynamique

**@Controller** : Retourne une vue (HTML) au lieu de JSON

**Model** : Conteneur pour passer les données au template

**th:field** : Liaison bidirectionnelle objet-formulaire

**th:if/th:unless** : Conditions dans le template

**th:each** : Boucle sur une collection

**@Valid** : Valider les données reçues

**BindingResult** : Contient les erreurs de validation

**RedirectAttributes** : Passer des données lors d'une redirection

---


**Lab 4 terminé ! Votre application web est maintenant opérationnelle ! 🎉**

Prochaines étapes :
- Lab 5 : Authentification et sécurité
- Lab 6 : API avancée et documentation
- Lab 7 : Déploiement en production

