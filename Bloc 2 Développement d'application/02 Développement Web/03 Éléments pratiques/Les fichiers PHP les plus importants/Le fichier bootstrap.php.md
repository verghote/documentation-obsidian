## 1. Rôle du fichier

Le fichier constitue le **point d'initialisation technique commun de l'application**.

Il est exécuté au démarrage des pages PHP afin de préparer l'environnement nécessaire au fonctionnement de l'application.

Il assure actuellement les opérations suivantes :

- définition du fuseau horaire ;
- démarrage de la session PHP ;
- définition des chemins principaux du projet ;
- chargement automatique des classes avec Composer ;
- installation du gestionnaire global d'erreurs ;
- chargement des contraintes SQL ;
- transmission des contraintes au gestionnaire d'erreurs.

Le fichier ne contient **aucune logique métier**. Il prépare uniquement l'environnement technique de l'application.

---

# 2. Ordre d'initialisation

Les opérations sont effectuées dans l'ordre suivant :

```text
autoload.php
    |
    +-- 1. Fuseau horaire
    |
    +-- 2. Session PHP
    |
    +-- 3. Définition des chemins
    |
    +-- 4. Chargement de Composer
    |
    +-- 5. Installation du gestionnaire d'erreurs
    |
    +-- 6. Chargement des contraintes SQL
    |
    +-- 7. Transmission des contraintes à Erreur
    |
    v
Application prête à fonctionner
```

Cet ordre est important, notamment parce que les classes `Config` et `Erreur` doivent être disponibles avant leur utilisation.

---

# 3. Déclaration du typage strict

Le fichier commence par :

```php
declare(strict_types=1);
```

Cette instruction active le **typage strict** pour le fichier PHP.

Elle permet notamment de rendre les contrôles de types plus stricts lors des appels de fonctions et de méthodes.

Le projet utilise donc une approche où les types déclarés dans le code PHP doivent être respectés explicitement.

---

# 4. Importation des classes

Le fichier utilise deux classes du projet :

```php
use ClasseTechnique\Config;
use ClasseTechnique\Erreur;
```

Ces déclarations permettent d'utiliser directement :

```php
Config
```

et :

```php
Erreur
```

au lieu d'écrire leur nom complet :

```php
ClasseTechnique\Config
ClasseTechnique\Erreur
```

L'autoloading de Composer, chargé plus loin dans le fichier, permet ensuite de retrouver automatiquement les fichiers correspondant à ces classes.

---

# 5. Initialisation du fuseau horaire

Le fuseau horaire de l'application est défini avec :

```php
date_default_timezone_set('Europe/Paris');
```

Toutes les fonctions PHP utilisant la date et l'heure utilisent alors le fuseau :

```text
Europe/Paris
```

Cela permet d'avoir un comportement cohérent pour les opérations telles que :

```php
date()
```

```php
DateTime
```

ou les autres fonctionnalités PHP dépendant du fuseau horaire.

Cette configuration est réalisée dès le démarrage afin que les traitements suivants utilisent le même fuseau horaire.

---

# 6. Gestion de la session

Le fichier vérifie d'abord si une session PHP existe déjà :

```php
if (session_status() === PHP_SESSION_NONE) {
    session_start();
}
```

## Pourquoi effectuer ce test ?

`session_start()` ne doit pas être appelé inutilement si une session est déjà active.

Le test :

```php
session_status() === PHP_SESSION_NONE
```

signifie :

> aucune session PHP n'est actuellement active.

Dans ce cas seulement, le fichier démarre la session.

L'application peut ensuite utiliser $_SESSION dans les pages.

# 7. Définition des chemins du projet

Le fichier définit trois constantes permettant de retrouver les principaux répertoires de l'application.

## 7.1 `DOSSIER_WWW`

```php
define(
    "DOSSIER_WWW",
    dirname(__DIR__) . DIRECTORY_SEPARATOR . 'public'
);
```

Cette constante désigne le répertoire :

```text
public/
```

Le commentaire du fichier indique qu'il s'agit du :

> Répertoire public accessible par le navigateur

L'utilisation de `dirname(__DIR__)` permet de construire le chemin à partir de l'emplacement du fichier bootstrap.php

---

## 7.2 `DOSSIER_RACINE`

```php
define(
    "DOSSIER_RACINE",
    dirname(DOSSIER_WWW)
);
```

Cette constante désigne la **racine du projet**.

La relation entre les deux constantes est :

```text
DOSSIER_RACINE
      |
      +-- public/
            |
            +-- DOSSIER_WWW
```

Autrement dit :

```php
DOSSIER_WWW
```

désigne le dossier `public`, tandis que :

```php
DOSSIER_RACINE
```

désigne le dossier qui contient `public`.

---

## 7.3 `DOSSIER_CONFIG`

```php
const DOSSIER_CONFIG =
    DOSSIER_RACINE . DIRECTORY_SEPARATOR . 'config';
```

Cette constante désigne le répertoire contenant les fichiers de configuration :

```text
config/
```

L'organisation générale peut donc être représentée ainsi :

```text
Projet/
│
├── config/
│
├── public/
│
├── vendor/
│
└── ...
```

Avec :

```text
DOSSIER_RACINE → Projet/
DOSSIER_CONFIG → Projet/config/
DOSSIER_WWW    → Projet/public/
```

---

# 8. Utilisation de `DIRECTORY_SEPARATOR`

Les chemins sont construits avec :

```php
DIRECTORY_SEPARATOR
```

au lieu d'écrire directement `/`.

Par exemple :

```php
DOSSIER_RACINE . DIRECTORY_SEPARATOR . 'config'
```

permet à PHP d'utiliser le séparateur correspondant au système d'exploitation.

