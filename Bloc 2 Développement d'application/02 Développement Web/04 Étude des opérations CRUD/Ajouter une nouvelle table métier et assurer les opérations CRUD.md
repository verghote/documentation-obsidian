## Objectif

Ce guide décrit le processus complet permettant d'ajouter une nouvelle entité métier dans l'application et de lui assurer les opérations CRUD :

- **Create** : ajouter un enregistrement ;
- **Read** : consulter les enregistrements ;
- **Update** : modifier un enregistrement ;
- **Delete** : supprimer un enregistrement.

L'architecture repose sur deux niveaux :

- `Table` : classe technique qui fournit le fonctionnement générique des opérations CRUD et des validations ;
- classe métier : classe dérivée de `Table` qui décrit la table et contient les règles propres au métier.

Le développeur **ne doit pas réécrire le CRUD dans chaque classe métier**.

Le principe général est :

```text
Base de données
       │
       ▼
Classe métier
       │
       │ hérite de Table
       ▼
Classe technique Table
       │
       ▼
Endpoints AJAX
       │
       ▼
Interface utilisateur
```

# 1. Préparer la table SQL

Avant de créer la classe métier, il faut connaître précisément la structure de la table.

Pour chaque table, identifier :

- le nom de la table ;
- la clé primaire ;
- les colonnes ;
- le type de chaque colonne ;
- les colonnes obligatoires ;
- les colonnes facultatives ;
- les colonnes générées automatiquement ;
- les colonnes qui peuvent être insérées ;
- les colonnes qui peuvent être modifiées ;
- les contraintes d'unicité ;
- les clés étrangères ;
- les règles métier particulières.

## Exemple

```sql
create table lieu
(
    id  varchar(20) not null,
    nom varchar(120) not null,

    primary key (id),

    unique (nom)
);
```

La classe métier devra être cohérente avec cette structure.

# 2. Identifier les responsabilités de la classe métier

La classe métier doit uniquement contenir ce qui est spécifique à l'entité.

Elle doit notamment définir :

- la table SQL représentée ;
- la clé primaire ;
- les colonnes gérées par l'application ;
- les règles de validation des colonnes ;
- les contraintes d'unicité ;
- les règles métier ;
- les méthodes de consultation spécifiques.

Elle ne doit pas contenir :

- le code générique d'insertion ;
- le code générique de modification ;
- le code générique de suppression ;
- le code générique de validation ;
- la gestion générale de la connexion à la base.

Ces responsabilités appartiennent à `Table`.

# 3. Créer la classe métier

Fichier :

```text
src/ClasseMetier/Lieu.php
```

Structure minimale :

```php
<?php

declare(strict_types=1);

namespace ClasseMetier;

use ClasseTechnique\ColumnText;
use ClasseTechnique\ContrainteUnique;
use ClasseTechnique\Table;
use ClasseTechnique\TextCase;

class Lieu extends Table
{
    protected function configure(): void
    {
        $this->table = 'lieu';
        $this->primaryKey = 'id';

        $this->addColumn('id', new ColumnText(
            required: true,
            insertable: true,
            updatable: false,
            maxLength: 20,
            casse: TextCase::Upper
        ));

        $this->addColumn('nom', new ColumnText(
            required: true,
            insertable: true,
            updatable: true,
            minLength: 3,
            maxLength: 120
        ));

        $this->addUniqueConstraint(
            new ContrainteUnique(
                ['nom'],
                'Un lieu avec ce nom existe déjà.'
            )
        );
    }
}
```

## Important

La méthode :

```php
protected function configure(): void
```

est le point d'entrée de la configuration de la classe métier.

Le constructeur de `Table` appelle automatiquement cette méthode.

Il ne faut donc pas recréer dans la classe métier un constructeur qui reproduit le fonctionnement de `Table`.

# 4. Déclarer la clé primaire

La clé primaire est déclarée avec :

```php
$this->primaryKey = 'id';
```

Elle permet notamment à `Table` d'effectuer les opérations nécessitant l'identification d'un enregistrement :

- recherche ;
- modification ;
- suppression.

