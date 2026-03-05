# 🔧 AMÉLIORATIONS DÉTAILLÉES - Lab 1 Spring Boot
## Guide Complet d'Amélioration du Lab Pédagogique

---

# SECTION 1 : AMÉLIORATION - CONCEPTS FONDAMENTAUX (À AJOUTER)

**Localisation** : Entre "Introduction" et "Architecture du projet"

## À Ajouter : Section Nouvelle

```markdown
## Concepts Fondamentaux

### Qu'est-ce que l'Architecture MVC en Couches ?

L'architecture MVC en couches sépare l'application en 4 niveaux :

**1. Couche Présentation** (Controllers)
- Reçoit les requêtes utilisateur
- Appelle les services
- Retourne les réponses
- Role : Orchestrer les demandes

**2. Couche Service** (Business Logic)
- Contient la logique métier
- Valide les données
- Orchestre les opérations
- Role : Encapsuler la logique

**3. Couche Persistance** (Repositories)
- Accède à la base de données
- Exécute les requêtes SQL/JPQL
- Retourne les entités
- Role : Abstraire l'accès aux données

**4. Couche Modèle** (Entities)
- Représente les données métier
- Mappe les objets aux tables
- Contient les validations
- Role : Modéliser les données

### Pourquoi cette Séparation ?

| Avantage | Détail |
|----------|--------|
| **Maintenabilité** | Changer le BD sans toucher à la logique |
| **Testabilité** | Tester chaque couche isolément |
| **Réutilisabilité** | Réutiliser les services dans différents contrôleurs |
| **Scalabilité** | Ajouter facilement de nouvelles fonctionnalités |

### Exemple : Créer un Produit

```
1. L'utilisateur clique sur "Créer"
                    ↓
2. Controller reçoit la requête
                    ↓
3. Service valide les données
                    ↓
4. Service appelle le Repository
                    ↓
5. Repository insère en BD
                    ↓
6. Réponse retour à l'utilisateur
```

### Qu'est-ce que JPA ?

**JPA** = Java Persistence API = Standard Java pour l'ORM

```
Classe Java (Objet)  ←→  JPA  ←→  Table SQL (Données)
    Produit                     produit
  (nomProduit)        @Entity   (nom_produit)
  (prixProduit)       @Column   (prix_produit)
```

**JPA fait la traduction automatiquement** 🎯

### Qu'est-ce qu'un Repository ?

Un Repository est une **interface qui fournit des méthodes pour accéder à la BD**.

```java
// Sans Repository (accès direct au driver JDBC)
// ❌ Beaucoup de code, pas maintenable
Connection conn = DriverManager.getConnection(...);
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("SELECT * FROM produit");

// Avec Repository (Spring Data JPA)
// ✅ Simple, maintenable
List<Produit> produits = produitRepository.findAll();
```

**Spring Data crée automatiquement les implémentations** 🪄

### Qu'est-ce qu'une Interface Service ?

Une interface Service **force un contrat** entre le Repository et les Controllers.

```java
// L'interface définit ce que le service DOIT faire
public interface ProduitService {
    Produit saveProduit(Produit produit);
    Produit getProduitById(Long id);
    List<Produit> getAllProduits();
}

// L'implémentation COMMENT le faire
@Service
public class ProduitServiceImpl implements ProduitService {
    // Détails d'implémentation
}
```

**Avantage** : Si on change d'implémentation, le contrat ne change pas

---

# SECTION 2 : AMÉLIORATION - ANNOTATIONS JPA DÉTAILLÉES (À AJOUTER)

**Localisation** : Partie 2, avant la classe Produit

## À Insérer : Sous-section Nouvelle

```markdown
### Étape 2.1 : Comprendre les Annotations JPA

Avant d'écrire le code, comprendre chaque annotation :

#### @Entity - Marque une classe comme entité persistée

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
@Table(name = "product")  // ← Le nom de la table en BD
public class Produit {
    // ... 
}
```

**Utilité** : Si vous voulez que le nom Java ≠ nom BD

#### @Id - Désigne la clé primaire

```java
@Entity
public class Produit {
    @Id
    private Long idProduit;  // ← C'est la clé primaire
    
