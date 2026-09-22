Voici la version revue et complétée de la documentation de **`ColumnTextarea`**. Elle intègre la séparation claire entre la méthode **`sanitize()`** (pour le filtrage XSS, l'assainissement HTML et l'encodage) et la méthode **`checkValidity()`** (pour la validation des contraintes).

# Documentation : La classe `ColumnTextarea`

## Rôle

La classe `ColumnTextarea` contrôle et assainit une donnée texte destinée à contenir un contenu long et éventuellement structuré.

Elle est utilisée pour les colonnes correspondant généralement à des zones de saisie multilignes :

- commentaires ;
    
- descriptions ;
    
- observations ;
    
- textes enrichis.
    

Elle hérite de la classe `Column` et conserve donc les règles communes :

- **Nettoyage automatique préalable (`sanitize`)** ;
    
- Caractère obligatoire ou facultatif (`Required`) ;
    
- Utilisation lors d'un ajout (`Insertable`) ;
    
- Utilisation lors d'une modification (`Updatable`) ;
    
- Gestion des messages d'erreur.
    

Elle ajoute des traitements et contrôles spécifiques aux contenus textuels longs, notamment le nettoyage du code HTML et la protection contre les failles XSS.

## Héritage

Plaintext

```
Column
   |
   +---- Value
   +-- Required
   +-- Insertable
   +-- Updatable
   |
   +---- ColumnTextarea
            +-- EncoderHtml
            +-- AcceptHtml
            +-- BalisesAutorisees
```

`ColumnTextarea` utilise le fonctionnement général de `Column` puis surcharge :

- **`sanitize()`** : pour filtrer et nettoyer le contenu HTML avant toute validation ;
    
- **`checkValidity()`** : pour effectuer les contrôles de conformité adaptés aux textes longs.
    

## Utilisation dans une classe métier

PHP

```
$col = new ColumnTextarea(
    required: false,
    encoderHtml: true
);

$this->addColumn('description', $col);
```

Cette colonne :

- accepte une absence de valeur ;
    
- nettoie et encadre le stockage sécurisé du contenu.
    

## Propriétés spécifiques

### `EncoderHtml`

PHP

```
public readonly bool $EncoderHtml;
```

Indique si le contenu doit être intégralement encodé en HTML avant stockage.

Lorsque cette option est activée (`true`), les caractères spéciaux HTML sont transformés via `htmlspecialchars()`.

_Exemple :_

Valeur saisie :

HTML

```
<script>alert('test')</script>
```

devient après `sanitize()` :

HTML

```
&lt;script&gt;alert('test')&lt;/script&gt;
```

Le navigateur affichera alors le texte brut au lieu d'exécuter le script.

### `AcceptHtml`

PHP

```
public readonly bool $AcceptHtml;
```

Indique si le contenu HTML est autorisé partiellement.

- `acceptHtml: true` : Le contenu conserve les balises autorisées et le code malveillant est retiré par le nettoyage métier (`Std::nettoyerHtml()`).
    
- `acceptHtml: false` : Les balises non incluses dans `$BalisesAutorisees` sont supprimées via `strip_tags()`.
    

### `BalisesAutorisees`

PHP

```
public readonly array $BalisesAutorisees;
```

Liste des balises HTML conservées lorsque `acceptHtml` est à `false`.

**Valeur par défaut :**

PHP

```
[
    '<br>',
    '<span>',
    '<b>',
    '<i>',
    '<strong>',
    '<ul>',
    '<li>',
    '<img>',
    '<a>',
    '<div>'
]
```

## Le constructeur

PHP

```
public function __construct(
    bool $required = true,
    bool $insertable = true,
    bool $updatable = true,
    bool $encoderHtml = false,
    bool $acceptHtml = true,
    array $balisesAutorisees = [...]
)
```

|**Paramètre**|**Rôle**|
|---|---|
|**`required`**|La valeur est-elle obligatoire ?|
|**`insertable`**|La colonne participe-t-elle aux insertions (`INSERT`) ?|
|**`updatable`**|La colonne peut-elle être modifiée (`UPDATE`) ?|
|**`encoderHtml`**|Encode tout le contenu HTML en caractères d'échappement.|
|**`acceptHtml`**|Autorise ou supprime les balises HTML.|
|**`balisesAutorisees`**|Liste des balises HTML conservées si `acceptHtml = false`.|

