## 1. Objectif

`ImageManager` est le gestionnaire technique spécialisé dans le **stockage des fichiers images**.

Il hérite de `FileManager` et reprend donc toutes ses fonctionnalités de gestion des fichiers sur disque.

Il ajoute une fonctionnalité spécifique aux images :

- redimensionner automatiquement une image avant son enregistrement ;
- limiter sa largeur ;
- limiter sa hauteur ;
- conserver ses proportions lors du redimensionnement ;
- enregistrer directement l'image traitée dans le répertoire de destination.

Le redimensionnement est effectué avec la bibliothèque `Gumlet\ImageResize`.

# 2. Positionnement dans l'architecture

`ImageManager` intervient après la récupération et la validation du fichier image.

Le traitement standard est :

```text
Requête HTTP
     │
     ▼
Requete
     │
     │ récupération de $_FILES
     ▼
InputFileImg
     │
     │ validation de l'image
     │ - taille
     │ - extension
     │ - type MIME
     │ - règles configurées
     ▼
ImageManager
     │
     ├── vérification du nom
     ├── redimensionnement éventuel
     └── sauvegarde physique
     │
     ▼
Répertoire des images
     │
     ▼
ReponseJson
```

Les responsabilités sont donc séparées :

|Composant|Responsabilité|
|---|---|
|`Requete`|Récupération des données de la requête HTTP|
|`InputFileImg`|Validation et encapsulation du fichier image|
|`ImageManager`|Traitement et stockage physique de l'image|
|`FileManager`|Gestion générale du stockage sur disque|
|`ReponseJson`|Retour des résultats au client|
# 3. Héritage de `FileManager`

`ImageManager` est une spécialisation de :

```php
class ImageManager extends FileManager
```

Il dispose donc de toutes les fonctionnalités publiques de `FileManager`.

Il est notamment possible d'utiliser :

```php
$imageManager->existe($nom);
$imageManager->getLesFichiers();
$imageManager->genererNomUnique($nom);
$imageManager->supprimer($nom);
$imageManager->remplacer($source, $nom);
```

La méthode spécifique aux images est :

```php
$imageManager->copierImage($inputFileImg, $nom);
```

### Principe

`FileManager` sait gérer un fichier.

`ImageManager` sait gérer un fichier **image** avec, en plus, la possibilité de le redimensionner avant son stockage.

# 4. Configuration

Le comportement de `ImageManager` est déterminé par les paramètres transmis au constructeur.

Exemple :

```php
$imageManager = new ImageManager(
    DOSSIER_IMAGE,
    [
        'maxWidth' => 1200,
        'maxHeight' => 800,
        'resized' => true,
    ]
);
```

Les paramètres spécifiques à `ImageManager` sont :

|Paramètre|Type|Défaut|Rôle|
|---|---|---|---|
|`maxWidth`|`int`|`0`|Largeur maximale en pixels|
|`maxHeight`|`int`|`0`|Hauteur maximale en pixels|
|`resized`|`bool`|`false`|Active ou désactive le redimensionnement|

# 5. Signification de `maxWidth`

```php
'maxWidth' => 1200
```

indique que l'image ne doit pas dépasser une largeur de 1200 pixels lorsque le redimensionnement est activé.

Si l'image fait déjà moins de 1200 pixels de largeur, elle n'est pas agrandie.

Exemple :

```text
Image source : 800 × 600
maxWidth     : 1200

Résultat     : 800 × 600
```

En revanche :

```text
Image source : 2400 × 1600
maxWidth     : 1200

Résultat     : 1200 × 800
```

Le rapport largeur/hauteur est conservé.


# 6. Signification de `maxHeight`

```php
'maxHeight' => 800
```

indique que l'image ne doit pas dépasser une hauteur de 800 pixels lorsque le redimensionnement est activé.

Exemple :

```text
Image source : 1200 × 600
maxHeight    : 800

Résultat     : 1200 × 600
```

Si l'image dépasse la hauteur maximale :

```text
Image source : 1200 × 1600
maxHeight    : 800

Résultat     : 600 × 800
```

Les proportions de l'image sont conservées.

# 7. Utilisation de `maxWidth` et `maxHeight`

Lorsque les deux dimensions sont renseignées :

```php
[
    'maxWidth' => 1200,
    'maxHeight' => 800,
]
```

l'image est adaptée à la zone maximale :

```text
1200 × 800
```

tout en conservant son ratio d'aspect.

Il s'agit d'un ajustement de type **best-fit**.

### Exemple

Image source :

```text
2400 × 1200
```

Configuration :

```text
maxWidth  = 1200
maxHeight = 800
```

Résultat :

```text
1200 × 600
```

L'image respecte les deux limites sans être déformée.

# 8. Les dimensions `0`

La valeur `0` signifie qu'aucune limite n'est définie pour cette dimension.

Exemple :

```php
[
    'maxWidth' => 1200,
    'maxHeight' => 0,
    'resized' => true,
]
```

