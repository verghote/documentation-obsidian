# Présentation

La classe `ColumnInt` représente une **colonne métier contenant un nombre entier**.

Elle hérite de la classe `Column` et ajoute les contrôles spécifiques aux valeurs numériques entières :

- vérification que la valeur est bien un entier ;
- contrôle d'une valeur minimale ;
- contrôle d'une valeur maximale.

Elle est utilisée dans les classes métier pour représenter les colonnes SQL contenant des nombres entiers :

- identifiants numériques ;
- quantités ;
- âges ;
- compteurs ;
- valeurs de classement.

Exemple :

```php
$age = new ColumnInt(
    required: true,
    min: 1,
    max: 99
);

$this->addColumn('age', $age);
```

La colonne `age` devra contenir un entier compris entre 1 et 99.

# Héritage

`ColumnInt` hérite de `Column`.

Elle possède donc les propriétés communes :

```
Column
 |
 +-- Value
 +-- Required
 +-- Insertable
 +-- Updatable
 |
 +-- ColumnInt
```

Elle bénéficie notamment :

- de la gestion de la valeur ;
- du contrôle des champs obligatoires ;
- de la gestion des messages d'erreur.

Elle ajoute les contraintes numériques propres aux entiers.

# Le constructeur

```php
public function __construct(
    bool $required = true,
    bool $insertable = true,
    bool $updatable = true,
    ?int $min = null,
    ?int $max = null
)
```

Le constructeur permet de définir les règles associées à la colonne entière.

# Paramètre `required`

Indique si une valeur est obligatoire.

Exemple :

```php
$quantite = new ColumnInt(
    required: true
);
```

Les valeurs suivantes seront refusées :

```text
null
""
"    "
```

Une colonne facultative :

```php
$nombreEnfants = new ColumnInt(
    required: false
);
```

peut ne contenir aucune valeur.

# Valeur minimale

Paramètre :

```php
min
```

Permet de définir la plus petite valeur acceptée.

Exemple :

```php
$age = new ColumnInt(
    min: 4
);
```

Valeurs acceptées :

```text
4
10
99
```

Valeurs refusées :

```text
0
2
3
```

Message obtenu :

```text
La valeur ne peut pas être inférieure à 4.
```

# Valeur maximale

Paramètre :

```php
max
```

Permet de définir la plus grande valeur acceptée.

Exemple :

```php
$age = new ColumnInt(
    max: 99
);
```

Valeurs refusées :

```text
100
150
```

Message obtenu :

```text
La valeur ne peut pas être supérieure à 99.
```

# Exemple métier

Dans une classe représentant des catégories de coureurs :

```php
$ageMin = new ColumnInt(
    required: true,
    min: 4,
    max: 99
);

$this->addColumn('ageMin', $ageMin);
```

La colonne :

```text
ageMin
```

doit obligatoirement contenir un entier compris entre :

```text
4 et 99
```

# La méthode `checkValidity()`

```php
public function checkValidity(): bool
```

Cette méthode réalise les contrôles dans l'ordre suivant :

1. vérification des règles communes de `Column` ;
2. vérification que la valeur est un entier ;
3. contrôle de la valeur minimale ;
4. contrôle de la valeur maximale ;
5. conversion de la valeur en entier.

## Exemple de validation réussie

```php
$age = new ColumnInt();

$age->Value = "25";

if ($age->checkValidity()) {

    echo $age->Value;

}
```

Après validation :

```php
$age->Value === 25
```

La valeur initialement reçue sous forme de texte est convertie en entier.

# Exemple de validation échouée

```php
$age = new ColumnInt();

$age->Value = "vingt";

if (!$age->checkValidity()) {

    echo $age->getValidationMessage();

}
```

Résultat :

```text
La valeur doit être un entier.
```

# Conversion automatique

Une valeur issue d'un formulaire arrive généralement sous forme de chaîne :

```php
$_POST['age']
```

contient :

```text
"25"
```

Après validation :

```php
ColumnInt->Value
```

contient :

```php
25
```

Cette conversion permet de manipuler ensuite une vraie valeur numérique dans l'application.

# Utilisation avec les opérations CRUD

Les propriétés héritées permettent également de contrôler les opérations SQL.

Exemple :

```php
$id = new ColumnInt(
    required: true,
    insertable: false,
    updatable: false
);
```

Cette colonne :

- existe dans la table ;
- est obligatoire ;
- n'est jamais fournie lors d'un ajout ;
- ne peut jamais être modifiée.

Cas typique :

```text
id auto-incrémenté par la base de données
```

# Exemple complet dans une classe métier

```php
protected function defineColumns(): void
{
    $colonne = new ColumnInt(
        required: true,
        min: 1,
        max: 999
    );

    $this->addColumn('classement', $colonne);
}
```

La colonne `classement` accepte uniquement :

```text
1 à 999
```

# Gestion des erreurs

Lorsqu'une validation échoue :

```php
if (!$colonne->checkValidity()) {

    echo $colonne->getValidationMessage();

}
```

Messages possibles :

```text
Veuillez renseigner ce champ.
```

```text
La valeur doit être un entier.
```

```text
La valeur ne peut pas être inférieure à 1.
```

```text
La valeur ne peut pas être supérieure à 999.
```

# Bonnes pratiques

Les règles doivent être définies dans la classe métier.

Exemple :

```php
protected function defineColumns(): void
{
    $age = new ColumnInt(
        required: true,
        min: 4,
        max: 99
    );

    $this->addColumn('age', $age);
}
```

Ainsi toutes les opérations utilisent automatiquement les mêmes règles :

- ajout ;
- modification ;
- import ;
- traitements internes.

# À retenir

`ColumnInt` permet de représenter une colonne entière avec ses règles métier.

Elle centralise :

- l'obligation de présence ;
- le type attendu ;
- les limites autorisées ;
- la conversion de la valeur.

Une seule déclaration dans la classe métier garantit que les données manipulées par l'application restent cohérentes.