    // Autres champs
}
```

**Règles** :
- Chaque entité DOIT avoir un @Id
- L'@Id doit être unique
- Généralement de type Long ou String

#### @GeneratedValue - Auto-incrémentation

```java
@Entity
public class Produit {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long idProduit;  // ← 1, 2, 3, 4... (auto)
}
```

**Stratégies** :
- `IDENTITY` : Laisse la BD générer (MySQL auto-increment)
- `SEQUENCE` : Utilise une séquence (PostgreSQL)
- `UUID` : Identifiant unique global (UUID)
- `AUTO` : JPA choisit la meilleure (par défaut)

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
    @Temporal(TemporalType.DATE)  // Stocke uniquement la date
    private LocalDate dateCreation;
    
    @Temporal(TemporalType.TIMESTAMP)  // Date + heure
    private LocalDateTime lastModified;
}
```

#### @PrePersist et @PostPersist - Hooks du Cycle de Vie

```java
@Entity
public class Produit {
    @Temporal(TemporalType.TIMESTAMP)
    private LocalDateTime createdAt;
    
    @PrePersist  // ← Exécuté juste avant l'insertion en BD
    public void beforeSave() {
        this.createdAt = LocalDateTime.now();
    }
    
    @PostPersist  // ← Exécuté juste après l'insertion
    public void afterSave() {
        System.out.println("✅ Produit sauvegardé !");
    }
}
```

**Cycle de Vie JPA** :
```
Création          @PrePersist      Sauvegarde       @PostPersist
  de l'objet   →      ↓      →    en BD        →     (logs, etc.)
```

### Résumé des Annotations

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
```

---

# SECTION 3 : AMÉLIORATION - REPOSITORY PROGRESSIVE (À REMPLACER)

**Localisation** : Partie 3 - Remplacer par une progression

## Code Actuel ❌

```markdown
### Étape 3.1 : Créer le Repository

```java
public interface ProduitRepository extends JpaRepository<Produit, Long> {
    
    @Query("SELECT p FROM Produit p WHERE p.prixProduit BETWEEN ?1 AND ?2")
    List<Produit> findByPriceRange(Double minPrice, Double maxPrice);
    
    // Autres méthodes...
}
```
```

## Code Amélioré ✅

```markdown
### Étape 3.1 : Comprendre JpaRepository

Avant d'écrire le Repository, regardez cet héritage :

```
          Repository (racine)
               ↑
          CrudRepository (CRUD de base)
               ↑
         JpaRepository (additions JPA)
```

**Ce que vous hérité automatiquement** :

| Méthode | Rôle |
|---------|------|
| `save(T)` | Insérer ou mettre à jour |
| `findById(ID)` | Récupérer par clé primaire |
| `findAll()` | Récupérer tous les enregistrements |
| `deleteById(ID)` | Supprimer par clé primaire |
| `count()` | Compter les enregistrements |
| `exists()` | Vérifier l'existence |

**Vous n'avez rien à implémenter !** Spring Data le fait pour vous.

### Étape 3.2 : Créer votre premier Repository

```java
package tn.iset.produits.repositories;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import tn.iset.produits.entities.Produit;

/**
 * Repository pour Produit
 * 
 * JpaRepository<Produit, Long> signifie :
 * - Gérer les entités Produit
 * - La clé primaire est de type Long
 */
@Repository
public interface ProduitRepository extends JpaRepository<Produit, Long> {
    
    // Pour le moment, rien à ajouter !
    // Les méthodes CRUD sont héritées automatiquement
}
```

**C'est tout ce dont vous avez besoin pour faire CRUD !**

### Étape 3.3 : Tester votre Repository

Avant d'ajouter des méthodes, testez ce qui est déjà là :

```java
@SpringBootTest
class ProduitRepositoryTest {
    
    @Autowired
    ProduitRepository repository;
    
    @Test
    void testSave() {
        // CREATE
        Produit p = new Produit("Laptop", 1500.0, LocalDate.now());
        Produit saved = repository.save(p);
        assertThat(saved.getId()).isNotNull();
    }
    
    @Test
    void testFindById() {
        // READ
        Optional<Produit> p = repository.findById(1L);
        assertThat(p).isPresent();
    }
    
    @Test
    void testFindAll() {
        // READ ALL
        List<Produit> produits = repository.findAll();
        assertThat(produits).isNotEmpty();
    }
    
    @Test
    void testDelete() {
        // DELETE
        repository.deleteById(1L);
        assertThat(repository.existsById(1L)).isFalse();
    }
}
```

