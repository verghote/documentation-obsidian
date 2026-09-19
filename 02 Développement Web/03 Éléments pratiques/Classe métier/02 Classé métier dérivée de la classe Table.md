## Introduction

La classe `Table` est la classe technique de base utilisée par les classes métier représentant une table de la base de données.

Elle fournit automatiquement les fonctionnalités communes nécessaires à la gestion des données :

- connexion et accès à la base de données ;
- validation des colonnes ;
- contrôle des valeurs obligatoires ;
- contrôle des formats ;
- vérification des contraintes d'unicité ;
- ajout d'enregistrements ;
- modification d'enregistrements ;
- suppression d'enregistrements.

Une classe métier qui hérite de `Table` **ne doit donc pas réécrire le fonctionnement général du CRUD**.

Elle décrit uniquement ce qui est spécifique à l'entité métier :

- la table représentée ;
- la clé primaire ;
- les colonnes utilisées par l'application ;
- les contraintes particulières ;
- les règles métier ;
- les traitements spécifiques ;
- les méthodes de consultation nécessaires.

# 1. Structure générale

Une classe métier dérivée de `Table` possède généralement la structure suivante :

```php
class Club extends Table
{
    protected function configure(): void
    {
        // Configuration de la table
    }

    protected function beforeChange(): bool
    {
        // Règles avant ajout ou modification
        return true;
    }

    protected function beforeDelete(mixed $id): bool
    {
        // Règles avant suppression
        return true;
    }

    public static function getAll(): array
    {
        // Consultation spécifique
    }
}
```

**Attention :** les méthodes `before...()` ne sont ajoutées que lorsqu'un traitement particulier est nécessaire.

Il ne faut pas créer des méthodes vides simplement pour respecter cette structure.

# 2. La méthode `configure()`

La méthode :

```php
protected function configure(): void
```

est la méthode essentielle de la classe métier.

Elle est définie comme méthode abstraite dans `Table` et doit donc être redéfinie par chaque classe métier.

Elle est appelée automatiquement par `Table` lors de la création de l'objet.

Elle permet de décrire la table utilisée par la classe métier.

Elle doit notamment définir :

- le nom de la table SQL ;
- la clé primaire ;
- les colonnes gérées par l'application ;
- les contraintes d'unicité éventuelles.

## Exemple

```php
protected function configure(): void
{
    $this->table = 'club';
    $this->primaryKey = 'id';

    $this->addColumn('nom', new ColumnText(
        required: true,
        maxLength: 60
    ));
}
```

# 3. Définir la table SQL

Le nom de la table est défini avec :

```php
$this->table = 'nom_table';
```

Exemple :

```php
$this->table = 'club';
```

La clé primaire est définie avec :

```php
$this->primaryKey = 'nom_colonne';
```

Exemple :

```php
$this->primaryKey = 'id';
```

Ces informations permettent à `Table` de réaliser automatiquement les opérations génériques :

- recherche d'un enregistrement ;
- ajout ;
- modification ;
- suppression ;
- contrôle d'existence.

# 4. Déclarer les colonnes

Les colonnes utilisées par l'application sont déclarées avec :

```php
$this->addColumn('nomColonne', $column);
```

Chaque colonne est associée à une classe spécialisée adaptée à son type de données.

Par exemple :

- `ColumnText`
- `ColumnInt`
- `ColumnDate`
- `ColumnEmail`
- `ColumnUrl`
- `ColumnBool`
- `ColumnList`

## Exemple

```php
$this->addColumn('nom', new ColumnText(
    required: true,
    minLength: 3,
    maxLength: 60
));
```

La déclaration d'une colonne permet notamment de définir :

- si elle est obligatoire ;
- son type ;
- son format ;
- ses longueurs minimale et maximale ;
- son expression régulière éventuelle ;
- si elle peut être utilisée lors d'un ajout ;
- si elle peut être utilisée lors d'une modification ;
- les transformations éventuelles de sa valeur.

# 5. Exemple avec une colonne texte

```php
$this->addColumn('nom', new ColumnText(
    required: true,
    insertable: true,
    updatable: true,
    pattern: '^[A-Z]( ?[A-Z]+)*$',
    maxLength: 30,
    casse: TextCase::Upper,
    supprimerAccent: true
));
```

Cette déclaration signifie notamment que :

- le champ est obligatoire ;
- il peut être fourni lors d'un ajout ;
- il peut être fourni lors d'une modification ;
- il possède une longueur maximale de 30 caractères ;
- il doit respecter l'expression régulière indiquée ;
- le texte peut être converti en majuscules ;
- les accents peuvent être supprimés.

La classe `Table` et les classes `Column` prennent alors en charge les validations génériques.

La classe métier n'a pas besoin de refaire ces contrôles.

# 6. Ne pas déclarer inutilement les colonnes gérées par la base

