# 🎓 LAB 2 - RELATIONS ONETOMANY ET REQUÊTES AVANCÉES JPQL
## Gestion des Produits et Catégories avec Spring Data JPA

---

## 📋 TABLE DES MATIÈRES

1. [Objectifs Pédagogiques](#objectifs-pédagogiques)
2. [Structure du Projet](#structure-du-projet)
3. [Ressources Fournies](#ressources-fournies)
4. [Partie 1 : Entité Catégorie](#partie-1--entité-catégorie)
5. [Partie 2 : Relation ManyToOne dans Produit](#partie-2--relation-manytoone-dans-produit)
6. [Partie 3 : Repository Catégorie](#partie-3--repository-catégorie)
7. [Partie 4 : Repository Produit Avancé](#partie-4--repository-produit-avancé)
8. [Partie 5 : Service Catégorie](#partie-5--service-catégorie)
9. [Partie 6 : Service Produit Enrichi](#partie-6--service-produit-enrichi)
10. [Partie 7 : Tests Unitaires](#partie-7--tests-unitaires)
11. [Partie 8 : Initialisation et Affichage](#partie-8--initialisation-et-affichage)
12. [Points Clés à Retenir](#points-clés-à-retenir)


---

## OBJECTIFS PÉDAGOGIQUES

À la fin de ce lab, vous serez capable de :

✅ **Modéliser** les relations OneToMany et ManyToOne  
✅ **Créer** deux entités liées avec les bonnes annotations  
✅ **Écrire** des requêtes JPQL avancées avec @Query  
✅ **Interroger** les données par id de relation  
✅ **Trier** les données de manière simple et complexe  
✅ **Organiser** le code en couches Service/Repository  
✅ **Tester** les fonctionnalités avec JUnit 5  
✅ **Appliquer** les bonnes pratiques Spring Data

---

## STRUCTURE DU PROJET

Vous devez organiser votre projet selon cette structure :

```
produits-management-lab2/
│
├── src/main/java/tn/iset/produits/
│   ├── entities/
│   │   ├── Produit.java              (EXISTANT du Lab 1, à modifier)
│   │   └── Categorie.java            (NOUVEAU)
│   │
│   ├── repositories/
│   │   ├── ProduitRepository.java    (EXISTANT, à enrichir)
│   │   └── CategorieRepository.java  (NOUVEAU)
│   │
│   ├── services/
│   │   ├── ProduitService.java       (EXISTANT, à enrichir)
│   │   ├── ProduitServiceImpl.java    (EXISTANT, à enrichir)
│   │   ├── CategorieService.java     (NOUVEAU)
│   │   └── CategorieServiceImpl.java  (NOUVEAU)
│   │
│   └── ProduitsApplication.java      (À modifier pour initialiser les catégories)
│
├── src/test/java/tn/iset/produits/
│   └── services/
│       ├── ProduitServiceTest.java    (EXISTANT, à enrichir)
│       └── CategorieServiceTest.java  (NOUVEAU)
│
├── src/main/resources/
│   └── application.properties
│
└── pom.xml
```

---

## RESSOURCES FOURNIES

Vous recevez :

📁 **Code source du Lab 1 (Entité Produit, Repository, Service)

📁 **Votre Lab 2** : Énoncé complet avec toutes les étapes

**À votre charge** : Implémenter chaque partie selon les instructions

---

---

# PARTIE 1 : ENTITÉ CATÉGORIE

## 🎯 Objectif

Créer une entité JPA pour les catégories avec une relation OneToMany vers les produits.

## 📌 Diagramme de la Relation

```
┌────────────────┐              ┌────────────────┐
│   Catégorie    │  1    *      │    Produit     │
├────────────────┤◄───────────►├────────────────┤
│ - idCat        │   OneToMany  │ - idProduit    │
│ - nomCat       │   ManyToOne  │ - nomProduit   │
│ - description  │              │ - prixProduit  │
│ - produits[]   │              │ - categorie    │
└────────────────┘              └────────────────┘
```

## 📝 Instructions

### Étape 1.1 : Créer le fichier Categorie.java

**Chemin** : `src/main/java/tn/iset/produits/entities/Categorie.java`

**À faire** :

```java
package tn.iset.produits.entities;

import jakarta.persistence.*;
import jakarta.validation.constraints.*;
import lombok.*;

import java.util.ArrayList;
import java.util.List;

/**
 * Entité Catégorie
 * 
 * Représente une catégorie de produits
 * Relation : Une catégorie contient plusieurs produits
 */
@Entity
@Table(name = "categorie")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Categorie {
    
    // ========== ATTRIBUTS ==========
    
    /**
     * Clé primaire auto-incrémentée
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id_cat")
    private Long idCat;
    
    /**
     * Nom de la catégorie
     * Validations :
     * - @NotBlank : Obligatoire
     * - @Size : Entre 3 et 100 caractères
     */
    @NotBlank(message = "Le nom de la catégorie est obligatoire")
    @Size(min = 3, max = 100, message = "Le nom doit contenir entre 3 et 100 caractères")
    @Column(name = "nom_cat", nullable = false, length = 100)
    private String nomCat;
    
    /**
     * Description de la catégorie
     */
    @Column(name = "description_cat", length = 255)
    private String descriptionCat;
    
    /**
     * Relation OneToMany avec Produit
     * 
     * mappedBy = "categorie" :
     * - Indique que le côté Produit gère la relation (avec @ManyToOne)
     * - Cette liste ne crée pas de colonne en BD
     * 
     * cascade = CascadeType.ALL :
     * - Si on supprime une catégorie, les produits sont supprimés
     * 
     * orphanRemoval = true :
     * - Si on retire un produit de la liste, il est supprimé
     * 
     * = new ArrayList<>() :
     * - Initialisation pour éviter NullPointerException
     */
    @OneToMany(
        mappedBy = "categorie",
        cascade = CascadeType.ALL,
        orphanRemoval = true
    )
    private List<Produit> produits = new ArrayList<>();
    
    // ========== MÉTHODES UTILITAIRES ==========
    
    /**
     * Ajouter un produit à cette catégorie
     */
    public void addProduit(Produit produit) {
        this.produits.add(produit);
        produit.setCategorie(this);
    }
    
    /**
     * Retirer un produit de cette catégorie
     */
    public void removeProduit(Produit produit) {
        this.produits.remove(produit);
        produit.setCategorie(null);
    }
    
    @Override
    public String toString() {
        return String.format(
            "Categorie{id=%d, nom='%s', produits=%d}",
            idCat, nomCat, produits.size()
        );
    }
}
```

## ✅ CHECKPOINT 1.1

**Vérifiez que** :
- [ ] La classe Categorie est créée dans `entities/`
- [ ] Toutes les annotations JPA sont présentes (@Entity, @Id, @GeneratedValue)
- [ ] Les validations sont appliquées (@NotBlank, @Size)
- [ ] La relation @OneToMany est correcte
- [ ] La classe compile sans erreurs

---

# PARTIE 2 : RELATION MANYTOONE DANS PRODUIT

## 🎯 Objectif

Ajouter la relation ManyToOne dans l'entité Produit (récupérée du Lab 1).

## 📝 Instructions

### Étape 2.1 : Modifier Produit.java

**Chemin** : `src/main/java/tn/iset/produits/entities/Produit.java`

**Ajouter** la relation ManyToOne **après les attributs existants** :

```java
// ========== RELATION MANYTOONE (À AJOUTER) ==========

/**
 * Relation ManyToOne avec Catégorie
 * 
 * @ManyToOne :
 * - Plusieurs produits pour une catégorie
 * - Cette entité possède la clé étrangère (colonne id_cat)
 * 
 * @JoinColumn(name = "id_cat") :
 * - Crée la colonne id_cat en BD (clé étrangère)
 * 
 * fetch = FetchType.EAGER :
 * - Charge la catégorie automatiquement quand on charge un produit
 */
@ManyToOne(fetch = FetchType.EAGER)
@JoinColumn(name = "id_cat")
private Categorie categorie;

// ========== FIN DE LA RELATION ==========
```

**Important** : 
- Ajoutez aussi le getter/setter pour `categorie` (Lombok le génère avec @Data)
- Modifiez le `toString()` pour afficher la catégorie

```java
@Override
public String toString() {
    return String.format(
        "Produit{id=%d, nom='%s', prix=%.2f, categorie=%s}",
        idProduit, nomProduit, prixProduit,
        categorie != null ? categorie.getNomCat() : "SANS CATÉGORIE"
    );
}
```

## ✅ CHECKPOINT 2.1

**Vérifiez que** :
- [ ] L'attribut categorie est ajouté dans Produit
- [ ] L'annotation @ManyToOne est présente
- [ ] @JoinColumn(name = "id_cat") est correct
- [ ] Le getter/setter sont générés par Lombok
- [ ] Le toString() inclut la catégorie
- [ ] Produit compile sans erreurs

## 📊 Résultat en BD

Après cette étape, la migration Hibernate crée :

```sql
-- Table Catégorie
CREATE TABLE categorie (
  id_cat BIGINT PRIMARY KEY AUTO_INCREMENT,
  nom_cat VARCHAR(100) NOT NULL,
  description_cat VARCHAR(255)
);

-- Table Produit modifiée
ALTER TABLE produit ADD COLUMN id_cat BIGINT;
ALTER TABLE produit ADD FOREIGN KEY (id_cat) REFERENCES categorie(id_cat);
```

---

# PARTIE 3 : REPOSITORY CATÉGORIE

## 🎯 Objectif

Créer une interface Repository pour les catégories avec des requêtes courantes.

## 📝 Instructions

### Étape 3.1 : Créer CategorieRepository.java

**Chemin** : `src/main/java/tn/iset/produits/repositories/CategorieRepository.java`

```java
package tn.iset.produits.repositories;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;
import tn.iset.produits.entities.Categorie;

import java.util.List;
import java.util.Optional;

/**
 * Repository pour l'entité Catégorie
 * 
 * Héritage automatique des méthodes CRUD :
 * - save(Categorie)
 * - findById(Long)
 * - findAll()
 * - delete(Categorie)
 * - count()
 */
@Repository
public interface CategorieRepository extends JpaRepository<Categorie, Long> {
    
    // ========== RECHERCHE PAR NOM ==========
    
    /**
     * Trouver une catégorie par son nom exact
     * 
     * Spring Data génère automatiquement :
     * SELECT * FROM categorie WHERE nom_cat = ?
     */
    Optional<Categorie> findByNomCat(String nomCat);
    
    /**
     * Trouver les catégories contenant un texte
     * 
     * Spring Data génère automatiquement :
     * SELECT * FROM categorie WHERE nom_cat LIKE %?%
     */
    List<Categorie> findByNomCatContains(String keyword);
    
    /**
     * Trouver les catégories commençant par un texte
     * 
     * Spring Data génère automatiquement :
     * SELECT * FROM categorie WHERE nom_cat LIKE ?%
     */
    List<Categorie> findByNomCatStartsWith(String prefix);
    
    // ========== TRI ==========
    
    /**
     * Trouver les catégories triées par nom (croissant)
     * 
     * Spring Data génère automatiquement :
     * SELECT * FROM categorie ORDER BY nom_cat ASC
     */
    List<Categorie> findByOrderByNomCatAsc();
    
    /**
     * Trouver les catégories triées par nom (décroissant)
     * 
     * Spring Data génère automatiquement :
     * SELECT * FROM categorie ORDER BY nom_cat DESC
     */
    List<Categorie> findByOrderByNomCatDesc();
    
    // ========== REQUÊTES PERSONNALISÉES AVEC @Query ==========
    
    /**
     * Compter les catégories ayant au moins N produits
     * 
     * JPQL - SELECT COUNT(DISTINCT c) FROM Categorie c WHERE SIZE(c.produits) >= :minProducts
     */
    @Query("SELECT COUNT(DISTINCT c) FROM Categorie c WHERE SIZE(c.produits) >= :minProduits")
    Long countCategoriesWithMinProducts(@Param("minProduits") int minProduits);
    
    /**
     * Trouver les catégories avec au moins N produits
     * 
     * JPQL - SELECT c FROM Categorie c WHERE SIZE(c.produits) > :minProduits
     */
    @Query("SELECT c FROM Categorie c WHERE SIZE(c.produits) > :minProduits")
    List<Categorie> findCategoriesWithMinProducts(@Param("minProduits") int minProduits);
}
```

## ✅ CHECKPOINT 3.1

**Vérifiez que** :
- [ ] CategorieRepository est créé dans `repositories/`
- [ ] Il étend JpaRepository<Categorie, Long>
- [ ] Au moins 5 méthodes de recherche sont implémentées
- [ ] Les annotations @Query sont correctes
- [ ] La classe compile sans erreurs

---

# PARTIE 4 : REPOSITORY PRODUIT AVANCÉ

## 🎯 Objectif

Enrichir le Repository Produit du Lab 1 avec des requêtes avancées et des recherches par catégorie.

## 📝 Instructions

### Étape 4.1 : Enrichir ProduitRepository.java

**Chemin** : `src/main/java/tn/iset/produits/repositories/ProduitRepository.java`

**Ajouter** les méthodes suivantes au Repository existant :

```java
// ========== RECHERCHE PAR CATÉGORIE (À AJOUTER) ==========

/**
 * Trouver les produits d'une catégorie donnée
 * 
 * Spring Data génère automatiquement :
 * SELECT * FROM produit WHERE id_cat = ?
 */
List<Produit> findByCategorie(Categorie categorie);

/**
 * Trouver les produits d'une catégorie par son ID
 * 
 * Spring Data supporte les chemins d'accès imbriqués :
 * SELECT * FROM produit p WHERE p.id_cat = ?
 */
List<Produit> findByCategorieIdCat(Long idCat);

/**
 * Trouver les produits d'une catégorie triés par nom
 * 
 * Spring Data génère automatiquement :
 * SELECT * FROM produit WHERE id_cat = ? ORDER BY nom_produit ASC
 */
List<Produit> findByCategorieOrderByNomProduitAsc(Categorie categorie);

// ========== REQUÊTES @QUERY AVEC CATÉGORIE (À AJOUTER) ==========

/**
 * Trouver les produits d'une catégorie avec un prix minimum
 * 
 * JPQL :
 * SELECT p FROM Produit p 
 * WHERE p.categorie = :categorie AND p.prixProduit >= :minPrix
 * ORDER BY p.prixProduit DESC
 */
@Query("SELECT p FROM Produit p " +
       "WHERE p.categorie = :categorie AND p.prixProduit >= :minPrix " +
       "ORDER BY p.prixProduit DESC")
List<Produit> findByCategorieAndMinPrice(
    @Param("categorie") Categorie categorie,
    @Param("minPrix") Double minPrix);

/**
 * Trouver les produits d'une catégorie dans une plage de prix
 * 
 * JPQL :
 * SELECT p FROM Produit p 
 * WHERE p.categorie.idCat = :idCat AND p.prixProduit BETWEEN :minPrix AND :maxPrix
 */
@Query("SELECT p FROM Produit p " +
       "WHERE p.categorie.idCat = :idCat AND p.prixProduit BETWEEN :minPrix AND :maxPrix")
List<Produit> findByCategoryAndPriceRange(
    @Param("idCat") Long idCat,
    @Param("minPrix") Double minPrix,
    @Param("maxPrix") Double maxPrix);

/**
 * Compter les produits d'une catégorie
 * 
 * JPQL :
 * SELECT COUNT(p) FROM Produit p WHERE p.categorie.idCat = :idCat
 */
@Query("SELECT COUNT(p) FROM Produit p WHERE p.categorie.idCat = :idCat")
Long countByCategory(@Param("idCat") Long idCat);

// ========== TRI AVANCÉ (À AJOUTER) ==========

/**
 * Trier les produits par catégorie, puis par nom, puis par prix
 * 
 * JPQL :
 * SELECT p FROM Produit p 
 * ORDER BY p.categorie.nomCat ASC, p.nomProduit ASC, p.prixProduit DESC
 */
@Query("SELECT p FROM Produit p " +
       "ORDER BY p.categorie.nomCat ASC, p.nomProduit ASC, p.prixProduit DESC")
List<Produit> findAllOrderByCategoryThenNameThenPrice();
```

## ✅ CHECKPOINT 4.1

**Vérifiez que** :
- [ ] ProduitRepository contient au moins 6 nouvelles méthodes
- [ ] Les méthodes pour rechercher par catégorie sont implémentées
- [ ] Les requêtes @Query avec JPQL sont correctes
- [ ] Le tri avancé est implémenté
- [ ] Le Repository compile sans erreurs

---

# PARTIE 5 : SERVICE CATÉGORIE

## 🎯 Objectif

Créer les interfaces et implémentations Service pour les catégories.

## 📝 Instructions

### Étape 5.1 : Créer CategorieService.java

**Chemin** : `src/main/java/tn/iset/produits/services/CategorieService.java`

```java
package tn.iset.produits.services;

import tn.iset.produits.entities.Categorie;

import java.util.List;

/**
 * Interface Service pour les catégories
 * 
 * Définit le contrat (QUOI faire) sans les détails d'implémentation
 */
public interface CategorieService {
    
    /**
     * Sauvegarder une catégorie
     */
    Categorie saveCategorie(Categorie categorie);
    
    /**
     * Récupérer une catégorie par son ID
     */
    Categorie getCategorieById(Long id);
    
    /**
     * Récupérer toutes les catégories
     */
    List<Categorie> getAllCategories();
    
    /**
     * Mettre à jour une catégorie
     */
    Categorie updateCategorie(Long id, Categorie categorieMaj);
    
    /**
     * Supprimer une catégorie
     */
    void deleteCategorie(Long id);
    
    /**
     * Rechercher une catégorie par nom
     */
    Categorie findByNom(String nom);
    
    /**
     * Rechercher les catégories contenant un texte
     */
    List<Categorie> findByNomContains(String keyword);
    
    /**
     * Récupérer les catégories triées par nom
     */
    List<Categorie> findAllOrderByNom();
    
    /**
     * Compter les catégories avec au moins N produits
     */
    Long countCategoriesWithMinProducts(int minProduits);
    
    /**
     * Récupérer le nombre total de catégories
     */
    Long countCategories();
}
```

### Étape 5.2 : Créer CategorieServiceImpl.java

**Chemin** : `src/main/java/tn/iset/produits/services/CategorieServiceImpl.java`

```java
package tn.iset.produits.services;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import tn.iset.produits.entities.Categorie;
import tn.iset.produits.repositories.CategorieRepository;

import java.util.List;

/**
 * Implémentation du Service Catégorie
 * 
 * Contient la logique métier pour les catégories
 */
@Service
@Slf4j
@RequiredArgsConstructor
public class CategorieServiceImpl implements CategorieService {
    
    private final CategorieRepository categorieRepository;
    
    @Override
    @Transactional
    public Categorie saveCategorie(Categorie categorie) {
        log.info("Sauvegarde de la catégorie : {}", categorie.getNomCat());
        
        // Validation
        if (categorie == null || categorie.getNomCat() == null || categorie.getNomCat().isBlank()) {
            throw new IllegalArgumentException("Le nom de la catégorie est obligatoire");
        }
        
        // Sauvegarde
        Categorie saved = categorieRepository.save(categorie);
        log.info("✅ Catégorie sauvegardée avec l'ID : {}", saved.getIdCat());
        
        return saved;
    }
    
    @Override
    @Transactional(readOnly = true)
    public Categorie getCategorieById(Long id) {
        log.info("Recherche de la catégorie avec l'ID : {}", id);
        
        return categorieRepository.findById(id)
            .orElseThrow(() -> {
                log.warn("Catégorie non trouvée avec l'ID : {}", id);
                return new RuntimeException("Catégorie non trouvée avec l'ID : " + id);
            });
    }
    
    @Override
    @Transactional(readOnly = true)
    public List<Categorie> getAllCategories() {
        log.info("Récupération de toutes les catégories");
        List<Categorie> categories = categorieRepository.findAll();
        log.info("Nombre de catégories trouvées : {}", categories.size());
        return categories;
    }
    
    @Override
    @Transactional
    public Categorie updateCategorie(Long id, Categorie categorieMaj) {
        log.info("Mise à jour de la catégorie avec l'ID : {}", id);
        
        Categorie existant = getCategorieById(id);
        
        if (categorieMaj.getNomCat() != null && !categorieMaj.getNomCat().isBlank()) {
            existant.setNomCat(categorieMaj.getNomCat());
        }
        
        if (categorieMaj.getDescriptionCat() != null) {
            existant.setDescriptionCat(categorieMaj.getDescriptionCat());
        }
        
        Categorie updated = categorieRepository.save(existant);
        log.info("✅ Catégorie mise à jour avec succès");
        
        return updated;
    }
    
    @Override
    @Transactional
    public void deleteCategorie(Long id) {
        log.info("Suppression de la catégorie avec l'ID : {}", id);
        
        if (!categorieRepository.existsById(id)) {
            log.warn("Tentative de suppression d'une catégorie inexistante : {}", id);
            throw new RuntimeException("Catégorie non trouvée avec l'ID : " + id);
        }
        
        categorieRepository.deleteById(id);
        log.info("✅ Catégorie supprimée avec succès");
    }
    
    @Override
    @Transactional(readOnly = true)
    public Categorie findByNom(String nom) {
        log.info("Recherche de la catégorie avec le nom : {}", nom);
        
        return categorieRepository.findByNomCat(nom)
            .orElseThrow(() -> new RuntimeException("Catégorie non trouvée : " + nom));
    }
    
    @Override
    @Transactional(readOnly = true)
    public List<Categorie> findByNomContains(String keyword) {
        log.info("Recherche des catégories contenant : {}", keyword);
        return categorieRepository.findByNomCatContains(keyword);
    }
    
    @Override
    @Transactional(readOnly = true)
    public List<Categorie> findAllOrderByNom() {
        log.info("Récupération des catégories triées par nom");
        return categorieRepository.findByOrderByNomCatAsc();
    }
    
    @Override
    @Transactional(readOnly = true)
    public Long countCategoriesWithMinProducts(int minProduits) {
        log.info("Comptage des catégories avec au moins {} produits", minProduits);
        return categorieRepository.countCategoriesWithMinProducts(minProduits);
    }
    
    @Override
    @Transactional(readOnly = true)
    public Long countCategories() {
        return categorieRepository.count();
    }
}
```

## ✅ CHECKPOINT 5.1

**Vérifiez que** :
- [ ] CategorieService créée dans `services/`
- [ ] CategorieServiceImpl crée dans `services/`
- [ ] L'interface définit au moins 9 méthodes
- [ ] L'implémentation contient la logique métier
- [ ] La classe est annotée avec @Service
- [ ] Les deux classes compilent sans erreurs

---

# PARTIE 6 : SERVICE PRODUIT ENRICHI

## 🎯 Objectif

Enrichir le Service Produit du Lab 1 avec des méthodes de recherche par catégorie.

## 📝 Instructions

### Étape 6.1 : Enrichir ProduitService.java

**Chemin** : `src/main/java/tn/iset/produits/services/ProduitService.java`

**Ajouter** les méthodes suivantes à l'interface existante :

```java
// ========== RECHERCHE PAR CATÉGORIE (À AJOUTER) ==========

/**
 * Récupérer tous les produits d'une catégorie
 */
List<Produit> findByCategorie(Categorie categorie);

/**
 * Récupérer tous les produits d'une catégorie par son ID
 */
List<Produit> findByCategorieIdCat(Long idCat);

/**
 * Récupérer les produits d'une catégorie avec un prix minimum
 */
List<Produit> findByCategorieAndMinPrice(Categorie categorie, Double minPrix);

/**
 * Récupérer les produits d'une catégorie dans une plage de prix
 */
List<Produit> findByCategoryAndPriceRange(Long idCat, Double minPrix, Double maxPrix);

/**
 * Compter les produits d'une catégorie
 */
Long countByCategory(Long idCat);

// ========== TRI AVANCÉ (À AJOUTER) ==========

/**
 * Récupérer tous les produits triés par catégorie, puis nom, puis prix
 */
List<Produit> findAllOrderByCategoryThenNameThenPrice();

/**
 * Récupérer les produits d'une catégorie triés par nom
 */
List<Produit> findByCategorieOrderByNom(Categorie categorie);
```

### Étape 6.2 : Implémenter dans ProduitServiceImpl.java

**Ajouter** les implémentations (avec @Transactional(readOnly = true)) :

```java
@Override
@Transactional(readOnly = true)
public List<Produit> findByCategorie(Categorie categorie) {
    log.info("Recherche des produits pour la catégorie : {}", categorie.getNomCat());
    return produitRepository.findByCategorie(categorie);
}

@Override
@Transactional(readOnly = true)
public List<Produit> findByCategorieIdCat(Long idCat) {
    log.info("Recherche des produits pour l'ID catégorie : {}", idCat);
    return produitRepository.findByCategorieIdCat(idCat);
}

@Override
@Transactional(readOnly = true)
public List<Produit> findByCategorieAndMinPrice(Categorie categorie, Double minPrix) {
    log.info("Recherche produits : catégorie={}, minPrix={}", categorie.getNomCat(), minPrix);
    return produitRepository.findByCategorieAndMinPrice(categorie, minPrix);
}

@Override
@Transactional(readOnly = true)
public List<Produit> findByCategoryAndPriceRange(Long idCat, Double minPrix, Double maxPrix) {
    log.info("Recherche produits plage prix : idCat={}, min={}, max={}", idCat, minPrix, maxPrix);
    return produitRepository.findByCategoryAndPriceRange(idCat, minPrix, maxPrix);
}

@Override
@Transactional(readOnly = true)
public Long countByCategory(Long idCat) {
    return produitRepository.countByCategory(idCat);
}

@Override
@Transactional(readOnly = true)
public List<Produit> findAllOrderByCategoryThenNameThenPrice() {
    log.info("Recherche tous les produits triés par catégorie, nom, prix");
    return produitRepository.findAllOrderByCategoryThenNameThenPrice();
}

@Override
@Transactional(readOnly = true)
public List<Produit> findByCategorieOrderByNom(Categorie categorie) {
    log.info("Recherche produits de {} triés par nom", categorie.getNomCat());
    return produitRepository.findByCategorieOrderByNomProduitAsc(categorie);
}
```

## ✅ CHECKPOINT 6.1

**Vérifiez que** :
- [ ] ProduitService a 7 nouvelles méthodes
- [ ] ProduitServiceImpl implémente toutes les nouvelles méthodes
- [ ] Les logs sont en place
- [ ] @Transactional(readOnly = true) est utilisé correctement
- [ ] Les deux fichiers compilent sans erreurs

---

# PARTIE 7 : TESTS UNITAIRES

## 🎯 Objectif

Créer des tests JUnit 5 pour valider le fonctionnement des services et repositories.

## 📝 Instructions

### Étape 7.1 : Tests pour CategorieService

**Chemin** : `src/test/java/tn/iset/produits/services/CategorieServiceTest.java`

```java
package tn.iset.produits.services;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import tn.iset.produits.entities.Categorie;
import tn.iset.produits.repositories.CategorieRepository;

import java.util.List;

import static org.assertj.core.api.Assertions.*;
import static org.junit.jupiter.api.Assertions.*;

/**
 * Tests JUnit 5 pour CategorieService
 * 
 * Pattern AAA (Arrange-Act-Assert) :
 * - ARRANGE : Préparer les données
 * - ACT : Exécuter l'action
 * - ASSERT : Vérifier les résultats
 */
@SpringBootTest
class CategorieServiceTest {
    
    @Autowired
    private CategorieService categorieService;
    
    @Autowired
    private CategorieRepository categorieRepository;
    
    @BeforeEach
    void setUp() {
        // Nettoyer la BD avant chaque test
        categorieRepository.deleteAll();
    }
    
    // ==================== TESTS CRUD ====================
    
    @Test
    @DisplayName("Créer une catégorie valide")
    void testSaveCategorieSuccess() {
        // ARRANGE
        Categorie categorie = Categorie.builder()
            .nomCat("Électronique")
            .descriptionCat("Produits électroniques")
            .build();
        
        // ACT
        Categorie saved = categorieService.saveCategorie(categorie);
        
        // ASSERT
        assertThat(saved).isNotNull();
        assertThat(saved.getIdCat()).isGreaterThan(0);
        assertThat(saved.getNomCat()).isEqualTo("Électronique");
    }
    
    @Test
    @DisplayName("Récupérer une catégorie par ID")
    void testGetCategorieById() {
        // ARRANGE
        Categorie saved = categorieService.saveCategorie(
            Categorie.builder().nomCat("Vêtements").build()
        );
        
        // ACT
        Categorie found = categorieService.getCategorieById(saved.getIdCat());
        
        // ASSERT
        assertThat(found).isNotNull();
        assertThat(found.getNomCat()).isEqualTo("Vêtements");
    }
    
    @Test
    @DisplayName("Récupérer toutes les catégories")
    void testGetAllCategories() {
        // ARRANGE
        categorieService.saveCategorie(Categorie.builder().nomCat("Cat1").build());
        categorieService.saveCategorie(Categorie.builder().nomCat("Cat2").build());
        
        // ACT
        List<Categorie> all = categorieService.getAllCategories();
        
        // ASSERT
        assertThat(all).hasSize(2);
    }
    
    @Test
    @DisplayName("Mettre à jour une catégorie")
    void testUpdateCategorie() {
        // ARRANGE
        Categorie saved = categorieService.saveCategorie(
            Categorie.builder().nomCat("Old").build()
        );
        Categorie updates = Categorie.builder().nomCat("New").build();
        
        // ACT
        Categorie updated = categorieService.updateCategorie(saved.getIdCat(), updates);
        
        // ASSERT
        assertThat(updated.getNomCat()).isEqualTo("New");
    }
    
    @Test
    @DisplayName("Supprimer une catégorie")
    void testDeleteCategorie() {
        // ARRANGE
        Categorie saved = categorieService.saveCategorie(
            Categorie.builder().nomCat("ToDelete").build()
        );
        
        // ACT
        categorieService.deleteCategorie(saved.getIdCat());
        
        // ASSERT
        assertThrows(RuntimeException.class, () -> {
            categorieService.getCategorieById(saved.getIdCat());
        });
    }
    
    // ==================== TESTS RECHERCHE ====================
    
    @Test
    @DisplayName("Rechercher une catégorie par nom")
    void testFindByNom() {
        // ARRANGE
        categorieService.saveCategorie(Categorie.builder().nomCat("Unique").build());
        
        // ACT
        Categorie found = categorieService.findByNom("Unique");
        
        // ASSERT
        assertThat(found).isNotNull();
        assertThat(found.getNomCat()).isEqualTo("Unique");
    }
    
    @Test
    @DisplayName("Rechercher les catégories contenant un texte")
    void testFindByNomContains() {
        // ARRANGE
        categorieService.saveCategorie(Categorie.builder().nomCat("Électronique Grand").build());
        categorieService.saveCategorie(Categorie.builder().nomCat("Électronique Petit").build());
        categorieService.saveCategorie(Categorie.builder().nomCat("Vêtements").build());
        
        // ACT
        List<Categorie> results = categorieService.findByNomContains("Électronique");
        
        // ASSERT
        assertThat(results).hasSize(2);
        assertThat(results).allMatch(c -> c.getNomCat().contains("Électronique"));
    }
    
    // ==================== TESTS VALIDATION ====================
    
    @Test
    @DisplayName("Sauvegarder une catégorie avec nom NULL doit échouer")
    void testSaveCategorieWithNullNom() {
        // ARRANGE
        Categorie categorie = new Categorie();
        categorie.setNomCat(null);
        
        // ACT & ASSERT
        assertThrows(IllegalArgumentException.class, () -> {
            categorieService.saveCategorie(categorie);
        });
    }
    
    @Test
    @DisplayName("Sauvegarder une catégorie avec nom vide doit échouer")
    void testSaveCategorieWithBlankNom() {
        // ARRANGE
        Categorie categorie = new Categorie();
        categorie.setNomCat("   ");
        
        // ACT & ASSERT
        assertThrows(IllegalArgumentException.class, () -> {
            categorieService.saveCategorie(categorie);
        });
    }
}
```

### Étape 7.2 : Enrichir ProduitServiceTest

**Ajouter** les tests pour les recherches par catégorie :

```java
// ==================== TESTS RECHERCHE PAR CATÉGORIE (À AJOUTER) ====================

@Test
@DisplayName("Trouver les produits d'une catégorie")
void testFindByCategorie() {
    // ARRANGE
    Categorie cat = categorieService.saveCategorie(
        Categorie.builder().nomCat("Électronique").build()
    );
    produitService.saveProduit(Produit.builder()
        .nomProduit("iPhone").prixProduit(999.0)
        .categorie(cat).build());
    produitService.saveProduit(Produit.builder()
        .nomProduit("Samsung").prixProduit(899.0)
        .categorie(cat).build());
    
    // ACT
    List<Produit> results = produitService.findByCategorie(cat);
    
    // ASSERT
    assertThat(results).hasSize(2);
    assertThat(results).allMatch(p -> p.getCategorie().getIdCat().equals(cat.getIdCat()));
}

@Test
@DisplayName("Trouver les produits d'une catégorie par son ID")
void testFindByCategorieIdCat() {
    // ARRANGE
    Categorie cat = categorieService.saveCategorie(
        Categorie.builder().nomCat("Téléphones").build()
    );
    produitService.saveProduit(Produit.builder()
        .nomProduit("Produit1").prixProduit(500.0)
        .categorie(cat).build());
    
    // ACT
    List<Produit> results = produitService.findByCategorieIdCat(cat.getIdCat());
    
    // ASSERT
    assertThat(results).isNotEmpty();
}

@Test
@DisplayName("Trouver les produits d'une catégorie dans une plage de prix")
void testFindByCategoryAndPriceRange() {
    // ARRANGE
    Categorie cat = categorieService.saveCategorie(
        Categorie.builder().nomCat("Électronique").build()
    );
    produitService.saveProduit(Produit.builder()
        .nomProduit("Produit500").prixProduit(500.0)
        .categorie(cat).build());
    produitService.saveProduit(Produit.builder()
        .nomProduit("Produit1500").prixProduit(1500.0)
        .categorie(cat).build());
    
    // ACT
    List<Produit> results = produitService.findByCategoryAndPriceRange(
        cat.getIdCat(), 600.0, 2000.0
    );
    
    // ASSERT
    assertThat(results).hasSize(1);
    assertThat(results.get(0).getNomProduit()).isEqualTo("Produit1500");
}
```

## ✅ CHECKPOINT 7.1

**Vérifiez que** :
- [ ] CategorieServiceTest.java est créée avec au moins 8 tests
- [ ] ProduitServiceTest.java est enrichie avec au moins 3 tests catégorie
- [ ] Tous les tests passent (vérifier avec `mvn test`)
- [ ] Le pattern AAA (Arrange-Act-Assert) est utilisé
- [ ] Les assertions utilisent AssertJ

---

# PARTIE 8 : INITIALISATION ET AFFICHAGE

## 🎯 Objectif

Initialiser des données de test et les afficher pour vérifier le fonctionnement.

## 📝 Instructions

### Étape 8.1 : Modifier ProduitsApplication.java

**Ajouter** l'initialisation des catégories :

```java
package tn.iset.produits;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import tn.iset.produits.entities.Categorie;
import tn.iset.produits.entities.Produit;
import tn.iset.produits.services.CategorieService;
import tn.iset.produits.services.ProduitService;

import java.time.LocalDate;
import java.util.List;

@SpringBootApplication
@Slf4j
@RequiredArgsConstructor
public class ProduitsApplication implements CommandLineRunner {
    
    private final ProduitService produitService;
    private final CategorieService categorieService;
    
    public static void main(String[] args) {
        SpringApplication.run(ProduitsApplication.class, args);
    }
    
    @Override
    public void run(String... args) throws Exception {
        log.info("========================================");
        log.info("🚀 Initialisation de l'application...");
        log.info("========================================");
        
        // Vérifier s'il y a déjà des données
        if (produitService.countProduits() > 0) {
            log.info("✅ Données existantes détectées. Initialisation skippée.");
            afficherDonnees();
            return;
        }
        
        // Créer les catégories
        log.info("\n📁 Création des catégories...");
        Categorie catElectronique = categorieService.saveCategorie(
            Categorie.builder()
                .nomCat("Électronique")
                .descriptionCat("Produits électroniques et informatiques")
                .build()
        );
        log.info("✅ Catégorie créée : {}", catElectronique);
        
        Categorie catVetements = categorieService.saveCategorie(
            Categorie.builder()
                .nomCat("Vêtements")
                .descriptionCat("Vêtements et accessoires")
                .build()
        );
        log.info("✅ Catégorie créée : {}", catVetements);
        
        Categorie catChaussures = categorieService.saveCategorie(
            Categorie.builder()
                .nomCat("Chaussures")
                .descriptionCat("Chaussures de tous les styles")
                .build()
        );
        log.info("✅ Catégorie créée : {}", catChaussures);
        
        // Créer les produits avec catégories
        log.info("\n📦 Création des produits...");
        produitService.saveProduit(Produit.builder()
            .nomProduit("iPhone 15 Pro")
            .prixProduit(1099.0)
            .dateCreation(LocalDate.now())
            .categorie(catElectronique)
            .build());
        
        produitService.saveProduit(Produit.builder()
            .nomProduit("Samsung Galaxy S24")
            .prixProduit(899.0)
            .dateCreation(LocalDate.now())
            .categorie(catElectronique)
            .build());
        
        produitService.saveProduit(Produit.builder()
            .nomProduit("T-Shirt Coton")
            .prixProduit(29.99)
            .dateCreation(LocalDate.now())
            .categorie(catVetements)
            .build());
        
        produitService.saveProduit(Produit.builder()
            .nomProduit("Sneakers Nike")
            .prixProduit(89.99)
            .dateCreation(LocalDate.now())
            .categorie(catChaussures)
            .build());
        
        log.info("✅ {} produits créés", produitService.countProduits());
        
        // Afficher les données
        afficherDonnees();
    }
    
    private void afficherDonnees() {
        log.info("\n========================================");
        log.info("📋 AFFICHAGE DES DONNÉES");
        log.info("========================================");
        
        // Afficher les catégories
        log.info("\n🏷️ CATÉGORIES ({} au total) :", categorieService.countCategories());
        List<Categorie> categories = categorieService.findAllOrderByNom();
        categories.forEach(cat -> {
            log.info("  - {} : {} produit(s)", cat.getNomCat(), cat.getProduits().size());
        });
        
        // Afficher les produits par catégorie
        log.info("\n📦 PRODUITS PAR CATÉGORIE :");
        categories.forEach(cat -> {
            log.info("\n  {}", cat.getNomCat().toUpperCase());
            List<Produit> produits = produitService.findByCategorieIdCat(cat.getIdCat());
            produits.forEach(p -> {
                log.info("    - {} : {}.00 DT", p.getNomProduit(), p.getPrixProduit());
            });
        });
        
        // Afficher tous les produits triés
        log.info("\n📈 TOUS LES PRODUITS (triés par catégorie, nom, prix) :");
        List<Produit> tousLesProduites = produitService.findAllOrderByCategoryThenNameThenPrice();
        tousLesProduites.forEach(p -> {
            log.info("  - {} ({}) : {}.00 DT", 
                p.getNomProduit(), 
                p.getCategorie() != null ? p.getCategorie().getNomCat() : "SANS CAT",
                p.getPrixProduit());
        });
        
        log.info("\n========================================");
        log.info("✅ Initialisation terminée !");
        log.info("========================================\n");
    }
}
```

## ✅ CHECKPOINT 8.1

**Vérifiez que** :
- [ ] ProduitsApplication.java est modifiée
- [ ] Les catégories sont créées au démarrage
- [ ] Les produits sont liés aux catégories
- [ ] L'application démarre sans erreurs
- [ ] Les logs d'initialisation s'affichent correctement

---

# POINTS CLÉS À RETENIR

## 🎯 Relations JPA

```java
// Côté Un (Catégorie)
@OneToMany(mappedBy = "categorie")
private List<Produit> produits = new ArrayList<>();

// Côté Plusieurs (Produit)
@ManyToOne
@JoinColumn(name = "id_cat")
private Categorie categorie;
```

**Important** :
- `mappedBy` indique que la relation est gérée par le côté ManyToOne
- `@JoinColumn` crée la colonne de clé étrangère
- Toujours initialiser les listes avec `new ArrayList<>()`

## 🔍 Requêtes Spring Data

```java
// Généré automatiquement
List<Produit> findByCategorieIdCat(Long idCat);

// Personnalisé avec @Query
@Query("SELECT p FROM Produit p WHERE p.categorie.idCat = :id")
List<Produit> findByCat(@Param("id") Long id);
```

## 🧪 Tests AAA

```java
@Test
void testFunction() {
    // ARRANGE - Préparer les données
    Categorie cat = new Categorie();
    
    // ACT - Exécuter l'action
    Produit result = service.find(cat);
    
    // ASSERT - Vérifier les résultats
    assertThat(result).isNotNull();
}
```




---

## ✅ CRITÈRES D'ACCEPTATION

Votre Lab 2 est réussi si :

✅ L'entité Categorie est créée correctement  
✅ La relation OneToMany fonctionne  
✅ Le Repository Categorie a au moins 5 méthodes  
✅ Le Repository Produit est enrichi pour les catégories  
✅ Le Service Categorie est complet  
✅ Le Service Produit inclut les recherches par catégorie  
✅ Au moins 11 tests passent (8 Categorie + 3 Produit catégorie)  
✅ L'application démarre sans erreurs  
✅ Les données s'affichent correctement  
✅ Aucun warning à la compilation  

---

## 🚀 COMMANDES UTILES

```bash
# Compiler le projet
mvn clean compile

# Lancer tous les tests
mvn test

# Lancer un test spécifique
mvn test -Dtest=CategorieServiceTest

# Lancer l'application
mvn spring-boot:run

# Nettoyer les fichiers générés
mvn clean
```

---

## 💡 CONSEILS POUR RÉUSSIR

1. **Suivez l'ordre** : Ne pas sauter de parties
2. **Compilez régulièrement** : Vérifier qu'il n'y a pas d'erreurs
3. **Testez au fur et à mesure** : Chaque partie après sa création
4. **Lisez les logs** : Ils vous aideront à déboguer
5. **Respectez les conventions** : Noms, structure, format du code
6. **Documentez votre code** : Javadoc et commentaires utiles
7. **Demandez de l'aide** : Si vous êtes bloqué(e)

---

**Bon courage ! 🎓 Vous avez tous les outils pour réussir ce Lab 2 !**

