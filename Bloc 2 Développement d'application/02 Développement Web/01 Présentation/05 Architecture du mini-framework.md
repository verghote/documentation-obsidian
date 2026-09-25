# Architecture du mini-framework PHP

## 1. Philosophie

Ce mini-framework repose sur un principe simple :

> **Une organisation conventionnelle des fichiers permet de limiter la configuration et de simplifier le développement.**

Il ne cherche pas à reproduire un framework généraliste complexe.

L'objectif est de disposer d'une architecture :

- simple à comprendre ;
- rapide à utiliser ;
- cohérente ;
- facilement maintenable ;
- capable d'évoluer progressivement.

Le framework fournit les mécanismes communs nécessaires à l'application, mais laisse au développeur la maîtrise du code PHP, HTML et JavaScript.
### Principe essentiel

> **Ne pas ajouter une abstraction tant qu'elle ne répond pas à un besoin réel.**

Les couches `Service` et `Validateur`, par exemple, sont utilisées lorsque la complexité du projet le justifie, et non systématiquement.

# 2. Organisation générale

Le projet est organisé autour de modules fonctionnels.

```text
projet/
│
├── public/
│   │
│   ├── categorie/
│   │   ├── liste/
│   │   ├── ajouter/
│   │   └── modifier/
│   │
│   ├── coureur/
│   │   └── ...
│   │
│   └── composant/
│
├── src/
│   ├── ClasseTechnique/
│   ├── ClasseMetier/
│   ├── Service/
│   └── Validateur/
│
├── config/
│
├── view/
│   ├── interface.php
│   ├── header.php
│   └── footer.php
│
└── bootstrap/
    └── autoload.php
```

La structure physique du projet reflète donc sa structure fonctionnelle.

# 3. Une interface = un répertoire

Une interface Web possède son propre répertoire.

```text
liste/
├── index.php
├── index.html
├── index.js
└── ajax/
    ├── ajouter.php
    ├── modifier.php
    └── supprimer.php
```

Les fichiers sont facultatifs selon les besoins.

|Fichier|Rôle|
|---|---|
|`index.php`|Prépare et lance la page|
|`index.html`|Contenu HTML spécifique|
|`index.js`|Comportement JavaScript|
|`ajax/`|Traitements serveur appelés par JavaScript|

Cette convention est au cœur du fonctionnement du framework.

# 4. Le rôle des fichiers

## `index.php`

C'est le contrôleur de l'interface.

Il :
- récupère les données nécessaires ;
- configure la classe `Page` ;
- transmet éventuellement des données à JavaScript ;
- active les mécanismes nécessaires ;
- déclenche l'affichage.

Exemple :

```php
$page = new Page();

$page
    ->setTitre("Liste des catégories")
    ->setDonnee("lesCategories", Categorie::getAll())
    ->avecJeton()
    ->afficher();
```

Le contrôleur **prépare la page**.

## `index.html`

Il contient uniquement le HTML propre à l'interface.

```html
<h1>Liste des catégories</h1>

<table id="lesCategories">
</table>
```

La structure générale du document HTML est fournie par le framework.

Le développeur ne reconstruit donc pas le `<html>`, `<head>`, `<body>`, les menus, le header, etc.

## `index.js`

Il contient le comportement de l'interface :

- événements ;
- manipulation du DOM ;
- interactions utilisateur ;
- appels AJAX ;
- traitement des données côté navigateur.

Exemple :

```javascript
const lesCategories = getData("lesCategories");
```

## `ajax/`

Ce répertoire contient les traitements serveur déclenchés depuis JavaScript.

```text
ajax/
├── ajouter.php
├── modifier.php
└── supprimer.php
```

Chaque fichier correspond à une opération.

# 5. Fonctionnement d'une page

Le cycle d'une page est volontairement simple :

```text
Navigateur
    │
    │ HTTP
    ▼
index.php
    │
    ▼
Page
    │
    ├── titre
    ├── données
    ├── ressources
    └── CSRF
    │
    ▼
interface.php
    │
    ├── structure commune
    ├── menus
    ├── index.html
    └── index.js
    │
    ▼
Page HTML
    │
    ▼
Navigateur
```

La séparation des responsabilités est donc :

```text
PHP         → prépare la page
HTML        → décrit l'interface
JavaScript  → gère les interactions
```

# 6. PHP vers JavaScript

Lorsqu'une donnée est déjà disponible côté serveur, elle peut être transmise directement à JavaScript.
### PHP

```php
$page->setDonnee("lesCategories", Categorie::getAll());
```
### JavaScript