Cela rend la construction des chemins plus portable.

---

# 9. Chargement automatique des classes avec Composer

Le fichier charge ensuite l'autoloader de Composer :

```php
require DOSSIER_RACINE . '/vendor/autoload.php';
```

Composer fournit un mécanisme permettant de charger automatiquement les classes PHP nécessaires.

Grâce à cette instruction, les classes du projet peuvent être utilisées sans effectuer manuellement des `require` pour chaque classe.

Par exemple :

```php
Config::chargerPhp('contrainte');
```

peut utiliser automatiquement la classe :

```php
ClasseTechnique\Config
```

si elle est correctement déclarée dans l'autoloading du projet.

---

# 10. Installation du gestionnaire global d'erreurs

Une fois les classes disponibles, le fichier installe le gestionnaire global d'erreurs :

```php
Erreur::installerGestionnaire();
```

Cette opération configure la classe `Erreur` pour prendre en charge la gestion des erreurs de l'application.

Le détail du traitement effectué par cette méthode appartient à la classe :

```text
ClasseTechnique\Erreur
```

Le fichier se contente de **déclencher son installation**.

# 11. Chargement des contraintes SQL 

Le fichier charge ensuite une configuration PHP appelée :

```text
contrainte
```

avec :

```php
$contraintes = Config::chargerPhp('contrainte');
```

La classe `Config` est donc responsable du chargement de cette configuration.

Le fichier bootstrap.php ne définit pas lui-même le contenu des contraintes.

Il demande simplement à `Config` de charger la configuration correspondante.

On peut représenter le fonctionnement ainsi :

```text
autoload.php
     |
     v
Config::chargerPhp('contrainte')
     |
     v
configuration "contrainte"
     |
     v
$contraintes
```

# 12. Transmission des contraintes au gestionnaire d'erreurs

Une fois les contraintes chargées, elles sont transmises à la classe `Erreur` :

```php
Erreur::definirLesContraintes($contraintes);
```

Le fonctionnement est donc :

```text
Configuration
     v
Config::chargerPhp('contrainte')
     v
$contraintes
     v
Erreur::definirLesContraintes()
```

La classe `Erreur` dispose ainsi des contraintes chargées par `Config`.

La documentation détaillée du format de ces contraintes doit être recherchée dans :

```text
config/
```

et dans l'implémentation de :

```php
Config::chargerPhp()
```

et :

```php
Erreur::definirLesContraintes()
```

# 13. Gestion de l'absence de configuration

Le chargement des contraintes est placé dans un bloc `try/catch` :

```php
try {

    $contraintes = Config::chargerPhp('contrainte');

    Erreur::definirLesContraintes($contraintes);

} catch (Exception $e) {

    // Pas de contrainte configurée
    // ou configuration absente :
    // on laisse l'application démarrer.
}
```

Cela signifie que si une exception survient lors du chargement ou de la définition des contraintes, elle est interceptée.

L'application est alors autorisée à continuer son démarrage.

Le commentaire précise le comportement attendu :

```text
Pas de contrainte configurée
ou configuration absente :
on laisse l'application démarrer.
```

Il s'agit donc d'une **configuration facultative** du point de vue du démarrage de l'application.

# 14. Conséquence du `try/catch`

Il est important de noter que le `catch` est volontairement vide :

```php
catch (Exception $e) {
}
```

L'exception n'est donc pas propagée à la page appelante.

Le démarrage continue.

Cela permet par exemple de faire fonctionner l'application même si le fichier de configuration des contraintes n'existe pas.

Cependant, ce comportement signifie également qu'une erreur lors du chargement de cette configuration n'est pas signalée directement à cet endroit.

Le comportement précis dépend donc de la manière dont `Config::chargerPhp()` et le gestionnaire `Erreur` sont implémentés.

# 15. Utilisation par les pages de l'application

Une page PHP peut charger ce fichier au début de son exécution :

```php
require $_SERVER['DOCUMENT_ROOT'] . '/../bootstrap/autoload.php';
```

# 16. Résumé

|Fonction|Code|Rôle|
|---|---|---|
|Typage strict|`declare(strict_types=1)`|Renforce le contrôle des types|
|Fuseau horaire|`date_default_timezone_set()`|Définit `Europe/Paris`|
|Session|`session_start()`|Démarre la session si nécessaire|
|Répertoire public|`DOSSIER_WWW`|Définit le chemin vers `public/`|
|Racine|`DOSSIER_RACINE`|Définit la racine du projet|
|Configuration|`DOSSIER_CONFIG`|Définit le chemin vers `config/`|
|Autoload|`vendor/autoload.php`|Charge automatiquement les classes|
|Erreurs|`Erreur::installerGestionnaire()`|Installe la gestion globale des erreurs|
|Contraintes|`Config::chargerPhp('contrainte')`|Charge les contraintes configurées|
|Contraintes|`Erreur::definirLesContraintes()`|Transmet les contraintes au gestionnaire d'erreurs|

---

# 19. Vue d'ensemble

Le fichier peut être considéré comme le **bootstrap technique** du projet :

```text
                         bootstrap.php
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
          ▼                   ▼                   ▼
    Initialisation          Chemins            Composer
          │                   │                   │
          ├─ Fuseau           ├─ public/          └─ Classes
          └─ Session          ├─ racine
                              └─ config/
                                  │
                                  ▼
                           Gestion des erreurs
                                  │
                                  ▼
                         Chargement des contraintes
                                  │
                                  ▼
                         Application prête
```

Le principe architectural est donc :

> **Préparer une seule fois l'environnement technique commun afin que les pages applicatives puissent se concentrer sur leur propre logique.**