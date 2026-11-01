## Présentation

La classe `ColumnText` représente une colonne métier contenant une chaîne de caractères.

Elle hérite de la classe `Column` et associe :

- **Un nettoyage et assainissement automatique (`sanitize`)** : normalisation métier (titres), suppression d'accents, espaces superflus, transformation de la casse ;
    
- **Des règles de validation** : longueur minimale et maximale, contrôle par expression régulière.
    

Elle est utilisée dans les classes métier pour représenter les colonnes SQL de type texte : `VARCHAR`, `CHAR`, `TEXT`.

PHP

```
$nom = new ColumnText(
    required: true,
    maxLength: 50,
    casse: TextCase::Word,
    supprimerEspaceSuperflu: true
);

$this->addColumn('nom', $nom);
```

_Ici, la colonne `nom` sera nettoyée, remise en forme (première lettre de chaque mot en majuscule, espaces nettoyés) et devra comporter au maximum 50 caractères._

## Héritage

`ColumnText` hérite de `Column`. Elle possède donc les propriétés et méthodes communes :

Plaintext

```
Column
 |
 +-- Value
 +-- Required
 +-- Insertable
 +-- Updatable
 |
 +-- ColumnText
      +-- Pattern
      +-- MinLength
      +-- MaxLength
      +-- Casse
      +-- SupprimerAccent
      +-- SupprimerEspaceSuperflu
```

Elle bénéficie notamment :

- de la gestion de la valeur ;
    
- du déclenchement automatique du nettoyage via `sanitize()` ;
    
- du contrôle des champs obligatoires (`Required`) ;
    
- de la gestion des messages d'erreur.
    

## Le constructeur

PHP

```
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

## Nettoyage et assainissement : La méthode `sanitize()`

PHP

```
public function sanitize(mixed $value): mixed
```

Cette méthode est appelée automatiquement avant les contrôles de validation. Elle réalise les transformations dans l'ordre suivant :

1. **Nettoyage métier de base** : suppression des espaces invisibles/parasites (via `Std::nettoyerTitre()`) ;
    
2. **Suppression des accents** (si `supprimerAccent: true`) ;
    
3. **Suppression des espaces superflus** (si `supprimerEspaceSuperflu: true`) ;
    
4. **Transformation de la casse** (selon la propriété `Casse`).
    

### 1. Suppression des accents

- **Paramètre** : `supprimerAccent: true`
    
- **Exemple** : `"Élodie"` ➔ `"Elodie"`
    
- **Usages** : noms de fichiers, identifiants techniques, slugs.
    

### 2. Suppression des espaces superflus

- **Paramètre** : `supprimerEspaceSuperflu: true`
    
- **Exemple** : `"Jean Dupont "` ➔ `"Jean Dupont"`
    

### 3. Gestion de la casse (`TextCase`)

L'énumération `TextCase` définit les règles de transformation de la casse :

PHP

```
enum TextCase
{
    case None;   // Aucun changement ("Jean DUPONT" -> "Jean DUPONT")
    case Upper;  // Tout en majuscules ("jean dupont" -> "JEAN DUPONT")
    case Lower;  // Tout en minuscules ("Jean DUPONT" -> "jean dupont")
    case Word;   // Majuscule à chaque mot ("JEAN DUPONT" -> "Jean Dupont")
    case First;  // Majuscule sur la 1re lettre uniquement ("jEAN" -> "Jean")
}
```

## Règles de validation (exécutées sur la valeur nettoyée)

### 1. Contrôle du caractère obligatoire (`Required`)

Rejette `null` ou une chaîne vide.

PHP

```
$nom = new ColumnText(required: true);
```

- **Valeurs refusées** : `null`, `""`, `" "`.
    

### 2. Longueur minimale (`minLength`) et maximale (`maxLength`)

- **`minLength`** : rejet si le texte comporte moins de $N$ caractères.
    
- **`maxLength`** : rejet si le texte dépasse $N$ caractères.
    

PHP

```
$nom = new ColumnText(minLength: 3, maxLength: 30);
```

### 3. Expression régulière (`pattern`)

Contrôle que le texte respecte un format précis (regex).

PHP

```
$id = new ColumnText(
    required: true,
    pattern: '^(M(10|[0-9])|[A-Z]{2})$',
    minLength: 2,
    maxLength: 3
);
```

- **Valeurs acceptées** : `"EA"`, `"M0"`, `"M10"`.
    
- **Valeurs refusées** : `"abc"`, `"A1234"`, `"12AB"`.
    

> **Note importante** : L'expression régulière est contrôlée **après** l'exécution de `sanitize()`. Si vous avez activé `casse: TextCase::Upper`, votre motif regex peut utiliser directement des majuscules (`^[A-Z]+$`), car la valeur aura déjà été passée en majuscules.

## La méthode `checkValidity()`

PHP

```
public function checkValidity(): bool
```

Cette méthode orchestre la validation complète :

Plaintext

```
[Appel de checkValidity()]
         │
         ▼
 1. Appel de parent::checkValidity()
         │
         ├───▶ Exécute $this->Value = $this->sanitize($this->Value)
         │
         └───▶ Vérifie la règle Required
         │
         ▼
 2. Si la valeur nettoyée est vide et optionnelle ➔ Retourne true
         │
         ▼
 3. Contrôle du Pattern (regex) sur la valeur nettoyée
         │
         ▼
 4. Contrôle de MinLength et MaxLength sur la valeur nettoyée
         │
         ▼
 5. Validation réussie (la valeur nettoyée est conservée dans $this->Value)
```

### Exemple d'exécution complet

PHP

```
$nom = new ColumnText(
    required: true,
    maxLength: 50,
    casse: TextCase::Word,
    supprimerEspaceSuperflu: true
);

$nom->Value = "  JEAN     DUPONT ";

if ($nom->checkValidity()) {
    echo $nom->Value; // Affiche : "Jean Dupont"
}
```

## Gestion des erreurs

En cas d'échec de la validation, le message est accessible via `getValidationMessage()` :

PHP

```
if (!$nom->checkValidity()) {
    echo $nom->getValidationMessage();
}
```

**Exemples de messages générés :**

- `"Veuillez renseigner ce champ."`
    
- `"La valeur transmise n'est pas valide."`
    
- `"Veuillez réduire ce texte afin de ne pas dépasser 50 caractères."`
    
- `"Veuillez allonger ce texte pour qu'il comporte au moins 3 caractères. Il en compte actuellement 1."`
    

## Bonnes pratiques

Déclarez toujours les règles de vos colonnes dans la méthode `defineColumns()` de vos classes métier (`Table`) :

PHP

```
protected function defineColumns(): void
{
    $nom = new ColumnText(
        required: true,
        minLength: 2,
        maxLength: 50,
        casse: TextCase::Word,
        supprimerEspaceSuperflu: true
    );

    $this->addColumn('nom', $nom);
}
```

Grâce à cette déclaration centralisée, toutes les entrées utilisateur (formulaires, requêtes HTTP, API, imports) seront automatiquement **assainies, transformées et validées** de manière uniforme avant d'être enregistrées en base de données.