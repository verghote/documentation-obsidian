
Lorsqu'un champ d'une table contient des données saisies par l'utilisateur à l'aide de l'éditeur **TinyMCE**, on utilise un objet `ColumnTextarea`.

La méthode `sanitize()` de `ColumnTextarea` utilise alors la méthode :

```php
Std::nettoyerHtml()
```

Cette méthode fait appel au composant **HTMLPurifier**, qui nettoie le HTML fourni par l'utilisateur afin de limiter les risques d'attaques de type **XSS (Cross-Site Scripting)** tout en conservant la mise en forme autorisée.

La chaîne d'utilisation est donc :

```text
Utilisateur
    ↓
TinyMCE
    ↓
ColumnTextarea
    ↓
sanitize()
    ↓
Std::nettoyerHtml()
    ↓
HTMLPurifier
```

Le développeur utilise donc directement `ColumnTextarea` ; l'utilisation de **HTMLPurifier est transparente** et est prise en charge par les différentes classes techniques.
## 1. Le principe

Dans l'application, le développeur **n'utilise pas directement HTMLPurifier**.

Lorsqu'un champ doit accepter du contenu HTML, il utilise simplement la classe :

```
new ColumnTextarea(
    acceptHtml: true
);
```

La sécurité est ensuite prise en charge automatiquement par les différentes classes techniques.

Le développeur travaille donc avec `ColumnTextarea` et **n'a pas besoin de connaître le fonctionnement interne de HTMLPurifier**.

## 2. Ce que fait le développeur

Par exemple, pour un champ contenant du HTML provenant de TinyMCE :

```
$description = new ColumnTextarea(
    required: true,
    acceptHtml: true
);

$description->Value = $_POST['description'];

if ($description->checkValidity()) {
    // La valeur nettoyée peut être utilisée
}
```

C'est tout ce que le développeur a besoin de connaître.

Il fournit une valeur à `ColumnTextarea` puis appelle :

```
$description->checkValidity();
```

La chaîne de traitement est ensuite automatique.

## 3. Les paramètres de `ColumnTextarea`

`ColumnTextarea` possède plusieurs paramètres. Certains concernent directement le traitement et le nettoyage de la valeur.

```
new ColumnTextarea(
    required: true,
    insertable: true,
    updatable: true,
    encoderHtml: false,
    acceptHtml: true,
    balisesAutorisees: [...]
);
```

### `required`

Indique si le champ est obligatoire.

```
required: true
```

Ce paramètre intervient dans la validation du champ, mais **pas dans le nettoyage HTML**.

### `insertable` et `updatable`

Ces paramètres indiquent si la colonne peut être utilisée respectivement lors d'une insertion ou d'une modification.

Ils concernent donc les opérations CRUD et **n'interviennent pas directement dans le nettoyage HTML**.

### `acceptHtml`

C'est le paramètre principal concernant le HTML.

```
acceptHtml: true
```

Lorsque `true`, le HTML est accepté mais il est tout de même nettoyé par HTMLPurifier.

Il ne faut donc pas interpréter :

```
acceptHtml: true
```

comme « accepter n'importe quel HTML ».

Le HTML reste soumis au nettoyage de sécurité.

Lorsque `false`, `ColumnTextarea` utilise à la place :

```
strip_tags()
```

avec la liste définie par `balisesAutorisees`.

### `encoderHtml`

Ce paramètre détermine si le HTML doit ensuite être encodé avec :

```
htmlspecialchars()
```

Par exemple :

```
encoderHtml: true
```

transforme les caractères HTML en entités.

Il faut distinguer :

- **nettoyer** le HTML : HTMLPurifier ;
- **encoder** le HTML : `htmlspecialchars()`.

Ce sont deux opérations différentes.

### `balisesAutorisees`

Ce paramètre contient les balises autorisées lorsque :

```
acceptHtml: false
```

Par exemple :

```
balisesAutorisees: [
    '<br>',
    '<strong>',
    '<em>'
]
```

La classe utilise alors :

```
strip_tags($valeur, $this->BalisesAutorisees);
```

Cette liste intervient donc dans le mode où l'on souhaite conserver seulement certaines balises.

## 4. Les classes travaillent ensemble

```
Développeur
     │
     ▼
ColumnTextarea
     │
     ▼
Column
     │
     ▼
Std
     │
     ▼
HTMLPurifier
```

Chaque classe possède une responsabilité différente.

### `ColumnTextarea`

C'est la classe utilisée par le développeur pour un champ texte multiligne.

Elle sait notamment :

- si le champ est obligatoire ;
- si le HTML est accepté ;
- si le HTML doit être encodé ;
- quelles balises conserver lorsque le HTML n'est pas accepté globalement.

Elle surcharge la méthode :

```
sanitize()
```

### `Column`

`Column` est la classe mère.

Elle fournit notamment la méthode :

```
checkValidity()
```

Cette méthode :

1. nettoie la valeur ;
2. vérifie si le champ est obligatoire ;
3. retourne `true` ou `false`.

Le point important est que `Column` appelle :

```
$this->sanitize();
```

Grâce au polymorphisme, si l'objet réel est un `ColumnTextarea`, c'est automatiquement :

```
ColumnTextarea::sanitize()
```

qui est exécutée.

### `Std`

`Std` joue le rôle d'intermédiaire technique.

`ColumnTextarea` appelle simplement :

```
Std::nettoyerHtml($valeur);
```

`Std` se charge alors d'utiliser HTMLPurifier.

Cette organisation évite de faire dépendre directement les classes métier de la bibliothèque externe.

### `HTMLPurifier`

HTMLPurifier est le composant externe qui effectue réellement le nettoyage du HTML.

Le développeur n'a normalement **pas à manipuler directement cet objet**.

## 5. Le parcours réel d'une valeur

Supposons que TinyMCE envoie :

```
<p>Bonjour <strong>Guy</strong></p>
```

Le développeur fait simplement :

```
$description->Value = $_POST['description'];

$description->checkValidity();
```

Le parcours est alors :

```
$_POST['description']
        │
        ▼
Column::checkValidity()
        │
        ▼
ColumnTextarea::sanitize()
        │
        ▼
Std::nettoyerHtml()
        │
        ▼
HTMLPurifier
        │
        ▼
HTML nettoyé
        │
        ▼
$description->Value
```

La valeur finale présente dans `Value` est donc la **version nettoyée**.

## 6. Pourquoi cette organisation est intéressante ?

Le développeur utilise une interface simple :

```
$champ->checkValidity();
```

mais plusieurs classes travaillent derrière cette méthode.

Cela permet de séparer les responsabilités :

|Classe|Responsabilité|
|---|---|
|`ColumnTextarea`|Gestion d'un champ texte pouvant contenir du HTML|
|`Column`|Gestion commune des colonnes et validation|
|`Std`|Services techniques communs|
|`HTMLPurifier`|Nettoyage et sécurisation du HTML|

Le développeur utilise donc **une classe métier simple**, tandis que la complexité technique est encapsulée dans les classes de l'architecture.

# Remarque

Si un projet ne possède :

- aucun `ColumnTextarea` ;
- aucun contenu HTML saisi par l'utilisateur ;
- aucun éditeur TinyMCE ;
- aucune utilisation de `Std::nettoyerHtml()` ;

alors **HTMLPurifier devient inutile** et peut être retiré.