### Étape 3.4 : Ajouter des Méthodes Personnalisées

Maintenant que CRUD fonctionne, ajoutez de la recherche :

```java
@Repository
public interface ProduitRepository extends JpaRepository<Produit, Long> {
    
    // Spring Data génère le SQL automatiquement !
    
    // Recherche par prix exact
    List<Produit> findByPrixProduit(Double prix);
    
    // Recherche par plage de prix
    List<Produit> findByPrixProduitGreaterThanEqual(Double minPrix);
    List<Produit> findByPrixProduitLessThanEqual(Double maxPrix);
    List<Produit> findByPrixProduitBetween(Double min, Double max);
    
    // Recherche par nom (contient)
    List<Produit> findByNomProduitContainingIgnoreCase(String keyword);
}
```

**🪄 Spring Data traduit automatiquement** :
- `findByPrixProduit(1000)` → `SELECT * FROM produit WHERE prix_produit = 1000`
- `findByPrixProduitBetween(1000, 2000)` → `SELECT * FROM produit WHERE prix_produit BETWEEN 1000 AND 2000`

### Étape 3.5 : Requêtes Personnalisées avec @Query

Quand les méthodes générées ne suffisent pas, utilisez `@Query` :

```java
@Repository
public interface ProduitRepository extends JpaRepository<Produit, Long> {
    
    // Requête JPQL (Language indépendant de la BD)
    @Query("SELECT p FROM Produit p WHERE p.prixProduit > :minPrix ORDER BY p.prixProduit ASC")
    List<Produit> findProduitsPlusChers(@Param("minPrix") Double prix);
    
    // Requête SQL native (spécifique à la BD)
    @Query(value = "SELECT * FROM produit WHERE nom_produit LIKE %:keyword%", nativeQuery = true)
    List<Produit> searchByKeyword(@Param("keyword") String keyword);
}
```

**JPQL vs SQL** :
- **JPQL** : Parler d'objets Java (`Produit`, `prixProduit`)
- **SQL** : Parler de tables/colonnes (`produit`, `prix_produit`)

Préférez JPQL (plus portable).

### Checkpoint #3 : Avant de continuer...

- [ ] J'ai créé ProduitRepository extends JpaRepository<Produit, Long>
- [ ] Je comprends que les méthodes CRUD sont héritées
- [ ] Je sais comment utiliser save(), findAll(), deleteById()
- [ ] Je comprends comment Spring Data génère le SQL

Si une case n'est pas cochée, relisez la section 3.
```

---

# SECTION 4 : AMÉLIORATION - TESTS PROGRESSIFS (À REMPLACER)

**Localisation** : Partie 6

## Contenu à Remplacer

```markdown
## Partie 6 : Tests Unitaires avec JUnit 5

### Étape 6.1 : Créer la Classe de Test

Créez `src/test/java/tn/iset/produits/services/ProduitServiceTest.java` :

