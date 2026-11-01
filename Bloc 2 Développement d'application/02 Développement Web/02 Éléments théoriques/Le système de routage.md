
# Le système de routage

## Principe

Le routage consiste à faire transiter toutes les requêtes HTTP par un **point d'entrée unique** appelé *front controller*.

Au lieu d'appeler directement un script PHP, le navigateur envoie une URL "virtuelle" :

```
/ajax/coureur/getByCategorie
```

Le serveur Web (Apache ou Nginx) redirige alors cette requête vers un unique fichier, par exemple :

```
/ajax/index.php
```

À l'aide d'une règle de réécriture (*URL rewriting*).

Le front controller analyse ensuite l'URL reçue afin de déterminer quel contrôleur doit être exécuté.

Le schéma de fonctionnement est le suivant :

```
Navigateur
      │
      ▼
URL : /ajax/coureur/getByCategorie
      │
      ▼
Apache (RewriteRule)
      │
      ▼
ajax/index.php
      │
      ▼
Analyse de l'URL
      │
      ▼
Contrôleur correspondant
      │
      ▼
Réponse JSON
```

---

# Exemple

Sans routage :

```
appelAjax()
        │
        ▼
ajax/getlescoureurs.php
```

Avec routage :

```
appelAjax()
        │
        ▼
/ajax/coureur/getlescoureurs
        │
        ▼
Apache
        │
        ▼
ajax/index.php
        │
        ▼
require controleur/coureur/getlescoureurs.php
```

Le contrôleur final reste souvent exactement le même.

Seul le chemin permettant d'y accéder change.

---

# Avantages

Le routage présente plusieurs intérêts.

## Un seul point d'entrée

Toutes les requêtes passent par le même fichier.

Il est donc possible d'y centraliser :

- le chargement de l'autoloader ;
- certaines vérifications de sécurité ;
- le traitement des erreurs ;
- la journalisation des accès.

---

## Des URL plus propres

Au lieu d'obtenir :

```
/ajax/getlescoureurs.php
```

on obtient :

```
/ajax/coureur/getlescoureurs
```

Les URL sont plus lisibles et indépendantes de l'organisation physique des fichiers.

---

## Évolution facilitée

Les contrôleurs peuvent être déplacés ou renommés sans modifier les URL publiques.

Le routage fait la correspondance entre l'URL et le fichier réel.

---

## Fonctionnalités avancées

Les grands frameworks utilisent le routage pour gérer :

- les paramètres dans l'URL ;
- les contraintes sur les paramètres ;
- les middlewares ;
- l'authentification ;
- les droits d'accès ;
- les versions d'API.

---

# Inconvénients

Le routage introduit également une couche supplémentaire.

La requête ne va plus directement vers le contrôleur.

Elle passe obligatoirement par un intermédiaire.

Cela implique :

- une configuration du serveur Web ;
- un mécanisme de dispatching ;
- une logique supplémentaire à maintenir.

Le fonctionnement est donc moins immédiat pour un débutant.

---

# Pourquoi ce choix n'a pas été retenu dans ce mini-framework ?

L'objectif du mini-framework est avant tout pédagogique.

Son principe d'organisation est volontairement simple :

```
fonctionnalite/
    index.php
    index.html
    index.js
    ajax/
```

Chaque fonctionnalité constitue un ensemble autonome regroupant :

- le contrôleur de la page ;
- la vue HTML ;
- le code JavaScript ;
- les contrôleurs AJAX associés.

Lorsqu'un développeur ouvre un répertoire, il retrouve immédiatement tous les fichiers liés à cette fonctionnalité.

Cette organisation est très lisible et facilite la maintenance.

L'introduction d'un système de routage n'apporterait que peu de bénéfices :

- les URL ne sont pas destinées à être saisies par l'utilisateur ;
- les scripts AJAX sont appelés uniquement par la fonction `appelAjax()` ;
- le chargement de l'autoloader reste limité à une seule ligne ;
- la sécurisation des contrôleurs est déjà centralisée grâce à la classe `Ajax`.

Le routage ajouterait donc une couche d'abstraction supplémentaire sans simplifier réellement le développement.

# Conclusion

Le système de routage est un élément essentiel des grands frameworks PHP tels que Symfony ou Laravel, où il permet de gérer des centaines de contrôleurs, des URL complexes et de nombreuses fonctionnalités transversales.

Dans ce mini-framework, les besoins sont très différents.

L'organisation des fonctionnalités par répertoire, associée à des contrôleurs AJAX indépendants, offre une architecture :

- simple ;
- explicite ;
- facile à comprendre ;
- facile à maintenir.

La mise en place d'un système de routage alourdirait inutilement cette architecture sans apporter de gain significatif.

Le choix a donc été fait de privilégier la simplicité, conformément à la philosophie générale du framework : **chaque fonctionnalité est autonome et directement accessible, sans couche d'abstraction supplémentaire.**