## Clé primaire générée automatiquement

Si la base utilise :

```sql
id int auto_increment primary key
```

la classe métier indique :

```php
$this->primaryKey = 'id';
```

mais ne déclare généralement pas `id` comme une colonne modifiable.

La base est responsable de générer sa valeur.

# 5. Déclarer les colonnes

Chaque colonne gérée par l'application est déclarée avec :

```php
$this->addColumn('nomColonne', new Column...);
```

Le type de colonne doit correspondre à la nature de la donnée.

Exemples :

```php
ColumnText
ColumnInt
ColumnDate
ColumnEmail
ColumnUrl
ColumnList
```

Exemple :

```php
$this->addColumn('nom', new ColumnText(
    required: true,
    insertable: true,
    updatable: true,
    minLength: 3,
    maxLength: 120
));
```

La déclaration permet notamment de définir :

- le caractère obligatoire ;
- les longueurs minimales et maximales ;
- le format ;
- les valeurs autorisées ;
- la casse ;
- les transformations éventuelles ;
- l'autorisation d'insertion ;
- l'autorisation de modification.

# 6. Déterminer `insertable` et `updatable`

Pour chaque colonne, il faut se poser deux questions :

> La valeur peut-elle être fournie lors d'un ajout ?

> La valeur peut-elle être modifiée ultérieurement ?

Exemple :

```php
$this->addColumn('id', new ColumnText(
    required: true,
    insertable: true,
    updatable: false
));
```

Ici :

- `id` peut être fourni lors de la création ;
- `id` ne peut plus être modifié.

Autre exemple :

```php
$this->addColumn('dateCreation', new ColumnDate(
    required: true,
    insertable: true,
    updatable: false
));
```

La date de création peut être enregistrée lors de l'ajout mais ne doit jamais être modifiée.

# 7. Déclarer les contraintes d'unicité

Toute contrainte `UNIQUE` définie dans SQL doit être représentée dans la classe métier.

SQL :

```sql
unique (nom)
```

Classe métier :

```php
$this->addUniqueConstraint(
    new ContrainteUnique(
        ['nom'],
        'Un lieu avec ce nom existe déjà.'
    )
);
```

Pour une contrainte portant sur plusieurs colonnes :

```sql
unique (nom, prenom)
```

on écrit :

```php
$this->addUniqueConstraint(
    new ContrainteUnique(
        ['nom', 'prenom'],
        'Un étudiant portant ce nom et ce prénom existe déjà.'
    )
);
```

## Règle

La structure SQL et la configuration de la classe métier doivent rester cohérentes.

Une contrainte présente dans SQL ne doit pas être oubliée dans la classe métier.

# 8. Ajouter les règles métier

Certaines règles ne peuvent pas être décrites simplement avec les propriétés d'une colonne.

Elles doivent être placées dans les méthodes prévues par `Table`.

## `beforeChange()`

Utiliser `beforeChange()` pour une règle commune à l'ajout et à la modification.

```php
protected function beforeChange(): bool
{
    if ($this->getValue('ageMin') >= $this->getValue('ageMax')) {

        $this->addError(
            'ageMin',
            "L'âge minimum doit être inférieur à l'âge maximum."
        );

        return false;
    }

    return true;
}
```

## `beforeInsert()`

Utiliser `beforeInsert()` lorsqu'une règle concerne uniquement l'ajout.

Exemple :

```php
protected function beforeInsert(): bool
{
    // Traitement spécifique à la création.

    return true;
}
```

## `beforeUpdate()`

Utiliser `beforeUpdate()` lorsqu'une règle concerne uniquement la modification.

```php
protected function beforeUpdate(mixed $id): bool
{
    // Traitement spécifique à la modification.

    return true;
}
```

## `beforeDelete()`

Utiliser `beforeDelete()` pour empêcher une suppression interdite par les règles métier.

```php
protected function beforeDelete(mixed $id): bool
{
    // Vérification de l'utilisation de l'enregistrement.

    if (...) {
        $this->addError(
            'global',
            'Suppression impossible.'
        );

        return false;
    }

    return true;
}
```