```java
package tn.iset.produits.services;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import tn.iset.produits.entities.Produit;

import java.time.LocalDate;

import static org.assertj.core.api.Assertions.*;

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
    
    /**
     * setUp() est exécuté AVANT chaque test
     * Utile pour initialiser les données
     */
    @BeforeEach
    void setUp() {
        // Facultatif : nettoyer la BD avant chaque test
        // produitRepository.deleteAll();
    }
    
    // ============================================
    // CAS NORMAL : Happy Path
    // ============================================
    
    /**
     * Test: Créer un produit valide
     * Donné : Un produit avec tous les champs
     * Quand : On l'enregistre
     * Alors : Il est sauvegardé avec un ID
     */
    @Test
    void testSaveProduitSuccess() {
        // GIVEN (Donné)
        Produit produit = new Produit(
            "PC Portable",
            1850.00,
            LocalDate.now()
        );
        
        // WHEN (Quand)
        Produit saved = produitService.saveProduit(produit);
        
        // THEN (Alors)
        assertThat(saved).isNotNull();
        assertThat(saved.getId()).isGreaterThan(0);
        assertThat(saved.getNomProduit()).isEqualTo("PC Portable");
    }
    
    /**
     * Test : Récupérer un produit
     */
    @Test
    void testGetProduitById() {
        // GIVEN
        Produit saved = produitService.saveProduit(
            new Produit("PC", 1000.0, LocalDate.now())
        );
        
        // WHEN
        Produit found = produitService.getProduitById(saved.getId());
        
        // THEN
        assertThat(found).isNotNull();
        assertThat(found.getId()).isEqualTo(saved.getId());
    }
    
    /**
     * Test : Mettre à jour un produit
     */
    @Test
    void testUpdateProduit() {
        // GIVEN
        Produit saved = produitService.saveProduit(
            new Produit("PC Old", 1000.0, LocalDate.now())
        );
        
        // WHEN
        saved.setNomProduit("PC New");
        Produit updated = produitService.saveProduit(saved);
        
        // THEN
        assertThat(updated.getNomProduit()).isEqualTo("PC New");
    }
    
    /**
     * Test : Supprimer un produit
     */
    @Test
    void testDeleteProduit() {
        // GIVEN
        Produit saved = produitService.saveProduit(
            new Produit("PC", 1000.0, LocalDate.now())
        );
        
        // WHEN
        produitService.deleteProduit(saved.getId());
        
        // THEN
        assertThrows(RuntimeException.class, () -> {
            produitService.getProduitById(saved.getId());
        });
    }
    
    // ============================================
    // CAS D'ERREUR : Gestion Exceptions
    // ============================================
    
    /**
     * Test : Sauvegarder un produit NULL
     * Attendu : Levée d'exception
     */
    @Test
    void testSaveProduitNull() {
        // WHEN/THEN
        assertThrows(IllegalArgumentException.class, () -> {
            produitService.saveProduit(null);
        }, "Doit lever IllegalArgumentException");
    }
    
    /**
     * Test : Récupérer un produit inexistant
     */
    @Test
    void testGetProduitNotFound() {
        // WHEN/THEN
        assertThrows(RuntimeException.class, () -> {
            produitService.getProduitById(999L);
        }, "Doit lever RuntimeException pour ID inexistant");
    }
    
    /**
     * Test : Sauvegarder un produit avec prix zéro
     * Cas limites : Edge case important
     */
    @Test
    void testSaveProduitWithZeroPrice() {
        // GIVEN
        Produit produit = new Produit("PC", 0.0, LocalDate.now());
        
        // WHEN/THEN
        assertThrows(IllegalArgumentException.class, () -> {
            produitService.saveProduit(produit);
        }, "Prix ne peut pas être zéro");
    }
    
    /**
     * Test : Sauvegarder produit avec prix négatif
     */
    @Test
    void testSaveProduitWithNegativePrice() {
        // GIVEN
        Produit produit = new Produit("PC", -100.0, LocalDate.now());
        
        // WHEN/THEN
        assertThrows(IllegalArgumentException.class, () -> {
            produitService.saveProduit(produit);
        }, "Prix ne peut pas être négatif");
    }
    
    /**
     * Test : Sauvegarder produit avec nom vide
     */
    @Test
    void testSaveProduitWithBlankName() {
        // GIVEN
        Produit produit = new Produit("   ", 1000.0, LocalDate.now());
        
        // WHEN/THEN
        assertThrows(IllegalArgumentException.class, () -> {
            produitService.saveProduit(produit);
        }, "Nom ne peut pas être vide");
    }
    
    // ============================================
    // STATISTIQUES
    // ============================================
    
    @Test
    void testCountProduits() {
        // GIVEN
        long initialCount = produitService.countProduits();
        produitService.saveProduit(
            new Produit("PC", 1000.0, LocalDate.now())
        );
        
        // WHEN
        long newCount = produitService.countProduits();
        
        // THEN
        assertThat(newCount).isEqualTo(initialCount + 1);
    }
    
    @Test
    void testGetAllProduits() {
        // WHEN
        var produits = produitService.getAllProduits();
        
        // THEN
        assertThat(produits)
            .isNotEmpty()
            .hasSizeGreaterThanOrEqualTo(1);
    }
}
```

### Étape 6.2 : Exécuter les Tests

1. Clic droit sur la classe → **Run 'ProduitServiceTest'**
2. Ou appuyez sur **Shift+Ctrl+F10** (Windows/Linux)
3. Ou **Shift+Cmd+R** (Mac)