Une classe métier ne doit pas nécessairement déclarer toutes les colonnes présentes physiquement dans la table SQL.

Elle doit déclarer les colonnes **qui sont manipulées par l'application**.

## Exemple : clé primaire auto-incrémentée

Si la table contient :

```sql
id INT AUTO_INCREMENT PRIMARY KEY
```

la base de données génère elle-même la valeur de `id`.

On indique donc :

```php
$this->primaryKey = 'id';
```

mais on ne déclare pas nécessairement :

```php
$this->addColumn('id', ...);
```

Exemple :

```php
protected function configure(): void
{
    $this->table = 'projet';
    $this->primaryKey = 'id';

    $this->addColumn('nom', new ColumnText(
        required: true
    ));
}
```

# 7. Colonnes calculées par la base de données

Une colonne dont la valeur est directement calculée par SQL ne doit pas être manipulée comme une colonne saisie par l'utilisateur.

Par exemple :

```sql
montant AS prix * quantite
```

La valeur est calculée par la base de données.

La classe métier n'a donc pas à demander cette valeur à l'utilisateur.

# 8. Colonnes calculées par les règles métier

Certaines valeurs ne sont pas saisies par l'utilisateur mais sont déterminées par une règle métier.

Par exemple, dans la classe `Coureur`, la catégorie peut être déterminée automatiquement à partir de la date de naissance.

Dans ce cas, la valeur peut être calculée dans un hook métier.

```php
protected function beforeChange(): bool
{
    return $this->affecterCategorie();
}
```

La méthode privée peut alors réaliser le calcul :

```php
private function affecterCategorie(): bool
{
    $idCategorie = Categorie::getIdDepuisDateNaissance(
        (string)$this->getValue('dateNaissance')
    );

    if ($idCategorie === null) {
        $this->addError(
            'dateNaissance',
            'Aucune catégorie trouvée.'
        );

        return false;
    }

    $this->setValue('idCategorie', $idCategorie);

    return true;
}
```

Cette organisation permet de séparer :

- le déclenchement du traitement dans `beforeChange()` ;
- l'implémentation du traitement dans une méthode privée.

# 9. Contraintes d'unicité

Les contraintes d'unicité sont déclarées dans `configure()`.

## Unicité sur une colonne

```php
$this->addUniqueConstraint(
    new ContrainteUnique(
        ['nom'],
        'Ce nom existe déjà.'
    )
);
```

## Unicité sur plusieurs colonnes

Il est également possible de définir une unicité composée.

```php
$this->addUniqueConstraint(
    new ContrainteUnique(
        ['nom', 'prenom', 'dateNaissance'],
        'Cet enregistrement existe déjà.'
    )
);
```

La contrainte est alors appliquée à la combinaison des trois colonnes.

# 10. Les hooks métier

La classe `Table` prévoit des méthodes particulières, appelées **hooks**, permettant à une classe métier d'intervenir à certains moments du fonctionnement du CRUD.

Ces méthodes sont appelées automatiquement par `Table`.

Le développeur n'a donc normalement **pas besoin de les appeler lui-même**.

Les principaux hooks sont :

|Méthode|Moment d'exécution|Utilisation|
|---|---|---|
|`beforeChange()`|avant un ajout ou une modification|règle commune aux deux opérations|
|`beforeInsert()`|avant un ajout|traitement spécifique à l'ajout|
|`beforeUpdate()`|avant une modification|traitement spécifique à la modification|
|`beforeDelete()`|avant une suppression|contrôle avant suppression|
|`afterInsert()`|après un ajout|traitement après création|
|`afterUpdate()`|après une modification|traitement après modification|
|`afterDelete()`|après une suppression|traitement après suppression|

Toutes ces méthodes ne sont évidemment pas obligatoires.

**On ne redéfinit un hook que lorsqu'un traitement particulier est nécessaire.**

# 11. `beforeChange()`

La méthode :

```php
protected function beforeChange(): bool
```

est exécutée avant un ajout et avant une modification.

Elle permet de factoriser une règle métier commune aux deux opérations.

## Exemple

```php
protected function beforeChange(): bool
{
    return $this->affecterCategorie();
}
```

Dans cet exemple, la catégorie d'un coureur est automatiquement déterminée avant toute écriture en base.

Si le traitement est impossible :

```php
protected function beforeChange(): bool
{
    if (...) {
        $this->addError(
            'dateNaissance',
            'Impossible de déterminer la catégorie.'
        );

        return false;
    }

    return true;
}
```

Retourner `false` permet d'empêcher la poursuite de l'opération.

# 12. `beforeInsert()`

La méthode :

```php
protected function beforeInsert(): bool
```

est appelée uniquement avant un ajout.

Elle est utilisée lorsqu'un traitement concerne spécifiquement la création d'un enregistrement.