Seule la largeur est limitée.

Autre exemple :

```php
[
    'maxWidth' => 0,
    'maxHeight' => 800,
    'resized' => true,
]
```

Seule la hauteur est limitée.

Si les deux dimensions sont à `0` :

```php
[
    'maxWidth' => 0,
    'maxHeight' => 0,
    'resized' => true,
]
```

aucun redimensionnement n'est effectué.

# 10. Activation du redimensionnement

Le redimensionnement n'est effectué que lorsque :

```php
'resized' => true
```

et qu'au moins une dimension maximale est définie.

Le comportement peut donc être résumé ainsi :

|`resized`|`maxWidth`|`maxHeight`|Traitement|
|---|---|---|---|
|`false`|0|0|Copie simple|
|`false`|1200|800|Copie simple|
|`true`|0|0|Copie simple|
|`true`|1200|0|Limitation de largeur|
|`true`|0|800|Limitation de hauteur|
|`true`|1200|800|Best-fit|

### Point important

Même si `maxWidth` et `maxHeight` sont configurés, le redimensionnement n'a pas lieu si :

```php
'resized' => false
```

Dans ce cas, `ImageManager` délègue directement le stockage à `FileManager`.

# 10. Comportement de `copierImage()`

La méthode principale est :

```php
$imageManager->copierImage(
    $inputFileImg,
    $nouveauNom
);
```

Elle réalise les opérations suivantes :

```text
InputFileImg
     │
     │ fichier temporaire
     ▼
ImageManager::copierImage()
     │
     ├── redimensionnement désactivé ?
     │       │
     │       └── oui → FileManager::copier()
     │
     └── redimensionnement activé
             │
             ▼
       ImageResize
             │
             ▼
       sauvegarde
             │
             ▼
       suppression du temporaire
```

# 11. Cas avec redimensionnement

Lorsque le redimensionnement est actif :

```php
'resized' => true
```

`ImageManager` :

1. prépare le nom de destination ;
2. charge l'image temporaire ;
3. détermine les dimensions à appliquer ;
4. redimensionne l'image si nécessaire ;
5. sauvegarde l'image dans le répertoire ;
6. supprime le fichier temporaire ;
7. retourne le nom réellement utilisé.

Le développeur n'a pas à appeler directement `ImageResize`.

# 12. Conservation des proportions

Le redimensionnement est effectué proportionnellement.

L'image n'est donc pas étirée pour atteindre artificiellement les dimensions maximales.

Cela permet de préserver le ratio d'aspect original.

# 15. Validation avec `InputFileImg`

`ImageManager` ne doit pas être utilisé comme mécanisme de validation du fichier reçu.

Le contrôleur doit d'abord créer un `InputFileImg` :

```php
$inputFileImg = new InputFileImg($fichier, $lesParametres);
```

Puis vérifier sa validité :

```php
if (!$inputFileImg->checkValidity()) {
    ReponseJson::envoyerErreur($inputFileImg->getValidationMessage());
}
```

Cette étape permet d'appliquer les contraintes définies dans la configuration.

Par exemple :

```php
return [
    'maxSize' => 2 * 1024 * 1024,
    'lesExtensions' => ['jpg', 'jpeg', 'png', 'webp'],
    'lesTypes' => [
        'image/jpeg',
        'image/png',
        'image/webp'
    ],
];
```

### Principe

`InputFileImg` répond à la question :

> « Cette image reçue est-elle autorisée ? »

`ImageManager` répond à la question :

> « Comment cette image doit-elle être stockée physiquement ? »

# 13. Exemple complet : téléversement d'une image

Le contrôleur de téléversement suit cette organisation :

```php
declare(strict_types=1);

use ClasseTechnique\Config;
use ClasseTechnique\ImageManager;
use ClasseTechnique\InputFileImg;
use ClasseTechnique\ReponseJson;
use ClasseTechnique\Requete;

require $_SERVER['DOCUMENT_ROOT'] . '/../bootstrap/bootstrap.php';

Requete::exigerPost();

$fichier = Requete::getFile('fichier');

if (!$fichier) {
    ReponseJson::envoyerErreur("Aucun fichier n'a été transmis.");
}

$lesParametres = Config::chargerPhp('image');

$inputFileImg = new InputFileImg($fichier,$lesParametres);

if (!$inputFileImg->checkValidity()) {
    ReponseJson::envoyerErreur($inputFileImg->getValidationMessage()
    );
}

$imageManager = new ImageManager(DOSSIER_IMAGE, $lesParametres);

$imageManager->copierImage($inputFileImg, $inputFileImg->getValue());

ReponseJson::envoyerLesDonnees($imageManager->getLesFichiers());
```

# 14. Le nom de l'image

Le nom transmis à `copierImage()` est :

```php
$inputFileImg->getValue()
```

Ce nom doit être utilisé plutôt que de récupérer directement un nom depuis la requête HTTP.

Exemple :

```php
$nom = $inputFileImg->getValue();

$imageManager->copierImage($inputFileImg, $nom);
```

