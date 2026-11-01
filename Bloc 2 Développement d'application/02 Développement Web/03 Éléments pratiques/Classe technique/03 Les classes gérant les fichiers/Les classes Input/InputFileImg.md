### Présentation

`InputFileImg` est une spécialisation de `InputFile` dédiée au contrôle des fichiers images.

Elle hérite de l'intégralité du cycle de validation de `InputFile` (vérification du téléversement HTTP, taille, extension, type MIME, nettoyage du nom) et **surcharge la méthode `validationSpecifique()`** pour vérifier les propriétés physiques de l'image.

Elle ne réalise **aucun redimensionnement ni stockage**, opérations dévolues à `ImageManager` (utilisant `Gumlet\ImageResize`).

### Paramètres spécifiques aux images

En plus des paramètres de `InputFile`, `InputFileImg` prend en charge dans son tableau `$lesParametres` :

|**Paramètre**|**Type**|**Rôle**|
|---|---|---|
|`maxWidth`|`int`|Largeur maximale autorisée en pixels (`0` = aucune limite).|
|`maxHeight`|`int`|Hauteur maximale autorisée en pixels (`0` = aucune limite).|
|`resized`|`bool`|Indique si l'image subira un redimensionnement ultérieur.|

```php
$parametresPhoto = [
    'maxSize' => 2_000_000,
    'lesExtensions' => ['jpg', 'jpeg', 'png', 'webp'],
    'lesTypes' => ['image/jpeg', 'image/png', 'image/webp'],
    'maxWidth' => 1200,
    'maxHeight' => 800,
    'resized' => false // Si true, les contrôles de dimensions maximales sont ignorés
];

$inputImg = new InputFileImg($_FILES['photo'], $parametresPhoto);
```

### Validation spécifique aux images : `validationSpecifique()`

La méthode `validationSpecifique()` intervient à la fin du cycle de validation :

1. **Gestion du redimensionnement automatique** :
    
    Si `$resized === true`, la méthode retourne immédiatement `true`. Les dimensions limites seront appliquées lors de la sauvegarde par `ImageManager`.
    
2. **Contrôle de validité binaire de l'image** :
    
    Exécute `getimagesize()` sur le fichier temporaire (`getTmpName()`). Si la fonction retourne `false`, le fichier n'est pas une image valide.
    
3. **Contrôle des dimensions maximales** :
    
    - Si `maxWidth > 0` et que la largeur réelle dépasse la limite, le fichier est refusé.
        
    - Si `maxHeight > 0` et que la hauteur réelle dépasse la limite, le fichier est refusé.
        
### Accesseurs spécifiques

```php
$inputImg->getMaxWidth();  // Largeur maximale configurée
$inputImg->getMaxHeight(); // Hauteur maximale configurée
$inputImg->getResized();   // État de l'option de redimensionnement
```