## Exemple

```php
protected function beforeInsert(): bool
{
    // Traitement spécifique à la création

    return true;
}
```

Par exemple :

- génération d'une référence ;
- initialisation d'une valeur ;
- contrôle spécifique à la création.

# 13. `beforeUpdate()`

La méthode :

```php
protected function beforeUpdate(mixed $id): bool
```

est appelée uniquement avant une modification.

Elle est utilisée lorsqu'une règle ne concerne que la mise à jour.

## Exemple

```php
protected function beforeUpdate(mixed $id): bool
{
    // Contrôle spécifique à la modification

    return true;
}
```

# 14. `beforeDelete()`

La méthode :

```php
protected function beforeDelete(mixed $id): bool
```

est appelée avant une suppression.

Elle permet d'interdire la suppression lorsqu'une règle métier ne l'autorise pas.

## Exemple

```php
protected function beforeDelete(mixed $id): bool
{
    if ($categorieEstUtilisee) {
        $this->addError(
            'global',
            'Cette catégorie possède encore des coureurs.'
        );

        return false;
    }

    return true;
}
```

Dans cet exemple, une catégorie ne peut pas être supprimée si elle est encore utilisée.

# 15. Les méthodes `after...()`

Les hooks `after...()` permettent d'effectuer un traitement après une opération réussie.

Par exemple :

```php
protected function afterInsert(mixed $id): bool
{
    // Traitement après création

    return true;
}
```

Ils peuvent notamment être utilisés lorsqu'une opération doit entraîner un traitement complémentaire.

Comme pour les hooks `before...()`, ils ne doivent être redéfinis que lorsqu'un traitement est nécessaire.

# 16. Ordre général des traitements

Lors d'une opération CRUD, `Table` réalise automatiquement les différentes étapes prévues par son fonctionnement.

Par exemple, lors d'un ajout, le traitement peut être représenté ainsi :

```text
Données reçues
      ↓
Validation des colonnes
      ↓
Contrôle des contraintes d'unicité
      ↓
beforeChange()
      ↓
beforeInsert()
      ↓
INSERT en base
      ↓
afterInsert()
```

Pour une modification :

```text
Données reçues
      ↓
Validation des colonnes
      ↓
Contrôle des contraintes d'unicité
      ↓
beforeChange()
      ↓
beforeUpdate()
      ↓
UPDATE en base
      ↓
afterUpdate()
```

Pour une suppression :

```text
Demande de suppression
      ↓
beforeDelete()
      ↓
DELETE en base
      ↓
afterDelete()
```

L'ordre exact dépend de l'implémentation de `Table`. Il faut donc considérer ce schéma comme une représentation du principe général et non comme une liste de traitements à reproduire dans la classe métier.

# 17. Méthodes métier privées

Une classe métier peut contenir des méthodes privées permettant de factoriser des traitements internes.

Ces méthodes ne sont pas destinées à être appelées directement depuis l'extérieur.

## Exemple

```php
private function affecterCategorie(): bool
{
    $idCategorie =
        Categorie::getIdDepuisDateNaissance(
            (string)$this->getValue('dateNaissance')
        );

    if ($idCategorie === null) {
        $this->addError(
            'dateNaissance',
            'Aucune catégorie trouvée.'
        );

        return false;
    }

    $this->setValue('idCategorie', $idCategorie);

    return true;
}
```

Le hook peut alors simplement appeler cette méthode :

```php
protected function beforeChange(): bool
{
    return $this->affecterCategorie();
}
```

Cette organisation rend la classe plus lisible.

# 18. Méthodes métier publiques

Une classe métier peut également proposer des méthodes publiques correspondant à des traitements propres au domaine.

Exemple :

```php
public function changerCategorie(): bool
{
    // Traitement métier spécifique

    return true;
}
```

Ces méthodes ne doivent pas reproduire les fonctionnalités déjà fournies par `Table`.

Elles doivent représenter une véritable opération métier.

# 19. Méthodes de consultation

Une classe métier peut proposer des méthodes permettant d'effectuer des recherches ou des consultations spécifiques.

Lorsqu'une consultation ne nécessite pas l'état d'un objet particulier, ces méthodes sont généralement statiques.

## Exemple

```php
public static function getAll(): array
{
    $sql = "
        SELECT id, nom
        FROM club
        ORDER BY nom
    ";

    $select = new Select();

    return $select->getRows($sql);
}
```

Une recherche peut également recevoir des paramètres.

```php
public static function getByNom(string $nom): array
{
    $sql = "
        SELECT id, nom
        FROM club
        WHERE nom LIKE :nom
        ORDER BY nom
    ";

    $select = new Select();

    return $select->getRows(
        $sql,
        ['nom' => "%$nom%"]
    );
}
```

L'utilisation de paramètres nommés permet d'éviter de construire directement les valeurs dans la requête SQL et protège notamment contre les injections SQL.

