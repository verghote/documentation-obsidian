# Présentation

`InputFileImg` est une spécialisation de la classe `InputFile` dédiée aux fichiers images téléversés.

Elle conserve toute la logique générique de validation d'un fichier fournie par `InputFile` et ajoute uniquement les contrôles spécifiques aux images.

Son rôle est uniquement de déterminer si le fichier reçu correspond bien aux contraintes d'une image.

Elle ne réalise pas :

- la copie du fichier ;
- l'enregistrement dans le répertoire de stockage ;
- le redimensionnement de l'image ;
- la suppression du fichier.

Ces opérations sont prises en charge par `FileManager` et `ImageManager`.

# Architecture
```
             Input
              │
              ▼
         InputFile
              │
              ▼
         InputFileImg
```

`InputFileImg` hérite de `InputFile`.

Elle utilise le mécanisme d'extension prévu par la méthode :

```php
protected function validationSpecifique(): bool
````

appelée automatiquement par :

```php
InputFile::checkValidity()
```

# Hiérarchie complète

```
                 Input
                  │
                  ▼
             InputFile
                  │
        ┌─────────┴─────────┐
        │                   │
 InputFileImg          autres spécialisations
```

La gestion physique des fichiers est séparée :

```
              FileManager
                  │
        ┌─────────┴─────────┐
        │                   │
   PdfManager         ImageManager
                              │
                              ▼
                   Redimensionnement image
```

# Responsabilités

## Responsabilités héritées de InputFile

`InputFileImg` bénéficie automatiquement de toutes les validations générales :

- contrôle du téléversement PHP ;
- contrôle de la présence du fichier ;
- contrôle de la taille maximale ;
- contrôle de l'extension ;
- contrôle du type MIME réel ;
- préparation du nom du fichier ;
- suppression des accents ;
- gestion des doublons ;
- validation du nom final.

## Responsabilités spécifiques aux images

`InputFileImg` ajoute :

- vérification que le fichier est réellement une image ;
- contrôle du format interne de l'image ;
- récupération des dimensions ;
- contrôle largeur minimale ;
- contrôle largeur maximale ;
- contrôle hauteur minimale ;
- contrôle hauteur maximale.

# Ce que InputFileImg ne fait pas

La classe ne :

- déplace pas le fichier temporaire ;
- ne crée pas le fichier définitif ;
- ne modifie pas l'image ;
- ne redimensionne pas l'image.

Le redimensionnement appartient à :

```
ImageManager
        │
        ▼
Gumlet\ImageResize
```

Cette séparation permet de conserver une classe de validation indépendante du stockage.

# Création d'un objet InputFileImg

Le contrôleur récupère le fichier transmis :

```php
$fichier = Requete::getFile('image');
```

Puis crée l'objet :

```php
$image = new InputFileImg($fichier);
```

À ce stade aucune validation n'est encore réalisée.

L'objet contient uniquement :

- le nom original ;
- le fichier temporaire PHP ;
- la taille ;
- le code d'erreur PHP.

# Configuration

La configuration est fournie automatiquement par `ImageManager`.

Le contrôleur ne configure jamais directement `InputFileImg`.

Exemple :

```php
$imageManager = new ImageManager(Config::chargerPhp('photo'));

$imageManager->ajouter($image);
```

Lors de l'appel à `ajouter()`, `ImageManager` transmet :

```
[
    'extensions',
    'types',
    'maxSize',
    'rename',
    'sansAccent',
    'repertoire'
]
```

Puis les paramètres spécifiques aux images :

```
[
    'minWidth',
    'maxWidth',
    'minHeight',
    'maxHeight',
    'formatsImages'
]
```

# Paramètres spécifiques aux images

minWidth :  Largeur minimale autorisée.

Une image de largeur inférieure sera refusée.

maxWidth :  Largeur maximale autorisée.

## minHeight : Hauteur minimale autorisée.

## maxHeight :  Hauteur maximale autorisée.

## formatsImages

Liste des formats internes autorisés.

Ces valeurs correspondent aux constantes PHP :

```
[
    IMAGETYPE_JPEG,
    IMAGETYPE_PNG,
    IMAGETYPE_WEBP
]
```

Le contrôle est réalisé avec getimagesize()

Cette vérification complète le contrôle MIME car le type envoyé par le navigateur n'est pas considéré comme fiable.

# Cycle de validation

Lorsque `checkValidity()` est appelé :

```
InputFile::checkValidity()
        ▼
Vérification upload PHP
        ▼
Contrôle taille
        ▼
Contrôle extension
        ▼
Contrôle MIME réel
        ▼
validationSpecifique()
        ▼
InputFileImg
contrôle image
        ▼
Préparation du nom final
```

# Validation spécifique image

La méthode :

```php
protected function validationSpecifique(): bool
```

effectue les contrôles suivants.

## Vérification du fichier image

Le fichier temporaire doit exister :

```php
is_file($this->getTmpName())
```

## Lecture des informations image

La méthode :

```php
getimagesize()
```

permet de récupérer :

- largeur ;
- hauteur ;
- type réel.

Exemple :

```
[
    0 => largeur,
    1 => hauteur,
    2 => type image
]
```

## Contrôle du format réel

Exemple :

```php
if (!in_array($infos[2], $formatsImages,  true)) {
    return false;
}
```

Une image déguisée avec une extension autorisée sera donc refusée.

## Contrôle des dimensions

Exemple :

```
maxWidth = 1200
maxHeight = 800
```

Une image :

```
1600 x 900
```

sera refusée.

# Récupération des dimensions

La méthode :

```
getDimensions()
```

retourne :

```
[
    'largeur' => 1200,
    'hauteur' => 800
]
```

ou :

```
null
```

si l'image ne peut pas être lue.

# Exemple complet

```php
$imageManager = new ImageManager(
    Config::chargerPhp('photo')
);


$fichier = Requete::getFile('image');


$image = new InputFileImg($fichier);


if (!$imageManager->ajouter($image)) {

    ReponseJson::envoyerLesErreurs([
        'global' => [
            $image->getValidationMessage()
        ]
    ]);

}
```

Le contrôleur reste volontairement simple.

La chaîne complète est :

```
Contrôleur
    ▼
Requete::getFile()
    ▼
InputFileImg
    ▼
ImageManager
    ▼
Validation
    ▼
Redimensionnement éventuel
    ▼
Stockage
```

# Répartition des responsabilités

|Classe|Responsabilité|
|---|---|
|`InputFile`|Validation générique d'un fichier téléversé|
|`InputFileImg`|Validation complémentaire des images|
|`FileManager`|Gestion physique commune des fichiers|
|`PdfManager`|Gestion des documents PDF|
|`ImageManager`|Gestion spécifique des images et redimensionnement|
|Contrôleur|Réception de la requête et orchestration|

---

# Principe général

`InputFileImg` suit le même principe architectural que le reste du système :

- les contrôles sont regroupés dans les classes responsables ;
- la validation est indépendante du stockage ;
- les traitements spécifiques sont ajoutés par spécialisation ;
- les gestionnaires restent responsables des opérations physiques.

Ainsi :

```
InputFile = validation commune

InputFileImg = validation image

FileManager = stockage

ImageManager = traitement image spécifique
```

Cette séparation permet d'ajouter facilement d'autres types de fichiers sans modifier le fonctionnement existant.