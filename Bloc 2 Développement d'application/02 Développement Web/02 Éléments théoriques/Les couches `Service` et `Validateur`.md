# Objectif

Dans une application simple, un contrôleur AJAX peut :

- récupérer les paramètres de la requête ;
- vérifier leur validité ;
- appliquer quelques règles métier ;
- appeler directement la classe métier ;
- retourner la réponse JSON.

Cette approche est parfaitement acceptable lorsque les traitements sont peu nombreux.

En revanche, lorsque plusieurs contrôleurs réalisent les mêmes validations ou appliquent les mêmes règles métier, le code devient rapidement répétitif.

Les couches **Service** et **Validateur** ont pour objectif de supprimer ces répétitions.

Elles ne sont donc **pas obligatoires**.

Leur mise en place est justifiée uniquement lorsque plusieurs contrôleurs partagent les mêmes traitements.

# Architecture

Sans ces couches, l'organisation est la suivante :

```
Contrôleur AJAX
        │
        ▼
Classe métier
        │
        ▼
Base de données
```

Avec un validateur et un service :

```
Contrôleur AJAX
        │
        ▼
Service
        │
        ├── Validation
        │
        ├── Règles métier
        │
        ▼
Classe métier
        │
        ▼
Base de données
```

Le contrôleur devient alors un simple orchestrateur.

# Exemple

On souhaite retourner les coureurs correspondant :

- à un sexe ;
- à un club ;
- à une catégorie.

Les trois paramètres sont transmis en AJAX.

# Version sans Service ni Validateur

Le contrôleur doit :

- récupérer les paramètres ;
- contrôler leur format ;
- retourner les erreurs éventuelles ;
- appeler la classe métier.

```php
<?php

declare(strict_types=1);

require $_SERVER['DOCUMENT_ROOT'] . '/../bootstrap/autoload.php';

use ClasseMetier\Coureur;
use ClasseTechnique\Ajax;
use ClasseTechnique\ReponseJson;
use ClasseTechnique\Requete;

Ajax::post();

$idCategorie = Requete::postString('idCategorie');
$idClub      = Requete::postString('idClub');
$sexe        = strtoupper(Requete::postString('sexe'));

$erreurs = [];

// Validation du sexe
if (!preg_match('/^[MF*]$/', $sexe)) {
    $erreurs['sexe'] = "Le sexe n'est pas conforme.";
}

// Validation du club
if (!preg_match('/^080\d{3}$|^\*$/', $idClub)) {
    $erreurs['idClub'] = "L'identifiant du club n'est pas conforme.";
}

// Validation de la catégorie
if (!preg_match('/^[A-Z](?:[A-Z]|\d|10)$|^\*$/', $idCategorie)) {
    $erreurs['idCategorie'] = "L'identifiant de la catégorie n'est pas conforme.";
}

if (!empty($erreurs)) {
    ReponseJson::envoyerLesErreurs($erreurs);
}

ReponseJson::envoyerLesDonnees(Coureur::getBySexeClubCategorie($sexe, $idClub, $idCategorie)
);
```

Le contrôleur contient maintenant :

- la gestion HTTP ;
- la validation ;
- la logique métier ;
- l'accès aux données.

Si plusieurs contrôleurs réalisent les mêmes validations, tout ce code devra être recopié.

# Version avec les couches Service et Validateur

Le contrôleur devient beaucoup plus simple.

```php
<?php

declare(strict_types=1);

require $_SERVER['DOCUMENT_ROOT'] . '/../bootstrap/autoload.php';

use ClasseTechnique\Ajax;
use ClasseTechnique\ReponseJson;
use ClasseTechnique\Requete;
use Service\CoureurService;

Ajax::post();

$idCategorie = Requete::postString('idCategorie');
$idClub      = Requete::postString('idClub');
$sexe        = strtoupper(Requete::postString('sexe'));

ReponseJson::envoyerLesDonnees(
    CoureurService::getBySexeClubCategorie($sexe, $idClub, $idCategorie)
);
```

Le contrôleur ne contient plus que :

- la récupération des paramètres ;
- l'appel du service ;
- l'envoi de la réponse.

# Le rôle du Validateur

Le validateur regroupe toutes les règles de contrôle des paramètres.

Exemple :

```php
(new CoureurValidateur())
    ->validerSexe($sexe)
    ->validerIdClub($idClub)
    ->validerIdCategorie($idCategorie);
```

Il connaît :

- les formats attendus ;
- les expressions régulières ;
- les messages d'erreur.

Ainsi, la règle :

> un identifiant de club est composé de six chiffres commençant par 080

n'est écrite qu'une seule fois dans l'application.

Si cette règle évolue, une seule classe est à modifier.

# Le rôle du Service

Le service constitue l'interface entre les contrôleurs et les classes métier.

Son rôle est de :

- appeler le validateur ;
- appliquer les règles métier ;
- appeler la classe métier ;
- retourner le résultat.

Exemple :

```php
public static function getBySexeClubCategorie(string $sexe, string $idClub, string $idCategorie): array {

    $validateur = (new CoureurValidateur())
        ->validerSexe($sexe, true)
        ->validerIdClub($idClub, true)
        ->validerIdCategorie($idCategorie, true);

    if (!$validateur->estValide()) {
        ReponseJson::envoyerLesErreurs($validateur->getErreurs());
    }

    return Coureur::getBySexeClubCategorie($sexe, $idClub, $idCategorie);
}
```

Le contrôleur ignore complètement la façon dont les validations sont réalisées.

# Répartition des responsabilités

| Couche | Responsabilité |
|---------|----------------|
| Contrôleur AJAX | Récupérer les paramètres et appeler le service |
| Validateur | Vérifier le format des données |
| Service | Appliquer les règles métier et appeler la classe métier |
| Classe métier | Accéder aux données |

# Quand utiliser ces deux couches ?

La réponse est simple :  **uniquement lorsqu'elles apportent une véritable simplification.**

Elles sont particulièrement intéressantes lorsque :

- plusieurs contrôleurs utilisent les mêmes paramètres ;
- les mêmes validations sont répétées ;
- plusieurs traitements appliquent les mêmes règles métier ;
- plusieurs contrôleurs appellent les mêmes méthodes de la classe métier.

Dans ce cas, les couches Service et Validateur évitent la duplication du code.

# Quand ne pas les utiliser ?

Pour un traitement très simple, appelé une seule fois, il est souvent préférable de conserver un contrôleur direct.

Par exemple :

```php
Ajax::post();

$id = Requete::postInt('id');

ReponseJson::envoyerLesDonnees(Club::getById($id));
```

Créer un `ClubService` et un `ClubValidateur` uniquement pour encapsuler ces trois lignes n'apporterait aucune valeur.

# Philosophie

Le framework ne cherche pas à imposer une architecture.

Les couches **Service** et **Validateur** sont des outils destinés à éviter la duplication de code.

Lorsqu'elles permettent de mutualiser des traitements, elles améliorent la maintenabilité.

Dans le cas contraire, elles ajoutent une complexité inutile et il est préférable de s'en passer.