**Résultat attendu** :
```
✅ testSaveProduitSuccess
✅ testGetProduitById
✅ testUpdateProduit
✅ testDeleteProduit
✅ testSaveProduitNull
✅ testGetProduitNotFound
✅ testSaveProduitWithZeroPrice
✅ testSaveProduitWithNegativePrice
✅ testSaveProduitWithBlankName
✅ testCountProduits
✅ testGetAllProduits

Tests passés: 11/11 ✅
```

### Étape 6.3 : Analyser les Résultats

Chaque test vous enseigne quelque chose :

| Test | Enseigne |
|------|----------|
| testSaveProduitSuccess | Les cas normaux fonctionnent |
| testSaveProduitNull | La validation existe |
| testSaveProduitWithZeroPrice | Les validations métier marchent |
| testCountProduits | Les statistiques sont correctes |

### Étape 6.4 : Couvrir Plus de Cas

Ajoutez des tests pour :

```java
// Test de concurrence (2 créations simultanées)
@Test
void testConcurrentSave() throws InterruptedException {
    Thread t1 = new Thread(() -> 
        produitService.saveProduit(new Produit("PC1", 1000, LocalDate.now()))
    );
    Thread t2 = new Thread(() -> 
        produitService.saveProduit(new Produit("PC2", 2000, LocalDate.now()))
    );
    
    t1.start();
    t2.start();
    t1.join();
    t2.join();
    
    assertThat(produitService.countProduits()).isGreaterThanOrEqualTo(2);
}

// Test de performance
@Test
void testPerformance() {
    long start = System.currentTimeMillis();
    
    for (int i = 0; i < 100; i++) {
        produitService.saveProduit(
            new Produit("PC" + i, 1000.0, LocalDate.now())
        );
    }
    
    long duration = System.currentTimeMillis() - start;
    assertThat(duration).isLessThan(5000); // Moins de 5 secondes
}
```

### ✅ Checkpoint #6 : Tests

- [ ] Tous les tests passent
- [ ] Je comprends la structure AAA (Arrange-Act-Assert)
- [ ] Je sais écrire un test pour un cas normal
- [ ] Je sais écrire un test pour une exception
- [ ] Je comprends pourquoi les tests sont importants

Si une case n'est pas cochée, relisez la section 6.
```

---

# SECTION 5 : AMÉLIORATION - SCRIPT MySQL

**À Ajouter** : Nouvelle section dans Partie 5

```markdown
### Étape 5.1 : Script de Création BD

Avant de lancer l'application, créez la BD et l'utilisateur :

#### Option 1 : En ligne de commande MySQL

```bash
# Ouvrir MySQL
mysql -u root -p

# Exécuter les commandes
```

Puis dans MySQL :

```sql
-- 1. Créer la base de données
CREATE DATABASE spring_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 2. Créer l'utilisateur
CREATE USER 'springuser'@'localhost' IDENTIFIED BY 'springpass';

-- 3. Donner les permissions
GRANT ALL PRIVILEGES ON spring_db.* TO 'springuser'@'localhost';
FLUSH PRIVILEGES;

-- 4. Vérifier
SHOW GRANTS FOR 'springuser'@'localhost';
```

#### Option 2 : Via un fichier SQL (Recommandé)

Créez `src/main/resources/init.sql` :

```sql
-- Script d'initialisation de la base de données
-- Exécutez ceci avant de lancer l'application

CREATE DATABASE IF NOT EXISTS spring_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE USER IF NOT EXISTS 'springuser'@'localhost' IDENTIFIED BY 'springpass';

GRANT ALL PRIVILEGES ON spring_db.* TO 'springuser'@'localhost';

FLUSH PRIVILEGES;

-- Montrer que c'est prêt
SHOW DATABASES LIKE 'spring_db';
```

Puis :

```bash
mysql -u root -p < init.sql
```

#### Option 3 : Utiliser Docker (Plus Facile!)

```bash
# Créer un conteneur MySQL
docker run -d \
  --name mysql-spring \
  -e MYSQL_DATABASE=spring_db \
  -e MYSQL_USER=springuser \
  -e MYSQL_PASSWORD=springpass \
  -e MYSQL_ROOT_PASSWORD=root \
  -p 3306:3306 \
  mysql:8.0

# Vérifier que c'est running
docker ps

# Accéder à MySQL
mysql -h localhost -u springuser -p spring_db
# Password: springpass
```

**Je recommande Docker !** ✅

### Étape 5.2 : Vérifier la Connexion

Dans IntelliJ, ouvrez **Database**:

