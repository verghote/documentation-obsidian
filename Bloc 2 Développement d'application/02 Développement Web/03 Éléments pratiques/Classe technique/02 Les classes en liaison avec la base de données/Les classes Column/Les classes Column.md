Les classes `Column` permettent de définir les colonnes métier d'une classe héritant de `Table`.

Chaque colonne associe :

- un type de donnée ;
- des règles de validation ;
- des contraintes utilisées lors des opérations CRUD.

Les colonnes sont déclarées dans la méthode `defineColumns()` de la classe métier.

Exemple général :

```php
$col = new ColumnText(
    required:true,
    maxLength:50
);

$this->addColumn('nom', $col);
````

# Column

## Rôle

Classe de base de toutes les colonnes métier.

Elle gère :

- la valeur de la colonne ;
- le caractère obligatoire ;
- l'utilisation lors d'un ajout ;
- l'utilisation lors d'une modification ;
- le message d'erreur.

Elle n'est jamais utilisée directement.

# ColumnText

## Rôle

Contrôle une chaîne de caractères courte.

Contrôles possibles :

- longueur minimale ;
- longueur maximale ;
- expression régulière ;
- transformation de casse ;
- suppression des accents ;
- suppression des espaces inutiles.

Exemple :

```php
$col = new ColumnText(
    required:true,
    minLength:2,
    maxLength:30
);

$this->addColumn('nom', $col);
```

# ColumnInt

## Rôle

Contrôle une valeur entière.

Contrôles possibles :

- valeur minimale ;
- valeur maximale.

Exemple :

```php
$col = new ColumnInt(
    required:true,
    min:0,
    max:120
);

$this->addColumn('age', $col);
```

# ColumnDate

## Rôle

Contrôle une date au format :

```
AAAA-MM-JJ
```

Contrôles possibles :

- date minimale ;
- date maximale.

Exemple :

```php
$col = new ColumnDate(
    required:true,
    min:'1900-01-01'
);

$this->addColumn('dateNaissance', $col);
```

# ColumnBool

## Rôle

Contrôle une valeur booléenne.

Convertit les valeurs provenant des formulaires HTML :

- 0 / 1 ;
- true / false.

Exemple :

```php
$col = new ColumnBool(
    required:true
);

$this->addColumn('actif', $col);
```

# ColumnList

## Rôle

Contrôle qu'une valeur appartient à une liste définie.

Exemple :

```php
$col = new ColumnList(
    values:[
        'Administrateur',
        'Utilisateur'
    ]
);

$this->addColumn('role', $col);
```

Valeurs acceptées :

```
Administrateur
Utilisateur
```

# ColumnEmail

## Rôle

Contrôle une adresse électronique.

Contrôles réalisés :

- format de l'adresse ;
- existence du domaine ;
- longueur maximale.

Exemple :

```php
$col = new ColumnEmail(
    required:true,
    maxLength:100
);

$this->addColumn('email', $col);
```

# ColumnUrl

## Rôle

Contrôle une adresse Internet.

Contrôles réalisés :

- format de l'URL ;
- éventuellement accessibilité de la ressource.

Exemple :

```php
$col = new ColumnUrl(
    required:false
);

$this->addColumn('siteWeb', $col);
```

# ColumnTextarea

## Rôle

Contrôle un texte long pouvant contenir plusieurs lignes.

Gestion :

- contenu HTML ;
- encodage HTML ;
- filtrage des balises autorisées ;
- détection de contenus dangereux.

Exemple :

```php
$col = new ColumnTextarea(
    required:false,
    acceptHtml:true
);

$this->addColumn('description', $col);
```

# Tableau récapitulatif

|Classe|Type de donnée|Utilisation|
|---|---|---|
|Column|Classe de base|Toutes les colonnes métier|
|ColumnText|Texte court|Nom, libellé, code|
|ColumnInt|Entier|Age, quantité, compteur|
|ColumnDate|Date|Date de naissance, date création|
|ColumnBool|Booléen|Actif, validé, disponible|
|ColumnList|Liste de valeurs|Statut, catégorie, type|
|ColumnEmail|Adresse email|Contact|
|ColumnUrl|Adresse web|Site internet|
|ColumnTextarea|Texte long|Description, commentaire|

# Exemple complet dans une classe métier

```
protected function defineColumns(): void
{
    $this->addColumn(
        'nom',
        new ColumnText(
            required:true,
            maxLength:50
        )
    );

    $this->addColumn(
        'age',
        new ColumnInt(
            min:0,
            max:120
        )
    );

    $this->addColumn(
        'dateNaissance',
        new ColumnDate()
    );

    $this->addColumn(
        'email',
        new ColumnEmail()
    );
}
```

# Principe à retenir

Dans une classe métier :

```
Table
 |
 +-- Column
      |
      +-- ColumnText
      +-- ColumnInt
      +-- ColumnDate
      +-- ColumnBool
      +-- ColumnList
      +-- ColumnEmail
      +-- ColumnUrl
      +-- ColumnTextarea
```

Le choix de la classe `Column` permet de centraliser les contrôles et d'éviter de dupliquer les validations dans chaque classe métier.