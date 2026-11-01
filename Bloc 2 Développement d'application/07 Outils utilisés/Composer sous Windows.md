# Introduction

**Composer** est le gestionnaire de dépendances officiel de PHP.

Son rôle principal est de :

- installer des bibliothèques PHP ;
- mettre à jour les dépendances d'un projet ;
- gérer automatiquement le chargement des classes (autoloading) selon la norme **PSR-4** ;
- faciliter la création de projets PHP modernes.

Même lorsqu'un projet ne dépend d'aucune bibliothèque externe, Composer peut être utilisé uniquement pour son **autoloader**, ce qui évite d'écrire son propre `spl_autoload_register()`.

---
# Installation sous Windows

## Prérequis

Avant d'installer Composer, il faut disposer de :

- PHP installé
- PHP accessible depuis la variable d'environnement `PATH`
- Une version récente de PHP (8.x recommandée)

Pour vérifier l'installation de PHP :

```bash
php -v
```

Exemple :

```text
PHP 8.4.15 (cli) (built: Nov 18 2025 18:38:40) (ZTS Visual C++ 2022 x64)
Copyright (c) The PHP Group
Zend Engine v4.4.15, Copyright (c) Zend Technologies
    with Zend OPcache v8.4.15, Copyright (c), by Zend Technologies
    with Xdebug v3.4.7, Copyright (c) 2002-2025, by Derick Rethans
```

Particularité : en salle 27 Php n'est pas accessible depuis la variable d'environnement car il a été installé avec WampServer qui propose plusieurs versions de php.  
---

## Télécharger Composer

Télécharger l'installateur officiel : [https://getcomposer.org/download/](https://getcomposer.org/download/)

Choisir : **Composer-Setup.exe**

---

## Installation

