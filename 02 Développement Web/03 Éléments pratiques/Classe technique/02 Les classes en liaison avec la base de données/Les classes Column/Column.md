# Présentation

La classe `Column` est la classe de base utilisée pour représenter une **colonne métier persistée** dans une classe métier.

Une colonne métier associe :

- une valeur ;
- des règles de validation ;
- des informations nécessaires aux opérations CRUD.

Elle constitue le lien entre :

- la classe métier représentant une table ;
- les données manipulées par l'application ;
- la base de données.

Les classes `Column` sont utilisées dans les classes métier qui héritent généralement de `Table`.

Exemple :

```php
class Categorie extends Table
{
    protected function defineColumns(): void
    {
        $nom = new ColumnText(
            required: true,
            maxLength: 50
        );

        $this->addColumn('nom', $nom);
    }
}
```

Ici, la colonne SQL `nom` est représentée par un objet `ColumnText` qui hérite de `Column`.

# Rôle de `Column`

`Column` est une classe abstraite.

Elle ne doit jamais être utilisée directement.

Elle fournit le comportement commun à toutes les colonnes :

- gestion de la valeur ;
- gestion du caractère obligatoire ;
- contrôle avant insertion ou modification ;
- gestion des messages d'erreur.

Les classes spécialisées héritent de `Column` :

```
Column
 |
 +-- ColumnText
 |
 +-- ColumnInt
 |
 +-- ColumnDate
 |
 +-- ColumnList
 |
 +-- ...
```

Chaque classe fille ajoute ses propres contrôles.

Exemple :

- `ColumnInt` vérifie qu'une valeur est un entier ;
- `ColumnText` contrôle une chaîne de caractères ;
- `ColumnDate` contrôle une date.

# La propriété `Value`

```php
public mixed $Value;
```

Cette propriété contient la valeur associée à la colonne.

Exemple :

```php
$age = new ColumnInt();

$age->Value = 25;
```

La valeur est ensuite utilisée par les opérations CRUD.

Lors d'une modification :

```php
$categorie->setValue('ageMin', 10);
```

la valeur est stockée dans la colonne correspondante.

# La propriété `Required`

```php
public readonly bool $Required;
```

Indique si la colonne doit obligatoirement contenir une valeur.

Par défaut :

```php
new ColumnText();
```

équivaut à :

```php
new ColumnText(required: true);
```

Une colonne obligatoire :

```php
$nom = new ColumnText(required: true);
```

refusera :

```text
null
```

ou :

```text
"    "
```

Une colonne facultative :

```php
$commentaire = new ColumnText(required: false);
```

acceptera une valeur vide.

# Les propriétés CRUD

## `Insertable`

```php
public readonly bool $Insertable;
```

Indique si la colonne peut être utilisée lors d'une insertion.

Exemple : Une clé primaire générée automatiquement :

```php
$id = new ColumnInt(insertable: false);
```

La colonne existe dans la table mais ne doit pas apparaître dans une requête `INSERT`.

## `Updatable`

```php
public readonly bool $Updatable;
```

Indique si la colonne peut être modifiée.

Exemple : Une date de création :

```php
$dateCreation = new ColumnDate(
    insertable: true,
    updatable: false
);
```

La valeur est enregistrée lors de la création mais ne peut plus être changée.

# La méthode `checkValidity()`

```php
public function checkValidity(): bool
```

Cette méthode réalise les contrôles communs à toutes les colonnes.

Elle vérifie principalement :

- qu'une valeur obligatoire est présente.

Exemple :

```php
$nom = new ColumnText();

$nom->Value = '';

if (!$nom->checkValidity()) {
    echo $nom->getValidationMessage();
}
```

Résultat :

```text
Veuillez renseigner ce champ.
```


Les classes filles redéfinissent cette méthode pour ajouter leurs propres contrôles.

Exemple :

```php
class ColumnInt extends Column
{
    public function checkValidity(): bool
    {
        if (!parent::checkValidity()) {
            return false;
        }

        // contrôle spécifique des entiers

        return true;
    }
}
```

La classe fille commence toujours par appeler :

```php
parent::checkValidity()
```

afin de conserver les contrôles communs.

# Récupérer un message d'erreur

Lorsqu'une validation échoue, le message associé peut être obtenu avec :

```php
getValidationMessage()
```

Exemple :

```php
if (!$colonne->checkValidity()) {
    echo $colonne->getValidationMessage();
}
```


# Exemple complet dans une classe métier

```php
protected function defineColumns(): void
{
    $colonne = new ColumnText(
        required: true,
        maxLength: 30
    );

    $this->addColumn('nom', $colonne);


    $colonne = new ColumnInt(
        required: true,
        min: 1,
        max: 99
    );

    $this->addColumn('age', $colonne);
}
```

La classe métier définit alors :

- quelles colonnes existent ;
- leur type ;
- leurs règles de validation ;
- leurs règles CRUD.

# À retenir

`Column` représente une donnée persistée dans l'application.

Elle permet de centraliser :

- la structure des données ;
- les règles de validation ;
- les contraintes CRUD.

Les classes métier ne manipulent pas directement les valeurs SQL : elles utilisent des objets `Column` spécialisés.