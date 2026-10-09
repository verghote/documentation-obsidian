# Rôle

`ColumnList` est une classe de validation utilisée pour représenter une colonne dont la valeur doit obligatoirement appartenir à une liste de choix prédéfinie.

Elle hérite de la classe abstraite `Column` et ajoute un contrôle d'appartenance à un ensemble de valeurs autorisées.

Elle est particulièrement adaptée aux colonnes SQL qui représentent :

- un état ;
- un type ;
- une catégorie limitée ;
- une option parmi plusieurs choix connus.

## Exemples d'utilisation

Dans une classe métier, une colonne représentant un état peut être définie ainsi :

```php
$col = new ColumnList(
    values: [
        'Inscrit',
        'Payé',
        'Annulé'
    ]
);

$this->addColumn('etat', $col);
````

La colonne `etat` ne pourra accepter que ces trois valeurs.

# Paramètres du constructeur

```
public function __construct(
    bool $required = true,
    bool $insertable = true,
    bool $updatable = true,
    array $values = [],
    TextCase $casse = TextCase::None
)
```

## required

Indique si la valeur est obligatoire.

Exemple :

```
required: true
```

La valeur doit être renseignée.

Valeurs refusées :

```
null
""
"   "
```

Le contrôle est hérité de la classe `Column`.

## insertable

Indique si la colonne peut être renseignée lors d'un ajout.

Exemple :

```
insertable: false
```

Cas d'utilisation :

- valeur définie automatiquement ;
- valeur calculée par l'application.

## updatable

Indique si la colonne peut être modifiée.

Exemple :

```
updatable: false
```

Cas courant :

Une colonne représentant un statut initial qui ne doit plus être modifiée après création.

## values

Définit la liste des valeurs autorisées.

Exemple :

```
values: [
    'Homme',
    'Femme',
    'Autre'
]
```

Seules ces valeurs seront acceptées.

## casse

Définit une éventuelle transformation de casse avant le contrôle.

Le paramètre utilise l'énumération :

```
TextCase
```

Valeurs disponibles :

|Valeur|Action|
|---|---|
|`TextCase::None`|aucune modification|
|`TextCase::Upper`|conversion en majuscules|
|`TextCase::Lower`|conversion en minuscules|

# Fonctionnement de la validation

La méthode :

```
checkValidity()
```

effectue les opérations suivantes.

## 1 - Vérification de l'obligation

La première vérification est héritée de `Column`.

Exemple :

```
required: true
```

Valeur :

```
""
```

Résultat :

```
Veuillez renseigner ce champ.
```

## 2 - Transformation éventuelle de la casse

Avant de comparer la valeur avec la liste autorisée, une transformation peut être appliquée.

Exemple :

```
$col = new ColumnList(
    values: [
        'oui',
        'non'
    ],
    casse: TextCase::Lower
);
```

Valeur reçue :

```
OUI
```

Transformation :

```
oui
```

La valeur est ensuite comparée à la liste.

## 3 - Vérification de l'appartenance à la liste

Exemple :

```
$col = new ColumnList(
    values: [
        'Actif',
        'Inactif'
    ]
);
```

Valeurs acceptées :

```
Actif
Inactif
```

Valeur refusée :

```
Supprimé
```

Message retourné :

```
Veuillez saisir une des valeurs autorisées.
```

# Exemple complet dans une classe métier

```
protected function defineColumns(): void
{
    $col = new ColumnList(
        required: true,
        values: [
            'Brouillon',
            'Publié',
            'Archivé'
        ]
    );

    $this->addColumn('statut', $col);
}
```

Cette déclaration garantit que la colonne `statut` ne pourra contenir qu'une valeur prévue par le modèle métier.

# Récupération des valeurs autorisées

La méthode :

```
getValues()
```

permet de récupérer la liste définie.

Exemple :

```
$valeurs = $col->getValues();
```

Résultat :

```
[
    'Brouillon',
    'Publié',
    'Archivé'
]
```

Cette méthode peut être utilisée pour alimenter automatiquement :

- une liste déroulante HTML (`select`) ;
- une liste de boutons radio ;
- une validation côté client.

# Exemple avec une interface utilisateur

Déclaration métier :

```
$col = new ColumnList(
    values: [
        'Petit',
        'Moyen',
        'Grand'
    ]
);
```

L'application peut générer automatiquement :

```
<select>
    <option>Petit</option>
    <option>Moyen</option>
    <option>Grand</option>
</select>
```

La même source d'information sert donc :

- à la validation serveur ;
- à la construction de l'interface utilisateur.

# Exemple avec gestion de casse

```
$col = new ColumnList(
    values: [
        'oui',
        'non'
    ],
    casse: TextCase::Lower
);
```

Valeurs reçues :

|Valeur envoyée|Valeur après traitement|Résultat|
|---|---|---|
|`oui`|`oui`|Acceptée|
|`OUI`|`oui`|Acceptée|
|`Oui`|`oui`|Acceptée|
|`peut-être`|`peut-être`|Refusée|

# Limites

`ColumnList` convient lorsque les valeurs autorisées sont connues dans le code.

Exemples adaptés :

- civilité ;
- état d'une commande ;
- type d'utilisateur ;
- niveau d'accès.

Elle n'est pas adaptée lorsque les valeurs évoluent régulièrement.

Dans ce cas, il faut plutôt utiliser :

- une table de référence en base de données ;
- une relation entre tables ;
- une classe métier dédiée.

# Résumé

|Fonction|Gestion|
|---|---|
|Valeur obligatoire|Oui|
|Liste de valeurs autorisées|Oui|
|Transformation majuscule/minuscule|Oui|
|Génération de listes utilisateur|Possible|
|Validation CRUD|Oui|
|Gestion de données dynamiques|Non|

`ColumnList` permet donc de sécuriser simplement les colonnes dont les valeurs possibles sont limitées et connues à l'avance.