# 20. Une classe métier peut-elle contenir uniquement des méthodes statiques ?

Oui, dans certains cas.

Si une classe métier sert uniquement à effectuer des consultations et ne gère pas directement les opérations d'ajout, de modification ou de suppression, elle peut ne contenir que des méthodes statiques de consultation.

Par exemple :

```php
class Club extends Table
{
    protected function configure(): void
    {
        $this->table = 'club';
        $this->primaryKey = 'id';

        // Configuration nécessaire à la classe
    }

    public static function getAll(): array
    {
        // ...
    }

    public static function getByNom(string $nom): array
    {
        // ...
    }
}
```

La présence de méthodes statiques ne signifie cependant pas qu'une classe métier doit obligatoirement être entièrement statique.

Le choix dépend de son rôle dans l'application.

# 21. Structure complète recommandée

Une classe métier peut être organisée dans l'ordre suivant :

```php
class Coureur extends Table
{
    /*
     * Configuration de la table
     */
    protected function configure(): void
    {
        $this->table = 'coureur';
        $this->primaryKey = 'id';

        $this->addColumn('nom', new ColumnText(
            required: true,
            maxLength: 30
        ));

        $this->addColumn('prenom', new ColumnText(
            required: true,
            maxLength: 30
        ));

        $this->addColumn('dateNaissance', new ColumnDate(
            required: true
        ));

        $this->addUniqueConstraint(
            new ContrainteUnique(
                ['nom', 'prenom', 'dateNaissance'],
                'Ce coureur existe déjà.'
            )
        );
    }


    /*
     * Hooks métier
     */
    protected function beforeChange(): bool
    {
        return $this->affecterCategorie();
    }


    /*
     * Méthodes métier privées
     */
    private function affecterCategorie(): bool
    {
        // Traitement métier

        return true;
    }


    /*
     * Méthodes métier publiques
     */
    public function exemple(): bool
    {
        // Traitement métier

        return true;
    }


    /*
     * Méthodes de consultation
     */
    public static function getAll(): array
    {
        // Requête de consultation

        return [];
    }


    public static function getByNom(string $nom): array
    {
        // Requête de recherche

        return [];
    }
}
```

# 22. Organisation recommandée d'une classe métier

Pour conserver une organisation homogène dans le projet, l'ordre suivant est recommandé :

1. Documentation éventuelle de la classe ;
2. `configure()` ;
3. contraintes et règles métier via les hooks `before...` et `after...` ;
4. méthodes métier privées ;
5. méthodes métier publiques ;
6. méthodes de consultation ;
7. méthodes utilitaires éventuelles.

L'ordre peut être adapté lorsque cela améliore la lisibilité, mais toutes les classes métier du projet devraient suivre autant que possible la même organisation.

# 23. Répartition des responsabilités

La séparation entre `Table` et les classes métier est essentielle.

|Élément|Responsable|
|---|---|
|Accès à la base de données|`Table`|
|CRUD générique|`Table`|
|Validation générale des colonnes|`Column` + `Table`|
|Valeurs obligatoires|`Column` + `Table`|
|Formats des données|`Column` + `Table`|
|Contraintes d'unicité|`Table`|
|Nom de la table|Classe métier|
|Clé primaire|Classe métier|
|Colonnes métier|Classe métier|
|Règles métier|Classe métier|
|Traitements spécifiques|Classe métier|
|Requêtes spécifiques|Classe métier|

# 24. Ce que la classe métier ne doit pas faire

Une classe métier ne doit pas :

- réécrire le CRUD ;
- recréer les validations déjà fournies par `Column` et `Table` ;
- gérer elle-même la connexion à la base ;
- reconstruire manuellement les requêtes génériques d'ajout, de modification ou de suppression ;
- dupliquer les contrôles déjà réalisés par le framework ;
- déclarer inutilement les colonnes dont la valeur est gérée exclusivement par la base de données.

Elle doit se concentrer sur **ce qui est propre au métier**.

# 25. Résumé

Une classe métier dérivée de `Table` peut être vue comme une description de l'entité métier.

```text
                 Classe métier
                       │
        ┌──────────────┼──────────────┐
        │              │              │
   Configuration   Règles métier   Consultations
        │              │              │
    configure()    before...()      get...
        │              │
        └──────────────┼──────────────┘
                       │
                       ▼
                     Table
                       │
             ┌─────────┼─────────┐
             │         │         │
           INSERT    UPDATE    DELETE
```

La règle essentielle à retenir est :

> **`Table` fournit la mécanique générale ; la classe métier fournit les informations et les règles propres au métier.**

Une bonne classe métier doit donc rester aussi simple que possible : elle configure son entité, ajoute ses règles métier et fournit les consultations spécifiques dont l'application a besoin.