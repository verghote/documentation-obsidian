## Présentation

La classe `ColumnText` représente une **colonne métier contenant une chaîne de caractères**.

Elle hérite de la classe `Column` et ajoute les contrôles spécifiques aux textes :

- longueur minimale et maximale ;
- contrôle par expression régulière ;
- transformation de la casse ;
- suppression éventuelle des accents ;
- suppression des espaces inutiles.

Elle est utilisée dans les classes métier pour représenter les colonnes SQL de type texte :

- `VARCHAR` ;
- `CHAR` ;
- `TEXT`.

Exemple :

```php
$nom = new ColumnText(
    required: true,
    maxLength: 50
);

$this->addColumn('nom', $nom);
```

La colonne `nom` devra obligatoirement contenir une chaîne de caractères d'au maximum 50 caractères.

# Héritage

`ColumnText` hérite de `Column`.

Elle possède donc les propriétés communes :

```text
Column
 |
 +-- Value
 +-- Required
 +-- Insertable
 +-- Updatable
 |
 +-- ColumnText
```

Elle bénéficie notamment :

- de la gestion de la valeur ;
- du contrôle des champs obligatoires ;
- de la gestion des messages d'erreur.

# Le constructeur

```php
public function __construct(
    bool $required = true,
    bool $insertable = true,
    bool $updatable = true,
    ?string $pattern = null,
    ?int $minLength = null,
    ?int $maxLength = null,
    TextCase $casse = TextCase::None,
    bool $supprimerAccent = false,
    bool $supprimerEspaceSuperflu = false
)
```

Le constructeur permet de définir toutes les règles associées à la colonne texte.

# Paramètre `required`

Détermine si la valeur est obligatoire.

Exemple :

```php
$nom = new ColumnText(
    required: true
);
```

Valeurs refusées :

```text
null
""
"     "
```

Exemple d'un champ facultatif :

```php
$commentaire = new ColumnText(
    required: false
);
```

# Contrôle de longueur

## Longueur minimale

Paramètre :

```php
minLength
```

Exemple :

```php
$nom = new ColumnText(
    minLength: 3
);
```

Les valeurs suivantes seront refusées :

```text
"A"
"AB"
```

## Longueur maximale

Paramètre :

```php
maxLength
```

Exemple :

```php
$nom = new ColumnText(
    maxLength: 30
);
```

Une valeur dépassant 30 caractères sera refusée.

# Contrôle par expression régulière

Paramètre :

```php
pattern
```

Permet d'imposer un format précis.

Exemple :

```php
$id = new ColumnText(
    pattern: '^[A-Z]{2}[0-9]{3}$'
);
```

Valeurs acceptées :

```text
AB123
XY456
```

Valeurs refusées :

```text
abc
A1234
12AB
```

## Exemple métier

Dans une catégorie de coureurs :

```php
$id = new ColumnText(
    required: true,
    pattern: '^(M(10|[0-9])|[A-Z]{2})$',
    minLength: 2,
    maxLength: 3
);
```

Cette règle autorise par exemple :

```text
EA
M0
M10
```

# Gestion de la casse

La classe utilise l'énumération :

```php
enum TextCase
{
    case None;
    case Upper;
    case Lower;
    case Word;
    case First;
}
```

Elle permet de transformer automatiquement le texte avant son enregistrement.

## Aucun changement

```php
casse: TextCase::None
```

La valeur est conservée.

Exemple :

```text
Jean DUPONT
```

reste :

```text
Jean DUPONT
```

## Tout en majuscules

```php
casse: TextCase::Upper
```

Exemple :

```text
jean dupont
```

devient :

```text
JEAN DUPONT
```

## Tout en minuscules

```php
casse: TextCase::Lower
```

Exemple :

```text
Jean DUPONT
```

devient :

```text
jean dupont
```


## Première lettre de chaque mot en majuscule

```php
casse: TextCase::Word
```

Exemple :

```text
JEAN DUPONT
```

devient :

```text
Jean Dupont
```

## Première lettre du texte en majuscule

```php
casse: TextCase::First
```

Exemple :

```text
jEAN
```

devient :

```text
Jean
```

# Suppression des accents

Paramètre :

```php
supprimerAccent
```

Exemple :

```php
$nomFichier = new ColumnText(
    supprimerAccent: true
);
```

Transformation :

```text
Élodie
```

devient :

```text
Elodie
```

Cette option est utile notamment pour :

- les noms de fichiers ;
- les identifiants techniques ;
- les recherches sans accent.

# Suppression des espaces superflus

Paramètre :

```php
supprimerEspaceSuperflu
```

Cette option remplace plusieurs espaces consécutifs par un seul.

Exemple :

Avant :

```text
Jean     Dupont
```

Après validation :

```text
Jean Dupont
```

# La méthode `checkValidity()`

```php
public function checkValidity(): bool
```

Cette méthode réalise les contrôles dans l'ordre suivant :

1. vérification des règles communes de `Column` ;
2. suppression des espaces inutiles ;
3. suppression éventuelle des accents ;
4. transformation de la casse ;
5. contrôle du format avec l'expression régulière ;
6. contrôle de la longueur ;
7. enregistrement de la valeur transformée.

# Exemple complet

```php
$nom = new ColumnText(
    required: true,
    maxLength: 50,
    casse: TextCase::Word,
    supprimerEspaceSuperflu: true
);

$nom->Value = "  JEAN     DUPONT ";

if ($nom->checkValidity()) {

    echo $nom->Value;

}
```

Résultat :

```text
Jean Dupont
```

# Gestion des erreurs

Si une validation échoue :

```php
if (!$nom->checkValidity()) {

    echo $nom->getValidationMessage();

}
```

Exemples de messages :

```text
Veuillez renseigner ce champ.
```

ou :

```text
Veuillez réduire ce texte afin de ne pas dépasser 50 caractères.
```

# Bonnes pratiques

## Définir les règles dans la classe métier

Les règles doivent être déclarées dans la classe représentant la table.

Exemple :

```php
protected function defineColumns(): void
{
    $nom = new ColumnText(
        required: true,
        minLength: 3,
        maxLength: 50,
        casse: TextCase::Word
    );

    $this->addColumn('nom', $nom);
}
```

La validation est alors centralisée et utilisée par toutes les opérations :

- ajout ;
- modification ;
- import ;
- API.

# À retenir

`ColumnText` permet de déclarer une colonne texte avec ses règles métier.

Elle évite de disperser les contrôles dans l'application.

Une seule définition permet de garantir que la donnée respecte toujours :

- son format ;
- sa longueur ;
- sa présentation ;
- ses contraintes métier.