```javascript
const lesCategories = getData("lesCategories");
```

Il n'est donc pas nécessaire de faire un appel AJAX simplement pour récupérer une donnée déjà connue lors du chargement de la page.

# 7. Communication AJAX

Lorsqu'une interaction nécessite une communication avec le serveur après le chargement de la page :

```text
index.js
   │
   │ appelAjax()
   ▼
ajax/traitement.php
   │
   ▼
Requete
   │
   ▼
Service / Métier
   │
   ▼
ReponseJson
   │
   ▼
index.js
```

Le JavaScript utilise :

```javascript
appelAjax(...)
```

Les contrôleurs AJAX utilisent les classes techniques du framework pour :

- contrôler la requête ;
- récupérer les paramètres ;
- vérifier le CSRF ;
- appeler le traitement nécessaire ;
- retourner une réponse JSON.

# 8. Les classes du framework

Les classes sont organisées selon leur responsabilité.
## Classes techniques

```text
src/ClasseTechnique/
```

Elles fournissent les mécanismes génériques du framework.

Principales classes :

```text
Page
InterfaceHtml
Requete
Ajax
Jeton
ReponseJson
Erreur
```

Elles ne contiennent pas la logique propre à l'application.

## Classes métier

```text
src/ClasseMetier/
```

Elles représentent les objets et opérations propres à l'application.

Exemples :

```text
Coureur
Course
Categorie
Club
```

## Services

```text
src/Service/
```

Ils servent à orchestrer un traitement lorsque celui-ci devient suffisamment complexe pour justifier une couche supplémentaire.

```text
Contrôleur
    ↓
Service
    ↓
Métier
```

**Un service n'est pas obligatoire.**

## Validateurs

```text
src/Validateur/
```

Ils regroupent les règles de validation réutilisables.

```text
Contrôleur
    ↓
Validateur
    ↓
Métier
```

**Un validateur n'est pas obligatoire.**


# 9. Vue synthétique de l'architecture

```text
                         NAVIGATEUR
                              │
                    ┌─────────┴─────────┐
                    │                   │
                 Page HTTP          AJAX HTTP
                    │                   │
                    ▼                   ▼
                index.php          ajax/*.php
                    │                   │
                    │             ┌─────┴─────┐
                    │             │           │
                    │          Service     Métier
                    │          facultatif
                    │
                    ▼
                   Page
                    │
                    ▼
             InterfaceHtml
                    │
                    ▼
              Vue commune
                    │
                    ▼
             HTML + JavaScript
```

Les classes techniques sont utilisées transversalement pour fournir les mécanismes communs :

```text
Page
InterfaceHtml
Requete
Ajax
Jeton
ReponseJson
Erreur
```

# 10. Exemple d'organisation

Pour une gestion des catégories :

```text
categorie/
│
├── liste/
│   ├── index.php
│   ├── index.html
│   ├── index.js
│   └── ajax/
│       └── supprimer.php
│
├── ajouter/
│   ├── index.php
│   ├── index.html
│   └── index.js
│
└── modifier/
    ├── index.php
    ├── index.html
    └── index.js
```

Le développeur sait immédiatement :

- où se trouve chaque interface ;
- où se trouve son HTML ;
- où se trouve son JavaScript ;
- où se trouvent ses traitements AJAX.

Il n'a pas besoin de consulter une configuration pour comprendre l'organisation du projet.

# 11. Les règles à retenir

L'architecture peut être résumée en quelques règles.
### Organisation

> **Un module regroupe un domaine fonctionnel.**

> **Une interface possède son propre répertoire.**

> **Les conventions de nommage et d'emplacement remplacent une grande partie de la configuration.**

### Page

> **`index.php` prépare la page.**

> **`index.html` contient son HTML.**

> **`index.js` gère son comportement.**

> **`ajax/` contient les traitements serveur déclenchés par JavaScript.**

### Architecture

> **Les classes techniques fournissent les mécanismes communs.**

> **Les classes métier portent la logique propre à l'application.**

> **Les services et validateurs sont ajoutés uniquement lorsque cela est nécessaire.**

### Principe général

> **La complexité de l'architecture doit rester proportionnelle à la complexité du projet.**

# 12. En une phrase

Le fonctionnement du framework peut être résumé ainsi :

> **Une organisation conventionnelle des répertoires permet de séparer clairement le contrôleur PHP, le HTML, le JavaScript et les traitements AJAX, tandis que les classes techniques centralisent les mécanismes communs et que les couches métier supplémentaires ne sont introduites qu'en fonction des besoins.**