1. **View** → **Tool Windows** → **Database**
2. Cliquez sur le **+** (Ajouter source)
3. **MySQL**
4. Remplissez :
   - **Host** : `localhost`
   - **Port** : `3306`
   - **User** : `springuser`
   - **Password** : `springpass`
   - **Database** : `spring_db`
5. **Test Connection**

✅ Si c'est vert, c'est bon !
```

---

# SECTION 6 : PIÈGES COURANTS (À AJOUTER)

**À Ajouter** : Nouvelle section avant "Exercices d'évaluation"

```markdown
## ⚠️ Pièges Courants et Comment les Éviter

### Piège 1️⃣ : Oublier @Entity

❌ **Erreur Courante** :
```java
public class Produit {  // ← Pas d'annotation !
    private Long id;
}
```

**Symptôme** :
```
Unknown entity: tn.iset.produits.entities.Produit
```

✅ **Correction** :
```java
@Entity  // ← Ajouter cette annotation
public class Produit {
    @Id
    private Long id;
}
```

---

### Piège 2️⃣ : Repository sans Paramètres de Type

❌ **Erreur Courante** :
```java
public interface ProduitRepository extends JpaRepository {
    // ← Pas de <Produit, Long>
}
```

✅ **Correction** :
```java
public interface ProduitRepository extends JpaRepository<Produit, Long> {
    //                                        ↑               ↑
    //                                    Type Entité    Type ID
}
```

---

### Piège 3️⃣ : Service sans @Service

❌ **Erreur Courante** :
```java
public class ProduitServiceImpl implements ProduitService {
    // ← Pas d'annotation
}
```

**Symptôme** :
```
No qualifying bean of type 'ProduitService' found
```

✅ **Correction** :
```java
@Service
public class ProduitServiceImpl implements ProduitService {
    // ← Spring crée maintenant le bean
}
```

---

### Piège 4️⃣ : @Autowired sans setters/constructeur

❌ **Erreur Courante** :
```java
@Service
public class ProduitServiceImpl {
    
    @Autowired
    private ProduitRepository repository;
    
    // ← Pas de constructeur ou setter !
}
```

✅ **Correction Moderns (Avec Lombok)** :
```java
@Service
@RequiredArgsConstructor  // ← Génère un constructeur
public class ProduitServiceImpl {
    
    private final ProduitRepository repository;  // ← final obligatoire
}
```

---

### Piège 5️⃣ : Oublier @Transactional pour les modifications

❌ **Erreur Courante** :
```java
@Service
public class ProduitServiceImpl {
    
    public void saveProduit(Produit p) {  // ← Pas d'annotation
        repository.save(p);
    }
}
```

**Problème** : Les modifications ne sont pas "committées"

✅ **Correction** :
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

### Piège 6️⃣ : Paramètres @Query mal nommés

❌ **Erreur Courante** :
```java
@Query("SELECT p FROM Produit p WHERE p.prixProduit > :minPrice")
List<Produit> findByPrice(@Param("minPrix") Double prix);
                          // ↑ Noms différents !
```

**Symptôme** :
```
Parameter binding error: ... Parameter 'minPrice' not found
```

✅ **Correction** :
```java
@Query("SELECT p FROM Produit p WHERE p.prixProduit > :minPrix")
List<Produit> findByPrice(@Param("minPrix") Double prix);
                          // ↑ Même nom !
```

---

### Piège 7️⃣ : Oublier la Clé Primaire

❌ **Erreur Courante** :
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

✅ **Correction** :
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

### Piège 8️⃣ : Connection timeout BD

❌ **Symptôme** :
```
Communications link failure
The last packet sent successfully to the server was 0 milliseconds ago
```

✅ **Solutions** :
1. Vérifier que MySQL est running : `docker ps`
2. Vérifier les crédentiales dans `application.properties`
3. Vérifier le port (par défaut 3306)

```bash
# Tester la connexion
mysql -h localhost -u springuser -p -e "USE spring_db; SHOW TABLES;"
```

---

### Piège 9️⃣ : NullPointerException dans les Tests

❌ **Erreur Courante** :
```java
@Test
void testSave() {
    Produit p = new Produit("PC", 1000, LocalDate.now());
    // repository n'est pas @Autowired !
    repository.save(p);  // ← NPE ici
}
```

