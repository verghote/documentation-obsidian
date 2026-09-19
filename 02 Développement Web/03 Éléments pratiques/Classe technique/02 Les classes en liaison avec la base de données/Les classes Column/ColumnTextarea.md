# Rôle

La classe `ColumnTextarea` contrôle une donnée texte destinée à contenir un contenu long et éventuellement structuré.

Elle est utilisée pour les colonnes correspondant généralement à des zones de saisie multilignes :

- commentaires ;
- descriptions ;
- observations ;
- textes enrichis.

Elle hérite de la classe `Column` et conserve donc les règles communes :

- caractère obligatoire ou facultatif ;
- utilisation lors d'un ajout ;
- utilisation lors d'une modification ;
- gestion des messages d'erreur.

Elle ajoute des contrôles spécifiques aux contenus textuels longs, notamment la gestion du HTML.

# Héritage

```text
Column
   |
   |
   +---- ColumnTextarea
````

`ColumnTextarea` utilise le fonctionnement général de `Column` puis complète la méthode :

```
checkValidity()
```

afin d'effectuer des contrôles adaptés aux textes longs.

# Utilisation dans une classe métier

Exemple :

```
$col = new ColumnTextarea(
    required: false,
    encoderHtml: true
);

$this->addColumn('description', $col);
```

Cette colonne :

- accepte une absence de valeur ;
- autorise un stockage sécurisé du contenu HTML.

# Propriétés spécifiques

## EncoderHtml

```
public readonly bool $EncoderHtml;
```

Indique si le contenu doit être encodé en HTML avant stockage.

Lorsque cette option est activée, les caractères spéciaux HTML sont transformés.

Exemple :

Valeur saisie :

```
<script>alert('test')</script>
```

devient :

```
&lt;script&gt;alert('test')&lt;/script&gt;
```

Le navigateur affichera alors le texte au lieu d'exécuter le script.

Cette option est utilisée lorsque le contenu doit être affiché comme du texte brut.

## AcceptHtml

```
public readonly bool $AcceptHtml;
```

Indique si le contenu HTML est autorisé.

Exemple :

```
new ColumnTextarea(
    acceptHtml:true
);
```

Le contenu peut conserver certaines balises HTML.

À l'inverse :

```
new ColumnTextarea(
    acceptHtml:false
);
```

les balises non autorisées seront supprimées.

## BalisesAutorisees

```
public readonly array $BalisesAutorisees;
```

Liste des balises HTML conservées lorsque le HTML est accepté partiellement.

Valeur par défaut :

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

Exemple :

Un texte contenant :

```
<p>Bonjour</p>
<strong>Bienvenue</strong>
```

pourra conserver :

```
<strong>Bienvenue</strong>
```

mais supprimer les balises non prévues.

# Constructeur

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

Paramètres hérités :

|Paramètre|Rôle|
|---|---|
|required|La valeur est-elle obligatoire ?|
|insertable|La colonne participe-t-elle aux ajouts ?|
|updatable|La colonne peut-elle être modifiée ?|

Paramètres spécifiques :

|Paramètre|Rôle|
|---|---|
|encoderHtml|Encode tout le contenu HTML|
|acceptHtml|Autorise le HTML|
|balisesAutorisees|Liste des balises conservées|

# Validation avec checkValidity()

La validation se déroule en plusieurs étapes.

## 1. Vérification des règles communes

La méthode commence par appeler :

```
parent::checkValidity()
```

Cela vérifie notamment :

- qu'une valeur obligatoire est présente ;
- qu'une chaîne vide n'est pas acceptée lorsqu'elle est obligatoire.

## 2. Détection des contenus dangereux

La classe recherche certaines séquences pouvant représenter un risque :

- scripts JavaScript ;
- commandes SQL ;
- commentaires SQL.

Exemples détectés :

```
<script>
```

ou :

```
DROP TABLE
```

ou :

```
--
```

Si une séquence interdite est trouvée :

```
La valeur contient des caractères ou des mots interdits.
```

La validation échoue.

## 3. Encodage HTML

Lorsque :

```
$EncoderHtml = true
```

le contenu est transformé avec :

```
htmlspecialchars()
```

Cela protège contre l'exécution de code HTML ou JavaScript.

Exemple :

Avant :

```
<img src="photo.jpg">
```

Après encodage :

```
&lt;img src=&quot;photo.jpg&quot;&gt;
```

## 4. Suppression des balises interdites

Lorsque :

```
AcceptHtml = false
```

la méthode utilise :

```
strip_tags()
```

pour supprimer les balises non autorisées.

Les balises présentes dans :

```
BalisesAutorisees
```

sont conservées.

# Exemple complet

Définition d'une colonne commentaire :

```
$col = new ColumnTextarea(
    required:false,
    maxLength:2000
);

$this->addColumn('commentaire', $col);
```

ou avec HTML autorisé :

```
$col = new ColumnTextarea(
    required:false,
    encoderHtml:false,
    acceptHtml:true
);

$this->addColumn('description', $col);
```

# Différence avec ColumnText

|ColumnText|ColumnTextarea|
|---|---|
|Texte court|Texte long|
|Une ligne|Plusieurs lignes|
|Contrôle longueur et format|Contrôle contenu HTML|
|Expressions régulières possibles|Sécurisation du contenu riche|

# Points d'attention

## Le HTML accepté doit rester limité

Autoriser du HTML complet peut présenter des risques.

Il est préférable :

- d'autoriser uniquement certaines balises ;
- d'encoder le contenu lorsqu'il n'est pas destiné à être interprété ;
- de filtrer les contenus provenant des utilisateurs.

# À retenir

`ColumnTextarea` permet de gérer les champs texte longs.

Elle ajoute à `Column` :

- la gestion du contenu multiligne ;
- le contrôle des contenus dangereux ;
- l'encodage HTML ;
- le filtrage des balises autorisées.

Elle est adaptée aux champs contenant des descriptions, commentaires ou contenus enrichis.