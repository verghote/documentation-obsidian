# Rôle

`ColumnBool` est une classe de validation utilisée pour représenter une colonne métier contenant une valeur booléenne.

Elle hérite de la classe abstraite `Column` et ajoute la gestion spécifique des valeurs vraies/fausses.

Elle permet notamment de gérer correctement les valeurs provenant des formulaires HTML, car ces dernières sont généralement transmises sous forme de chaînes de caractères.

Elle est adaptée aux colonnes SQL de type :

- `BOOLEAN` ;
- `TINYINT(1)` ;
- indicateurs oui/non ;
- options activées ou désactivées.

# Exemples d'utilisation

Dans une classe métier :

```php
$col = new ColumnBool(
    required: true
);

$this->addColumn('actif', $col);
````

La colonne `actif` devra obligatoirement contenir une valeur booléenne.

# Paramètres du constructeur

```
public function __construct(
    bool $required = true,
    bool $insertable = true,
    bool $updatable = true
)
```

## required

Indique si la valeur est obligatoire.

Exemple :

```
required: true
```

La valeur doit être fournie.

Valeurs refusées :

```
null
""
"   "
```

Le contrôle est effectué par la classe mère `Column`.

## insertable

Indique si la colonne peut être renseignée lors d'une insertion.

Exemple :

```
insertable: false
```

Cas d'utilisation :

- une valeur définie automatiquement ;
- une option calculée par l'application.

## updatable

Indique si la colonne peut être modifiée.

Exemple :

```
updatable: false
```

Cas courant :

```
compteValide
```

Une validation initiale peut être enregistrée mais non modifiable ensuite par l'utilisateur.

# Fonctionnement de la validation

La méthode :

```
checkValidity()
```

réalise plusieurs contrôles.

## 1 - Vérification de l'obligation

La première étape est héritée de `Column`.

Exemple :

```
$col = new ColumnBool(
    required: true
);
```

Valeur :

```
null
```

Résultat :

```
Veuillez renseigner ce champ.
```

## 2 - Conversion des valeurs reçues

Les formulaires HTML transmettent souvent les valeurs sous forme de chaînes.

Par exemple :

```
<input type="checkbox" name="actif" value="1">
```

envoie :

```
"1"
```

et non :

```
true
```

`ColumnBool` réalise automatiquement la conversion.

# Valeurs acceptées

## Valeur vraie

Les valeurs suivantes sont converties en :

```
true
```

|Valeur reçue|Résultat|
|---|---|
|`true`|`true`|
|`1`|`true`|
|`"1"`|`true`|
|`"true"`|`true`|

## Valeur fausse

Les valeurs suivantes sont converties en :

```
false
```

|Valeur reçue|Résultat|
|---|---|
|`false`|`false`|
|`0`|`false`|
|`"0"`|`false`|
|`"false"`|`false`|

## Valeurs refusées

Toutes les autres valeurs sont rejetées.

Exemples :

```
oui
non
yes
2
abc
```

Message retourné :

```
La valeur doit être vraie ou fausse.
```

# Exemple complet dans une classe métier

```
protected function defineColumns(): void
{
    $col = new ColumnBool(
        required: true
    );

    $this->addColumn('visible', $col);
}
```

Cette colonne représente par exemple :

```
visible = true
```

ou :

```
visible = false
```

# Exemple avec un formulaire HTML

Formulaire :

```
<input 
    type="checkbox"
    name="publie"
    value="1"
>
```

Lorsque la case est cochée :

```
$_POST['publie'] = "1"
```

Après validation :

```
$column->Value === true
```

Lorsque la case est décochée, selon la gestion du formulaire :

```
$_POST['publie'] = "0"
```

Après validation :

```
$column->Value === false
```

# Utilisation avec les opérations CRUD

La conversion est particulièrement utile car elle garantit que les différentes sources de données utilisent toujours le même type.

Avant validation :

```
"1"
```

Après validation :

```
true
```

La couche SQL reçoit donc toujours une valeur cohérente.

# Exemple métier

Une table :

```
utilisateur
----------------
id
nom
actif
administrateur
```

Déclaration :

```
protected function defineColumns(): void
{
    $this->addColumn(
        'actif',
        new ColumnBool()
    );

    $this->addColumn(
        'administrateur',
        new ColumnBool()
    );
}
```

Les colonnes deviennent des propriétés métier clairement typées.

# Résumé

|Fonction|Gestion|
|---|---|
|Valeur obligatoire|Oui|
|Contrôle booléen|Oui|
|Conversion HTML (`"1"`, `"0"`)|Oui|
|Conversion texte (`"true"`, `"false"`)|Oui|
|Valeur finale typée `bool`|Oui|
|Compatible CRUD|Oui|

`ColumnBool` permet donc d'abstraire les différences entre les valeurs reçues par les formulaires, les données PHP et les valeurs attendues par la base de données.