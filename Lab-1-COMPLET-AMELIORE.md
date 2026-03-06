# Lab 1 - Architecture Spring Boot MVC : Couches Modèle, Persistance et Service


**Framework** : Spring Boot 3.2+  
**IDE** : IntelliJ IDEA Ultimate/Community  
**Durée estimée** : 6-8 heures  
**Niveau** : Intermédiaire  
**Version** : 2.0 (Améliorée)

---

## Table des Matières

1. [Objectifs Pédagogiques](#objectifs-pédagogiques)
2. [Prérequis](#prérequis)
3. [Concepts Fondamentaux](#concepts-fondamentaux)
4. [Architecture du Projet](#architecture-du-projet)
5. [Structure du Projet](#structure-du-projet)
6. [Partie 1 : Création du Projet](#partie-1--création-du-projet)
7. [Partie 2 : Entité JPA](#partie-2--entité-jpa)
8. [Partie 3 : Repository](#partie-3--repository)
9. [Partie 4 : Service](#partie-4--service)
10. [Partie 5 : Configuration BD](#partie-5--configuration-bd)
11. [Partie 6 : Tests Unitaires](#partie-6--tests-unitaires)
12. [Partie 7 : Initialisation Données](#partie-7--initialisation-données)
13. [Pièges Courants](#pièges-courants)
14. [Glossaire](#glossaire)
15. [Exercices d'Évaluation](#exercices-dévaluation)
16. [Évaluation Finale](#évaluation-finale)
17. [Ressources Complémentaires](#ressources-complémentaires)

---

## Objectifs Pédagogiques

À l'issue de ce lab, vous serez capable de :

✅ Créer et configurer un projet Spring Boot avec IntelliJ IDEA  
✅ Modéliser une entité JPA avec annotations et validations  
✅ Implémenter un repository Spring Data avec requêtes personnalisées  
✅ Concevoir une couche service avec interface et logique métier  
✅ Tester les opérations CRUD avec JUnit 5 (cas normal, erreur, limites)  
✅ Configurer et connecter une base de données MySQL  
✅ Appliquer les bonnes pratiques d'architecture en couches  
✅ Comprendre les patterns et principes SOLID  

---

## Prérequis

- **Java 17 ou supérieur** : Vérifier avec `java -version`
- **IntelliJ IDEA 2023.3+** : Community ou Ultimate
- **MySQL 8.0+ ou MariaDB 10.6+** : Ou Docker pour MySQL
- **Maven 3.8+** : Vérifier avec `mvn -version`
- **Connaissance de base** :
  - POO Java (héritage, polymorphisme)
  - SQL basique (SELECT, INSERT, UPDATE, DELETE)
  - Annotations Java (@interface)

**Vérification des prérequis** :
```bash
# Vérifier Java
java -version
# Devrait afficher : java version "17.x.x"

# Vérifier Maven
mvn -version
# Devrait afficher : Apache Maven 3.8.x

# Vérifier MySQL (si installé localement)
mysql -u root -p -e "SELECT VERSION();"
# OU lancer Docker
docker run -d --name mysql-test -e MYSQL_ROOT_PASSWORD=root mysql:8.0
docker ps
```

---

## Concepts Fondamentaux

### ⭐ Qu'est-ce que l'Architecture MVC en Couches ?

L'architecture MVC en couches sépare l'application en 4 niveaux logiques :

#### 1. Couche Présentation (Controllers - Lab 2)
- Reçoit les requêtes HTTP de l'utilisateur
- Appelle les services métier
- Retourne les réponses (JSON, HTML, XML)
- **Responsabilité** : Orchestrer les demandes utilisateur

#### 2. Couche Service (Business Logic)
- Contient la **logique métier** de l'application
- Valide les données reçues
- Orchestre les opérations complexes
- Encapsule les règles métier
- **Responsabilité** : Implémenter les règles de gestion

#### 3. Couche Persistance (Repositories)
- Accède à la base de données
- Exécute les requêtes SQL/JPQL
- Retourne les entités
- **Responsabilité** : Abstraire l'accès aux données

#### 4. Couche Modèle (Entities)
- Représente les données métier
- Mappe les objets Java aux tables BD
- Contient les validations de données
- **Responsabilité** : Modéliser les données

```
┌─────────────────────────────────────────────────┐
│          UTILISATEUR (Application Web)          │
└────────────────────┬────────────────────────────┘
                     │ Requête HTTP
                     ▼
┌─────────────────────────────────────────────────┐
│       COUCHE PRÉSENTATION (Controllers)         │
│         @RestController / @Controller           │
└────────────────────┬────────────────────────────┘
                     │ Appel Service
                     ▼
┌─────────────────────────────────────────────────┐
│        COUCHE SERVICE (Business Logic)          │
│           @Service + Interface                  │
│    (Validation, Orchestration, Logique)         │
└────────────────────┬────────────────────────────┘
                     │ Appel Repository
                     ▼
┌─────────────────────────────────────────────────┐
│     COUCHE PERSISTANCE (Repositories)           │
│       extends JpaRepository<T, ID>              │
│        (CRUD + Requêtes personnalisées)         │
└────────────────────┬────────────────────────────┘
                     │ Requête SQL/JPQL
                     ▼
┌─────────────────────────────────────────────────┐
│     COUCHE MODÈLE (Entities - JPA)              │
│          @Entity @Table @Column                 │
└────────────────────┬────────────────────────────┘
                     │ ORM Mapping
                     ▼
┌─────────────────────────────────────────────────┐
│        BASE DE DONNÉES MySQL                    │
│           Tables / Colonnes / Données           │
└─────────────────────────────────────────────────┘
```

#### Principes Appliqués

| Principe | Détail | Bénéfice |
|----------|--------|----------|
| **Séparation des Préoccupations** | Chaque couche = 1 responsabilité | Maintenance facile |
| **Inversion de Dépendance** | Couches dépendent d'abstractions (interfaces) | Testabilité |
| **Injection de Dépendances** | Spring gère le cycle de vie des beans | Flexibilité |
| **Single Responsibility** | Une classe = une raison de changer | Évolutivité |

#### Exemple : Créer un Produit

```
1. L'utilisateur clique sur "Créer Produit"
                    ↓
2. Controller reçoit la requête POST
   - Valide le format JSON
   - Appelle produitService.saveProduit()
                    ↓
3. Service valide les données
   - Vérifie que le nom n'est pas vide
   - Vérifie que le prix est > 0
   - Peut faire des appels BD pour vérifier l'unicité
                    ↓
4. Service appelle produitRepository.save()
                    ↓
5. Repository insère la ligne en BD
   - Exécute : INSERT INTO produit (nom, prix) VALUES (...)
                    ↓
6. BD retourne l'ID généré
                    ↓
7. Service retourne le produit créé au Controller
                    ↓
8. Controller retourne la réponse HTTP 201 Created
                    ↓
9. L'utilisateur reçoit la confirmation
```

### 🪄 Qu'est-ce que JPA ?

**JPA** = Java Persistence API = Standard Java pour l'ORM (Object-Relational Mapping)

**Rôle** : Traduire automatiquement entre objets Java et tables BD

```
Classe Java (Objet)        JPA         Table SQL (Données)
                         (Mapping)
public class Produit  ←→  @Entity  ←→  table produit
  private Long id        @Id            id BIGINT PRIMARY KEY
  private String nom     @Column        nom VARCHAR(100)
  private Double prix    @Column        prix DECIMAL(10,2)
```

**Sans JPA** (Ancien style JDBC) :
```java
// ❌ Beaucoup de code boilerplate
Connection conn = DriverManager.getConnection("jdbc:mysql://...");
Statement stmt = conn.createStatement();
stmt.execute("INSERT INTO produit (nom, prix) VALUES ('PC', 1000)");
stmt.close();
conn.close();
```

**Avec JPA** (Moderne) :
```java
// ✅ Simple, maintenable, orienté objet
Produit p = new Produit("PC", 1000);
produitRepository.save(p);
```

**Spring Data crée automatiquement les implémentations** 🪄

### 📚 Qu'est-ce qu'un Repository ?

Un **Repository** est une **interface qui fournit des méthodes pour accéder à la BD**.

Il encapsule la logique d'accès aux données et offre une abstraction.

```java
// Repository = Interface qui hérite de JpaRepository
public interface ProduitRepository extends JpaRepository<Produit, Long> {
    // Spring crée automatiquement l'implémentation !
}

// Utilisation simple
List<Produit> tous = produitRepository.findAll();
Optional<Produit> un = produitRepository.findById(1L);
produitRepository.save(produit);
produitRepository.delete(produit);
```

**Spring Data génère le SQL automatiquement** ✨

### 🎯 Qu'est-ce qu'une Interface Service ?

Une **interface Service** force un **contrat** entre le Repository et les Controllers.

```java
// L'interface définit LE QUOI (contrat)
public interface ProduitService {
    Produit saveProduit(Produit produit);
    Produit getProduitById(Long id);
    List<Produit> getAllProduits();
}

// L'implémentation définit LE COMMENT (détails)
@Service
public class ProduitServiceImpl implements ProduitService {
    
    @Autowired
    private ProduitRepository repository;
    
    @Override
    @Transactional
    public Produit saveProduit(Produit produit) {
        // Validation
        if (produit == null || produit.getNomProduit().isBlank()) {
            throw new IllegalArgumentException("Nom obligatoire");
        }
        
        // Sauvegarde
        return repository.save(produit);
    }
    
    @Override
    public Produit getProduitById(Long id) {
        return repository.findById(id)
            .orElseThrow(() -> new RuntimeException("Produit non trouvé"));
    }
    
    // Autres implémentations...
}
```

**Avantages** :
✅ Découplage (on peut changer l'implémentation)  
✅ Testabilité (on peut mocker l'interface)  
✅ Contrat clair (API définie)  
✅ Maintenabilité (responsabilités bien séparées)  

---

## Architecture du Projet

```
┌──────────────────────────────────────────────────────────┐
│                    ARCHITECTURE MVC                      │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────────────────────────────────────────┐    │
│  │              COUCHE PRÉSENTATION               │    │
│  │           (Web Controllers - Lab 2)            │    │
│  └─────────────────────┬──────────────────────────┘    │
│                        │                                 │
│                        ▼                                 │
│  ┌────────────────────────────────────────────────┐    │
│  │              COUCHE SERVICE                    │    │
│  │  ┌──────────────────────────────────────────┐ │    │
│  │  │  ProduitService (Interface)              │ │    │
│  │  │  - saveProduit(Produit): Produit         │ │    │
│  │  │  - getProduitById(Long): Produit         │ │    │
│  │  │  - getAllProduits(): List<Produit>       │ │    │
│  │  └──────────────────────────────────────────┘ │    │
│  │  ┌──────────────────────────────────────────┐ │    │
│  │  │  ProduitServiceImpl (Implémentation)      │ │    │
│  │  │  - Logique métier                        │ │    │
│  │  │  - Validation des données                │ │    │
│  │  │  - Orchestration des opérations          │ │    │
│  │  └──────────────────────────────────────────┘ │    │
│  └─────────────────────┬──────────────────────────┘    │
│                        │                                 │
│                        ▼                                 │
│  ┌────────────────────────────────────────────────┐    │
│  │           COUCHE PERSISTANCE                   │    │
│  │  ┌──────────────────────────────────────────┐ │    │
│  │  │  ProduitRepository (Spring Data JPA)     │ │    │
│  │  │  - CRUD automatique (héritée)            │ │    │
│  │  │  - Requêtes personnalisées               │ │    │
│  │  │  - findByPrixProduitBetween()            │ │    │
│  │  │  - findByNomProduitContaining()          │ │    │
│  │  └──────────────────────────────────────────┘ │    │
│  └─────────────────────┬──────────────────────────┘    │
│                        │                                 │
│                        ▼                                 │
│  ┌────────────────────────────────────────────────┐    │
│  │              COUCHE MODÈLE                     │    │
│  │  ┌──────────────────────────────────────────┐ │    │
│  │  │  Produit (Entité JPA)                    │ │    │
│  │  │  @Entity @Table @Column                  │ │    │
│  │  │  - Attributs persistés                   │ │    │
│  │  │  - Contraintes de validation             │ │    │
│  │  │  - Hooks du cycle de vie                 │ │    │
│  │  └──────────────────────────────────────────┘ │    │
│  └─────────────────────┬──────────────────────────┘    │
│                        │                                 │
│                        ▼                                 │
│  ┌────────────────────────────────────────────────┐    │
│  │           BASE DE DONNÉES MySQL                │    │
│  │              Table: produit                    │    │
│  │   id (PK) | nom | prix | dateCreation        │    │
│  └────────────────────────────────────────────────┘    │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Structure du Projet

```
produits-management/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── tn/
│   │   │       └── iset/
│   │   │           └── produits/
│   │   │               ├── ProduitsApplication.java
│   │   │               ├── entities/
│   │   │               │   └── Produit.java
│   │   │               ├── repositories/
│   │   │               │   └── ProduitRepository.java
│   │   │               └── services/
│   │   │                   ├── ProduitService.java        (Interface)
│   │   │                   └── ProduitServiceImpl.java     (Implémentation)
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── application-dev.properties
│   │       └── init.sql                                  (Nouveau!)
│   └── test/
│       └── java/
│           └── tn/iset/produits/
│               ├── ProduitsApplicationTests.java
│               └── services/
│                   └── ProduitServiceTest.java
└── pom.xml
```

---

## Partie 1 : Création du Projet

### Étape 1.1 : Initialiser le Projet Spring Boot

#### Méthode 1 : Via Spring Initializr dans IntelliJ

1. Ouvrez **IntelliJ IDEA**
2. **File** → **New** → **Project**
3. Sélectionnez **Spring Initializr** dans le menu gauche
4. Configurez les paramètres du projet :

```
Name                : produits-management
Location            : /chemin/vers/workspace
Language            : Java
Type                : Maven
Group               : tn.iset
Artifact            : produits
Package name        : tn.iset.produits
JDK                 : 17 ou supérieur
Java                : 17
Packaging           : Jar
```

5. Cliquez sur **Next**, puis sélectionnez les dépendances :

```
✅ Spring Web                    (Pour les controllers)
✅ Spring Data JPA              (Pour ORM + Repository)
✅ MySQL Driver                 (Pour connexion MySQL)
✅ Lombok                       (Pour réduire boilerplate)
✅ Spring Boot DevTools         (Pour rechargement auto)
✅ Spring Boot Test             (Pour les tests)
```

6. Cliquez sur **Create**

IntelliJ génère la structure du projet et télécharge les dépendances.

### Étape 1.2 : Vérifier le Fichier pom.xml

IntelliJ a généré le fichier `pom.xml`. Vérifiez qu'il contient :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.3</version>
        <relativePath/>
    </parent>
    
    <groupId>tn.iset</groupId>
    <artifactId>produits</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>produits-management</name>
    <description>Application de gestion des produits</description>
    
    <properties>
        <java.version>17</java.version>
    </properties>
    
    <dependencies>
        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!-- Base de données -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Outils de développement -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        
        <!-- Tests -->
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
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### ✅ Checkpoint #1 : Avant de Continuer...

Vérifiez que vous avez :

- [ ] Créé un projet Maven avec Spring Initializr
- [ ] Sélectionné les bonnes dépendances (JPA, Web, MySQL, Lombok)
- [ ] Le projet compile sans erreurs
- [ ] Vous voyez l'arborescence du projet dans IntelliJ
- [ ] Le pom.xml ressemble à celui ci-dessus

Si une case n'est pas cochée, revérifiez les étapes 1.1-1.2.

---

## Partie 2 : Entité JPA

### Étape 2.1 : Comprendre les Annotations JPA

Avant d'écrire le code, comprenons chaque annotation JPA :

#### @Entity - Marquer une classe comme entité persistée

```java
@Entity  // ← JPA va créer une table pour cette classe
public class Produit {
    // ...
}
```

**Ce que JPA fait** :
- Crée une table `produit` (par défaut = nom classe en minuscules)
- Mappe chaque champ à une colonne
- Permet la sérialisation/désérialisation

#### @Table - Personnaliser le nom de la table

```java
@Entity
@Table(name = "product")  // ← Le nom custom de la table en BD
public class Produit {
    // ... 
}
```

**Utilité** : Si vous voulez que le nom Java ≠ nom BD

#### @Id - Désigner la clé primaire

```java
@Entity
public class Produit {
    @Id
    private Long idProduit;  // ← C'est la clé primaire
    
    // Autres champs
}
```

**Règles** :
- Chaque entité DOIT avoir une @Id
- L'@Id doit être unique
- Généralement de type Long ou String

#### @GeneratedValue - Auto-incrémentation de la clé

```java
@Entity
public class Produit {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long idProduit;  // ← 1, 2, 3, 4... (auto-généré)
}
```

**Stratégies disponibles** :
- `IDENTITY` : Laisse la BD générer (MySQL auto-increment)
- `SEQUENCE` : Utilise une séquence (PostgreSQL préfère ça)
- `UUID` : Identifiant unique global (pour distribution)
- `AUTO` : JPA choisit la meilleure stratégie

#### @Column - Personnaliser une colonne

```java
@Entity
public class Produit {
    @Column(name = "product_name", length = 100, nullable = false)
    private String nomProduit;
    
    // Par défaut = nom champ en minuscules
    // length = taille max en BD
    // nullable = null autorisé ou pas
}
```

#### @Temporal - Pour les dates

```java
@Entity
public class Produit {
    @Temporal(TemporalType.DATE)  // Stocke uniquement la date (YYYY-MM-DD)
    private LocalDate dateCreation;
    
    @Temporal(TemporalType.TIMESTAMP)  // Date + heure complète
    private LocalDateTime lastModified;
}
```

#### @PrePersist et @PostPersist - Hooks du Cycle de Vie

```java
@Entity
public class Produit {
    @Temporal(TemporalType.TIMESTAMP)
    private LocalDateTime createdAt;
    
    @PrePersist  // ← Exécuté juste AVANT l'insertion en BD
    public void beforeSave() {
        this.createdAt = LocalDateTime.now();
        System.out.println("ℹ️ Avant d'insérer...");
    }
    
    @PostPersist  // ← Exécuté juste APRÈS l'insertion
    public void afterSave() {
        System.out.println("✅ Produit sauvegardé avec succès !");
    }
}
```

**Cycle de Vie JPA** :
```
Création        @PrePersist      Sauvegarde       @PostPersist
  de l'objet   →      ↓      →    en BD        →   (logs, audit)
```

#### Résumé des Annotations

| Annotation | Rôle | Exemple |
|-----------|------|---------|
| `@Entity` | Marquer comme entité | `@Entity public class Produit` |
| `@Table` | Nom custom table | `@Table(name="product")` |
| `@Id` | Clé primaire | `@Id private Long id` |
| `@GeneratedValue` | Auto-générer | `@GeneratedValue(strategy=IDENTITY)` |
| `@Column` | Personnaliser colonne | `@Column(length=100)` |
| `@Temporal` | Type de date | `@Temporal(TemporalType.DATE)` |
| `@PrePersist` | Avant insert | Initialiser timestamps |
| `@PostPersist` | Après insert | Logging, notifications |

### Étape 2.2 : Créer la Classe Produit

Créez le fichier `src/main/java/tn/iset/produits/entities/Produit.java` :

```java
package tn.iset.produits.entities;

import jakarta.persistence.*;
import jakarta.validation.constraints.*;
import lombok.*;

import java.time.LocalDate;

/**
 * Entité Produit mappée à la table 'produit'
 * 
 * JPA gère automatiquement :
 * - La création de la table
 * - L'insertion/mise à jour/suppression des lignes
 * - La conversion objet ↔ données BD
 */
@Entity
@Table(name = "produit")
@Data                           // Lombok : génère getter/setter/equals/hashCode
@NoArgsConstructor              // Lombok : génère constructeur sans paramètres
@AllArgsConstructor             // Lombok : génère constructeur avec tous les paramètres
@Builder                        // Lombok : génère le Builder pattern
public class Produit {
    
    // ========== ATTRIBUTS ==========
    
    /**
     * Clé primaire - Auto-incrémentée par la BD
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id_produit")
    private Long idProduit;
    
    /**
     * Nom du produit
     * Validations :
     * - @NotBlank : Ne peut pas être null ou vide
     * - @Size : Entre 3 et 100 caractères
     */
    @NotBlank(message = "Le nom du produit est obligatoire")
    @Size(min = 3, max = 100, message = "Le nom doit contenir entre 3 et 100 caractères")
    @Column(name = "nom_produit", length = 100, nullable = false)
    private String nomProduit;
    
    /**
     * Prix du produit
     * Validations :
     * - @NotNull : Doit être défini
     * - @Positive : Doit être > 0
     * - @DecimalMin : Au minimum 0.01
     */
    @NotNull(message = "Le prix est obligatoire")
    @Positive(message = "Le prix doit être positif")
    @DecimalMin(value = "0.01", message = "Le prix minimum est 0.01 DT")
    @Column(name = "prix_produit", nullable = false)
    private Double prixProduit;
    
    /**
     * Date de création du produit
     * Définie automatiquement par @PrePersist
     */
    @Temporal(TemporalType.DATE)
    @Column(name = "date_creation", nullable = false, updatable = false)
    private LocalDate dateCreation;
    
    // ========== HOOKS DU CYCLE DE VIE ==========
    
    /**
     * Exécuté automatiquement AVANT la première sauvegarde
     */
    @PrePersist
    protected void onCreate() {
        this.dateCreation = LocalDate.now();
    }
    
    // ========== MÉTHODES UTILITAIRES ==========
    
    @Override
    public String toString() {
        return String.format(
            "Produit{id=%d, nom='%s', prix=%.2f DT, date=%s}",
            idProduit, nomProduit, prixProduit, dateCreation
        );
    }
}
```

**Points Clés** :

1. **@Entity** : JPA reconnaît cette classe
2. **@Table(name = "produit")** : Crée une table MySQL appelée "produit"
3. **@Id + @GeneratedValue** : La BD génère l'ID (1, 2, 3...)
4. **@NotBlank, @Size, @Positive** : Validations de données
5. **@PrePersist** : Initialise automatiquement la date
6. **Lombok** : Réduit le boilerplate (getter/setter)

### ✅ Checkpoint #2 : Entité Créée

Vérifiez que vous avez :

- [ ] Créé la classe Produit dans `entities/`
- [ ] Classe compile sans erreurs
- [ ] Importé les bonnes annotations (@Entity, @Column, etc.)
- [ ] Compris le rôle de chaque annotation
- [ ] Vu le schéma de table dans IntelliJ (**Database** tab)

Si une case n'est pas cochée, relisez la section 2.

### 🤔 Réflexion Intermédiaire (5 min)

À ce stade, vous avez créé une entité Produit. Avant de passer à la couche Repository, réfléchissez :

**Q1** : Pourquoi mettre les validations (@NotBlank, @Size) dans l'entité et pas dans le service ?  
**Réponse** : Les validations proches des données garantissent l'intégrité même si les données viennent de plusieurs sources (API, fichier, etc.)

**Q2** : Qu'arriverait-il si on oubliait @Entity ?  
**Réponse** : Hibernate ne reconnaîtrait pas la classe, et les annotations @Column seraient ignorées. Erreur : "Cannot find entity mapping"

**Q3** : Pourquoi utiliser Lombok ?  
**Réponse** : Réduit le boilerplate (200+ lignes de code générées automatiquement)

---

## Partie 3 : Repository

### Étape 3.1 : Comprendre JpaRepository

Avant d'écrire le Repository, regardez cet héritage :

```
                    Repository
                        ↑
                CrudRepository (CRUD de base)
                        ↑
                  JpaRepository (additions JPA)
```

**Ce que vous hérité automatiquement quand vous extends JpaRepository** :

| Méthode Héritée | Rôle | Requête SQL Générée |
|-----------------|------|---------------------|
| `save(T)` | Insérer ou mettre à jour | INSERT ou UPDATE |
| `findById(ID)` | Récupérer par clé primaire | SELECT ... WHERE id = ? |
| `findAll()` | Récupérer TOUS les enregistrements | SELECT * |
| `findAll(Pageable)` | Paginer les résultats | SELECT * LIMIT X OFFSET Y |
| `deleteById(ID)` | Supprimer par clé primaire | DELETE WHERE id = ? |
| `count()` | Compter les enregistrements | SELECT COUNT(*) |
| `existsById(ID)` | Vérifier l'existence | SELECT COUNT(*) > 0 |

**Vous n'avez rien à implémenter !** Spring Data le fait automatiquement ✨

### Étape 3.2 : Créer votre Premier Repository

Créez le fichier `src/main/java/tn/iset/produits/repositories/ProduitRepository.java` :

```java
package tn.iset.produits.repositories;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import tn.iset.produits.entities.Produit;

/**
 * Repository pour l'entité Produit
 * 
 * JpaRepository<Produit, Long> signifie :
 * - Gérer les entités Produit
 * - La clé primaire est de type Long
 * 
 * Spring Data génère automatiquement :
 * - save(), findById(), findAll(), delete(), count(), etc.
 */
@Repository
public interface ProduitRepository extends JpaRepository<Produit, Long> {
    
    // Pour le moment, rien à ajouter !
    // Les méthodes CRUD sont héritées automatiquement
}
```

**C'est TOUT ce dont vous avez besoin pour faire CRUD !** 🎉

Les méthodes `save()`, `findById()`, `findAll()`, `deleteById()` sont déjà disponibles.

### Étape 3.3 : Ajouter des Méthodes Personnalisées (Progressivement)

Une fois que CRUD fonctionne, ajoutez de la recherche :

```java
package tn.iset.produits.repositories;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;
import tn.iset.produits.entities.Produit;

import java.util.List;
import java.util.Optional;

@Repository
public interface ProduitRepository extends JpaRepository<Produit, Long> {
    
    // ==================== RECHERCHE PAR PRIX ====================
    
    /**
     * Trouver les produits avec un prix exact
     * Spring Data traduit en : SELECT * FROM produit WHERE prix_produit = ?
     */
    List<Produit> findByPrixProduit(Double prix);
    
    /**
     * Trouver les produits plus chers qu'un seuil
     * Spring Data traduit en : SELECT * FROM produit WHERE prix_produit >= ?
     */
    List<Produit> findByPrixProduitGreaterThanEqual(Double minPrix);
    
    /**
     * Trouver les produits moins chers qu'un seuil
     * Spring Data traduit en : SELECT * FROM produit WHERE prix_produit <= ?
     */
    List<Produit> findByPrixProduitLessThanEqual(Double maxPrix);
    
    /**
     * Trouver les produits dans une plage de prix
     * Spring Data traduit en : SELECT * FROM produit WHERE prix_produit BETWEEN ? AND ?
     */
    List<Produit> findByPrixProduitBetween(Double minPrix, Double maxPrix);
    
    // ==================== RECHERCHE PAR NOM ====================
    
    /**
     * Recherche par nom contenant une chaîne (insensible à la casse)
     * Spring Data traduit en : SELECT * FROM produit WHERE LOWER(nom_produit) LIKE LOWER(?)
     */
    List<Produit> findByNomProduitContainingIgnoreCase(String keyword);
    
    /**
     * Trouver un produit par son nom exact
     */
    Optional<Produit> findByNomProduit(String nom);
    
    // ==================== REQUÊTES PERSONNALISÉES AVEC @Query ====================
    
    /**
     * Requête JPQL (Language indépendant de la BD)
     * Utilise les noms des propriétés Java
     */
    @Query("SELECT p FROM Produit p WHERE p.prixProduit > :minPrix ORDER BY p.prixProduit ASC")
    List<Produit> findProduitsPlusChers(@Param("minPrix") Double prix);
    
    /**
     * Requête SQL native (spécifique à MySQL)
     * Utilise les noms des colonnes BD
     */
    @Query(value = "SELECT * FROM produit WHERE nom_produit LIKE %:keyword% ORDER BY nom_produit ASC", 
           nativeQuery = true)
    List<Produit> searchByKeywordNative(@Param("keyword") String keyword);
    
    /**
     * Compter les produits d'une certaine gamme de prix
     */
    @Query("SELECT COUNT(p) FROM Produit p WHERE p.prixProduit >= :min AND p.prixProduit <= :max")
    Long countInPriceRange(@Param("min") Double min, @Param("max") Double max);
}
```

**Règles pour les noms de méthodes Spring Data** :

- `findBy` + NomPropriété = Rechercher par cette propriété
- `GreaterThan` / `LessThan` = > / <
- `GreaterThanEqual` / `LessThanEqual` = >= / <=
- `Between` = BETWEEN
- `Containing` = LIKE %text%
- `IgnoreCase` = Insensible à la casse
- `OrderBy` + Propriété = ORDER BY

**JPQL vs SQL Native** :
- **JPQL** : Parler d'objets Java (`Produit`, `prixProduit`) - Portable
- **SQL Native** : Parler de tables/colonnes (`produit`, `prix_produit`) - Moins portable

**Préférez JPQL** (plus portable entre BD) 🎯

### ✅ Checkpoint #3 : Repository Complet

Vérifiez que vous avez :

- [ ] Créé ProduitRepository extends JpaRepository<Produit, Long>
- [ ] Je comprends que les méthodes CRUD sont héritées
- [ ] Je sais utiliser save(), findAll(), deleteById()
- [ ] Je comprends comment Spring Data génère le SQL
- [ ] J'ai ajouté au moins 2-3 requêtes personnalisées

Si une case n'est pas cochée, relisez la section 3.

---

## Partie 4 : Service

### Étape 4.1 : Créer l'Interface Service

Créez le fichier `src/main/java/tn/iset/produits/services/ProduitService.java` :

```java
package tn.iset.produits.services;

import tn.iset.produits.entities.Produit;

import java.util.List;

/**
 * Interface Service pour les produits
 * 
 * Définit LE QUOI (contrat) sans détails d'implémentation
 * Permet le découplage et la testabilité
 */
public interface ProduitService {
    
    /**
     * Sauvegarder un produit (CREATE ou UPDATE)
     * @param produit le produit à sauvegarder
     * @return le produit sauvegardé avec l'ID générée
     * @throws IllegalArgumentException si les données sont invalides
     */
    Produit saveProduit(Produit produit);
    
    /**
     * Récupérer un produit par son ID
     * @param id l'ID du produit
     * @return le produit si trouvé
     * @throws RuntimeException si le produit n'existe pas
     */
    Produit getProduitById(Long id);
    
    /**
     * Récupérer TOUS les produits
     * @return liste de tous les produits
     */
    List<Produit> getAllProduits();
    
    /**
     * Mettre à jour un produit existant
     * @param id l'ID du produit
     * @param produitMaj les nouvelles données
     * @return le produit mis à jour
     */
    Produit updateProduit(Long id, Produit produitMaj);
    
    /**
     * Supprimer un produit par son ID
     * @param id l'ID du produit
     */
    void deleteProduit(Long id);
    
    /**
     * Récupérer les produits dans une plage de prix
     * @param minPrix le prix minimum
     * @param maxPrix le prix maximum
     * @return liste des produits dans la plage
     */
    List<Produit> getProduitsByPriceRange(Double minPrix, Double maxPrix);
    
    /**
     * Rechercher les produits par mot-clé
     * @param keyword mot-clé à rechercher
     * @return liste des produits trouvés
     */
    List<Produit> searchProduits(String keyword);
    
    /**
     * Compter le nombre total de produits
     * @return nombre de produits
     */
    Long countProduits();
}
```

### Étape 4.2 : Implémenter le Service

Créez le fichier `src/main/java/tn/iset/produits/services/ProduitServiceImpl.java` :

```java
package tn.iset.produits.services;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import tn.iset.produits.entities.Produit;
import tn.iset.produits.repositories.ProduitRepository;

import java.util.List;

/**
 * Implémentation du service Produit
 * 
 * Contient la logique métier :
 * - Validation des données
 * - Orchestration des opérations
 * - Gestion des erreurs
 * - Logging
 */
@Service
@Slf4j                          // Lombok : injecte un Logger
@RequiredArgsConstructor        // Lombok : génère un constructeur avec dépendances
public class ProduitServiceImpl implements ProduitService {
    
    // ========== INJECTION DE DÉPENDANCE ==========
    
    private final ProduitRepository produitRepository;
    
    // ========== IMPLÉMENTATION DES MÉTHODES ==========
    
    /**
     * Sauvegarder un produit avec validation
     */
    @Override
    @Transactional  // Les modifs sont auto-committées
    public Produit saveProduit(Produit produit) {
        log.info("Tentative de sauvegarde du produit: {}", produit.getNomProduit());
        
        // ===== VALIDATION =====
        if (produit == null) {
            log.warn("Tentative de sauvegarde d'un produit NULL");
            throw new IllegalArgumentException("Le produit ne peut pas être null");
        }
        
        if (produit.getNomProduit() == null || produit.getNomProduit().isBlank()) {
            log.warn("Tentative avec nom vide");
            throw new IllegalArgumentException("Le nom du produit ne peut pas être vide");
        }
        
        if (produit.getPrixProduit() == null || produit.getPrixProduit() <= 0) {
            log.warn("Tentative avec prix invalide: {}", produit.getPrixProduit());
            throw new IllegalArgumentException("Le prix doit être > 0");
        }
        
        // ===== SAUVEGARDE =====
        Produit saved = produitRepository.save(produit);
        log.info("✅ Produit sauvegardé avec succès | ID: {} | Nom: {}", 
                 saved.getIdProduit(), saved.getNomProduit());
        
        return saved;
    }
    
    /**
     * Récupérer un produit par son ID
     */
    @Override
    @Transactional(readOnly = true)  // Lecture seule (optimisation)
    public Produit getProduitById(Long id) {
        log.info("Recherche du produit avec l'ID: {}", id);
        
        return produitRepository.findById(id)
                .orElseThrow(() -> {
                    log.warn("Produit non trouvé avec l'ID: {}", id);
                    return new RuntimeException("Produit non trouvé avec l'ID: " + id);
                });
    }
    
    /**
     * Récupérer TOUS les produits
     */
    @Override
    @Transactional(readOnly = true)
    public List<Produit> getAllProduits() {
        log.info("Récupération de tous les produits");
        List<Produit> produits = produitRepository.findAll();
        log.info("Nombre de produits trouvés: {}", produits.size());
        return produits;
    }
    
    /**
     * Mettre à jour un produit
     */
    @Override
    @Transactional
    public Produit updateProduit(Long id, Produit produitMaj) {
        log.info("Mise à jour du produit avec l'ID: {}", id);
        
        // Récupérer le produit existant
        Produit existant = getProduitById(id);
        
        // Valider les nouvelles données
        if (produitMaj.getNomProduit() != null && !produitMaj.getNomProduit().isBlank()) {
            existant.setNomProduit(produitMaj.getNomProduit());
        }
        
        if (produitMaj.getPrixProduit() != null && produitMaj.getPrixProduit() > 0) {
            existant.setPrixProduit(produitMaj.getPrixProduit());
        }
        
        // Sauvegarder les modifications
        Produit updated = produitRepository.save(existant);
        log.info("✅ Produit mis à jour avec succès | ID: {}", updated.getIdProduit());
        
        return updated;
    }
    
    /**
     * Supprimer un produit
     */
    @Override
    @Transactional
    public void deleteProduit(Long id) {
        log.info("Suppression du produit avec l'ID: {}", id);
        
        // Vérifier que le produit existe
        if (!produitRepository.existsById(id)) {
            log.warn("Tentative de suppression d'un produit inexistant | ID: {}", id);
            throw new RuntimeException("Produit non trouvé avec l'ID: " + id);
        }
        
        produitRepository.deleteById(id);
        log.info("✅ Produit supprimé avec succès | ID: {}", id);
    }
    
    /**
     * Récupérer les produits dans une plage de prix
     */
    @Override
    @Transactional(readOnly = true)
    public List<Produit> getProduitsByPriceRange(Double minPrix, Double maxPrix) {
        log.info("Recherche de produits entre {} et {} DT", minPrix, maxPrix);
        List<Produit> produits = produitRepository.findByPrixProduitBetween(minPrix, maxPrix);
        log.info("Nombre de produits trouvés: {}", produits.size());
        return produits;
    }
    
    /**
     * Rechercher les produits par mot-clé
     */
    @Override
    @Transactional(readOnly = true)
    public List<Produit> searchProduits(String keyword) {
        log.info("Recherche de produits avec le mot-clé: '{}'", keyword);
        
        if (keyword == null || keyword.isBlank()) {
            log.warn("Mot-clé vide");
            return List.of();  // Retourner une liste vide
        }
        
        List<Produit> results = produitRepository.findByNomProduitContainingIgnoreCase(keyword);
        log.info("Nombre de résultats trouvés: {}", results.size());
        return results;
    }
    
    /**
     * Compter le nombre total de produits
     */
    @Override
    @Transactional(readOnly = true)
    public Long countProduits() {
        return produitRepository.count();
    }
}
```

**Points Clés** :

1. **@Service** : Spring reconnait cette classe comme un bean service
2. **@RequiredArgsConstructor** : Lombok génère le constructeur avec dépendances
3. **@Transactional** : Les modifications sont auto-committées
4. **@Transactional(readOnly = true)** : Optimisation pour les lectures
5. **@Slf4j** : Injecte un Logger pour les logs
6. **Validation** : Chaque méthode valide les données avant d'opérer
7. **Logging** : Chaque opération est loggée pour le debugging

### ✅ Checkpoint #4 : Service Complet

Vérifiez que vous avez :

- [ ] Créé l'interface ProduitService
- [ ] Créé la classe ProduitServiceImpl qui l'implémente
- [ ] Service injecte le Repository avec @RequiredArgsConstructor
- [ ] Chaque méthode valide les données
- [ ] Chaque méthode loggue ses opérations
- [ ] Comprenez pourquoi @Transactional est important

Si une case n'est pas cochée, relisez la section 4.

---

## Partie 5 : Configuration BD

### Étape 5.1 : Script de Création BD

Créez un fichier `src/main/resources/init.sql` :

```sql
-- Script d'initialisation de la base de données
-- Exécutez ce script AVANT de lancer l'application

-- ===== 1. Créer la base de données =====
CREATE DATABASE IF NOT EXISTS spring_db 
CHARACTER SET utf8mb4 
COLLATE utf8mb4_unicode_ci;

-- ===== 2. Créer l'utilisateur Spring =====
CREATE USER IF NOT EXISTS 'springuser'@'localhost' IDENTIFIED BY 'springpass';

-- ===== 3. Donner les permissions à l'utilisateur =====
GRANT ALL PRIVILEGES ON spring_db.* TO 'springuser'@'localhost';
FLUSH PRIVILEGES;

-- ===== 4. Montrer que c'est prêt =====
SHOW DATABASES LIKE 'spring_db';
SHOW GRANTS FOR 'springuser'@'localhost';
```

**Comment l'exécuter** :

```bash
# Option 1 : En ligne de commande
mysql -u root -p < src/main/resources/init.sql

# Option 2 : Dans un terminal MySQL interactif
mysql -u root -p
mysql> source /chemin/vers/init.sql;
```

### Étape 5.2 : Configuration application.properties

Créez ou modifiez le fichier `src/main/resources/application.properties` :

```properties
# ===== Serveur =====
server.port=8080
server.servlet.context-path=/api

# ===== Base de Données MySQL =====
spring.datasource.url=jdbc:mysql://localhost:3306/spring_db?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
spring.datasource.username=springuser
spring.datasource.password=springpass
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# ===== JPA/Hibernate Configuration =====
spring.jpa.hibernate.ddl-auto=update
# ddl-auto options:
#   - create       : Crée les tables (destructif)
#   - create-drop  : Crée au démarrage, supprime à l'arrêt
#   - update       : Mets à jour le schéma (sans perdre données)
#   - validate     : Valide le schéma (pas de modif)
#   - none         : Pas d'action sur le schéma

spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.use_sql_comments=true

# ===== Logging =====
logging.level.root=INFO
logging.level.tn.iset.produits=DEBUG
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE

# ===== Application =====
spring.application.name=produits-management
spring.profiles.active=dev
```

### Étape 5.3 : Configuration Profil Développement

Créez le fichier `src/main/resources/application-dev.properties` :

```properties
# Configuration spécifique au développement

spring.datasource.url=jdbc:mysql://localhost:3306/spring_db?useSSL=false&serverTimezone=UTC
spring.datasource.username=springuser
spring.datasource.password=springpass

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

logging.level.tn.iset.produits=DEBUG
logging.level.org.hibernate.SQL=DEBUG

# DevTools - Rechargement automatique
spring.devtools.restart.enabled=true
```

### Étape 5.4 : Tester la Connexion BD

#### Option 1 : Via IntelliJ IDEA

1. Allez dans l'onglet **Database** (côté droit)
2. Cliquez sur le **+** → **Data Source** → **MySQL**
3. Remplissez les infos :
   - **Host** : `localhost`
   - **Port** : `3306`
   - **Database** : `spring_db`
   - **User** : `springuser`
   - **Password** : `springpass`
4. Cliquez sur **Test Connection**

✅ Si c'est vert, la connexion marche !

#### Option 2 : Via Terminal

```bash
# Tester la connexion
mysql -h localhost -u springuser -p spring_db

# Mot de passe : springpass

# Vérifier que la table est créée
SHOW TABLES;
DESCRIBE produit;
```

### ✅ Checkpoint #5 : Configuration BD

Vérifiez que vous avez :

- [ ] Créé la BD avec le script init.sql
- [ ] Créé l'utilisateur springuser
- [ ] Configuration application.properties correcte
- [ ] Test de connexion dans IntelliJ réussi
- [ ] Hibernate a créé la table `produit` automatiquement

Si une case n'est pas cochée, revérifiez les étapes 5.1-5.4.

---

## Partie 6 : Tests Unitaires avec JUnit 5

### Étape 6.1 : Créer la Classe de Test

Créez le fichier `src/test/java/tn/iset/produits/services/ProduitServiceTest.java` :

```java
package tn.iset.produits.services;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import tn.iset.produits.entities.Produit;
import tn.iset.produits.repositories.ProduitRepository;

import java.time.LocalDate;
import java.util.List;

import static org.assertj.core.api.Assertions.*;
import static org.junit.jupiter.api.Assertions.*;

/**
 * Tests unitaires pour ProduitService
 * 
 * Stratégie de test :
 * 1. Cas normal (happy path)
 * 2. Cas d'erreur (exceptions)
 * 3. Cas limites (edge cases)
 */
@SpringBootTest
class ProduitServiceTest {
    
    @Autowired
    ProduitService produitService;
    
    @Autowired
    ProduitRepository produitRepository;
    
    /**
     * setUp() est exécuté AVANT chaque test
     */
    @BeforeEach
    void setUp() {
        // Nettoyage optionnel avant chaque test
        // produitRepository.deleteAll();
    }
    
    // ============================================
    // 1️⃣ CAS NORMAL : Happy Path
    // ============================================
    
    /**
     * Test: Créer un produit valide
     * 
     * Donné (GIVEN) : Un produit avec tous les champs
     * Quand (WHEN) : On l'enregistre
     * Alors (THEN) : Il est sauvegardé avec un ID
     */
    @Test
    @DisplayName("Sauvegarder un produit valide doit retourner l'objet avec un ID")
    void testSaveProduitSuccess() {
        // GIVEN
        Produit produit = new Produit(
            null,  // ID null (auto-généré)
            "PC Portable Dell",
            1850.00,
            LocalDate.now()
        );
        
        // WHEN
        Produit saved = produitService.saveProduit(produit);
        
        // THEN
        assertThat(saved).isNotNull();
        assertThat(saved.getIdProduit()).isGreaterThan(0);
        assertThat(saved.getNomProduit()).isEqualTo("PC Portable Dell");
        assertThat(saved.getPrixProduit()).isEqualTo(1850.00);
    }
    
    /**
     * Test : Récupérer un produit par ID
     */
    @Test
    @DisplayName("Récupérer un produit existant par son ID")
    void testGetProduitById() {
        // GIVEN
        Produit saved = produitService.saveProduit(
            new Produit(null, "PC Asus", 1500.0, LocalDate.now())
        );
        
        // WHEN
        Produit found = produitService.getProduitById(saved.getIdProduit());
        
        // THEN
        assertThat(found).isNotNull();
        assertThat(found.getIdProduit()).isEqualTo(saved.getIdProduit());
        assertThat(found.getNomProduit()).isEqualTo("PC Asus");
    }
    
    /**
     * Test : Mettre à jour un produit
     */
    @Test
    @DisplayName("Mettre à jour un produit modifie les champs")
    void testUpdateProduit() {
        // GIVEN
        Produit saved = produitService.saveProduit(
            new Produit(null, "PC Old", 1000.0, LocalDate.now())
        );
        
        Produit updates = new Produit(null, "PC New", 1200.0, null);
        
        // WHEN
        Produit updated = produitService.updateProduit(saved.getIdProduit(), updates);
        
        // THEN
        assertThat(updated.getNomProduit()).isEqualTo("PC New");
        assertThat(updated.getPrixProduit()).isEqualTo(1200.0);
    }
    
    /**
     * Test : Supprimer un produit
     */
    @Test
    @DisplayName("Supprimer un produit le rend inaccessible")
    void testDeleteProduit() {
        // GIVEN
        Produit saved = produitService.saveProduit(
            new Produit(null, "PC Toshiba", 1000.0, LocalDate.now())
        );
        Long idToDelete = saved.getIdProduit();
        
        // WHEN
        produitService.deleteProduit(idToDelete);
        
        // THEN
        assertThrows(RuntimeException.class, () -> {
            produitService.getProduitById(idToDelete);
        }, "Le produit ne doit plus exister après suppression");
    }
    
    /**
     * Test : Récupérer tous les produits
     */
    @Test
    @DisplayName("Récupérer tous les produits retourne une liste")
    void testGetAllProduits() {
        // GIVEN
        produitService.saveProduit(new Produit(null, "PC 1", 1000.0, LocalDate.now()));
        produitService.saveProduit(new Produit(null, "PC 2", 2000.0, LocalDate.now()));
        
        // WHEN
        List<Produit> all = produitService.getAllProduits();
        
        // THEN
        assertThat(all)
            .isNotEmpty()
            .hasSizeGreaterThanOrEqualTo(2);
    }
    
    // ============================================
    // 2️⃣ CAS D'ERREUR : Gestion Exceptions
    // ============================================
    
    /**
     * Test : Sauvegarder un produit NULL
     */
    @Test
    @DisplayName("Sauvegarder NULL doit lever IllegalArgumentException")
    void testSaveProduitNull() {
        // WHEN/THEN
        assertThrows(IllegalArgumentException.class, () -> {
            produitService.saveProduit(null);
        }, "Doit lever IllegalArgumentException pour null");
    }
    
    /**
     * Test : Récupérer un produit inexistant
     */
    @Test
    @DisplayName("Récupérer un produit inexistant doit lever RuntimeException")
    void testGetProduitNotFound() {
        // WHEN/THEN
        assertThrows(RuntimeException.class, () -> {
            produitService.getProduitById(999999L);
        }, "Doit lever RuntimeException pour ID inexistant");
    }
    
    /**
     * Test : Supprimer un produit inexistant
     */
    @Test
    @DisplayName("Supprimer un produit inexistant doit lever RuntimeException")
    void testDeleteNonExistentProduit() {
        // WHEN/THEN
        assertThrows(RuntimeException.class, () -> {
            produitService.deleteProduit(999999L);
        });
    }
    
    // ============================================
    // 3️⃣ CAS LIMITES : Edge Cases
    // ============================================
    
    /**
     * Test : Sauvegarder un produit avec prix zéro
     */
    @Test
    @DisplayName("Prix zéro doit lever IllegalArgumentException")
    void testSaveProduitWithZeroPrice() {
        // GIVEN
        Produit produit = new Produit(null, "PC", 0.0, LocalDate.now());
        
        // WHEN/THEN
        assertThrows(IllegalArgumentException.class, () -> {
            produitService.saveProduit(produit);
        }, "Prix ne peut pas être zéro");
    }
    
    /**
     * Test : Sauvegarder un produit avec prix négatif
     */
    @Test
    @DisplayName("Prix négatif doit lever IllegalArgumentException")
    void testSaveProduitWithNegativePrice() {
        // GIVEN
        Produit produit = new Produit(null, "PC", -100.0, LocalDate.now());
        
        // WHEN/THEN
        assertThrows(IllegalArgumentException.class, () -> {
            produitService.saveProduit(produit);
        }, "Prix ne peut pas être négatif");
    }
    
    /**
     * Test : Sauvegarder un produit avec nom vide
     */
    @Test
    @DisplayName("Nom vide doit lever IllegalArgumentException")
    void testSaveProduitWithBlankName() {
        // GIVEN
        Produit produit = new Produit(null, "   ", 1000.0, LocalDate.now());
        
        // WHEN/THEN
        assertThrows(IllegalArgumentException.class, () -> {
            produitService.saveProduit(produit);
        }, "Nom ne peut pas être vide");
    }
    
    /**
     * Test : Sauvegarder un produit avec nom NULL
     */
    @Test
    @DisplayName("Nom NULL doit lever IllegalArgumentException")
    void testSaveProduitWithNullName() {
        // GIVEN
        Produit produit = new Produit(null, null, 1000.0, LocalDate.now());
        
        // WHEN/THEN
        assertThrows(IllegalArgumentException.class, () -> {
            produitService.saveProduit(produit);
        });
    }
    
    /**
     * Test : Sauvegarder un produit avec prix NULL
     */
    @Test
    @DisplayName("Prix NULL doit lever IllegalArgumentException")
    void testSaveProduitWithNullPrice() {
        // GIVEN
        Produit produit = new Produit(null, "PC", null, LocalDate.now());
        
        // WHEN/THEN
        assertThrows(IllegalArgumentException.class, () -> {
            produitService.saveProduit(produit);
        });
    }
    
    // ============================================
    // 4️⃣ RECHERCHE ET STATISTIQUES
    // ============================================
    
    @Test
    @DisplayName("Rechercher par plage de prix retourne les bons produits")
    void testGetProduitsByPriceRange() {
        // GIVEN
        produitService.saveProduit(new Produit(null, "PC 500", 500.0, LocalDate.now()));
        produitService.saveProduit(new Produit(null, "PC 1500", 1500.0, LocalDate.now()));
        produitService.saveProduit(new Produit(null, "PC 2500", 2500.0, LocalDate.now()));
        
        // WHEN
        List<Produit> results = produitService.getProduitsByPriceRange(1000.0, 2000.0);
        
        // THEN
        assertThat(results).hasSize(1);
        assertThat(results.get(0).getNomProduit()).isEqualTo("PC 1500");
    }
    
    @Test
    @DisplayName("Rechercher par mot-clé retourne les bons produits")
    void testSearchProduits() {
        // GIVEN
        produitService.saveProduit(new Produit(null, "PC Dell", 1000.0, LocalDate.now()));
        produitService.saveProduit(new Produit(null, "Laptop Asus", 2000.0, LocalDate.now()));
        
        // WHEN
        List<Produit> results = produitService.searchProduits("Dell");
        
        // THEN
        assertThat(results).hasSize(1);
        assertThat(results.get(0).getNomProduit()).isEqualTo("PC Dell");
    }
    
    @Test
    @DisplayName("Compter les produits retourne le bon nombre")
    void testCountProduits() {
        // GIVEN
        long initialCount = produitService.countProduits();
        produitService.saveProduit(new Produit(null, "PC New", 1000.0, LocalDate.now()));
        
        // WHEN
        long newCount = produitService.countProduits();
        
        // THEN
        assertThat(newCount).isEqualTo(initialCount + 1);
    }
}
```

### Étape 6.2 : Exécuter les Tests

1. **Clic droit** sur la classe → **Run 'ProduitServiceTest'**
2. Ou appuyez sur **Shift+Ctrl+F10** (Windows/Linux)
3. Ou **Shift+Cmd+R** (Mac)

**Résultat attendu** :
```
✅ testSaveProduitSuccess
✅ testGetProduitById
✅ testUpdateProduit
✅ testDeleteProduit
✅ testGetAllProduits
✅ testSaveProduitNull
✅ testGetProduitNotFound
✅ testDeleteNonExistentProduit
✅ testSaveProduitWithZeroPrice
✅ testSaveProduitWithNegativePrice
✅ testSaveProduitWithBlankName
✅ testSaveProduitWithNullName
✅ testSaveProduitWithNullPrice
✅ testGetProduitsByPriceRange
✅ testSearchProduits
✅ testCountProduits

Tests passés: 16/16 ✅
```

### ✅ Checkpoint #6 : Tests Complets

Vérifiez que vous avez :

- [ ] Créé la classe de test ProduitServiceTest
- [ ] Tous les tests passent (16/16)
- [ ] Je comprends la structure AAA (Arrange-Act-Assert)
- [ ] Je sais écrire un test pour un cas normal
- [ ] Je sais écrire un test pour une exception
- [ ] Je comprends l'importance des tests

Si une case n'est pas cochée, relisez la section 6.

---

## Partie 7 : Initialisation des Données

### Étape 7.1 : Modifier la Classe Principale

Modifiez `src/main/java/tn/iset/produits/ProduitsApplication.java` :

```java
package tn.iset.produits;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import tn.iset.produits.entities.Produit;
import tn.iset.produits.services.ProduitService;

import java.time.LocalDate;

/**
 * Classe principale de l'application Spring Boot.
 * 
 * Implémente CommandLineRunner pour exécuter du code au démarrage.
 */
@SpringBootApplication
@Slf4j
@RequiredArgsConstructor
public class ProduitsApplication implements CommandLineRunner {
    
    private final ProduitService produitService;
    
    public static void main(String[] args) {
        SpringApplication.run(ProduitsApplication.class, args);
    }
    
    /**
     * Méthode exécutée au démarrage de l'application
     * Initialise les données de test
     */
    @Override
    public void run(String... args) throws Exception {
        log.info("========================================");
        log.info("🚀 Initialisation des données de test...");
        log.info("========================================");
        
        // Vérifier si des produits existent déjà
        if (produitService.countProduits() > 0) {
            log.info("ℹ️ Des produits existent déjà. Initialisation ignorée.");
            return;
        }
        
        // Créer des produits de démonstration
        Produit prod1 = Produit.builder()
                .nomProduit("PC Portable Asus VivoBook")
                .prixProduit(1850.00)
                .dateCreation(LocalDate.now())
                .build();
        
        Produit prod2 = Produit.builder()
                .nomProduit("PC Dell Inspiron 15")
                .prixProduit(2200.00)
                .dateCreation(LocalDate.now())
                .build();
        
        Produit prod3 = Produit.builder()
                .nomProduit("PC Toshiba Satellite")
                .prixProduit(1650.00)
                .dateCreation(LocalDate.now())
                .build();
        
        // Sauvegarder les produits
        produitService.saveProduit(prod1);
        produitService.saveProduit(prod2);
        produitService.saveProduit(prod3);
        
        log.info("✅ {} produits initialisés avec succès", produitService.countProduits());
        
        // Afficher les produits
        log.info("📋 Liste des produits :");
        produitService.getAllProduits().forEach(p -> 
            log.info("  - {} | {} DT", p.getNomProduit(), p.getPrixProduit())
        );
    }
}
```

### Étape 7.2 : Lancer l'Application

1. Clic droit sur `ProduitsApplication.java` → **Run**
2. Ou appuyez sur **Shift+F10**

**Vous devriez voir dans la console** :

```
========================================
🚀 Initialisation des données de test...
========================================

✅ 3 produits initialisés avec succès

📋 Liste des produits :
  - PC Portable Asus VivoBook | 1850.0 DT
  - PC Dell Inspiron 15 | 2200.0 DT
  - PC Toshiba Satellite | 1650.0 DT
```

### ✅ Checkpoint #7 : Application Lancée

Vérifiez que vous avez :

- [ ] L'application démarre sans erreurs
- [ ] Vous voyez les logs d'initialisation
- [ ] Les 3 produits s'affichent dans la console
- [ ] Pas d'erreurs Spring ou Hibernate
- [ ] La BD contient maintenant les données (vérifiable avec H2 ou MySQL)

Si une case n'est pas cochée, vérifiez les logs d'erreur.

---

## Pièges Courants

### ⚠️ Piège 1️⃣ : Oublier @Entity

**❌ Erreur Courante** :
```java
public class Produit {  // ← Pas d'annotation !
    private Long id;
}
```

**Symptôme** :
```
Unknown entity: tn.iset.produits.entities.Produit
```

**✅ Correction** :
```java
@Entity  // ← Ajouter cette annotation
public class Produit {
    @Id
    private Long id;
}
```

---

### ⚠️ Piège 2️⃣ : Repository sans Paramètres de Type

**❌ Erreur Courante** :
```java
public interface ProduitRepository extends JpaRepository {
    // ← Pas de <Produit, Long>
}
```

**Symptôme** :
```
Compilation error: expected 2 type arguments
```

**✅ Correction** :
```java
public interface ProduitRepository extends JpaRepository<Produit, Long> {
    //                                      ↑               ↑
    //                                  Type Entité    Type ID
}
```

---

### ⚠️ Piège 3️⃣ : Service sans @Service

**❌ Erreur Courante** :
```java
public class ProduitServiceImpl implements ProduitService {
    // ← Pas d'annotation
}
```

**Symptôme** :
```
No qualifying bean of type 'ProduitService' found
```

**✅ Correction** :
```java
@Service
public class ProduitServiceImpl implements ProduitService {
    // ← Spring crée maintenant le bean
}
```

---

### ⚠️ Piège 4️⃣ : @Autowired sans constructeur

**❌ Erreur Courante** :
```java
@Service
public class ProduitServiceImpl {
    @Autowired
    private ProduitRepository repository;
    // ← Pas de constructeur ou setter !
}
```

**✅ Correction Moderne (Avec Lombok)** :
```java
@Service
@RequiredArgsConstructor  // ← Génère un constructeur
public class ProduitServiceImpl {
    private final ProduitRepository repository;  // ← final obligatoire
}
```

---

### ⚠️ Piège 5️⃣ : Oublier @Transactional pour les modifications

**❌ Erreur Courante** :
```java
@Service
public class ProduitServiceImpl {
    public void saveProduit(Produit p) {  // ← Pas d'annotation
        repository.save(p);
    }
}
```

**Problème** : Les modifications peuvent ne pas être committées correctement

**✅ Correction** :
```java
@Service
public class ProduitServiceImpl {
    @Transactional
    public void saveProduit(Produit p) {
        repository.save(p);
    }
}
```

---

### ⚠️ Piège 6️⃣ : Paramètres @Query mal nommés

**❌ Erreur Courante** :
```java
@Query("SELECT p FROM Produit p WHERE p.prixProduit > :minPrice")
List<Produit> findByPrice(@Param("minPrix") Double prix);
                          // ↑ Noms différents !
```

**Symptôme** :
```
Parameter binding error: ... Parameter 'minPrice' not found
```

**✅ Correction** :
```java
@Query("SELECT p FROM Produit p WHERE p.prixProduit > :minPrix")
List<Produit> findByPrice(@Param("minPrix") Double prix);
                          // ↑ Même nom !
```

---

### ⚠️ Piège 7️⃣ : Oublier la Clé Primaire

**❌ Erreur Courante** :
```java
@Entity
public class Produit {
    // ← Pas d'@Id !
    private String nomProduit;
}
```

**Symptôme** :
```
Unable to find unique id property
```

**✅ Correction** :
```java
@Entity
public class Produit {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;  // ← Ajouter la PK
    
    private String nomProduit;
}
```

---

### ⚠️ Piège 8️⃣ : Connection Timeout BD

**❌ Symptôme** :
```
Communications link failure
The last packet sent successfully to the server was 0 milliseconds ago
```

**✅ Solutions** :
1. Vérifier que MySQL est running : `docker ps`
2. Vérifier les crédentiales dans `application.properties`
3. Vérifier le port (par défaut 3306)

```bash
# Tester la connexion
mysql -h localhost -u springuser -p -e "USE spring_db; SHOW TABLES;"
```

---

### ⚠️ Piège 9️⃣ : NullPointerException dans les Tests

**❌ Erreur Courante** :
```java
@Test
void testSave() {
    Produit p = new Produit();
    // repository n'est pas @Autowired !
    repository.save(p);  // ← NPE ici
}
```

**✅ Correction** :
```java
@SpringBootTest  // ← Important !
class ProduitServiceTest {
    
    @Autowired
    ProduitService produitService;
    
    @Test
    void testSave() {
        Produit p = new Produit();
        produitService.saveProduit(p);  // ← Marche maintenant
    }
}
```

---

### ⚠️ Piège 🔟 : Hibernate Crée les Tables Mais ne Les Rempli Pas

**❌ Problème** :
```
Table créée ✅
Mais aucune donnée ❌
```

**Raison** : `CommandLineRunner` n'a pas été implémenté, ou l'initialisation est ignorée

**✅ Solution** : Utiliser `CommandLineRunner` (voir Partie 7)

---

## Glossaire

### A

**Annotation** : Marque (décorateur) qui ajoute de la metadata au code
- Exemple : `@Entity`, `@Service`, `@Transactional`

**Assertion** : Vérification dans un test (assertTrue, assertEqual, etc.)

**Auto-increment** : L'ID s'incrémente automatiquement (1, 2, 3...)

**Autowired** : Annotation pour injecter les dépendances automatiquement

### B

**Bean** : Objet géré par Spring (créé, injecté, détruit par Spring)

**Builder Pattern** : Pattern créationnel pour créer des objets (`Produit.builder()`)

**BDD** : Base De Données

### C

**CRUD** : Create, Read, Update, Delete (4 opérations basiques)

**CrudRepository** : Interface avec CRUD de base

**Column** : Annotation pour mapper une propriété à une colonne BD

### D

**DAO** : Data Access Object (ancienne terminologie pour Repository)

**DTO** : Data Transfer Object (objet pour transférer les données)

**DDL** : Data Definition Language (CREATE, ALTER, DROP)

### E

**Entity** : Classe Java mappée à une table BD (`@Entity`)

**Enum** : Type avec valeurs prédéfinies (`GenerationType.IDENTITY`)

### H

**Hibernate** : ORM qui implémente JPA

**Hook** : Méthode appelée à un moment précis (`@PrePersist`, `@PostPersist`)

### I

**ID** : Identifiant unique (clé primaire)

**Injection de Dépendance** : Spring crée et injecte les beans automatiquement

**Interface** : Contrat que doit implémenter une classe

### J

**JPA** : Java Persistence API (standard ORM Java)

**JUnit** : Framework de test Java (JUnit 5 moderne)

**JPQL** : Java Persistence Query Language (requêtes orientées objets)

### L

**Lombok** : Librairie qui réduit le boilerplate (génère getter/setter)

### M

**Mapping** : Correspondance entre classe Java et table BD

**Mock** : Objet bidon pour tester sans dépendances réelles

**MySQL** : Système de gestion de base de données relationnelle

### O

**ORM** : Object-Relational Mapping (mappe objets aux tables)

**Optional** : Wrapper pour valeurs qui peuvent être null

### P

**PK** : Primary Key (clé primaire)

**Polymorphisme** : Une interface, plusieurs implémentations

### Q

**Query** : Requête BD (SELECT, INSERT, UPDATE, DELETE)

**@Query** : Annotation pour écrire des requêtes personnalisées

### R

**Repository** : Interface pour accéder aux données

**Reflection** : Capacité de Java à examiner/modifier le code au runtime

### S

**Schema** : Structure de la base de données (tables, colonnes)

**Service** : Classe avec logique métier (`@Service`)

**SQL** : Structured Query Language (langage pour BD relationnelles)

**Spring** : Framework Java pour applications

**@SpringBootTest** : Annotation pour tester l'app entière

### T

**Table** : Collection de lignes/colonnes en BD

**Transaction** : Opération atomique en BD (Commit/Rollback)

**Test** : Code qui vérifie que le code principal fonctionne

**Temporal** : Annotation pour les dates (`@Temporal`)

### U

**Unit Test** : Test d'une unité de code isolée

### V

**Validation** : Vérifier que les données sont valides

### W

**Web** : Application accessible via HTTP

---

## Exercices d'Évaluation

### Exercice 1 : Validation Avancée

Ajoutez les validations Bean Validation suivantes dans l'entité `Produit` :

```java
import jakarta.validation.constraints.*;

@Entity
public class Produit {
    
    // ===== Nom du produit =====
    @NotBlank(message = "Le nom du produit est obligatoire")
    @Size(min = 3, max = 100, message = "Le nom doit contenir entre 3 et 100 caractères")
    private String nomProduit;
    
    // ===== Prix du produit =====
    @NotNull(message = "Le prix est obligatoire")
    @Positive(message = "Le prix doit être positif")
    @DecimalMin(value = "0.01", message = "Le prix minimum est 0.01 DT")
    private Double prixProduit;
}
```

**À Tester** :
- Sauvegarder un produit avec un nom de 2 caractères (doit échouer)
- Sauvegarder un produit avec un prix de -100 (doit échouer)
- Sauvegarder un produit avec un nom vide (doit échouer)

---

### Exercice 2 : Méthodes de Recherche Personnalisées

Dans `ProduitRepository`, ajoutez les méthodes suivantes :

```java
public interface ProduitRepository extends JpaRepository<Produit, Long> {
    
    // Recherche par prix < seuil
    List<Produit> findByPrixProduitLessThan(Double prix);
    
    // Recherche insensible à la casse
    List<Produit> findByNomProduitContainingIgnoreCase(String keyword);
    
    // Trouver le produit le plus cher
    Optional<Produit> findFirstByOrderByPrixProduitDesc();
    
    // Trouver le produit le moins cher
    Optional<Produit> findFirstByOrderByPrixProduitAsc();
}
```

**Testez ces méthodes** dans `ProduitRepositoryTest` :

```java
@SpringBootTest
class ProduitRepositoryTest {
    
    @Autowired
    ProduitRepository repository;
    
    @Test
    void testFindByPrixLessThan() {
        // Insérer des produits
        // Rechercher avec findByPrixProduitLessThan(2000)
        // Vérifier les résultats
    }
    
    @Test
    void testFindMostExpensive() {
        // Insérer des produits
        // Appeler findFirstByOrderByPrixProduitDesc()
        // Vérifier que c'est le plus cher
    }
}
```

---

### Exercice 3 : Statistiques

Ajoutez les méthodes suivantes dans `ProduitService` et `ProduitServiceImpl` :

```java
public interface ProduitService {
    
    /**
     * Calculer le prix moyen de tous les produits
     */
    Double getAveragePrice();
    
    /**
     * Trouver le produit le plus cher
     */
    Produit getMostExpensiveProduct();
    
    /**
     * Trouver le produit le moins cher
     */
    Produit getCheapestProduct();
    
    /**
     * Compter le nombre de produits au-dessus d'un prix
     */
    Long countProductsAbovePrice(Double price);
}
```

**Implémentation** :

```java
@Override
@Transactional(readOnly = true)
public Double getAveragePrice() {
    List<Produit> all = getAllProduits();
    if (all.isEmpty()) return 0.0;
    return all.stream()
        .mapToDouble(Produit::getPrixProduit)
        .average()
        .orElse(0.0);
}

@Override
@Transactional(readOnly = true)
public Produit getMostExpensiveProduct() {
    return produitRepository.findFirstByOrderByPrixProduitDesc()
        .orElseThrow(() -> new RuntimeException("Aucun produit trouvé"));
}

// À compléter...
```

**Testez ces méthodes** :

```java
@Test
void testGetAveragePrice() {
    // Insérer prod 1000, 2000, 3000
    // Moyenne doit être 2000
    Double avg = produitService.getAveragePrice();
    assertThat(avg).isEqualTo(2000.0);
}
```

---

## Évaluation Finale

### Durée Estimée : 2-3 heures

### Partie A : Questions Théoriques (30-45 min)

**Question 1** (5 points) :  
Expliquez pourquoi la couche Service est importante même si le Repository peut faire CRUD directement. Donnez un exemple concret.

**Réponse Attendue** :
- Service encapsule la logique métier
- Service peut faire plusieurs opérations (validation + audit + logging)
- Service découple Controller de Repository
- Exemple : `saveProduit()` valide avant de sauvegarder

---

**Question 2** (5 points) :  
Quelle est la différence entre JPA et SQL ? Quand utiliser chacun ?

**Réponse Attendue** :
- JPA = Orienté objet (`Produit`, `prixProduit`)
- SQL = Orienté table (`produit`, `prix_produit`)
- JPA pour requêtes simples/standard
- SQL quand JPA ne suffit pas (`@Query` native)

---

**Question 3** (5 points) :  
Quel est le rôle du builder pattern dans Produit ?

**Réponse Attendue** :
- Créer des objets avec beaucoup de champs facilement
- Évite les constructeurs avec beaucoup de paramètres
- `Produit.builder().nomProduit(...).prixProduit(...).build()`

---

**Question 4** (5 points) :  
Pourquoi les tests sont-ils importants ? Donnez 2 avantages.

**Réponse Attendue** :
- Vérifier que le code fonctionne (regression testing)
- Documenter le comportement attendu (documentation vivante)
- Bonus : Refactoriser en confiance

---

**Question 5** (5 points) :  
Qu'est-ce que l'injection de dépendance et pourquoi c'est mieux que `new` ?

**Réponse Attendue** :
- Spring crée et gère les objets (beans)
- Moins couplé (dépendre d'interfaces, pas de classes concrètes)
- Testabilité (mocker facilement)
- Vs. `new` = création manuelle, couplage fort

---

### Partie B : Implémentation Pratique (90-120 min)

**Défi** : Créer un système de gestion de catégories

Vous allez :
1. Créer l'entité `Category`
2. Ajouter une relation Produit ↔ Category (Many-to-One)
3. Créer `CategoryRepository`
4. Créer `CategoryService` + interface
5. Écrire des tests pour Category

#### Étape 1 : Créer Category Entity

```java
@Entity
@Table(name = "categorie")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Category {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long idCategory;
    
    @NotBlank(message = "Le nom de la catégorie est obligatoire")
    @Column(length = 100, nullable = false)
    private String nameCategory;
    
    @OneToMany(mappedBy = "category", cascade = CascadeType.ALL)
    private List<Produit> produits = new ArrayList<>();
}
```

#### Étape 2 : Mettre à Jour Produit

```java
@Entity
public class Produit {
    // ... champs existants ...
    
    @ManyToOne
    @JoinColumn(name = "id_category")
    private Category category;  // ← Ajouter ça
}
```

#### Étape 3 : Créer CategoryRepository

```java
@Repository
public interface CategoryRepository extends JpaRepository<Category, Long> {
    // Ajouter des requêtes personnalisées si besoin
}
```

#### Étape 4 : Créer Service + Interface

```java
public interface CategoryService {
    Category saveCategory(Category category);
    Category getCategoryById(Long id);
    List<Category> getAllCategories();
    // ... autres méthodes ...
}

@Service
@RequiredArgsConstructor
public class CategoryServiceImpl implements CategoryService {
    // Implémentation
}
```

#### Étape 5 : Tests

```java
@SpringBootTest
class CategoryServiceTest {
    // Au moins 5 tests
}
```

**Critères d'Évaluation** :
- ✅ Entity correcte (30 points)
- ✅ Repository complet (20 points)
- ✅ Service + Interface (20 points)
- ✅ Tests passants (20 points)
- ✅ Code lisible et bien structuré (10 points)

---

### Partie C : Documentation & Réflexion (30-45 min)

Écrivez un document (1-2 pages) contenant :

1. **Architecture** : Décrire votre système Category (avec 1 diagramme)
2. **Choix de Design** : Pourquoi Many-to-One et pas Many-to-Many ?
3. **Améliorations** : 3 choses que vous pourriez ajouter
4. **Apprentissages** : Ce que vous avez appris de ce lab

---

### Barème Global

| Section | Points |
|---------|--------|
| Questions Théoriques | 25 points |
| Implémentation | 50 points |
| Documentation | 25 points |
| **TOTAL** | **100 points** |

### Grille de Correction

| Score | Note |
|-------|------|
| 90-100 | A (Excellent) |
| 80-89 | B (Très Bon) |
| 70-79 | C (Bon) |
| 60-69 | D (Acceptable) |
| < 60 | F (À Refaire) |

---

## Ressources Complémentaires

### Documentation Officielle

- **Spring Boot** : https://docs.spring.io/spring-boot/docs/current/reference/html/
- **Spring Data JPA** : https://docs.spring.io/spring-data/jpa/docs/current/reference/html/
- **JPA/Hibernate** : https://hibernate.org/orm/documentation/
- **MySQL** : https://dev.mysql.com/doc/

### Tutoriels et Articles

- **Baeldung Spring Tutorials** : https://www.baeldung.com/spring-tutorial
- **Spring Guides** : https://spring.io/guides
- **IntelliJ IDEA Spring Guide** : https://www.jetbrains.com/help/idea/spring-boot.html

### Outils

- **Spring Initializr** : https://start.spring.io/
- **Lombok Documentation** : https://projectlombok.org/features/
- **JUnit 5 User Guide** : https://junit.org/junit5/docs/current/user-guide/

### Livres Recommandés

- "Spring in Action" (Craig Walls)
- "Spring Boot in Action" (Craig Walls)
- "Effective Java" (Joshua Bloch) - Pour les meilleures pratiques
- "Test Driven Development" (Kent Beck) - Pour comprendre les tests

---

## Conclusion

Félicitations ! 🎉 Vous avez construit les fondations solides d'une application Spring Boot avec :

✅ **Architecture MVC en couches** - Bien structuré et maintenable  
✅ **Entité JPA avec validation** - Modèle de données robuste  
✅ **Repository Spring Data** - Accès données simplifié  
✅ **Couche service** - Logique métier centralisée  
✅ **Tests unitaires complets** - Cas normal, erreur, limites  
✅ **Initialisation de données** - Application prête à l'emploi  
✅ **Meilleures pratiques** - Patterns et principes SOLID  

**Prochaine étape** : Lab 2 - Création des contrôleurs Web et des vues pour interagir avec l'application via une interface HTTP/REST.

---

**Note Pédagogique** : Ce lab a été conçu pour favoriser l'apprentissage actif. Les parties marquées "⚠️ À COMPLÉTER" dans la version de base nécessitent votre réflexion et compréhension du code. Cette version améliorée fournit des explications complètes et des exemples fonctionnels.

**Bon apprentissage et bon développement !** 🚀