✅ **Correction** :
```java
@SpringBootTest  // ← Important !
class ProduitServiceTest {
    
    @Autowired
    ProduitService produitService;
    
    @Test
    void testSave() {
        Produit p = new Produit("PC", 1000, LocalDate.now());
        produitService.saveProduit(p);  // ← Marche maintenant
    }
}
```

---

### Piège 🔟 : Hibernate crée les tables MAIS ne les rempli pas

❌ **Problème** :
```
Table créée ✅
Mais aucune donnée ❌
```

**Raison** : `@PrePersist` n'est pas appelé automatiquement

✅ **Solution** : Utiliser `CommandLineRunner` (voir Partie 7)
```

---

# SECTION 7 : GLOSSAIRE (À AJOUTER)

```markdown
## 📚 Glossaire

### A

**Annotation** : Marque (décorator) qui ajoute de la metadata au code
- Exemple : `@Entity`, `@Service`, `@Transactional`

**Assertion** : Vérification dans un test (assertTrue, assertEqual, etc.)

**Auto-increment** : L'ID s'incrémente automatiquement (1, 2, 3...)

### B

**Bean** : Objet géré par Spring (créé, injecté, détruit par Spring)

**Builder Pattern** : Pattern créationnel pour créer des objets (Produit.builder())

### C

**CRUD** : Create, Read, Update, Delete (4 opérations basiques)

**CrudRepository** : Interface avec CRUD de base

### D

**DAO** : Data Access Object (ancienne terminologie pour Repository)

**DTO** : Data Transfer Object (objet pour transférer les données)

**Database** : Base de données (collection de tables)

### E

**Entity** : Classe Java mappée à une table BD (@Entity)

**Enum** : Type avec valeurs prédéfinies (GenerationType.IDENTITY)

### H

**Hibernate** : ORM qui implémente JPA

**Hook** : Méthode appelée à un moment précis (@PrePersist, @PostPersist)

### I

**ID** : Identifiant unique (clé primaire)

**Injection de Dépendance** : Spring crée et injecte les beans

**Interface** : Contrat que doit implémenter une classe

### J

**JPA** : Java Persistence API (standard ORM Java)

**JUnit** : Framework de test Java (JUnit 5 moderne)

**JPQL** : Java Persistence Query Language (requêtes orientées objets)

### M

**Mapping** : Correspondance entre classe Java et table BD

**Mock** : Objet bidon pour tester sans dépendances réelles

### O

**ORM** : Object-Relational Mapping (mappe objets aux tables)

### P

**PK** : Primary Key (clé primaire)

**Polymorphisme** : Une interface, plusieurs implémentations

### Q

**Query** : Requête BD (SELECT, INSERT, UPDATE, DELETE)

### R

**Repository** : Interface pour accéder aux données

**Reflection** : Capacité de Java à examiner/modifier le code au runtime

### S

**Schema** : Structure de la base de données (tables, colonnes)

**Service** : Classe avec logique métier (@Service)

**SQL** : Structured Query Language (langage pour BD relationnelles)

**Spring** : Framework Java pour applications

### T

**Table** : Collection de lignes/colonnes en BD

**Transaction** : Opération atomique en BD (Commit/Rollback)

**Test** : Code qui vérifie que le code principal fonctionne

### U

**Unit Test** : Test d'une unité de code isolée

### V

**Validation** : Vérifier que les données sont valides

### W

**Web** : Application accessible via HTTP

---

## Mots-clés à Retenir

| Terme | Exemple | Mémoriser |
|-------|---------|-----------|
| **Entity** | `@Entity public class Produit` | Objet → Table |
| **Repository** | `extends JpaRepository<Produit, Long>` | Accès données |
| **Service** | `@Service class ProduitServiceImpl` | Logique métier |
| **Transactional** | `@Transactional` | Modifier données |
| **Autowired** | `@Autowired ProduitRepository` | Injecter |
```

---

# SECTION 8 : EVALUATION FINALE (À AJOUTER)

```markdown
## 🎯 Évaluation Finale - Lab 1 Complet

### Durée Estimée : 2-3 heures

### Partie A : Questions Théoriques (30-45 min)

**Question 1** (5 points) :
Expliquez pourquoi la couche Service est importante même si le Repository 
peut faire CRUD directement. Donnez un exemple concret.