## Nettoyage et assainissement : La méthode `sanitize()`

PHP

```
public function sanitize(mixed $value): mixed
```

Cette méthode est appelée automatiquement avant les contrôles de validation. Elle réalise l'assainissement du texte dans l'ordre suivant :

Plaintext

```
[Chaîne brute transmise]
          │
          ▼
 1. Assainissement XSS générique (Std::nettoyerHtml)
          │
          ▼
 2. Si EncoderHtml == true ? ───▶ [htmlspecialchars()] ───▶ [Retourne le texte encodé]
          │ (non)
          ▼
 3. Si AcceptHtml == false ? ───▶ [strip_tags(..., BalisesAutorisees)]
          │ (oui)
          ▼
 [Valeur nettoyée enregistrée dans $this->Value]
```

### Exemple de comportement de `sanitize()`

- **Si `encoderHtml: true`** :
    
    `"<img src='photo.jpg'>"` ➔ `&lt;img src=&quot;photo.jpg&quot;&gt;`
    
- **Si `acceptHtml: false`** :
    
    `"<p>Bonjour</p><strong>Bienvenue</strong>"` ➔ `"Bonjour<strong>Bienvenue</strong>"` _(la balise `<p>` non autorisée est supprimée)_.
    

## Validation avec `checkValidity()`

PHP

```
public function checkValidity(): bool
```

La validation s'exécute **sur la valeur déjà nettoyée par `sanitize()`** :

1. **Appel de `parent::checkValidity()`** :
    
    - Exécute `$this->Value = $this->sanitize($this->Value)`.
        
    - Vérifie la règle `Required` (refuse une chaîne vide si le champ est obligatoire).
        
2. **Validation métier** :
    
    - Si la valeur est facultative et vide, elle est validée.
        
    - Si des contraintes supplémentaires sont définies, elles sont contrôlées sur la chaîne assainie.
        

## Exemple complet

### Définition d'un commentaire brut (sans HTML)

PHP

```
$col = new ColumnTextarea(
    required: false,
    encoderHtml: true
);

$this->addColumn('commentaire', $col);
```

### Définition d'une description avec HTML restreint

PHP

```
$col = new ColumnTextarea(
    required: true,
    encoderHtml: false,
    acceptHtml: false,
    balisesAutorisees: ['<br>', '<b>', '<i>', '<strong>']
);

$this->addColumn('description', $col);
```

## Comparaison : `ColumnText` vs `ColumnTextarea`

|**Critère**|**ColumnText**|**ColumnTextarea**|
|---|---|---|
|**Type de contenu**|Texte court (une ligne)|Texte long / multiligne|
|**Nettoyage principal**|Titre, espaces superflus, casse, accents|Sanitisation HTML, filtrage XSS|
|**Contrôles spécifiques**|Longueur (`MinLength`/`MaxLength`), Regex (`Pattern`)|Encodage HTML, filtrage par balises autorisées|
|**Champs SQL cibles**|`VARCHAR`, `CHAR`|`TEXT`, `MEDIUMTEXT`, `LONGTEXT`|

## Bonnes pratiques

1. **Le HTML accepté doit rester limité** : autoriser du HTML complet peut présenter des risques d'injection. Privilégiez toujours la liste blanche avec `balisesAutorisees`.
    
2. **Encoder par défaut** : si la donnée n'a pas besoin d'être affichée sous forme de mise en page HTML enrichie, activez `encoderHtml: true`.
    
3. **Déclarer dans la classe métier** : configurez vos champs dans `defineColumns()` afin que le nettoyage et la validation s'appliquent automatiquement sur toutes les opérations CRUD.
    

## À retenir

`ColumnTextarea` centralise la gestion des champs texte longs.

Elle associe :

1. **La gestion du contenu multiligne** ;
    
2. **L'assainissement automatique contre les injections XSS** ;
    
3. **L'encodage ou le filtrage fin des balises HTML (`sanitize`)** ;
    
4. **La validation des contraintes d'obligation (`checkValidity`)**.