# 1. Rappel sur la classe `Input` (classe abstraite)

La classe `Input` est la classe de base de tous les contrôles de saisie.

Elle fournit :

- la gestion de la valeur ;
- la notion de champ obligatoire ;
- le message d'erreur de validation.
## Propriétés principales

| Propriété | Type  | Description       |
| --------- | ----- | ----------------- |
| `Value`   | mixed | Valeur du champ   |
| `Require` | bool  | Champ obligatoire |

La méthode checkValidity()  vérifie que le champ est renseigné lorsque `Require=true`.

En cas d'échec  $input->getValidationMessage() retourne la raison de l'erreur.

# 2. Classe `InputFile`

`InputFile` hérite de `Input`.

Elle représente un fichier reçu dans `$_FILES`.

Elle assure :

- la validation du téléversement ;
- le contrôle de la taille ;
- le contrôle des extensions ;
- le contrôle du type MIME ;
- la normalisation du nom du fichier ;
- la suppression des accents ;
- la transformation de casse ;
- la gestion des doublons ;
- la préparation du nom final.

Elle **ne copie jamais le fichier**.

La copie est réalisée par `FileManager`.

# Construction

```php
$fichier = new InputFile($_FILES['document']);
```

Puis on configure les règles :

```php
$fichier->configurer([
    'extensions' => ['pdf'],
    'types' => ['application/pdf'],
    'maxSize' => 5 * 1024 * 1024,
    'rename' => true,
    'repertoire' => 'uploads/documents'
]);
```

# Paramètres de configuration

|Paramètre|Description|
|---|---|
|`extensions`|extensions autorisées|
|`types`|types MIME autorisés|
|`maxSize`|taille maximale|
|`rename`|renommage automatique en cas de doublon|
|`mode`|`insert` ou `update`|
|`require`|fichier obligatoire|
|`sansAccent`|suppression des accents|
|`casse`|`U`, `L` ou vide|
|`repertoire`|dossier de destination|

# Validation

```php
if ($fichier->checkValidity()) {

}
```

Les contrôles effectués sont :

1. téléversement PHP ;
2. taille ;
3. extension ;
4. type MIME ;
5. validation spécifique (classes dérivées) ;
6. préparation du nom ;
7. gestion des doublons.

Lorsque toutes les vérifications sont terminées : $fichier->isValide() retourne true

# Accesseurs utiles

|Méthode|Description|
|---|---|
|`getFile()`|tableau `$_FILES`|
|`getName()`|nom d'origine|
|`getNom()`|nom sans extension|
|`getExtension()`|extension|
|`getTmpName()`|fichier temporaire|
|`getMimeType()`|type MIME réel|
|`getSize()`|taille|
|`getError()`|code d'erreur PHP|
|`getNomFinal()`|nom retenu après validation|
|`getMode()`|mode courant|
|`getRepertoire()`|dossier de destination|
|`isValide()`|indique si le fichier est valide|

# Gestion des doublons

Deux modes existent.

## Sans renommage

```
'rename' => false
```

Si le fichier existe déjà :

```
photo.jpg
```

la validation échoue.

---

## Avec renommage

```
'rename' => true
```

Le nom devient automatiquement :

```
photo.jpg
photo(1).jpg
photo(2).jpg
...
```

Le nom réellement retenu est accessible avec :

```
$fichier->getNomFinal();
```

---

# Utilisation avec `FileManager`

Après validation :

```
$fichier = new InputFile($_FILES['document']);

$fichier->configurer([
    'extensions' => ['pdf'],
    'types' => ['application/pdf'],
    'maxSize' => 5 * 1024 * 1024,
    'rename' => true,
    'repertoire' => 'uploads/documents'
]);

if ($fichier->checkValidity()) {

    $manager = new FileManager();

    $manager->copy($fichier);

}
```

---

# 3. Classe `InputFileImg`

`InputFileImg` hérite directement de `InputFile`.

Elle reprend **toutes les fonctionnalités** de `InputFile` et ajoute des contrôles spécifiques aux images.

Elle permet notamment de :

- vérifier que le fichier est bien une image ;
- contrôler les dimensions ;
- imposer une largeur maximale ;
- imposer une hauteur maximale ;
- redimensionner automatiquement une image lors de la copie (via `FileManager`).

La validation générale reste entièrement assurée par `InputFile`.

La classe `InputFileImg` intervient uniquement au niveau de la méthode protégée :

```
validationSpecifique()
```

appelée automatiquement par `InputFile::checkValidity()`.

---

# Paramètres supplémentaires

|Paramètre|Description|
|---|---|
|`width`|largeur maximale ou largeur cible|
|`height`|hauteur maximale ou hauteur cible|
|`redimensionner`|autorise le redimensionnement automatique|

---

# Fonctionnement

Si :

```
redimensionner = false
```

les dimensions de l'image doivent respecter les limites configurées.

Si :

```
redimensionner = true
```

la validation accepte les images plus grandes, qui seront ensuite redimensionnées lors de la copie par `FileManager`.

---

# Exemple

```
$image = new InputFileImg($_FILES['photo']);

$image->configurer([
    'extensions' => ['jpg', 'jpeg', 'png'],
    'types' => ['image/jpeg', 'image/png'],
    'width' => 1200,
    'height' => 800,
    'redimensionner' => true,
    'rename' => true,
    'repertoire' => 'uploads/photos'
]);

if ($image->checkValidity()) {

    $manager = new FileManager();

    $manager->copy($image);

}
```