**Réponse Attendue** :
- Service encapsule la logique métier
- Service peut faire plusieurs opérations (validation + audit + logging)
- Service découple Controller de Repository
- Exemple : saveProduit() valide avant de sauvegarder

---

**Question 2** (5 points) :
Quelle est la différence entre JPA et SQL ? Quand utiliser chacun ?

**Réponse Attendue** :
- JPA = Orienté objet (Produit, prixProduit)
- SQL = Orienté table (produit, prix_produit)
- JPA pour requêtes simples/standard
- SQL quand JPA ne suffit pas (@Query native)

---

**Question 3** (5 points) :
Quel est le rôle du builder pattern dans Produit ?

**Réponse Attendue** :
- Créer des objets avec beaucoup de champs facilement
- Évite les constructeurs avec beaucoup de paramètres
- Produit.builder().nomProduit(...).prixProduit(...).build()

---

**Question 4** (5 points) :
Pourquoi les tests sont-ils importants ? Donnez 2 avantages.

**Réponse Attendue** :
- Vérifier que le code fonctionne (régression testing)
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
    
    @Column(length = 100, nullable = false)
    private String nameCategory;
    
    @OneToMany(mappedBy = "category", cascade = CascadeType.ALL)
    private List<Produit> produits = new ArrayList<>();
}
```

#### Étape 2 : Mettre à Jour Produit

```java
@Entity
@Table(name = "produit")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Produit {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long idProduit;
    
    // ... autres champs ...
    
    @ManyToOne
    @JoinColumn(name = "id_category")
    private Category category;  // ← Ajouter ça
}
```

#### Étape 3 : Créer CategoryRepository

Créez `CategoryRepository` avec des requêtes utiles.

#### Étape 4 : Créer Service + Interface

```java
public interface CategoryService {
    Category saveCategory(Category category);
    Category getCategoryById(Long id);
    List<Category> getAllCategories();
    // ... etc
}
```

#### Étape 5 : Tests

```java
@SpringBootTest
class CategoryServiceTest {
    // Écrire au moins 5 tests
}
```

**Critères d'Évaluation** :
- ✅ Entity correcte (30 points)
- ✅ Repository complet (20 points)
- ✅ Service + Interface (20 points)
- ✅ Tests passants (20 points)
- ✅ Code lisible (10 points)

### Partie C : Documentation & Réflexion (30-45 min)

Écrivez un document (1-2 pages) contenant :

1. **Architecture** : Décrire votre système Category (1 diagramme)
2. **Choix de Design** : Pourquoi Many-to-One et pas Many-to-Many ?
3. **Améliorations** : 3 choses que vous pourriez ajouter
4. **Apprentissages** : Ce que vous avez appris de ce lab

**Exemple de réponse** :

```
## Réflexion Lab 1

### 1. Architecture

Category --1--↑
               |
            Many-to-One
               |
          ---+--
         /         \
    Produit 1    Produit 2

### 2. Pourquoi Many-to-One ?

Un produit n'a qu'une catégorie (PC portable, pas PC + Téléphone)
Une catégorie peut avoir plusieurs produits

Si c'était Many-to-Many :
- Un produit pourrait être dans plusieurs catégories
- Plus complexe (table de jointure)

### 3. Améliorations Futures

1. Ajouter Sous-Catégories (hiérarchie)
2. Ajouter des Promotions par Catégorie
3. Calculer le prix moyen par catégorie (statistiques)

### 4. Apprentissages

- J'ai compris les relations JPA
- Les tests me donnent confiance
- Le pattern Service est vraiment utile
```

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
```

---

# RÉSUMÉ GLOBAL DES AMÉLIORATIONS

| Section | Amélioration | Impact |
|---------|-------------|--------|
| Concepts Fondamentaux | + 300 lignes | Architecture expliquée |
| Annotations JPA | + 200 lignes | Compréhension JPA |
| Repository Progressif | + 250 lignes | Approche graduelle |
| Tests Progressifs | + 400 lignes | Tests complets |
| Script MySQL | + 100 lignes | Prêt à l'emploi |
| Pièges Courants | + 300 lignes | Éviter erreurs |
| Glossaire | + 200 lignes | Clarté terminologie |
| Évaluation Finale | + 250 lignes | Test compétences |
| **TOTAL** | **~2000 lignes** | **Lab complet** |

---

**Ces améliorations transforment le lab d'une base solide à un excellent outil pédagogique !** ✨

