# Rôle

`ColumnDate` est une classe de validation utilisée pour représenter une colonne métier contenant une date.

Elle hérite de la classe abstraite `Column` et ajoute les contrôles spécifiques aux dates :

- présence obligatoire ou non ;
- format attendu `AAAA-MM-JJ` ;
- existence réelle de la date ;
- date minimale autorisée ;
- date maximale autorisée.

Elle est utilisée dans les classes métier qui représentent des tables SQL lorsque certaines colonnes contiennent des dates.

# Utilisation

Dans une classe métier héritant de `Table`, une colonne de type date est déclarée avec :

```php
$col = new ColumnDate(
    required: true,
    min: '2000-01-01',
    max: '2030-12-31'
);

$this->addColumn('dateNaissance', $col);
````

La colonne `dateNaissance` sera alors contrôlée automatiquement lors des opérations :

- ajout (`insert`) ;
- modification (`update`).

# Paramètres du constructeur

```
public function __construct(
    bool $required = true,
    bool $insertable = true,
    bool $updatable = true,
    ?string $min = null,
    ?string $max = null
)
```

# required

Indique si la valeur est obligatoire.

### Exemple

```
required: true
```

La colonne doit obligatoirement contenir une date.

```
required: false
```

La colonne peut rester vide.

## insertable

Indique si cette colonne peut être utilisée lors d'une création d'enregistrement.

Exemple :

```
insertable: false
```

Cas courant :

- date de création générée automatiquement ;
- date renseignée par la base de données.

## updatable

Indique si cette colonne peut être modifiée.

Exemple :

```
updatable: false
```

Cas courant :

```
dateCreation
```

Une date de création ne doit généralement jamais être modifiée après insertion.

## min

Définit la date minimale autorisée.

Format obligatoire :

```
AAAA-MM-JJ
```

Exemple :

```
min: '2005-01-01'
```

Une date antérieure sera refusée.

## max

Définit la date maximale autorisée.

Exemple :

```
max: '2026-12-31'
```

Une date postérieure sera refusée.

# Contrôles réalisés

Lors de l'appel :

```
$col->checkValidity();
```

la classe réalise plusieurs vérifications.

## 1 - Vérification de l'obligation

Héritée de `Column`.

Exemple :

```
required: true
```

Valeurs refusées :

```
null
""
"   "
```

Message retourné :

```
Veuillez renseigner ce champ.
```

## 2 - Vérification du format

Le format attendu est :

```
AAAA-MM-JJ
```

Exemples valides :

```
2026-07-28
2000-01-01
```

Exemples invalides :

```
28/07/2026
2026-7-28
2026-02-30
```

Message retourné :

```
Le format de la date est invalide.
```

## 3 - Vérification de l'existence réelle de la date

La classe utilise le calendrier réel.

Exemple :

```
2026-02-30
```

sera refusé car cette date n'existe pas.

## 4 - Contrôle de la date minimale

Exemple :

```
new ColumnDate(
    min: '2020-01-01'
);
```

Valeur reçue :

```
2019-12-31
```

Résultat :

```
La date doit être égale ou postérieure au 01/01/2020
```

## 5 - Contrôle de la date maximale

Exemple :

```
new ColumnDate(
    max: '2026-12-31'
);
```

Valeur reçue :

```
2027-01-01
```

Résultat :

```
La date doit être égale ou antérieure au 31/12/2026
```

# Exemple complet dans une classe métier

```
protected function defineColumns(): void
{
    $col = new ColumnDate(
        required: true,
        min: '1900-01-01',
        max: date('Y-m-d')
    );

    $this->addColumn('dateNaissance', $col);
}
```

Cette déclaration garantit que :

- une date de naissance est obligatoire ;
- elle doit être postérieure au 01/01/1900 ;
- elle ne peut pas être dans le futur.

# Valeur après validation

Avant validation :

```
$col->Value = '2026-07-28';
```

Après :

```
$col->checkValidity();
```

La valeur reste une chaîne au format :

```
2026-07-28
```

Ce format est directement compatible avec :

- MySQL ;
- PostgreSQL ;
- les échanges JSON ;
- les champs HTML de type `date`.

# Résumé

|Fonction|Gestion|
|---|---|
|Valeur obligatoire|Oui|
|Format AAAA-MM-JJ|Oui|
|Date réelle|Oui|
|Date minimale|Oui|
|Date maximale|Oui|
|Transformation de valeur|Non|
|Compatible CRUD|Oui|

`ColumnDate` permet donc de déclarer simplement une colonne date dans une classe métier tout en centralisant les règles de validation.