## Ne pas déclarer les hooks inutilisés

Il est inutile d'écrire :

```php
protected function beforeInsert(): bool
{
    return true;
}
```

si aucun traitement particulier n'est nécessaire.

On ne redéfinit un hook que lorsqu'il apporte réellement un comportement métier.

# 9. Ajouter les méthodes de consultation

Les méthodes de consultation spécifiques restent dans la classe métier.

Elles sont généralement `static`.

Exemple :

```php
public static function getAll(): array
{
    $sql = <<<SQL
        select id, nom
        from lieu
        order by nom
    SQL;

    $select = new Select();

    return $select->getRows($sql);
}
```

Autres exemples possibles :

```php
getListe()
getById()
getByNom()
getByNomPrenom()
getDisponibles()
getPourSelection()
```

Ces méthodes utilisent `Select` pour effectuer les requêtes de consultation.

# 10. Séparer CRUD générique et consultations métier

Il faut distinguer deux types de méthodes.

## Opérations CRUD

Elles sont fournies par `Table` :

```text
add()
modify()
delete()
```

Le développeur ne doit normalement pas les réécrire dans chaque classe métier.

## Consultations spécifiques

Elles sont écrites dans la classe métier :

```php
public static function getAll(): array
```

ou :

```php
public static function getByNom(string $nom): array
```

La classe métier ajoute donc uniquement les requêtes qui ont une signification pour son domaine.

# 11. Créer les endpoints

La classe métier ne doit pas être appelée directement depuis le navigateur.

Les opérations d'écriture passent par des endpoints du module.

Organisation recommandée :

```text
public/
└── lieu/
    ├── ajout/
    │   └── ajax/
    │       └── ajouter.php
    │
    ├── maj/
    │   └── ajax/
    │       ├── modifier.php
    │       └── supprimer.php
    │
    └── ...
```

On peut également prévoir des endpoints de consultation selon les besoins du module.

# 12. Endpoint d'ajout

L'endpoint d'ajout reçoit les colonnes à enregistrer.

Schéma :

```text
Interface
   │
   │ POST columns
   ▼
ajouter.php
   │
   ▼
Lieu->add(...)
   │
   ▼
Table
   │
   ├── validation
   ├── contraintes
   ├── règles métier
   └── INSERT
   │
   ▼
Base de données
```

Structure générale :

```php
require ...;

Requete::exigerPost();
Jeton::verifier();

$columns = ...;

$lieu = new Lieu();

if ($lieu->add($columns)) {
    ReponseJson::envoyerMessage(...);
} else {
    ReponseJson::envoyerLesErreurs(...);
}
```

L'endpoint ne doit pas reproduire les validations déjà réalisées par `Table` et les `Column`.

# 13. Endpoint de modification

Une modification nécessite deux informations :

- l'identifiant de l'enregistrement ;
- les colonnes à modifier.

La requête contient donc :

```text
primaryKey
columns
```

Exemple conceptuel :

```text
POST

primaryKey = 12

columns:
    nom = "Nouveau nom"
```

Le traitement suit le chemin :

```text
Navigateur
    │
    ▼
modifier.php
    │
    ▼
Lieu->modify(...)
    │
    ▼
Table
    │
    ├── validation
    ├── contrôle de la clé
    ├── contraintes
    ├── beforeChange()
    ├── beforeUpdate()
    └── UPDATE
    │
    ▼
Base
```

# 14. Endpoint de suppression

Une suppression nécessite uniquement l'identifiant.

Exemple :

```text
POST

primaryKey = 12
```

Le chemin est :

```text
Navigateur
    │
    ▼
supprimer.php
    │
    ▼
Lieu->delete(12)
    │
    ▼
Table
    │
    ├── beforeDelete()
    └── DELETE
    │
    ▼
Base
```

La suppression doit être protégée contre les requêtes non autorisées et contre les suppressions interdites par les règles métier.

# 15. Sécurité des endpoints

Les endpoints d'écriture doivent respecter le mécanisme de sécurité prévu par l'application.

Typiquement :

```php
Requete::exigerPost();
Jeton::verifier();
```