Ne pas cocher la case 'Developer mode'  (elle permet simplement de choisir son propre répertoire d'installation)

L'assistant d'installation demande :
- l'emplacement de `php.exe`  :  Sélectionner dans la liste **C:\wamp64\bin\php\php8.4.15\php.exe**
- l'ajout de Composer dans le `PATH` : cocher la case
- Définir les paramètres du Proxy : cocher la case et dans url : 192.168.10.2:8080

Une fois l'installation terminée, ouvrir une nouvelle invite de commandes.

Vérifier :

```bash
composer --version
```

Exemple :

```text
Composer version 2.10.2 2026-07-01 11:24:45
PHP version 8.4.15 (C:\wamp64\bin\php\php8.4.15\php.exe)
Run the "diagnose" command to get more detailed diagnostics output.
```

Composer est installé dans le répertoire **C:\ProgramData\ComposerSetup\bin**

---

# Utilisation avec PhpStorm

PhpStorm détecte généralement Composer automatiquement.

## Vérification

Ouvrir :

```
File
    Settings
        PHP
            Composer
```

Le chemin vers `composer.bat` doit être renseigné 

Exemple :

```
C:\ProgramData\ComposerSetup\bin\composer.bat
```

Si ce n'est pas le cas, utiliser **Browse...** pour le sélectionner.


## Terminal intégré

PhpStorm possède un terminal accessible par le raccourci Alt F12

Toutes les commandes Composer peuvent y être exécutées.

# Le fichier composer.json

Le fichier principal de Composer est :

```
composer.json
```

Exemple minimal :

```json
{  
  "autoload": {  
    "psr-4": {  
      "ClasseTechnique\\": "src/ClasseTechnique/",  
      "ClasseMetier\\": "src/ClasseMetier/"  
    }  
  }  
}
```

Important : si on n'a pas besoin de charger des composants (installer des dépendances) il est plus rapide de mettre à jour uniquement le fichier autolader.php en exécutant l'instruction **composer dump-autoload**

# Initialiser Composer

Dans un projet existant :

```bash
composer init
```

Composer pose plusieurs questions.

**Il est préférable  de créer directement le fichier `composer.json`.**

# Installer les dépendances

Installation d'une bibliothèque :

```bash
composer require monolog/monolog
```

Composer télécharge automatiquement :

- la bibliothèque
- ses dépendances
- l'autoloader
# Mettre à jour les dépendances

```bash
composer update
```

# Réinstaller les dépendances

Lorsqu'un projet est cloné depuis Git :

```bash
composer install
```

Cette commande lit le fichier :

```
composer.lock
```

et installe exactement les versions utilisées par le projet.

# Les principaux fichiers générés

Après installation :

```
Projet/
│
├── composer.json
├── composer.lock
├── vendor/
│   ├── autoload.php
│   └── ...
└── src/
```

Le dossier `vendor` contient :

- les bibliothèques téléchargées ;
- l'autoloader généré par Composer.

# Gestion des extensions PHP

PhpStorm possède une fonctionnalité d'assistance lors de la gestion de Composer qui peut proposer/ajouter les extensions PHP détectées comme utilisées par le projet.
C'est notamment utile parce que `composer.json` devient une **description complète des prérequis de l'application**.

```
{
    "require": {
        "php": "^8.1",
        "ext-pdo": "*",
        "ext-curl": "*",
        "phpmailer/phpmailer": "^7.0"
    }
}
```

signifie :

> Cette application nécessite PHP 8.1+, PDO, cURL et PHPMailer 7.x.

Pour savoir qui nécessite une extension

```
composer why ext-mbstring
```

  Réponse :
  ```
  Package "ext-mbstring *" found in version "8.4.15".  
__root__  -      requires ext-mbstring (*)   
mpdf/mpdf v8.3.1 requires ext-mbstring (*)
  ```
# Utiliser uniquement l'autoloader de Composer

L'un des intérêts majeurs de Composer est son autoloader.

Même sans dépendance externe, il peut remplacer un autoloader développé manuellement.

## Exemple

Projet :

```
Projet/
│
├── src/
│   ├── ClasseTechnique/
│   │      Erreur.php
│   └── ClasseMetier/
│          Produit.php
│
├── public/
│      index.php
│
└── composer.json
```

## Configuration PSR-4

```json
{
    "autoload": {
        "psr-4": {
            "ClasseTechnique\\": "src/ClasseTechnique/",
            "ClasseMetier\\": "src/ClasseMetier/"
        }
    }
}
```

Puis générer l'autoloader :

```bash
composer dump-autoload
```

Composer crée automatiquement :

```
vendor/autoload.php
```

## Chargement des classes

Dans le point d'entrée de l'application :

```php
require __DIR__ . '/../vendor/autoload.php';
```

Ensuite :

```php
use ClasseMetier\Produit;

$produit = new Produit();
```

Aucun `require` supplémentaire n'est nécessaire.

# Remplacer un autoloader manuel

Un autoloader classique utilise souvent :

```php
spl_autoload_register(function ($nomClasse) {

    ...
});
```

Avec Composer, cette partie disparaît complètement.

Il suffit de charger :

```php
require 'vendor/autoload.php';
```

Le chargement des classes est entièrement pris en charge.

---

# Utiliser un namespace unique

Il est généralement préférable d'utiliser un seul namespace racine.

Exemple :

```
src/
│
├── Technique/
├── Metier/
├── Controleur/
└── Repository/
```

Configuration :

```json
{
    "autoload": {
        "psr-4": {
            "MonProjet\\": "src/"
        }
    }
}
```

Une classe :

```php
namespace MonProjet\Metier;

class Produit
{
}
```

Utilisation :

```php
use MonProjet\Metier\Produit;
```

Cette organisation est plus évolutive.

# Régénérer l'autoloader

À chaque ajout, suppression ou déplacement de classe :

```bash
composer dump-autoload
```

Version optimisée pour la production :

```bash
composer dump-autoload -o
```


# Les commandes essentielles

| Commande                    | Description                        |
| --------------------------- | ---------------------------------- |
| `composer init`             | Initialise un projet Composer      |
| `composer install`          | Installe les dépendances           |
| `composer update`           | Met à jour les dépendances         |
| `composer require paquet`   | Ajoute une bibliothèque            |
| `composer remove paquet`    | Supprime une bibliothèque          |
| `composer dump-autoload`    | Régénère l'autoloader              |
| `composer dump-autoload -o` | Génère un autoloader optimisé      |
| `composer validate`         | Vérifie le fichier `composer.json` |
| `composer show`             | Liste les dépendances installées   |
# Bonnes pratiques

- Ne jamais modifier le contenu du dossier `vendor`.
- Versionner `composer.json`.
- Versionner `composer.lock` pour les applications.
- Ignorer le dossier `vendor` dans Git.
- Utiliser un namespace racine unique.
- Respecter la norme PSR-4.
- Régénérer l'autoloader après toute modification de l'arborescence.

# Conclusion

Composer est aujourd'hui un outil incontournable de l'écosystème PHP.

Même sans bibliothèque tierce, son autoloader constitue une solution robuste, rapide et conforme aux standards. Il remplace avantageusement les autoloaders développés manuellement tout en préparant le projet à intégrer facilement des dépendances externes à l'avenir.