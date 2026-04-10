# 🎓 LAB 3 - RELATION MANYTOMANY ET CONTROLLERS REST
## Gestion des Produits, Catégories et Fournisseurs avec Spring Data JPA + Spring MVC

---

## 📋 TABLE DES MATIÈRES

1. [Objectifs Pédagogiques](#objectifs-pédagogiques)
2. [Structure du Projet](#structure-du-projet)
3. [Ressources Fournies](#ressources-fournies)
4. [Partie 1 : Entité Fournisseur](#partie-1--entité-fournisseur)
5. [Partie 2 : Relation ManyToMany dans Produit](#partie-2--relation-manytomany-dans-produit)
6. [Partie 3 : Repository Fournisseur](#partie-3--repository-fournisseur)
7. [Partie 4 : Repository Produit Avancé](#partie-4--repository-produit-avancé)
8. [Partie 5 : Service Fournisseur](#partie-5--service-fournisseur)
9. [Partie 6 : Service Produit Enrichi](#partie-6--service-produit-enrichi)
10. [Partie 7 : Contrôleur Catégorie](#partie-7--contrôleur-catégorie)
11. [Partie 8 : Contrôleur Produit](#partie-8--contrôleur-produit)
12. [Partie 9 : Contrôleur Fournisseur](#partie-9--contrôleur-fournisseur)
13. [Partie 10 : Tests Unitaires](#partie-10--tests-unitaires)
14. [Partie 11 : Initialisation et Affichage](#partie-11--initialisation-et-affichage)
15. [Points Clés à Retenir](#points-clés-à-retenir)

---

## OBJECTIFS PÉDAGOGIQUES

À la fin de ce lab, vous serez capable de :

✅ **Modéliser** la relation ManyToMany entre deux entités  
✅ **Créer** une table de jointure avec `@JoinTable`  
✅ **Gérer** l'association bidirectionnelle ManyToMany  
✅ **Exposer** des API REST avec `@RestController`  
✅ **Créer** un contrôleur pour chaque entité (Produit, Catégorie, Fournisseur)  
✅ **Utiliser** les verbes HTTP : GET, POST, PUT, DELETE  
✅ **Retourner** des réponses HTTP appropriées avec `ResponseEntity`  
✅ **Tester** les endpoints avec JUnit 5 et MockMvc  
✅ **Appliquer** les bonnes pratiques REST et Spring Data

---

## STRUCTURE DU PROJET

Vous devez organiser votre projet selon cette structure :

```
produits-management-lab3/
│
├── src/main/java/tn/iset/produits/
│   ├── entities/
│   │   ├── Produit.java              (EXISTANT du Lab 2, à modifier)
│   │   ├── Categorie.java            (EXISTANT du Lab 2)
│   │   └── Fournisseur.java          (NOUVEAU)
│   │
│   ├── repositories/
│   │   ├── ProduitRepository.java    (EXISTANT, à enrichir)
│   │   ├── CategorieRepository.java  (EXISTANT)
│   │   └── FournisseurRepository.java (NOUVEAU)
│   │
│   ├── services/
│   │   ├── ProduitService.java       (EXISTANT, à enrichir)
│   │   ├── ProduitServiceImpl.java   (EXISTANT, à enrichir)
│   │   ├── CategorieService.java     (EXISTANT)
│   │   ├── CategorieServiceImpl.java (EXISTANT)
│   │   ├── FournisseurService.java   (NOUVEAU)
│   │   └── FournisseurServiceImpl.java (NOUVEAU)
│   │
│   ├── controllers/
│   │   ├── ProduitController.java    (NOUVEAU)
│   │   ├── CategorieController.java  (NOUVEAU)
│   │   └── FournisseurController.java (NOUVEAU)
│   │
│   └── ProduitsApplication.java      (À modifier pour initialiser les fournisseurs)
│
├── src/test/java/tn/iset/produits/
│   ├── services/
│   │   ├── ProduitServiceTest.java   (EXISTANT, à enrichir)
│   │   ├── CategorieServiceTest.java (EXISTANT)
│   │   └── FournisseurServiceTest.java (NOUVEAU)
│   │
│   └── controllers/
│       ├── ProduitControllerTest.java    (NOUVEAU)
│       ├── CategorieControllerTest.java  (NOUVEAU)
│       └── FournisseurControllerTest.java (NOUVEAU)
│
├── src/main/resources/
│   └── application.properties
│
└── pom.xml
```

---

## RESSOURCES FOURNIES

📁 **Code source du Lab 2** (Entités Produit + Catégorie, Repositories, Services)

📁 **Votre Lab 3** : Énoncé complet avec toutes les étapes

**À votre charge** : Implémenter chaque partie selon les instructions

---

# PARTIE 1 : ENTITÉ FOURNISSEUR

## 🎯 Objectif

Créer une entité JPA `Fournisseur` avec une relation **ManyToMany** vers les produits.

## 📌 Diagramme de la Relation

```
┌────────────────┐              ┌────────────────────┐              ┌────────────────┐
│   Catégorie    │  1    *      │      Produit        │  *    *      │  Fournisseur   │
├────────────────┤◄────────────►├─────────────────────┤◄────────────►├────────────────┤
│ - idCat        │  OneToMany   │ - idProduit         │  ManyToMany  │ - idFournisseur│
│ - nomCat       │  ManyToOne   │ - nomProduit        │              │ - nomFournisseur│
│ - description  │              │ - prixProduit       │              │ - email        │
│ - produits[]   │              │ - categorie         │              │ - telephone    │
└────────────────┘              │ - fournisseurs[]    │              │ - adresse      │
                                └─────────────────────┘              │ - produits[]   │
                                                                     └────────────────┘

               Table de jointure générée automatiquement :
               ┌──────────────────────────────────┐
               │       produit_fournisseur         │
               ├──────────────────────────────────┤
               │  id_produit  (FK → produit)       │
               │  id_fournisseur (FK → fournisseur)│
               └──────────────────────────────────┘
```


## 📝 Instructions

### Étape 1.1 : Créer le fichier Fournisseur.java

**Chemin** : `src/main/java/tn/iset/produits/entities/Fournisseur.java`

**À faire** :

```java
package tn.iset.produits.entities;

import jakarta.persistence.*;
import jakarta.validation.constraints.*;
import lombok.*;

import java.util.ArrayList;
import java.util.List;

/**
 * Entité Fournisseur
 *
 * Représente un fournisseur qui peut fournir plusieurs produits.
 * Relation : Un fournisseur fournit plusieurs produits ET un produit
 *            peut être fourni par plusieurs fournisseurs → ManyToMany
 */
@Entity
@Table(name = "____") // TODO: nom de la table en BD (indice : "fournisseur")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Fournisseur {

    // ========== ATTRIBUTS ==========

    /**
     * Clé primaire auto-incrémentée
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id_fournisseur")
    private Long idFournisseur;

    /**
     * Nom du fournisseur
     * Validations :
     * - @NotBlank : Obligatoire
     * - @Size : Entre 3 et 100 caractères
     */
    @NotBlank(message = "____") // TODO: message d'erreur parlant
    @Size(
        min = 3,
        max = 100,
        message = "Le nom doit contenir entre ____ et ____ caractères" // TODO: bornes min/max
    )
    @Column(
        name = "nom_fournisseur",
        nullable = ____ ,         // TODO: true ou false ?
        length = 100
    )
    private String nomFournisseur;

    /**
     * Email du fournisseur
     * Validation : @Email
     */
    @Email(message = "____") // TODO: message d'erreur pour email invalide
    @Column(
        name = "email",
        unique = true,
        length = 150
    )
    private String email;

    /**
     * Numéro de téléphone
     */
    @Column(name = "telephone", length = 20)
    private String telephone;

    /**
     * Adresse du fournisseur
     */
    @Column(name = "____", length = 255) // TODO: nom de colonne pour l'adresse
    private String adresse;

    /**
     * Relation ManyToMany avec Produit (côté inverse)
     *
     * mappedBy = "fournisseurs" :
     * - Indique que le côté Produit gère la relation (owner side)
     * - Cette liste ne crée pas de table de jointure ici
     *
     * cascade : Pas de CascadeType.REMOVE pour éviter de supprimer
     *           les produits en supprimant un fournisseur.
     */
    @ManyToMany(
        mappedBy = "____",         // TODO: nom de l'attribut côté Produit
        fetch = FetchType.LAZY
    )
    private List<Produit> produits = new ArrayList<>();

    // ========== MÉTHODES UTILITAIRES ==========

    /**
     * Ajouter un produit à ce fournisseur (cohérence bidirectionnelle)
     */
    public void addProduit(Produit produit) {
        this.produits.add(produit);
        produit.getFournisseurs().add(____); // TODO: this ou produit ?
    }

    /**
     * Retirer un produit de ce fournisseur (cohérence bidirectionnelle)
     */
    public void removeProduit(Produit produit) {
        this.produits.remove(produit);
        produit.getFournisseurs().remove(____); // TODO: même objet que ci-dessus
    }

    @Override
    public String toString() {
        return String.format(
            "Fournisseur{id=%d, nom='%s', email='%s', nbProduits=%d}",
            idFournisseur,
            nomFournisseur,
            email,
            ____  // TODO: taille de la liste produits
        );
    }
}
```

## ✅ CHECKPOINT 1.1

**Vérifiez que** :
- [ ] La classe `Fournisseur` est créée dans `entities/`
- [ ] Toutes les annotations JPA sont présentes (`@Entity`, `@Id`, `@GeneratedValue`)
- [ ] Les validations sont appliquées (`@NotBlank`, `@Size`, `@Email`)
- [ ] La relation `@ManyToMany(mappedBy = "...")` est présente
- [ ] La classe compile sans erreurs


## ✅ CHECKPOINT 1.1

**Vérifiez que** :
- [ ] La classe `Fournisseur` est créée dans `entities/`
- [ ] Toutes les annotations JPA sont présentes (`@Entity`, `@Id`, `@GeneratedValue`)
- [ ] Les validations sont appliquées (`@NotBlank`, `@Size`, `@Email`)
- [ ] La relation `@ManyToMany(mappedBy = "fournisseurs")` est présente
- [ ] La classe compile sans erreurs

---

# PARTIE 2 : RELATION MANYTOMANY DANS PRODUIT

## 🎯 Objectif

Ajouter la relation ManyToMany **owner side** dans l'entité `Produit`.

## 📝 Instructions

### Étape 2.1 : Modifier Produit.java

**Chemin** : `src/main/java/tn/iset/produits/entities/Produit.java`

**Ajouter** la relation ManyToMany **après la relation ManyToOne existante** :

```java
// ========== RELATION MANYTOMANY (À AJOUTER) ==========

/**
 * Relation ManyToMany avec Fournisseur (owner side)
 *
 * @ManyToMany :
 * - Plusieurs produits pour plusieurs fournisseurs
 *
 * @JoinTable :
 * - Crée la table de jointure "produit_fournisseur"
 * - joinColumns : colonne qui référence Produit (id_produit)
 * - inverseJoinColumns : colonne qui référence Fournisseur (id_fournisseur)
 *
 * fetch = FetchType.LAZY :
 * - Les fournisseurs ne sont chargés que si on y accède explicitement
 *   (évite les requêtes inutiles et les boucles infinies JSON)
 */
@ManyToMany(fetch = FetchType.LAZY)
@JoinTable(
    name = "produit_fournisseur",
    joinColumns = @JoinColumn(name = "id_produit"),
    inverseJoinColumns = @JoinColumn(name = "id_fournisseur")
)
private List<Fournisseur> fournisseurs = new ArrayList<>();

// ========== FIN DE LA RELATION ==========
```

**Modifiez également le `toString()`** pour afficher le nombre de fournisseurs :

```java
@Override
public String toString() {
    return String.format(
        "Produit{id=%d, nom='%s', prix=%.2f, categorie=%s, nbFournisseurs=%d}",
        idProduit, nomProduit, prixProduit,
        categorie != null ? categorie.getNomCat() : "SANS CATÉGORIE",
        fournisseurs != null ? fournisseurs.size() : 0
    );
}
```

## 📊 Résultat en BD

Après cette étape, Hibernate crée automatiquement :

```sql
-- Table Fournisseur
CREATE TABLE fournisseur (
  id_fournisseur BIGINT PRIMARY KEY AUTO_INCREMENT,
  nom_fournisseur VARCHAR(100) NOT NULL,
  email VARCHAR(150) UNIQUE,
  telephone VARCHAR(20),
  adresse VARCHAR(255)
);

-- Table de jointure ManyToMany
CREATE TABLE produit_fournisseur (
  id_produit     BIGINT NOT NULL,
  id_fournisseur BIGINT NOT NULL,
  PRIMARY KEY (id_produit, id_fournisseur),
  FOREIGN KEY (id_produit)     REFERENCES produit(id_produit),
  FOREIGN KEY (id_fournisseur) REFERENCES fournisseur(id_fournisseur)
);
```

## ✅ CHECKPOINT 2.1

**Vérifiez que** :
- [ ] L'attribut `fournisseurs` est ajouté dans `Produit`
- [ ] L'annotation `@ManyToMany` est présente
- [ ] `@JoinTable` avec `joinColumns` et `inverseJoinColumns` est correct
- [ ] La liste est initialisée avec `new ArrayList<>()`
- [ ] `Produit` compile sans erreurs

---

# PARTIE 3 : REPOSITORY FOURNISSEUR

## 🎯 Objectif

Créer une interface Repository pour les fournisseurs avec des requêtes courantes.

## 📝 Instructions

### Étape 3.1 : Créer FournisseurRepository.java

**Chemin** : `src/main/java/tn/iset/produits/repositories/FournisseurRepository.java`

```java
package tn.iset.produits.repositories;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;
import tn.iset.produits.entities.Fournisseur;

import java.util.List;
import java.util.Optional;

/**
 * Repository pour l'entité Fournisseur
 *
 * Héritage automatique des méthodes CRUD :
 * - save(Fournisseur)
 * - findById(Long)
 * - findAll()
 * - delete(Fournisseur)
 * - count()
 */
@Repository
public interface FournisseurRepository extends JpaRepository<Fournisseur, Long> {

    // ========== RECHERCHE PAR NOM ==========

    /**
     * Trouver un fournisseur par son nom exact
     * SELECT * FROM fournisseur WHERE nom_fournisseur = ?
     */
    Optional<Fournisseur> findByNomFournisseur(String nomFournisseur);

    /**
     * Trouver les fournisseurs dont le nom contient un texte
     * SELECT * FROM fournisseur WHERE nom_fournisseur LIKE %?%
     */
    List<Fournisseur> findByNomFournisseurContains(String keyword);

    /**
     * Trouver les fournisseurs dont le nom commence par un préfixe
     * SELECT * FROM fournisseur WHERE nom_fournisseur LIKE ?%
     */
    List<Fournisseur> findByNomFournisseurStartsWith(String prefix);

    // ========== RECHERCHE PAR EMAIL ==========

    /**
     * Trouver un fournisseur par son email
     */
    Optional<Fournisseur> findByEmail(String email);

    /**
     * Vérifier si un email existe déjà
     */
    boolean existsByEmail(String email);

    // ========== TRI ==========

    /**
     * Récupérer les fournisseurs triés par nom (croissant)
     * SELECT * FROM fournisseur ORDER BY nom_fournisseur ASC
     */
    List<Fournisseur> findByOrderByNomFournisseurAsc();

    /**
     * Récupérer les fournisseurs triés par nom (décroissant)
     */
    List<Fournisseur> findByOrderByNomFournisseurDesc();

    // ========== REQUÊTES PERSONNALISÉES AVEC @Query ==========

    /**
     * Trouver les fournisseurs d'un produit donné (par son ID)
     *
     * JPQL :
     * SELECT f FROM Fournisseur f JOIN f.produits p WHERE p.idProduit = :idProduit
     */
    @Query("SELECT f FROM Fournisseur f JOIN f.produits p WHERE p.idProduit = :idProduit")
    List<Fournisseur> findByProduitId(@Param("idProduit") Long idProduit);

    /**
     * Compter les fournisseurs ayant au moins N produits
     *
     * JPQL :
     * SELECT COUNT(DISTINCT f) FROM Fournisseur f WHERE SIZE(f.produits) >= :minProduits
     */
    @Query("SELECT COUNT(DISTINCT f) FROM Fournisseur f WHERE SIZE(f.produits) >= :minProduits")
    Long countFournisseursWithMinProducts(@Param("minProduits") int minProduits);

    /**
     * Trouver les fournisseurs ayant au moins N produits
     *
     * JPQL :
     * SELECT f FROM Fournisseur f WHERE SIZE(f.produits) > :minProduits
     */
    @Query("SELECT f FROM Fournisseur f WHERE SIZE(f.produits) > :minProduits")
    List<Fournisseur> findFournisseursWithMinProducts(@Param("minProduits") int minProduits);
}
```

## ✅ CHECKPOINT 3.1

**Vérifiez que** :
- [ ] `FournisseurRepository` est créé dans `repositories/`
- [ ] Il étend `JpaRepository<Fournisseur, Long>`
- [ ] Au moins 7 méthodes de recherche sont implémentées
- [ ] Les annotations `@Query` sont correctes
- [ ] La classe compile sans erreurs

---

# PARTIE 4 : REPOSITORY PRODUIT AVANCÉ

## 🎯 Objectif

Enrichir le Repository Produit du Lab 2 avec des requêtes liées aux fournisseurs.

## 📝 Instructions

### Étape 4.1 : Enrichir ProduitRepository.java

**Chemin** : `src/main/java/tn/iset/produits/repositories/ProduitRepository.java`

**Ajouter** les méthodes suivantes au Repository existant :

```java
// ========== RECHERCHE PAR FOURNISSEUR (À AJOUTER) ==========

/**
 * Trouver les produits d'un fournisseur donné
 *
 * Spring Data supporte la navigation des relations ManyToMany :
 * SELECT * FROM produit p JOIN produit_fournisseur pf ON p.id_produit = pf.id_produit
 * WHERE pf.id_fournisseur = ?
 */
List<Produit> findByFournisseursIdFournisseur(Long idFournisseur);

/**
 * Trouver les produits d'un fournisseur triés par nom
 */
List<Produit> findByFournisseursIdFournisseurOrderByNomProduitAsc(Long idFournisseur);

/**
 * Compter les produits d'un fournisseur
 *
 * JPQL :
 * SELECT COUNT(p) FROM Produit p JOIN p.fournisseurs f WHERE f.idFournisseur = :idFournisseur
 */
@Query("SELECT COUNT(p) FROM Produit p JOIN p.fournisseurs f WHERE f.idFournisseur = :idFournisseur")
Long countByFournisseur(@Param("idFournisseur") Long idFournisseur);

/**
 * Trouver les produits liés à un fournisseur avec un prix minimum
 *
 * JPQL :
 * SELECT p FROM Produit p JOIN p.fournisseurs f
 * WHERE f.idFournisseur = :idFournisseur AND p.prixProduit >= :minPrix
 * ORDER BY p.prixProduit DESC
 */
@Query("SELECT p FROM Produit p JOIN p.fournisseurs f " +
       "WHERE f.idFournisseur = :idFournisseur AND p.prixProduit >= :minPrix " +
       "ORDER BY p.prixProduit DESC")
List<Produit> findByFournisseurAndMinPrice(
    @Param("idFournisseur") Long idFournisseur,
    @Param("minPrix") Double minPrix);

/**
 * Trouver les produits sans fournisseur
 *
 * JPQL :
 * SELECT p FROM Produit p WHERE p.fournisseurs IS EMPTY
 */
@Query("SELECT p FROM Produit p WHERE p.fournisseurs IS EMPTY")
List<Produit> findProduitsWithoutFournisseur();
```

## ✅ CHECKPOINT 4.1

**Vérifiez que** :
- [ ] `ProduitRepository` contient au moins 5 nouvelles méthodes fournisseur
- [ ] Les requêtes `@Query` JPQL sont correctes
- [ ] Le Repository compile sans erreurs

---

# PARTIE 5 : SERVICE FOURNISSEUR

## 🎯 Objectif

Créer l'interface et l'implémentation `Service` pour les fournisseurs.

## 📝 Instructions

### Étape 5.1 : Créer FournisseurService.java

**Chemin** : `src/main/java/tn/iset/produits/services/FournisseurService.java`

```java
package tn.iset.produits.services;

import tn.iset.produits.entities.Fournisseur;

import java.util.List;

/**
 * Interface Service pour les fournisseurs
 *
 * Définit le contrat (QUOI faire) sans les détails d'implémentation
 */
public interface FournisseurService {

    /** Sauvegarder un fournisseur */
    ____ saveFournisseur(____ fournisseur); // TODO: type de retour et type du paramètre

    /** Récupérer un fournisseur par son ID */
    Fournisseur getFournisseurById(____ id); // TODO: type de l'ID

    /** Récupérer tous les fournisseurs */
    List<____> getAllFournisseurs(); // TODO: type générique de la liste

    /** Mettre à jour un fournisseur */
    Fournisseur updateFournisseur(Long id, Fournisseur ____); // TODO: nom du paramètre mis à jour

    /** Supprimer un fournisseur */
    void deleteFournisseur(Long ____); // TODO: type ou nom

    /** Rechercher un fournisseur par nom */
    Fournisseur findByNom(String ____); // TODO: nom du paramètre

    /** Rechercher les fournisseurs contenant un texte dans le nom */
    List<Fournisseur> findByNomContains(____ keyword); // TODO: type du paramètre

    /** Récupérer les fournisseurs triés par nom */
    List<Fournisseur> findAllOrderByNom();

    /** Lier un fournisseur à un produit */
    void addProduitToFournisseur(____ idFournisseur, ____ idProduit); // TODO: types

    /** Délier un fournisseur d'un produit */
    void removeProduitFromFournisseur(Long idFournisseur, Long idProduit);

    /** Compter les fournisseurs avec au moins N produits */
    Long countFournisseursWithMinProducts(int ____); // TODO: nom du paramètre

    /** Récupérer le nombre total de fournisseurs */
    Long countFournisseurs();
}
```

### Étape 5.2 : Créer FournisseurServiceImpl.java

**Chemin** : `src/main/java/tn/iset/produits/services/FournisseurServiceImpl.java`

```java
package tn.iset.produits.services;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import tn.iset.produits.entities.Fournisseur;
import tn.iset.produits.entities.Produit;
import tn.iset.produits.repositories.FournisseurRepository;
import tn.iset.produits.repositories.ProduitRepository;

import java.util.List;

/**
 * Implémentation du Service Fournisseur
 *
 * Contient la logique métier pour les fournisseurs
 */
@Service
@Slf4j
@RequiredArgsConstructor
public class FournisseurServiceImpl implements FournisseurService {

    private final FournisseurRepository fournisseurRepository;
    private final ProduitRepository produitRepository;

    @Override
    @Transactional
    public Fournisseur saveFournisseur(Fournisseur fournisseur) {
        log.info("Sauvegarde du fournisseur : {}", fournisseur != null ? fournisseur.getNomFournisseur() : "NULL");

        if (fournisseur == null || fournisseur.getNomFournisseur() == null
                || fournisseur.getNomFournisseur().isBlank()) {
            throw new IllegalArgumentException("____"); // TODO: message d'erreur
        }

        if (fournisseur.getEmail() != null
                && fournisseurRepository.existsByEmail(fournisseur.getEmail())) {
            throw new IllegalArgumentException("Un fournisseur avec cet email existe déjà : "
                    + fournisseur.getEmail());
        }

        Fournisseur saved = fournisseurRepository.____(fournisseur); // TODO: méthode CRUD à utiliser
        log.info("✅ Fournisseur sauvegardé avec l'ID : {}", saved.getIdFournisseur());
        return saved;
    }

    @Override
    @Transactional(readOnly = true)
    public Fournisseur getFournisseurById(Long id) {
        log.info("Recherche du fournisseur avec l'ID : {}", id);
        return fournisseurRepository.findById(id)
            .orElseThrow(() -> {
                log.warn("Fournisseur non trouvé avec l'ID : {}", id);
                return new RuntimeException("Fournisseur non trouvé avec l'ID : " + ____); // TODO: variable à afficher
            });
    }

    @Override
    @Transactional(readOnly = true)
    public List<Fournisseur> getAllFournisseurs() {
        log.info("Récupération de tous les fournisseurs");
        List<Fournisseur> list = fournisseurRepository.____(); // TODO: méthode pour récupérer tous les fournisseurs
        log.info("Nombre de fournisseurs trouvés : {}", list.size());
        return list;
    }

    @Override
    @Transactional
    public Fournisseur updateFournisseur(Long id, Fournisseur fournisseurMaj) {
        log.info("Mise à jour du fournisseur avec l'ID : {}", id);
        Fournisseur existant = getFournisseurById(id);

        if (fournisseurMaj.getNomFournisseur() != null
                && !fournisseurMaj.getNomFournisseur().isBlank()) {
            existant.setNomFournisseur(fournisseurMaj.getNomFournisseur());
        }
        if (fournisseurMaj.getEmail() != null) {
            existant.setEmail(fournisseurMaj.getEmail());
        }
        if (fournisseurMaj.getTelephone() != null) {
            existant.setTelephone(fournisseurMaj.getTelephone());
        }
        if (fournisseurMaj.getAdresse() != null) {
            existant.setAdresse(fournisseurMaj.getAdresse());
        }

        Fournisseur updated = fournisseurRepository.save(____); // TODO: objet à sauvegarder
        log.info("✅ Fournisseur mis à jour avec succès");
        return updated;
    }

    @Override
    @Transactional
    public void deleteFournisseur(Long id) {
        log.info("Suppression du fournisseur avec l'ID : {}", id);
        if (!fournisseurRepository.existsById(id)) {
            log.warn("Tentative de suppression d'un fournisseur inexistant : {}", id);
            throw new RuntimeException("Fournisseur non trouvé avec l'ID : " + id);
        }
        fournisseurRepository.delete(____); // TODO: objet ou méthode de suppression
        log.info("✅ Fournisseur supprimé avec succès");
    }

    @Override
    @Transactional(readOnly = true)
    public Fournisseur findByNom(String nom) {
        log.info("Recherche du fournisseur avec le nom : {}", nom);
        return fournisseurRepository.findByNomFournisseur(nom)
            .orElseThrow(() -> new RuntimeException("____")); // TODO: message si non trouvé
    }

    @Override
    @Transactional(readOnly = true)
    public List<Fournisseur> findByNomContains(String keyword) {
        log.info("Recherche des fournisseurs contenant : {}", keyword);
        return fournisseurRepository.findByNomFournisseurContains(____); // TODO: paramètre
    }

    @Override
    @Transactional(readOnly = true)
    public List<Fournisseur> findAllOrderByNom() {
        log.info("Récupération des fournisseurs triés par nom");
        return fournisseurRepository.____(); // TODO: méthode de tri par nom ASC
    }

    @Override
    @Transactional
    public void addProduitToFournisseur(Long idFournisseur, Long idProduit) {
        log.info("Association du produit {} au fournisseur {}", idProduit, idFournisseur);
        Fournisseur fournisseur = getFournisseurById(idFournisseur);
        Produit produit = produitRepository.findById(idProduit)
            .orElseThrow(() -> new RuntimeException("____")); // TODO: message d'erreur produit

        fournisseur.addProduit(produit);
        fournisseurRepository.save(fournisseur);
        log.info("✅ Produit {} associé au fournisseur {}", idProduit, idFournisseur);
    }

    @Override
    @Transactional
    public void removeProduitFromFournisseur(Long idFournisseur, Long idProduit) {
        log.info("Dissociation du produit {} du fournisseur {}", idProduit, idFournisseur);
        Fournisseur fournisseur = getFournisseurById(idFournisseur);
        Produit produit = produitRepository.findById(idProduit)
            .orElseThrow(() -> new RuntimeException("____")); // TODO: message d'erreur

        fournisseur.removeProduit(produit);
        fournisseurRepository.save(fournisseur);
        log.info("✅ Produit {} dissocié du fournisseur {}", idProduit, idFournisseur);
    }

    @Override
    @Transactional(readOnly = true)
    public Long countFournisseursWithMinProducts(int minProduits) {
        log.info("Comptage des fournisseurs avec au moins {} produits", minProduits);
        return fournisseurRepository.____(minProduits); // TODO: méthode de comptage
    }

    @Override
    @Transactional(readOnly = true)
    public Long countFournisseurs() {
        return fournisseurRepository.count();
    }
}
```

## ✅ CHECKPOINT 5.1

**Vérifiez que** :
- [ ] `FournisseurService` est créé dans `services/`
- [ ] `FournisseurServiceImpl` est créé dans `services/`
- [ ] L’interface définit au moins 12 méthodes
- [ ] La logique d’association/dissociation ManyToMany est correcte
- [ ] La classe est annotée avec `@Service`
- [ ] Les deux fichiers compilent sans erreurs

## ✅ CHECKPOINT 5.1

**Vérifiez que** :
- [ ] `FournisseurService` est créé dans `services/`
- [ ] `FournisseurServiceImpl` est créé dans `services/`
- [ ] L'interface définit au moins 12 méthodes
- [ ] La logique d'association/dissociation ManyToMany est correcte
- [ ] La classe est annotée avec `@Service`
- [ ] Les deux fichiers compilent sans erreurs

---

# PARTIE 6 : SERVICE PRODUIT ENRICHI

## 🎯 Objectif

Enrichir le Service Produit du Lab 2 avec des méthodes de recherche par fournisseur.

## 📝 Instructions

### Étape 6.1 : Enrichir ProduitService.java

**Ajouter** les méthodes suivantes à l'interface existante :

```java
// ========== RECHERCHE PAR FOURNISSEUR (À AJOUTER) ==========

/** Récupérer tous les produits d'un fournisseur */
List<Produit> findByFournisseurId(Long idFournisseur);

/** Récupérer les produits d'un fournisseur triés par nom */
List<Produit> findByFournisseurIdOrderByNom(Long idFournisseur);

/** Compter les produits d'un fournisseur */
Long countByFournisseur(Long idFournisseur);

/** Récupérer les produits d'un fournisseur avec un prix minimum */
List<Produit> findByFournisseurAndMinPrice(Long idFournisseur, Double minPrix);

/** Récupérer les produits sans fournisseur */
List<Produit> findProduitsWithoutFournisseur();
```

### Étape 6.2 : Implémenter dans ProduitServiceImpl.java

**Ajouter** les implémentations (avec `@Transactional(readOnly = true)`) :

```java
@Override
@Transactional(readOnly = true)
public List<Produit> findByFournisseurId(Long idFournisseur) {
    log.info("Recherche des produits pour l'ID fournisseur : {}", idFournisseur);
    return produitRepository.findByFournisseursIdFournisseur(idFournisseur);
}

@Override
@Transactional(readOnly = true)
public List<Produit> findByFournisseurIdOrderByNom(Long idFournisseur) {
    log.info("Recherche des produits (triés) pour le fournisseur : {}", idFournisseur);
    return produitRepository.findByFournisseursIdFournisseurOrderByNomProduitAsc(idFournisseur);
}

@Override
@Transactional(readOnly = true)
public Long countByFournisseur(Long idFournisseur) {
    return produitRepository.countByFournisseur(idFournisseur);
}

@Override
@Transactional(readOnly = true)
public List<Produit> findByFournisseurAndMinPrice(Long idFournisseur, Double minPrix) {
    log.info("Recherche produits fournisseur={} avec minPrix={}", idFournisseur, minPrix);
    return produitRepository.findByFournisseurAndMinPrice(idFournisseur, minPrix);
}

@Override
@Transactional(readOnly = true)
public List<Produit> findProduitsWithoutFournisseur() {
    log.info("Recherche des produits sans fournisseur");
    return produitRepository.findProduitsWithoutFournisseur();
}
```

## ✅ CHECKPOINT 6.1

**Vérifiez que** :
- [ ] `ProduitService` a 5 nouvelles méthodes fournisseur
- [ ] `ProduitServiceImpl` implémente toutes les nouvelles méthodes
- [ ] Les logs sont en place
- [ ] `@Transactional(readOnly = true)` est utilisé correctement

---

# PARTIE 7 : CONTRÔLEUR CATÉGORIE

## 🎯 Objectif

Créer un contrôleur REST pour exposer les opérations CRUD de `Catégorie` via HTTP.

## 📌 Tableau des Endpoints

| Méthode HTTP | URL                               | Action                        | Réponse |
|-------------|-----------------------------------|-------------------------------|---------|
| GET         | `/api/categories`                 | Lister toutes les catégories  | 200     |
| GET         | `/api/categories/{id}`            | Récupérer une catégorie       | 200/404 |
| GET         | `/api/categories/search?nom=xxx`  | Rechercher par nom            | 200     |
| POST        | `/api/categories`                 | Créer une catégorie           | 201     |
| PUT         | `/api/categories/{id}`            | Mettre à jour une catégorie   | 200/404 |
| DELETE      | `/api/categories/{id}`            | Supprimer une catégorie       | 204/404 |
| GET         | `/api/categories/count`           | Compter les catégories        | 200     |

## 📝 Instructions

### Étape 7.1 : Créer CategorieController.java

**Chemin** : `src/main/java/tn/iset/produits/controllers/CategorieController.java`

```java
package tn.iset.produits.controllers;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import tn.iset.produits.entities.Categorie;
import tn.iset.produits.services.CategorieService;

import java.util.List;

/**
 * Contrôleur REST pour l'entité Catégorie
 *
 * @RestController = @Controller + @ResponseBody
 * - Toutes les méthodes retournent du JSON automatiquement
 *
 * @RequestMapping("/api/categories") :
 * - Préfixe de toutes les URLs de ce contrôleur
 */
@RestController
@RequestMapping("/api/categories")
@RequiredArgsConstructor
@Slf4j
public class CategorieController {

    private final CategorieService categorieService;

    // ========== GET ALL ==========

    /**
     * GET /api/categories
     * Récupérer toutes les catégories triées par nom
     *
     * @return 200 OK + liste des catégories
     */
    @GetMapping
    public ResponseEntity<List<Categorie>> getAllCategories() {
        log.info("GET /api/categories - Récupération de toutes les catégories");
        List<Categorie> categories = categorieService.findAllOrderByNom();
        return ResponseEntity.ok(categories);
    }

    // ========== GET BY ID ==========

    /**
     * GET /api/categories/{id}
     * Récupérer une catégorie par son ID
     *
     * @param id identifiant de la catégorie
     * @return 200 OK + catégorie, ou 404 Not Found
     */
    @GetMapping("/{id}")
    public ResponseEntity<Categorie> getCategorieById(@PathVariable Long id) {
        log.info("GET /api/categories/{} - Récupération de la catégorie", id);
        try {
            Categorie categorie = categorieService.getCategorieById(id);
            return ResponseEntity.ok(categorie);
        } catch (RuntimeException e) {
            log.warn("Catégorie non trouvée : {}", id);
            return ResponseEntity.notFound().build();
        }
    }

    // ========== SEARCH BY NOM ==========

    /**
     * GET /api/categories/search?nom=xxx
     * Rechercher les catégories par nom
     *
     * @param nom texte à rechercher
     * @return 200 OK + liste des catégories correspondantes
     */
    @GetMapping("/search")
    public ResponseEntity<List<Categorie>> searchByNom(
            @RequestParam(name = "nom", defaultValue = "") String nom) {
        log.info("GET /api/categories/search?nom={}", nom);
        List<Categorie> results = categorieService.findByNomContains(nom);
        return ResponseEntity.ok(results);
    }

    // ========== COUNT ==========

    /**
     * GET /api/categories/count
     * Compter le nombre total de catégories
     *
     * @return 200 OK + nombre
     */
    @GetMapping("/count")
    public ResponseEntity<Long> countCategories() {
        log.info("GET /api/categories/count");
        return ResponseEntity.ok(categorieService.countCategories());
    }

    // ========== CREATE ==========

    /**
     * POST /api/categories
     * Créer une nouvelle catégorie
     *
     * @param categorie données de la catégorie (corps JSON)
     * @return 201 Created + catégorie créée, ou 400 Bad Request
     */
    @PostMapping
    public ResponseEntity<Categorie> createCategorie(@RequestBody Categorie categorie) {
        log.info("POST /api/categories - Création : {}", categorie.getNomCat());
        try {
            Categorie saved = categorieService.saveCategorie(categorie);
            return ResponseEntity.status(HttpStatus.CREATED).body(saved);
        } catch (IllegalArgumentException e) {
            log.error("Erreur de validation : {}", e.getMessage());
            return ResponseEntity.badRequest().build();
        }
    }

    // ========== UPDATE ==========

    /**
     * PUT /api/categories/{id}
     * Mettre à jour une catégorie existante
     *
     * @param id          identifiant de la catégorie
     * @param categorieMaj données de mise à jour (corps JSON)
     * @return 200 OK + catégorie mise à jour, ou 404 Not Found
     */
    @PutMapping("/{id}")
    public ResponseEntity<Categorie> updateCategorie(
            @PathVariable Long id,
            @RequestBody Categorie categorieMaj) {
        log.info("PUT /api/categories/{} - Mise à jour", id);
        try {
            Categorie updated = categorieService.updateCategorie(id, categorieMaj);
            return ResponseEntity.ok(updated);
        } catch (RuntimeException e) {
            log.warn("Catégorie non trouvée pour mise à jour : {}", id);
            return ResponseEntity.notFound().build();
        }
    }

    // ========== DELETE ==========

    /**
     * DELETE /api/categories/{id}
     * Supprimer une catégorie
     *
     * @param id identifiant de la catégorie
     * @return 204 No Content, ou 404 Not Found
     */
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteCategorie(@PathVariable Long id) {
        log.info("DELETE /api/categories/{}", id);
        try {
            categorieService.deleteCategorie(id);
            return ResponseEntity.noContent().build();
        } catch (RuntimeException e) {
            log.warn("Catégorie non trouvée pour suppression : {}", id);
            return ResponseEntity.notFound().build();
        }
    }
}
```

## ✅ CHECKPOINT 7.1

**Vérifiez que** :
- [ ] `CategorieController` est créé dans `controllers/`
- [ ] La classe est annotée avec `@RestController` et `@RequestMapping`
- [ ] Les 7 endpoints sont implémentés
- [ ] Chaque méthode retourne un `ResponseEntity` avec le bon code HTTP
- [ ] La classe compile sans erreurs

---

# PARTIE 8 : CONTRÔLEUR PRODUIT

## 🎯 Objectif

Créer un contrôleur REST pour exposer les opérations CRUD de `Produit` via HTTP, incluant les recherches par catégorie et par fournisseur.

## 📌 Tableau des Endpoints

| Méthode HTTP | URL                                         | Action                              | Réponse |
|-------------|---------------------------------------------|-------------------------------------|---------|
| GET         | `/api/produits`                             | Lister tous les produits            | 200     |
| GET         | `/api/produits/{id}`                        | Récupérer un produit                | 200/404 |
| GET         | `/api/produits/search?nom=xxx`              | Rechercher par nom                  | 200     |
| GET         | `/api/produits/categorie/{idCat}`           | Produits par catégorie              | 200     |
| GET         | `/api/produits/fournisseur/{idFourn}`       | Produits par fournisseur            | 200     |
| GET         | `/api/produits/sans-fournisseur`            | Produits sans fournisseur           | 200     |
| GET         | `/api/produits/count`                       | Compter les produits                | 200     |
| POST        | `/api/produits`                             | Créer un produit                    | 201     |
| PUT         | `/api/produits/{id}`                        | Mettre à jour un produit            | 200/404 |
| DELETE      | `/api/produits/{id}`                        | Supprimer un produit                | 204/404 |

## 📝 Instructions

### Étape 8.1 : Créer ProduitController.java

**Chemin** : `src/main/java/tn/iset/produits/controllers/ProduitController.java`

```java
package tn.iset.produits.controllers;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import tn.iset.produits.entities.Produit;
import tn.iset.produits.services.ProduitService;

import java.util.List;

/**
 * Contrôleur REST pour l'entité Produit
 *
 * Expose les opérations CRUD et les recherches avancées via HTTP
 */
@RestController
@RequestMapping("/api/produits")
@RequiredArgsConstructor
@Slf4j
public class ProduitController {

    private final ProduitService produitService;

    // ========== GET ALL ==========

    /**
     * GET /api/produits
     * Récupérer tous les produits (triés par catégorie, nom, prix)
     */
    @GetMapping
    public ResponseEntity<List<Produit>> getAllProduits() {
        log.info("GET /api/produits");
        List<Produit> produits = produitService.findAllOrderByCategoryThenNameThenPrice();
        return ResponseEntity.ok(produits);
    }

    // ========== GET BY ID ==========

    /**
     * GET /api/produits/{id}
     * Récupérer un produit par son ID
     */
    @GetMapping("/{id}")
    public ResponseEntity<Produit> getProduitById(@PathVariable Long id) {
        log.info("GET /api/produits/{}", id);
        try {
            Produit produit = produitService.getProduitById(id);
            return ResponseEntity.ok(produit);
        } catch (RuntimeException e) {
            return ResponseEntity.notFound().build();
        }
    }

    // ========== SEARCH BY NOM ==========

    /**
     * GET /api/produits/search?nom=xxx
     * Rechercher les produits par nom
     */
    @GetMapping("/search")
    public ResponseEntity<List<Produit>> searchByNom(
            @RequestParam(name = "nom", defaultValue = "") String nom) {
        log.info("GET /api/produits/search?nom={}", nom);
        List<Produit> results = produitService.findByNomContains(nom);
        return ResponseEntity.ok(results);
    }

    // ========== FILTER BY CATEGORIE ==========

    /**
     * GET /api/produits/categorie/{idCat}
     * Récupérer les produits d'une catégorie donnée
     */
    @GetMapping("/categorie/{idCat}")
    public ResponseEntity<List<Produit>> getProduitsByCategorie(@PathVariable Long idCat) {
        log.info("GET /api/produits/categorie/{}", idCat);
        List<Produit> produits = produitService.findByCategorieIdCat(idCat);
        return ResponseEntity.ok(produits);
    }

    // ========== FILTER BY FOURNISSEUR ==========

    /**
     * GET /api/produits/fournisseur/{idFournisseur}
     * Récupérer les produits d'un fournisseur donné
     */
    @GetMapping("/fournisseur/{idFournisseur}")
    public ResponseEntity<List<Produit>> getProduitsByFournisseur(
            @PathVariable Long idFournisseur) {
        log.info("GET /api/produits/fournisseur/{}", idFournisseur);
        List<Produit> produits = produitService.findByFournisseurId(idFournisseur);
        return ResponseEntity.ok(produits);
    }

    // ========== SANS FOURNISSEUR ==========

    /**
     * GET /api/produits/sans-fournisseur
     * Récupérer les produits sans fournisseur
     */
    @GetMapping("/sans-fournisseur")
    public ResponseEntity<List<Produit>> getProduitsWithoutFournisseur() {
        log.info("GET /api/produits/sans-fournisseur");
        List<Produit> produits = produitService.findProduitsWithoutFournisseur();
        return ResponseEntity.ok(produits);
    }

    // ========== COUNT ==========

    /**
     * GET /api/produits/count
     * Compter le nombre total de produits
     */
    @GetMapping("/count")
    public ResponseEntity<Long> countProduits() {
        log.info("GET /api/produits/count");
        return ResponseEntity.ok(produitService.countProduits());
    }

    // ========== CREATE ==========

    /**
     * POST /api/produits
     * Créer un nouveau produit
     */
    @PostMapping
    public ResponseEntity<Produit> createProduit(@RequestBody Produit produit) {
        log.info("POST /api/produits - Création : {}", produit.getNomProduit());
        try {
            Produit saved = produitService.saveProduit(produit);
            return ResponseEntity.status(HttpStatus.CREATED).body(saved);
        } catch (IllegalArgumentException e) {
            log.error("Erreur de validation : {}", e.getMessage());
            return ResponseEntity.badRequest().build();
        }
    }

    // ========== UPDATE ==========

    /**
     * PUT /api/produits/{id}
     * Mettre à jour un produit existant
     */
    @PutMapping("/{id}")
    public ResponseEntity<Produit> updateProduit(
            @PathVariable Long id,
            @RequestBody Produit produitMaj) {
        log.info("PUT /api/produits/{}", id);
        try {
            Produit updated = produitService.updateProduit(id, produitMaj);
            return ResponseEntity.ok(updated);
        } catch (RuntimeException e) {
            return ResponseEntity.notFound().build();
        }
    }

    // ========== DELETE ==========

    /**
     * DELETE /api/produits/{id}
     * Supprimer un produit
     */
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduit(@PathVariable Long id) {
        log.info("DELETE /api/produits/{}", id);
        try {
            produitService.deleteProduit(id);
            return ResponseEntity.noContent().build();
        } catch (RuntimeException e) {
            return ResponseEntity.notFound().build();
        }
    }
}
```

## ✅ CHECKPOINT 8.1

**Vérifiez que** :
- [ ] `ProduitController` est créé dans `controllers/`
- [ ] Les 10 endpoints sont implémentés
- [ ] Les codes HTTP sont corrects (200, 201, 204, 400, 404)
- [ ] La classe compile sans erreurs

---

# PARTIE 9 : CONTRÔLEUR FOURNISSEUR

## 🎯 Objectif

Créer un contrôleur REST pour exposer les opérations CRUD de `Fournisseur` et la gestion de la relation ManyToMany.

## 📌 Tableau des Endpoints

| Méthode HTTP | URL                                                       | Action                         | Réponse |
|-------------|-----------------------------------------------------------|--------------------------------|---------|
| GET         | `/api/fournisseurs`                                       | Lister tous les fournisseurs   | 200     |
| GET         | `/api/fournisseurs/{id}`                                  | Récupérer un fournisseur       | 200/404 |
| GET         | `/api/fournisseurs/search?nom=xxx`                        | Rechercher par nom             | 200     |
| GET         | `/api/fournisseurs/count`                                 | Compter les fournisseurs       | 200     |
| POST        | `/api/fournisseurs`                                       | Créer un fournisseur           | 201     |
| PUT         | `/api/fournisseurs/{id}`                                  | Mettre à jour un fournisseur   | 200/404 |
| DELETE      | `/api/fournisseurs/{id}`                                  | Supprimer un fournisseur       | 204/404 |
| POST        | `/api/fournisseurs/{id}/produits/{idProduit}`             | Lier un produit                | 200     |
| DELETE      | `/api/fournisseurs/{id}/produits/{idProduit}`             | Délier un produit              | 204     |

## 📝 Instructions

### Étape 9.1 : Créer FournisseurController.java

**Chemin** : `src/main/java/tn/iset/produits/controllers/FournisseurController.java`

```java
package tn.iset.produits.controllers;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import tn.iset.produits.entities.Fournisseur;
import tn.iset.produits.services.FournisseurService;

import java.util.List;

/**
 * Contrôleur REST pour l'entité Fournisseur
 *
 * Expose les opérations CRUD et la gestion de la relation ManyToMany
 * avec les produits via HTTP.
 */
@RestController
@RequestMapping("/api/fournisseurs")
@RequiredArgsConstructor
@Slf4j
public class FournisseurController {

    private final FournisseurService fournisseurService;

    // ========== GET ALL ==========

    /**
     * GET /api/fournisseurs
     * Récupérer tous les fournisseurs triés par nom
     */
    @GetMapping
    public ResponseEntity<List<Fournisseur>> getAllFournisseurs() {
        log.info("GET /api/fournisseurs");
        List<Fournisseur> fournisseurs = fournisseurService.findAllOrderByNom();
        return ResponseEntity.ok(fournisseurs);
    }

    // ========== GET BY ID ==========

    /**
     * GET /api/fournisseurs/{id}
     * Récupérer un fournisseur par son ID
     */
    @GetMapping("/{id}")
    public ResponseEntity<Fournisseur> getFournisseurById(@PathVariable Long id) {
        log.info("GET /api/fournisseurs/{}", id);
        try {
            Fournisseur fournisseur = fournisseurService.getFournisseurById(id);
            return ResponseEntity.ok(fournisseur);
        } catch (RuntimeException e) {
            return ResponseEntity.notFound().build();
        }
    }

    // ========== SEARCH BY NOM ==========

    /**
     * GET /api/fournisseurs/search?nom=xxx
     * Rechercher les fournisseurs par nom
     */
    @GetMapping("/search")
    public ResponseEntity<List<Fournisseur>> searchByNom(
            @RequestParam(name = "nom", defaultValue = "") String nom) {
        log.info("GET /api/fournisseurs/search?nom={}", nom);
        List<Fournisseur> results = fournisseurService.findByNomContains(nom);
        return ResponseEntity.ok(results);
    }

    // ========== COUNT ==========

    /**
     * GET /api/fournisseurs/count
     * Compter le nombre total de fournisseurs
     */
    @GetMapping("/count")
    public ResponseEntity<Long> countFournisseurs() {
        log.info("GET /api/fournisseurs/count");
        return ResponseEntity.ok(fournisseurService.countFournisseurs());
    }

    // ========== CREATE ==========

    /**
     * POST /api/fournisseurs
     * Créer un nouveau fournisseur
     *
     * Corps JSON attendu :
     * {
     *   "nomFournisseur": "TechDistrib",
     *   "email": "contact@techdistrib.tn",
     *   "telephone": "+216 71 000 000",
     *   "adresse": "Tunis, Tunisie"
     * }
     */
    @PostMapping
    public ResponseEntity<Fournisseur> createFournisseur(@RequestBody Fournisseur fournisseur) {
        log.info("POST /api/fournisseurs - Création : {}", fournisseur.getNomFournisseur());
        try {
            Fournisseur saved = fournisseurService.saveFournisseur(fournisseur);
            return ResponseEntity.status(HttpStatus.CREATED).body(saved);
        } catch (IllegalArgumentException e) {
            log.error("Erreur de validation : {}", e.getMessage());
            return ResponseEntity.badRequest().build();
        }
    }

    // ========== UPDATE ==========

    /**
     * PUT /api/fournisseurs/{id}
     * Mettre à jour un fournisseur existant
     */
    @PutMapping("/{id}")
    public ResponseEntity<Fournisseur> updateFournisseur(
            @PathVariable Long id,
            @RequestBody Fournisseur fournisseurMaj) {
        log.info("PUT /api/fournisseurs/{}", id);
        try {
            Fournisseur updated = fournisseurService.updateFournisseur(id, fournisseurMaj);
            return ResponseEntity.ok(updated);
        } catch (RuntimeException e) {
            return ResponseEntity.notFound().build();
        }
    }

    // ========== DELETE ==========

    /**
     * DELETE /api/fournisseurs/{id}
     * Supprimer un fournisseur
     */
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteFournisseur(@PathVariable Long id) {
        log.info("DELETE /api/fournisseurs/{}", id);
        try {
            fournisseurService.deleteFournisseur(id);
            return ResponseEntity.noContent().build();
        } catch (RuntimeException e) {
            return ResponseEntity.notFound().build();
        }
    }

    // ========== GESTION MANYTOMANY ==========

    /**
     * POST /api/fournisseurs/{id}/produits/{idProduit}
     * Lier un produit à ce fournisseur (association ManyToMany)
     *
     * Exemple : POST /api/fournisseurs/1/produits/3
     * → Le produit 3 est maintenant fourni par le fournisseur 1
     */
    @PostMapping("/{id}/produits/{idProduit}")
    public ResponseEntity<String> addProduitToFournisseur(
            @PathVariable Long id,
            @PathVariable Long idProduit) {
        log.info("POST /api/fournisseurs/{}/produits/{} - Association", id, idProduit);
        try {
            fournisseurService.addProduitToFournisseur(id, idProduit);
            return ResponseEntity.ok(
                "Produit " + idProduit + " associé au fournisseur " + id
            );
        } catch (RuntimeException e) {
            log.error("Erreur d'association : {}", e.getMessage());
            return ResponseEntity.notFound().build();
        }
    }

    /**
     * DELETE /api/fournisseurs/{id}/produits/{idProduit}
     * Délier un produit de ce fournisseur (dissociation ManyToMany)
     *
     * Exemple : DELETE /api/fournisseurs/1/produits/3
     * → Le produit 3 n'est plus fourni par le fournisseur 1
     */
    @DeleteMapping("/{id}/produits/{idProduit}")
    public ResponseEntity<Void> removeProduitFromFournisseur(
            @PathVariable Long id,
            @PathVariable Long idProduit) {
        log.info("DELETE /api/fournisseurs/{}/produits/{} - Dissociation", id, idProduit);
        try {
            fournisseurService.removeProduitFromFournisseur(id, idProduit);
            return ResponseEntity.noContent().build();
        } catch (RuntimeException e) {
            return ResponseEntity.notFound().build();
        }
    }
}
```

## ✅ CHECKPOINT 9.1

**Vérifiez que** :
- [ ] `FournisseurController` est créé dans `controllers/`
- [ ] Les 9 endpoints sont implémentés
- [ ] Les endpoints ManyToMany (POST et DELETE `/produits/{idProduit}`) fonctionnent
- [ ] Les codes HTTP sont corrects (200, 201, 204, 400, 404)
- [ ] La classe compile sans erreurs

---

# PARTIE 10 : TESTS UNITAIRES

## 🎯 Objectif

Créer des tests JUnit 5 pour valider les services et les contrôleurs.

## 📝 Instructions

### Étape 10.1 : Tests pour FournisseurService

**Chemin** : `src/test/java/tn/iset/produits/services/FournisseurServiceTest.java`

```java
package tn.iset.produits.services;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import tn.iset.produits.entities.Categorie;
import tn.iset.produits.entities.Fournisseur;
import tn.iset.produits.entities.Produit;
import tn.iset.produits.repositories.FournisseurRepository;
import tn.iset.produits.repositories.ProduitRepository;
import tn.iset.produits.repositories.CategorieRepository;

import java.util.List;

import static org.assertj.core.api.Assertions.*;
import static org.junit.jupiter.api.Assertions.*;

/**
 * Tests JUnit 5 pour FournisseurService
 * Pattern AAA (Arrange-Act-Assert)
 */
@SpringBootTest
class FournisseurServiceTest {

    @Autowired
    private FournisseurService fournisseurService;

    @Autowired
    private ProduitService produitService;

    @Autowired
    private CategorieService categorieService;

    @Autowired
    private FournisseurRepository fournisseurRepository;

    @Autowired
    private ProduitRepository produitRepository;

    @Autowired
    private CategorieRepository categorieRepository;

    @BeforeEach
    void setUp() {
        // Nettoyer dans l'ordre (ManyToMany d'abord)
        fournisseurRepository.deleteAll();
        produitRepository.deleteAll();
        categorieRepository.deleteAll();
    }

    // ==================== TESTS CRUD ====================

    @Test
    @DisplayName("Créer un fournisseur valide")
    void testSaveFournisseurSuccess() {
        // ARRANGE
        Fournisseur fournisseur = Fournisseur.builder()
            .nomFournisseur("TechDistrib")
            .email("contact@techdistrib.tn")
            .telephone("+216 71 000 000")
            .adresse("Tunis, Tunisie")
            .build();

        // ACT
        Fournisseur saved = fournisseurService.saveFournisseur(fournisseur);

        // ASSERT
        assertThat(saved).isNotNull();
        assertThat(saved.getIdFournisseur()).isGreaterThan(0);
        assertThat(saved.getNomFournisseur()).isEqualTo("TechDistrib");
    }

    @Test
    @DisplayName("Créer un fournisseur avec nom NULL doit échouer")
    void testSaveFournisseurWithNullNom() {
        // ARRANGE
        Fournisseur fournisseur = new Fournisseur();
        fournisseur.setNomFournisseur(null);

        // ACT & ASSERT
        assertThrows(IllegalArgumentException.class, () -> {
            fournisseurService.saveFournisseur(fournisseur);
        });
    }

    @Test
    @DisplayName("Récupérer un fournisseur par ID")
    void testGetFournisseurById() {
        // ARRANGE
        Fournisseur saved = fournisseurService.saveFournisseur(
            Fournisseur.builder().nomFournisseur("GlobalSupply").build()
        );

        // ACT
        Fournisseur found = fournisseurService.getFournisseurById(saved.getIdFournisseur());

        // ASSERT
        assertThat(found).isNotNull();
        assertThat(found.getNomFournisseur()).isEqualTo("GlobalSupply");
    }

    @Test
    @DisplayName("Récupérer tous les fournisseurs")
    void testGetAllFournisseurs() {
        // ARRANGE
        fournisseurService.saveFournisseur(
            Fournisseur.builder().nomFournisseur("Fourn1").build());
        fournisseurService.saveFournisseur(
            Fournisseur.builder().nomFournisseur("Fourn2").build());

        // ACT
        List<Fournisseur> all = fournisseurService.getAllFournisseurs();

        // ASSERT
        assertThat(all).hasSize(2);
    }

    @Test
    @DisplayName("Mettre à jour un fournisseur")
    void testUpdateFournisseur() {
        // ARRANGE
        Fournisseur saved = fournisseurService.saveFournisseur(
            Fournisseur.builder().nomFournisseur("OldName").build()
        );
        Fournisseur updates = Fournisseur.builder().nomFournisseur("NewName").build();

        // ACT
        Fournisseur updated = fournisseurService.updateFournisseur(
            saved.getIdFournisseur(), updates);

        // ASSERT
        assertThat(updated.getNomFournisseur()).isEqualTo("NewName");
    }

    @Test
    @DisplayName("Supprimer un fournisseur")
    void testDeleteFournisseur() {
        // ARRANGE
        Fournisseur saved = fournisseurService.saveFournisseur(
            Fournisseur.builder().nomFournisseur("ToDelete").build()
        );

        // ACT
        fournisseurService.deleteFournisseur(saved.getIdFournisseur());

        // ASSERT
        assertThrows(RuntimeException.class, () -> {
            fournisseurService.getFournisseurById(saved.getIdFournisseur());
        });
    }

    // ==================== TESTS MANYTOMANY ====================

    @Test
    @DisplayName("Associer un produit à un fournisseur (ManyToMany)")
    void testAddProduitToFournisseur() {
        // ARRANGE
        Categorie cat = categorieService.saveCategorie(
            Categorie.builder().nomCat("Électronique").build()
        );
        Produit produit = produitService.saveProduit(
            Produit.builder().nomProduit("Laptop").prixProduit(1200.0)
                .categorie(cat).build()
        );
        Fournisseur fournisseur = fournisseurService.saveFournisseur(
            Fournisseur.builder().nomFournisseur("TechDistrib").build()
        );

        // ACT
        fournisseurService.addProduitToFournisseur(
            fournisseur.getIdFournisseur(), produit.getIdProduit()
        );

        // ASSERT
        List<Produit> produitsDuFournisseur =
            produitService.findByFournisseurId(fournisseur.getIdFournisseur());
        assertThat(produitsDuFournisseur).hasSize(1);
        assertThat(produitsDuFournisseur.get(0).getNomProduit()).isEqualTo("Laptop");
    }

    @Test
    @DisplayName("Un produit peut avoir plusieurs fournisseurs (ManyToMany)")
    void testProduitPlusieurseFournisseurs() {
        // ARRANGE
        Categorie cat = categorieService.saveCategorie(
            Categorie.builder().nomCat("Cat").build()
        );
        Produit produit = produitService.saveProduit(
            Produit.builder().nomProduit("Produit Commun").prixProduit(500.0)
                .categorie(cat).build()
        );
        Fournisseur f1 = fournisseurService.saveFournisseur(
            Fournisseur.builder().nomFournisseur("Fournisseur A").build()
        );
        Fournisseur f2 = fournisseurService.saveFournisseur(
            Fournisseur.builder().nomFournisseur("Fournisseur B").build()
        );

        // ACT
        fournisseurService.addProduitToFournisseur(
            f1.getIdFournisseur(), produit.getIdProduit()
        );
        fournisseurService.addProduitToFournisseur(
            f2.getIdFournisseur(), produit.getIdProduit()
        );

        // ASSERT
        List<Produit> produitsF1 = produitService.findByFournisseurId(f1.getIdFournisseur());
        List<Produit> produitsF2 = produitService.findByFournisseurId(f2.getIdFournisseur());
        assertThat(produitsF1).hasSize(1);
        assertThat(produitsF2).hasSize(1);
    }

    @Test
    @DisplayName("Dissocier un produit d'un fournisseur")
    void testRemoveProduitFromFournisseur() {
        // ARRANGE
        Categorie cat = categorieService.saveCategorie(
            Categorie.builder().nomCat("Cat2").build()
        );
        Produit produit = produitService.saveProduit(
            Produit.builder().nomProduit("Produit X").prixProduit(300.0)
                .categorie(cat).build()
        );
        Fournisseur fournisseur = fournisseurService.saveFournisseur(
            Fournisseur.builder().nomFournisseur("Fourn X").build()
        );
        fournisseurService.addProduitToFournisseur(
            fournisseur.getIdFournisseur(), produit.getIdProduit()
        );

        // ACT
        fournisseurService.removeProduitFromFournisseur(
            fournisseur.getIdFournisseur(), produit.getIdProduit()
        );

        // ASSERT
        List<Produit> produits =
            produitService.findByFournisseurId(fournisseur.getIdFournisseur());
        assertThat(produits).isEmpty();
    }

    @Test
    @DisplayName("Trouver les produits sans fournisseur")
    void testFindProduitsWithoutFournisseur() {
        // ARRANGE
        Categorie cat = categorieService.saveCategorie(
            Categorie.builder().nomCat("Cat3").build()
        );
        produitService.saveProduit(
            Produit.builder().nomProduit("Produit Orphelin").prixProduit(200.0)
                .categorie(cat).build()
        );

        // ACT
        List<Produit> sansF = produitService.findProduitsWithoutFournisseur();

        // ASSERT
        assertThat(sansF).isNotEmpty();
        assertThat(sansF).allMatch(p -> p.getFournisseurs().isEmpty());
    }
}
```

### Étape 10.2 : Tests pour CategorieController avec MockMvc

**Chemin** : `src/test/java/tn/iset/produits/controllers/CategorieControllerTest.java`

```java
package tn.iset.produits.controllers;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import tn.iset.produits.entities.Categorie;
import tn.iset.produits.services.CategorieService;

import java.util.List;

import static org.mockito.ArgumentMatchers.*;
import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

/**
 * Tests MockMvc pour CategorieController
 *
 * @WebMvcTest : Lance uniquement la couche Web (pas le contexte Spring complet)
 * @MockBean   : Remplace le service par un mock (pas de BD réelle)
 */
@WebMvcTest(CategorieController.class)
class CategorieControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private CategorieService categorieService;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @DisplayName("GET /api/categories → 200 + liste")
    void testGetAllCategories() throws Exception {
        // ARRANGE
        List<Categorie> cats = List.of(
            Categorie.builder().idCat(1L).nomCat("Électronique").build(),
            Categorie.builder().idCat(2L).nomCat("Vêtements").build()
        );
        when(categorieService.findAllOrderByNom()).thenReturn(cats);

        // ACT & ASSERT
        mockMvc.perform(get("/api/categories"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.length()").value(2))
            .andExpect(jsonPath("$[0].nomCat").value("Électronique"));
    }

    @Test
    @DisplayName("GET /api/categories/{id} → 200 + catégorie")
    void testGetCategorieByIdFound() throws Exception {
        // ARRANGE
        Categorie cat = Categorie.builder().idCat(1L).nomCat("Électronique").build();
        when(categorieService.getCategorieById(1L)).thenReturn(cat);

        // ACT & ASSERT
        mockMvc.perform(get("/api/categories/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.nomCat").value("Électronique"));
    }

    @Test
    @DisplayName("GET /api/categories/{id} → 404 si non trouvé")
    void testGetCategorieByIdNotFound() throws Exception {
        // ARRANGE
        when(categorieService.getCategorieById(99L))
            .thenThrow(new RuntimeException("Non trouvé"));

        // ACT & ASSERT
        mockMvc.perform(get("/api/categories/99"))
            .andExpect(status().isNotFound());
    }

    @Test
    @DisplayName("POST /api/categories → 201 + catégorie créée")
    void testCreateCategorie() throws Exception {
        // ARRANGE
        Categorie input = Categorie.builder().nomCat("Nouvelles").build();
        Categorie saved = Categorie.builder().idCat(5L).nomCat("Nouvelles").build();
        when(categorieService.saveCategorie(any())).thenReturn(saved);

        // ACT & ASSERT
        mockMvc.perform(post("/api/categories")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(input)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.idCat").value(5))
            .andExpect(jsonPath("$.nomCat").value("Nouvelles"));
    }

    @Test
    @DisplayName("DELETE /api/categories/{id} → 204 No Content")
    void testDeleteCategorie() throws Exception {
        // ARRANGE
        doNothing().when(categorieService).deleteCategorie(1L);

        // ACT & ASSERT
        mockMvc.perform(delete("/api/categories/1"))
            .andExpect(status().isNoContent());
    }
}
```

### Étape 10.3 : Tests pour FournisseurController avec MockMvc

**Chemin** : `src/test/java/tn/iset/produits/controllers/FournisseurControllerTest.java`

```java
package tn.iset.produits.controllers;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import tn.iset.produits.entities.Fournisseur;
import tn.iset.produits.services.FournisseurService;

import java.util.List;

import static org.mockito.ArgumentMatchers.*;
import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(FournisseurController.class)
class FournisseurControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private FournisseurService fournisseurService;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @DisplayName("GET /api/fournisseurs → 200 + liste")
    void testGetAllFournisseurs() throws Exception {
        List<Fournisseur> list = List.of(
            Fournisseur.builder().idFournisseur(1L).nomFournisseur("TechDistrib").build(),
            Fournisseur.builder().idFournisseur(2L).nomFournisseur("GlobalSupply").build()
        );
        when(fournisseurService.findAllOrderByNom()).thenReturn(list);

        mockMvc.perform(get("/api/fournisseurs"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.length()").value(2));
    }

    @Test
    @DisplayName("POST /api/fournisseurs → 201 Created")
    void testCreateFournisseur() throws Exception {
        Fournisseur input = Fournisseur.builder()
            .nomFournisseur("Nouveau").email("n@n.tn").build();
        Fournisseur saved = Fournisseur.builder()
            .idFournisseur(10L).nomFournisseur("Nouveau").email("n@n.tn").build();
        when(fournisseurService.saveFournisseur(any())).thenReturn(saved);

        mockMvc.perform(post("/api/fournisseurs")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(input)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.idFournisseur").value(10));
    }

    @Test
    @DisplayName("POST /api/fournisseurs/{id}/produits/{idProduit} → 200")
    void testAddProduitToFournisseur() throws Exception {
        doNothing().when(fournisseurService).addProduitToFournisseur(1L, 2L);

        mockMvc.perform(post("/api/fournisseurs/1/produits/2"))
            .andExpect(status().isOk());
    }

    @Test
    @DisplayName("DELETE /api/fournisseurs/{id}/produits/{idProduit} → 204")
    void testRemoveProduitFromFournisseur() throws Exception {
        doNothing().when(fournisseurService).removeProduitFromFournisseur(1L, 2L);

        mockMvc.perform(delete("/api/fournisseurs/1/produits/2"))
            .andExpect(status().isNoContent());
    }

    @Test
    @DisplayName("DELETE /api/fournisseurs/{id} → 204 No Content")
    void testDeleteFournisseur() throws Exception {
        doNothing().when(fournisseurService).deleteFournisseur(1L);

        mockMvc.perform(delete("/api/fournisseurs/1"))
            .andExpect(status().isNoContent());
    }
}
```

## ✅ CHECKPOINT 10.1

**Vérifiez que** :
- [ ] `FournisseurServiceTest` a au moins 8 tests (dont 4 sur ManyToMany)
- [ ] `CategorieControllerTest` a au moins 5 tests MockMvc
- [ ] `FournisseurControllerTest` a au moins 5 tests MockMvc (dont 2 sur ManyToMany)
- [ ] Le pattern AAA est respecté
- [ ] Tous les tests passent (`mvn test`)

---

# PARTIE 11 : INITIALISATION ET AFFICHAGE

## 🎯 Objectif

Initialiser les données de test incluant les fournisseurs et les associations ManyToMany.

## 📝 Instructions

### Étape 11.1 : Modifier ProduitsApplication.java

```java
package tn.iset.produits;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import tn.iset.produits.entities.Categorie;
import tn.iset.produits.entities.Fournisseur;
import tn.iset.produits.entities.Produit;
import tn.iset.produits.services.CategorieService;
import tn.iset.produits.services.FournisseurService;
import tn.iset.produits.services.ProduitService;

import java.time.LocalDate;
import java.util.List;

@SpringBootApplication
@Slf4j
@RequiredArgsConstructor
public class ProduitsApplication implements CommandLineRunner {

    private final ProduitService produitService;
    private final CategorieService categorieService;
    private final FournisseurService fournisseurService;

    public static void main(String[] args) {
        SpringApplication.run(ProduitsApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        log.info("========================================");
        log.info("🚀 Initialisation de l'application...");
        log.info("========================================");

        if (produitService.countProduits() > 0) {
            log.info("✅ Données existantes détectées. Initialisation skippée.");
            afficherDonnees();
            return;
        }

        // ===== CATÉGORIES =====
        log.info("\n📁 Création des catégories...");
        Categorie catElectronique = categorieService.saveCategorie(
            Categorie.builder()
                .nomCat("Électronique")
                .descriptionCat("Produits électroniques et informatiques")
                .build()
        );
        Categorie catVetements = categorieService.saveCategorie(
            Categorie.builder()
                .nomCat("Vêtements")
                .descriptionCat("Vêtements et accessoires")
                .build()
        );
        Categorie catChaussures = categorieService.saveCategorie(
            Categorie.builder()
                .nomCat("Chaussures")
                .descriptionCat("Chaussures de tous les styles")
                .build()
        );
        log.info("✅ {} catégories créées", categorieService.countCategories());

        // ===== PRODUITS =====
        log.info("\n📦 Création des produits...");
        Produit iphone = produitService.saveProduit(Produit.builder()
            .nomProduit("iPhone 15 Pro")
            .prixProduit(1099.0)
            .dateCreation(LocalDate.now())
            .categorie(catElectronique)
            .build());

        Produit samsung = produitService.saveProduit(Produit.builder()
            .nomProduit("Samsung Galaxy S24")
            .prixProduit(899.0)
            .dateCreation(LocalDate.now())
            .categorie(catElectronique)
            .build());

        Produit tshirt = produitService.saveProduit(Produit.builder()
            .nomProduit("T-Shirt Coton")
            .prixProduit(29.99)
            .dateCreation(LocalDate.now())
            .categorie(catVetements)
            .build());

        Produit sneakers = produitService.saveProduit(Produit.builder()
            .nomProduit("Sneakers Nike")
            .prixProduit(89.99)
            .dateCreation(LocalDate.now())
            .categorie(catChaussures)
            .build());

        log.info("✅ {} produits créés", produitService.countProduits());

        // ===== FOURNISSEURS =====
        log.info("\n🏭 Création des fournisseurs...");
        Fournisseur techDistrib = fournisseurService.saveFournisseur(
            Fournisseur.builder()
                .nomFournisseur("TechDistrib")
                .email("contact@techdistrib.tn")
                .telephone("+216 71 000 001")
                .adresse("Centre Urbain Nord, Tunis")
                .build()
        );
        Fournisseur globalSupply = fournisseurService.saveFournisseur(
            Fournisseur.builder()
                .nomFournisseur("GlobalSupply")
                .email("info@globalsupply.tn")
                .telephone("+216 71 000 002")
                .adresse("La Marsa, Tunis")
                .build()
        );
        Fournisseur fashionPro = fournisseurService.saveFournisseur(
            Fournisseur.builder()
                .nomFournisseur("FashionPro")
                .email("vente@fashionpro.tn")
                .telephone("+216 71 000 003")
                .adresse("Sfax, Tunisie")
                .build()
        );
        log.info("✅ {} fournisseurs créés", fournisseurService.countFournisseurs());

        // ===== ASSOCIATIONS MANYTOMANY =====
        log.info("\n🔗 Création des associations Produit ↔ Fournisseur...");

        // iPhone fourni par TechDistrib ET GlobalSupply
        fournisseurService.addProduitToFournisseur(
            techDistrib.getIdFournisseur(), iphone.getIdProduit());
        fournisseurService.addProduitToFournisseur(
            globalSupply.getIdFournisseur(), iphone.getIdProduit());

        // Samsung fourni par GlobalSupply uniquement
        fournisseurService.addProduitToFournisseur(
            globalSupply.getIdFournisseur(), samsung.getIdProduit());

        // T-Shirt ET Sneakers fournis par FashionPro
        fournisseurService.addProduitToFournisseur(
            fashionPro.getIdFournisseur(), tshirt.getIdProduit());
        fournisseurService.addProduitToFournisseur(
            fashionPro.getIdFournisseur(), sneakers.getIdProduit());

        log.info("✅ Associations ManyToMany créées");

        // Afficher les données
        afficherDonnees();
    }

    private void afficherDonnees() {
        log.info("\n========================================");
        log.info("📋 AFFICHAGE DES DONNÉES");
        log.info("========================================");

        // Catégories
        log.info("\n🏷️ CATÉGORIES ({} au total) :", categorieService.countCategories());
        categorieService.findAllOrderByNom().forEach(cat ->
            log.info("  - {} : {} produit(s)", cat.getNomCat(), cat.getProduits().size())
        );

        // Fournisseurs
        log.info("\n🏭 FOURNISSEURS ({} au total) :", fournisseurService.countFournisseurs());
        fournisseurService.findAllOrderByNom().forEach(f ->
            log.info("  - {} ({}) : {} produit(s)",
                f.getNomFournisseur(), f.getEmail(), f.getProduits().size())
        );

        // Produits avec leurs fournisseurs
        log.info("\n📦 PRODUITS ET LEURS FOURNISSEURS :");
        produitService.findAllOrderByCategoryThenNameThenPrice().forEach(p -> {
            List<Fournisseur> fournisseurs =
                fournisseurService.findAllOrderByNom().stream()
                    .filter(f -> f.getProduits().stream()
                        .anyMatch(pr -> pr.getIdProduit().equals(p.getIdProduit())))
                    .toList();
            log.info("  - {} ({}) : {} fournisseur(s)",
                p.getNomProduit(),
                p.getCategorie() != null ? p.getCategorie().getNomCat() : "SANS CAT",
                fournisseurs.size());
        });

        // Produits sans fournisseur
        List<Produit> sansF = produitService.findProduitsWithoutFournisseur();
        if (!sansF.isEmpty()) {
            log.info("\n⚠️  PRODUITS SANS FOURNISSEUR ({}) :", sansF.size());
            sansF.forEach(p -> log.info("  - {}", p.getNomProduit()));
        }

        log.info("\n========================================");
        log.info("✅ API REST disponible sur http://localhost:8080");
        log.info("   GET  /api/produits");
        log.info("   GET  /api/categories");
        log.info("   GET  /api/fournisseurs");
        log.info("========================================\n");
    }
}
```

## ✅ CHECKPOINT 11.1

**Vérifiez que** :
- [ ] `ProduitsApplication.java` est modifiée
- [ ] Les fournisseurs sont créés au démarrage
- [ ] Les associations ManyToMany sont initialisées
- [ ] L'application démarre sans erreurs
- [ ] Les logs d'initialisation s'affichent correctement
- [ ] Les endpoints REST sont accessibles

---

# POINTS CLÉS À RETENIR

## 🎯 Relation ManyToMany

```java
// Owner side (Produit) — crée la table de jointure
@ManyToMany(fetch = FetchType.LAZY)
@JoinTable(
    name = "produit_fournisseur",
    joinColumns = @JoinColumn(name = "id_produit"),
    inverseJoinColumns = @JoinColumn(name = "id_fournisseur")
)
private List<Fournisseur> fournisseurs = new ArrayList<>();

// Inverse side (Fournisseur) — pointe vers l'owner
@ManyToMany(mappedBy = "fournisseurs")
private List<Produit> produits = new ArrayList<>();
```

**Important** :
- Toujours maintenir la cohérence bidirectionnelle avec des méthodes `addX` / `removeX`
- Utiliser `FetchType.LAZY` pour éviter les boucles infinies JSON
- La table de jointure est créée automatiquement par Hibernate

## 🌐 Contrôleurs REST

```java
@RestController              // = @Controller + @ResponseBody
@RequestMapping("/api/...")  // Préfixe de l'URL
public class MonController {

    @GetMapping            // HTTP GET
    @PostMapping           // HTTP POST  → 201 Created
    @PutMapping("/{id}")   // HTTP PUT   → 200 OK
    @DeleteMapping("/{id}")// HTTP DELETE → 204 No Content

    // Extraire l'ID de l'URL
    public ResponseEntity<T> methode(@PathVariable Long id) { ... }

    // Extraire le paramètre de requête (?nom=xxx)
    public ResponseEntity<T> methode(@RequestParam String nom) { ... }

    // Lire le corps JSON
    public ResponseEntity<T> methode(@RequestBody MonEntite entite) { ... }
}
```

## 🧪 Tests MockMvc

```java
@WebMvcTest(MonController.class)    // Contexte Web uniquement
class MonControllerTest {

    @Autowired MockMvc mockMvc;     // Client HTTP simulé
    @MockBean  MonService service;  // Service mocké (pas de BD)

    @Test
    void testGet() throws Exception {
        // ARRANGE
        when(service.findAll()).thenReturn(List.of(...));

        // ACT & ASSERT
        mockMvc.perform(get("/api/ressources"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.length()").value(2));
    }
}
```

---

## ✅ CRITÈRES D'ACCEPTATION

Votre Lab 3 est réussi si :

✅ L'entité `Fournisseur` est créée correctement  
✅ La relation `ManyToMany` fonctionne (table `produit_fournisseur` créée)  
✅ La cohérence bidirectionnelle est maintenue  
✅ Le Repository `Fournisseur` a au moins 7 méthodes  
✅ Le Service `Fournisseur` gère l'association/dissociation ManyToMany  
✅ `CategorieController` expose les 7 endpoints REST  
✅ `ProduitController` expose les 10 endpoints REST  
✅ `FournisseurController` expose les 9 endpoints REST (dont 2 ManyToMany)  
✅ Au moins 18 tests passent (8 FournisseurService + 5 CategorieController + 5 FournisseurController)  
✅ L'application démarre sans erreurs  
✅ Les endpoints sont accessibles via `http://localhost:8080`  
✅ Aucun warning à la compilation  

---

## 🚀 COMMANDES UTILES

```bash
# Compiler le projet
mvn clean compile

# Lancer tous les tests
mvn test

# Lancer un test spécifique
mvn test -Dtest=FournisseurServiceTest
mvn test -Dtest=FournisseurControllerTest

# Lancer l'application
mvn spring-boot:run

# Tester les endpoints avec curl
curl http://localhost:8080/api/categories
curl http://localhost:8080/api/produits
curl http://localhost:8080/api/fournisseurs

# Créer un fournisseur via curl
curl -X POST http://localhost:8080/api/fournisseurs \
  -H "Content-Type: application/json" \
  -d '{"nomFournisseur":"Test","email":"test@test.tn"}'

# Associer un produit au fournisseur 1
curl -X POST http://localhost:8080/api/fournisseurs/1/produits/2

# Nettoyer
mvn clean
```

---

## 💡 CONSEILS POUR RÉUSSIR

1. **Suivez l'ordre** : Ne pas sauter de parties
2. **Attention aux boucles JSON** : `@JsonIgnore` ou `@JsonManagedReference`/`@JsonBackReference` si besoin sur les relations bidirectionnelles
3. **FetchType.LAZY** : Toujours utiliser LAZY sur les ManyToMany pour éviter les N+1 queries
4. **Cohérence bidirectionnelle** : Maintenir la liste des deux côtés dans les méthodes `addX`/`removeX`
5. **MockMvc vs SpringBootTest** : Utiliser `@WebMvcTest` pour les contrôleurs (rapide), `@SpringBootTest` pour les services (avec vraie BD)
6. **Compilez régulièrement** : Vérifier qu'il n'y a pas d'erreurs après chaque partie
7. **Demandez de l'aide** : Si vous êtes bloqué(e)

---

**Bon courage ! 🎓 Vous avez tous les outils pour réussir ce Lab 3 !**