L'ordre général est :

```text
1. Charger l'environnement
2. Vérifier la méthode HTTP
3. Vérifier le jeton
4. Récupérer les paramètres
5. Instancier la classe métier
6. Appeler l'opération Table
7. Retourner une réponse JSON
```

Un endpoint ne doit jamais faire confiance directement aux données envoyées par le navigateur.

# 16. Gestion des erreurs

Les erreurs doivent être produites par les couches appropriées.

## Erreur de colonne

Exemple :

```text
Le nom est obligatoire.
```

Elle relève de la définition de la colonne.

## Erreur d'unicité

Exemple :

```text
Un lieu avec ce nom existe déjà.
```

Elle relève de :

```php
ContrainteUnique
```

## Erreur métier

Exemple :

```text
Cette catégorie possède des coureurs et ne peut pas être supprimée.
```

Elle relève d'un hook comme :

```php
beforeDelete()
```

L'endpoint récupère ensuite les erreurs et les transmet au frontend dans le format JSON standard de l'application.

# 17. Adapter le frontend

Le frontend utilise le mécanisme AJAX de l'application.

## Ajout

Envoyer :

```text
columns
```

Exemple :

```javascript
appelAjax({
    url: 'ajax/ajouter.php',
    data: {
        columns: {
            nom: inputNom.value
        }
    }
});
```

## Modification

Envoyer :

```text
primaryKey
columns
```

Exemple :

```javascript
appelAjax({
    url: 'ajax/modifier.php',
    data: {
        primaryKey: id,
        columns: {
            nom: inputNom.value
        }
    }
});
```

## Suppression

Envoyer :

```text
primaryKey
```

Exemple :

```javascript
appelAjax({
    url: 'ajax/supprimer.php',
    data: {
        primaryKey: id
    }
});
```

# 18. Ne pas dupliquer la logique métier dans JavaScript

Le JavaScript peut effectuer des contrôles destinés à améliorer l'expérience utilisateur.

Cependant, les règles métier importantes doivent être contrôlées côté serveur.

Par exemple :

```text
JavaScript
    │
    └── amélioration de l'expérience utilisateur

Classe métier / Table
    │
    └── validation réelle des données
```

Le serveur doit toujours considérer les données reçues comme non fiables.

# 19. Gérer les valeurs facultatives

Une colonne SQL nullable doit être traitée correctement.

Par exemple :

```sql
photo varchar(50) null
```

Lorsqu'aucune photo n'est fournie, il faut transmettre une valeur représentant réellement l'absence de donnée, généralement :

```php
null
```

et non :

```text
""
```

Cette distinction est importante pour conserver la cohérence entre PHP et SQL.

# 20. Vérifier les noms des paramètres

Les noms utilisés entre le frontend et les endpoints doivent être strictement cohérents.

Convention :

```text
primaryKey
columns
```

Exemple :

```javascript
data: {
    primaryKey: id,
    columns: {
        nom: valeur
    }
}
```

Il ne faut pas avoir par exemple :

```text
id
primarykey
primary_key
champs
colonnes
```

si l'endpoint attend :

```text
primaryKey
columns
```

# 21. Vérifier le retour JSON

Les endpoints AJAX doivent utiliser le mécanisme standard de réponse de l'application.

En cas de succès :

```php
ReponseJson::envoyerMessage(...);
```

En cas d'erreur :

```php
ReponseJson::envoyerLesErreurs(...);
```

Le JavaScript doit alors traiter ces réponses avec le mécanisme prévu par `appelAjax`.

# 22. Processus complet d'un ajout

Lorsqu'un utilisateur ajoute un lieu, le processus complet est :

```text
FORMULAIRE
    │
    │ POST columns
    ▼
ENDPOINT ajouter.php
    │
    │
    ▼
CLASSE MÉTIER Lieu
    │
    ▼
TABLE
    │
    ├── validation des colonnes
    │
    ├── contrôle des contraintes d'unicité
    │
    ├── beforeChange()
    │
    ├── beforeInsert()
    │
    └── INSERT
    │
    ▼
BASE DE DONNÉES
    │
    ▼
RÉPONSE JSON
    │
    ▼
JAVASCRIPT
    │
    ▼
MISE À JOUR DE L'INTERFACE
```

