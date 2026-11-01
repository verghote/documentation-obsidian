# `interface.php`

## Rôle

`interface.php` est le **gabarit principal (layout)** de l'application.

Toutes les pages du site utilisent ce fichier afin de produire une structure HTML homogène.

Il est responsable de :

- construire la structure HTML complète ;
- charger Bootstrap et les feuilles de style communes ;
- charger les ressources spécifiques à la page ;
- intégrer automatiquement le JavaScript associé à la page ;
- transmettre des données PHP au JavaScript ;
- afficher l'en-tête, le contenu et le pied de page.

Il joue donc le rôle de **layout** dans une architecture MVC.

# Architecture générale

Une page est construite suivant le schéma suivant :

```
                interface.php
                     │
        ┌────────────┼────────────┐
        │            │            │
     header.php   page.html   footer.php
```

Le contrôleur prépare un objet `Page`.

`interface.php` utilise ensuite cet objet afin de construire la page HTML.


# Fonctionnement complet

Le déroulement est le suivant :

```
Contrôleur
        │
        ▼
Préparation de l'objet Page
        │
        ▼
interface.php
        │
        ├── Bootstrap
        ├── CSS
        ├── JS
        ├── Token CSRF
        ├── Données JSON
        ├── Header
        ├── Page HTML
        └── Footer
```

---

# Pourquoi cette architecture ?

Cette organisation présente plusieurs avantages.

## Séparation des responsabilités

- Le contrôleur prépare les données.
- Le layout (structure générale de la page ou gabarit) construit la page.
- Les fichiers HTML décrivent uniquement le contenu.
- Le JavaScript gère les interactions.

Chaque composant possède une responsabilité claire.

---
## Convention plutôt que configuration

Le framework applique une convention simple :

```
client.php
↓
client.html
↓
client.js
```

Il n'est pas nécessaire de déclarer explicitement ces fichiers.

Le framework les détecte automatiquement.

---
## Mutualisation

Les éléments communs :

- Bootstrap
- Header
- Footer
- Style général

ne sont écrits qu'une seule fois.

---
## Évolutivité

L'ajout d'une nouvelle page nécessite généralement :

```
produit.php
produit.html
produit.js (facultatif)
```

Aucune modification du layout n'est nécessaire.

---

# Résumé

`interface.php` est le **layout principal** du framework.

Il centralise :

- la structure HTML ;
- le chargement des ressources ;
- la sécurité CSRF ;
- la communication PHP ⇄ JavaScript ;
- l'intégration des composants communs.

Il applique le principe :

> **Le contrôleur prépare les données ; le layout construit la page.**

Cette séparation simplifie le développement, améliore la maintenabilité et favorise une architecture MVC claire.