Le `ImageManager` applique ensuite la règle :

```php
'renommerSiExiste'
```

héritée de `FileManager`.

# 15. Gestion des collisions

`ImageManager` hérite du comportement de `FileManager` concernant les noms existants.

Avec :

```php
'renommerSiExiste' => false
```

une image existante provoque une `UserException`.

Avec :

```php
'renommerSiExiste' => true
```

un nouveau nom est généré automatiquement.

Exemple :

```text
photo.jpg
photo(1).jpg
photo(2).jpg
```

Lorsque le renommage automatique est activé, `copierImage()` retourne le nom réellement utilisé.

Il est donc possible de faire :

```php
$nomStocke = $imageManager->copierImage(
    $inputFileImg,
    $inputFileImg->getValue()
);
```

Si le nom initial était déjà utilisé :

```php
$nomStocke === 'photo(1).jpg';
```

# 16. Suppression d'une image

La suppression ne nécessite pas `ImageManager`.

Puisque `ImageManager` hérite de `FileManager`, il est parfaitement possible d'utiliser :

```php
$fileManager = new FileManager(DOSSIER_IMAGE);

$fileManager->supprimer($nomFichier);
```

C'est d'ailleurs le choix effectué dans le contrôleur de suppression fourni.

Le redimensionnement n'intervient pas lors de la suppression.

Il n'est donc pas nécessaire d'instancier un `ImageManager` uniquement pour supprimer une image.

# 17. Exemple complet de suppression

```php
declare(strict_types=1);

use ClasseTechnique\FileManager;
use ClasseTechnique\ReponseJson;
use ClasseTechnique\Requete;

require $_SERVER['DOCUMENT_ROOT']
    . '/../bootstrap/bootstrap.php';

Requete::exigerPost();

$nomFichier = Requete::postString('nomFichier');

if (!$nomFichier) {
    ReponseJson::envoyerErreur("Aucun fichier n'a été spécifié.");
}

$fileManager = new FileManager(DOSSIER_IMAGE);

if (!$fileManager->supprimer($nomFichier)) {
    ReponseJson::envoyerErreur("Le fichier n'a pas pu être supprimé.");
}

ReponseJson::envoyerLesDonnees($fileManager->getLesFichiers());
```

### Pourquoi utiliser `FileManager` ici ?

Parce que l'opération réalisée est uniquement :

```text
supprimer un fichier
```

Elle ne nécessite aucune fonctionnalité spécifique aux images.

`FileManager` est donc suffisant.

# 18. Cycle de vie d'une image

Pour un téléversement :

```text
                     REQUÊTE HTTP
                          │
                          ▼
                      Requete
                          │
                          │ fichier
                          ▼
                    InputFileImg
                          │
                          │ validation
                          ▼
                    ImageManager
                          │
              ┌───────────┴───────────┐
              │                       │
        resized = false          resized = true
              │                       │
              ▼                       ▼
        FileManager             ImageResize
              │                       │
              │                       ▼
              │                  Redimensionnement
              │                       │
              └───────────┬───────────┘
                          ▼
                  Fichier sur disque
                          │
                          ▼
                    getLesFichiers()
                          │
                          ▼
                     ReponseJson
```

# 19. Configuration type pour les images

Une configuration peut par exemple être définie ainsi :

```php
return [
    'repertoire' => substr(
        DOSSIER_IMAGE,
        strlen(DOSSIER_WWW)
    ),

    'maxSize' => 2 * 1024 * 1024,

    'lesExtensions' => [
        'jpg',
        'jpeg',
        'png',
        'webp'
    ],

    'lesTypes' => [
        'image/jpeg',
        'image/png',
        'image/webp'
    ],

    'renommerSiExiste' => false,

    'maxWidth' => 1200,
    'maxHeight' => 1200,
    'resized' => true,
];
```

Cette configuration permet de séparer clairement :

### Validation du fichier

```php
'maxSize'
'lesExtensions'
'lesTypes'
```

### Gestion du stockage

```php
'renommerSiExiste'
```

### Traitement de l'image

```php
'maxWidth'
'maxHeight'
'resized'
```



# 20. Méthodes héritées utilisables avec `ImageManager`

Comme `ImageManager` hérite de `FileManager`, les méthodes suivantes restent disponibles :

|Méthode|Utilisation|
|---|---|
|`getRepertoire()`|Récupérer le répertoire de stockage|
|`getLesFichiers()`|Lister les images présentes|
|`existe($nom)`|Vérifier l'existence d'une image|
|`genererNomUnique($nom)`|Générer un nom disponible|
|`copier($source, $nom)`|Copier un fichier sans traitement spécifique|
|`remplacer($source, $nom)`|Remplacer un fichier existant|
|`supprimer($nom)`|Supprimer une image|
|`copierImage($file, $nom)`|Stocker une image avec redimensionnement éventuel|

La méthode spécifique à `ImageManager` est donc :

```php
copierImage()
```