# 23. Processus complet d'une modification

```text
FORMULAIRE
    │
    │ POST primaryKey + columns
    ▼
ENDPOINT modifier.php
    │
    ▼
CLASSE MÉTIER
    │
    ▼
TABLE
    │
    ├── validation des colonnes
    ├── contrôle de la clé primaire
    ├── contrôle des contraintes d'unicité
    ├── beforeChange()
    ├── beforeUpdate()
    └── UPDATE
    │
    ▼
BASE DE DONNÉES
    │
    ▼
RÉPONSE JSON
    │
    ▼
JAVASCRIPT
```

# 24. Processus complet d'une suppression

```text
BOUTON SUPPRIMER
    │
    ▼
confirmation utilisateur
    │
    │ POST primaryKey
    ▼
ENDPOINT supprimer.php
    │
    ▼
CLASSE MÉTIER
    │
    ▼
TABLE
    │
    ├── beforeDelete()
    │
    └── DELETE
    │
    ▼
BASE DE DONNÉES
    │
    ▼
RÉPONSE JSON
    │
    ▼
JAVASCRIPT
    │
    ▼
SUPPRESSION DE L'ÉLÉMENT DANS L'INTERFACE
```

# 25. Processus complet d'une consultation

Une consultation spécifique est généralement réalisée par une méthode statique de la classe métier.

Exemple :

```php
public static function getAll(): array
{
    $sql = <<<SQL
        select id, nom
        from lieu
        order by nom
    SQL;

    $select = new Select();

    return $select->getRows($sql);
}
```

Le chemin devient :

```text
PAGE / ENDPOINT
       │
       ▼
Lieu::getAll()
       │
       ▼
Select
       │
       ▼
Base de données
       │
       ▼
array
       │
       ▼
Vue / JSON / JavaScript
```

# 26. Checklist de création d'une nouvelle table métier

## Base de données

- [ ]  La table SQL existe.
- [ ]  La clé primaire est identifiée.
- [ ]  Les colonnes obligatoires sont identifiées.
- [ ]  Les colonnes facultatives sont identifiées.
- [ ]  Les colonnes générées automatiquement sont identifiées.
- [ ]  Les colonnes non modifiables sont identifiées.
- [ ]  Les contraintes `UNIQUE` sont identifiées.
- [ ]  Les clés étrangères sont identifiées.
- [ ]  Les règles métier sont identifiées.

## Classe métier

- [ ]  La classe hérite de `Table`.
- [ ]  Le namespace est correct.
- [ ]  `configure()` est définie.
- [ ]  `$this->table` est défini.
- [ ]  `$this->primaryKey` est défini.
- [ ]  Les colonnes nécessaires sont déclarées avec `addColumn()`.
- [ ]  Le bon type `Column...` est utilisé.
- [ ]  `required` est correctement défini.
- [ ]  `insertable` est correctement défini.
- [ ]  `updatable` est correctement défini.
- [ ]  Les règles de validation sont définies.
- [ ]  Toutes les contraintes `UNIQUE` sont déclarées.
- [ ]  Les hooks nécessaires sont définis.
- [ ]  Les hooks inutiles ne sont pas déclarés.
- [ ]  Les méthodes de consultation nécessaires sont ajoutées.

## Endpoints

- [ ]  Endpoint d'ajout créé si nécessaire.
- [ ]  Endpoint de modification créé si nécessaire.
- [ ]  Endpoint de suppression créé si nécessaire.
- [ ]  Méthode HTTP vérifiée.
- [ ]  Jeton vérifié pour les opérations d'écriture.
- [ ]  Paramètres correctement récupérés.
- [ ]  Classe métier utilisée pour les opérations CRUD.
- [ ]  Réponse JSON conforme au standard de l'application.

## Frontend

