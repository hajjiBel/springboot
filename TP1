# Workshop 1 – Spring Boot MVC (Niveau débutant)

## 🎯 Objectifs pédagogiques

À la fin de ce workshop, vous serez capable de :

- Créer un projet Spring Boot avec Spring Web et Thymeleaf. [file:9]
- Comprendre la structure d’un projet Spring Boot (classe main, packages, dossiers `static` et `templates`, `pom.xml`). [file:9]
- Créer un premier contrôleur MVC, une vue, et échanger des données entre contrôleur et vue. [file:9]
- Utiliser Thymeleaf pour afficher des données dynamiques et intégrer un fichier CSS. [file:9]
- Construire une mini-application MVC de gestion de personnes (formulaire + liste). [file:9]
- Écrire quelques tests de la couche MVC avec Spring Boot et JUnit (en option). [file:9]

---

## 0. Pré-requis

- Connaissances de base en Java (classes, méthodes, packages). [file:9]
- Java 17+ installé (vérifier avec `java -version`). [file:9]
- IDE : **Spring Tool Suite (STS)** recommandé, ou IntelliJ / VS Code avec extensions Spring. [file:9]
- Accès à Internet pour utiliser **start.spring.io**. [file:9]

> 💡 Question de réflexion  
> Expliquez en 2 phrases la différence entre :  
> - une application Java “classique” (avec `public static void main`)  
> - une application web Spring Boot.

---

## 1. Découvrir Spring, Spring Boot et MVC

### 1.1. Intuition

- Spring est un framework Java qui simplifie le développement d’applications modernes (web, microservices, etc.). [file:9]
- Spring Boot est une surcouche qui réduit énormément la configuration (auto-configuration, serveur embarqué, etc.). [file:9]
- Spring MVC implémente le modèle **Modèle – Vue – Contrôleur** pour les applications web. [file:9]

### 1.2. Ressources officielles (à montrer en cours)

- Site des projets Spring : https://spring.io/projects [file:9]  
- Spring Boot : https://spring.io/projects/spring-boot [file:9]

> ❓ Question intermédiaire  
> En une phrase : à quoi sert Spring Boot par rapport au “Spring classique” avec beaucoup de XML ?

---

## 2. Création du projet Spring Boot

### 2.1. Utilisation de Spring Initializr

1. Aller sur : https://start.spring.io [file:9]
2. Paramétrer le projet :
   - Project : Maven
   - Language : Java
   - Spring Boot : version stable 3.x
   - Group : `com.example`
   - Artifact : `springboot-mvc-workshop`
3. Cliquer sur **Add Dependencies** et choisir :
   - `Spring Web`
   - `Thymeleaf`
   - (Optionnel) `Spring Boot DevTools` [file:9]
4. Générer le projet (`Generate`) et extraire le .zip, puis l’importer dans l’IDE.

> ❓ Question intermédiaire  
> Quelles sont les 3 dépendances principales que vous avez ajoutées, et à quoi servent-elles ?

### 2.2. Schéma de la structure du projet

Une fois le projet importé, vous devez voir quelque chose comme :

```text
springboot-mvc-workshop
 ├─ src
 │  ├─ main
 │  │  ├─ java
 │  │  │   └─ com.example.springbootmvc
 │  │  │        └─ SpringbootMvcWorkshopApplication.java   (classe main)
 │  │  └─ resources
 │  │       ├─ static/          (CSS, images, JS…)
 │  │       ├─ templates/       (vues Thymeleaf)
 │  │       └─ application.properties
 │  └─ test
 │      └─ java                 (tests unitaires)
 └─ pom.xml                     (gestion des dépendances Maven)