- [ ]  Le formulaire est correctement relié à l'endpoint.
- [ ]  `columns` est utilisé pour un ajout.
- [ ]  `primaryKey` + `columns` sont utilisés pour une modification.
- [ ]  `primaryKey` est utilisé pour une suppression.
- [ ]  Les erreurs sont affichées.
- [ ]  Le message de succès est affiché.
- [ ]  L'interface est mise à jour après une opération réussie.

# 27. Ce qui appartient à chaque couche

|Responsabilité|Classe / composant|
|---|---|
|Structure physique de la table|SQL|
|Connexion à la base|`Table` / classes techniques|
|CRUD générique|`Table`|
|Validation d'une colonne|`Column...`|
|Contraintes d'unicité|`ContrainteUnique`|
|Nom de la table|Classe métier|
|Clé primaire|Classe métier|
|Colonnes métier|Classe métier|
|Règles métier|Classe métier|
|Requêtes de consultation spécifiques|Classe métier + `Select`|
|Sécurité HTTP|Endpoint|
|Lecture des paramètres HTTP|Endpoint|
|Réponse HTTP/JSON|Endpoint / `ReponseJson`|
|Affichage|Frontend|
|Appels AJAX|Frontend / `appelAjax`|

# 28. Règle fondamentale de l'architecture

Lorsqu'une nouvelle table est ajoutée, il ne faut pas repartir de zéro.

Le développeur doit raisonner ainsi :

```text
Que doit savoir la classe métier ?
```

et non :

```text
Comment vais-je refaire le CRUD ?
```

La classe `Table` connaît la mécanique générale.

La classe métier connaît le métier.

Les endpoints assurent la communication HTTP.

Le frontend assure l'interaction avec l'utilisateur.

Ainsi :

```text
                 APPLICATION

        ┌─────────────────────────┐
        │       FRONTEND          │
        │ formulaire / JS / AJAX  │
        └────────────┬────────────┘
                     │
                     ▼
        ┌─────────────────────────┐
        │       ENDPOINT          │
        │ HTTP / sécurité / JSON  │
        └────────────┬────────────┘
                     │
                     ▼
        ┌─────────────────────────┐
        │     CLASSE MÉTIER       │
        │ configuration / métier  │
        └────────────┬────────────┘
                     │
                     ▼
        ┌─────────────────────────┐
        │         TABLE           │
        │ CRUD / validation       │
        └────────────┬────────────┘
                     │
                     ▼
        ┌─────────────────────────┐
        │      BASE SQL           │
        └─────────────────────────┘
```

# 29. Références techniques

Les développeurs doivent consulter en priorité les classes techniques suivantes :

```text
src/ClasseTechnique/Table.php
src/ClasseTechnique/Column.php
src/ClasseTechnique/ColumnText.php
src/ClasseTechnique/ColumnInt.php
src/ClasseTechnique/ColumnDate.php
src/ClasseTechnique/ColumnList.php
src/ClasseTechnique/ContrainteUnique.php
src/ClasseTechnique/Select.php
src/ClasseTechnique/Requete.php
src/ClasseTechnique/ReponseJson.php
```

Ils doivent également consulter une classe métier existante suffisamment représentative avant de créer une nouvelle classe.

# 30. Résumé du processus

Pour ajouter une nouvelle entité métier :

```text
1. Créer / vérifier la table SQL
              ↓
2. Identifier sa structure et ses règles
              ↓
3. Créer la classe métier qui hérite de Table
              ↓
4. Implémenter configure()
              ↓
5. Déclarer les colonnes
              ↓
6. Déclarer les contraintes d'unicité
              ↓
7. Ajouter les hooks métier nécessaires
              ↓
8. Ajouter les méthodes de consultation
              ↓
9. Créer les endpoints nécessaires
              ↓
10. Connecter le frontend avec appelAjax
              ↓
11. Tester ajout / consultation / modification / suppression
              ↓
12. Tester également les erreurs et règles métier
```

L'objectif est de conserver une séparation claire des responsabilités :

> **SQL décrit les données.**

> **`Table` fournit la mécanique CRUD.**

> **La classe métier décrit les règles du domaine.**

> **Les endpoints exposent les opérations à HTTP.**

> **Le frontend permet à l'utilisateur de les exploiter.**