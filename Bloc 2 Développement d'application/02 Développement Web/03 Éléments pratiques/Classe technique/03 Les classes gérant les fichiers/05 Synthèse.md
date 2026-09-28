## 1. Vue d'ensemble de l'Architecture

La gestion du téléversement et du stockage de fichiers repose sur une séparation stricte des responsabilités entre deux objets principaux :

- **`InputFile`** : Représente le fichier temporaire reçu (provenant de `$_FILES`), gère la validation des règles (taille, extension, type MIME) et le nettoyage du nom.
    
- **`FileManager`** : Gère l'emplacement physique définitif sur le disque (création, remplacement atomique, suppression, redimensionnement).
    

```
                      ┌──────────────────────┐
                      │     FileManager      │ (Classe abstraite de base)
                      │ (Stockage & Sécurité)│
                      └──────────▲───────────┘
                                 │
                 ┌───────────────┴───────────────┐
                 │                               │
      ┌──────────┴──────────┐         ┌──────────┴──────────┐
      │    ImageManager     │         │     PdfManager      │
      │ (Traitement images) │         │ (Stockage de PDF)   │
      └─────────────────────┘         └─────────────────────┘
```

## 2. La Classe Abstraite : `FileManager`

`FileManager` sert de socle commun. Elle ne peut pas être instanciée directement : elle impose un cadre de sécurité strict et fournit des opérations réutilisables.

### Responsabilités principales

- **Sécurité & Intégrité** : Elle s'assure que le répertoire cible existe, refuse toute tentative de traversée de répertoire (`basename`), et valide les extensions de manière stricte (conversion en minuscules).
    
- **Remplacement atomique (`remplacerPhysiquement`)** : Pour éviter qu'un fichier soit corrompu si l'écriture échoue au milieu du processus, le nouveau fichier est copié sous un nom temporaire (`tempnam`) puis renommé (`rename`) pour écraser l'ancien.
    
- **Gestion des doublons (`renommerSiNecessaire`)** : Lors d'un ajout avec renommage automatique (`$rename = true`), si `image.jpg` existe déjà, elle génère automatiquement `image(1).jpg`, `image(2).jpg`, etc.
    

## 3. Les Spécialisations : `ImageManager` et `PdfManager`

Chaque classe fille hérite de toute la logique de stockage mais personnalise son comportement à travers deux mécanismes :

### A. La classe `PdfManager`

C'est la spécialisation la plus simple. Elle se contente de restreindre le périmètre de sécurité :

- **Extensions autorisées** : `['pdf']`
    
- **Types MIME autorisés** : `['application/pdf']`
    
- **Passe-plat de configuration** : Transmet la taille maximale (`$maxSize`) et l'option de nettoyage d'accents (`$sansAccent`) à l'objet `InputFile` via la méthode `configurerInputFile()`.
    

### B. La classe `ImageManager`

Elle surcharge l'étape de copie grâce au polymorphisme de la méthode `copierFichier()`.

Si le fichier est une instance de `InputFileImg` et que l'option `$redimensionner` est activée, elle prend le relais sur la copie standard pour traiter le fichier via la bibliothèque **`Gumlet\ImageResize`** :

1. **Redimensionnement proportionnel à la largeur** (`resizeToWidth`) si seule `$width` est renseignée.
    
2. **Redimensionnement proportionnel à la hauteur** (`resizeToHeight`) si seule `$height` est renseignée.
    
3. **Ajustement optimal** (`resizeToBestFit`) si les deux dimensions sont renseignées (l'image tiendra dans un cadre sans être déformée).
    

## 4. Exemple Concret d'Utilisation

### Cas 1 : Téléversement d'une image de profil avec redimensionnement

PHP

```
use ClasseTechnique\ImageManager;
use ClasseTechnique\InputFileImg;

// 1. Instanciation du gestionnaire d'images
// Répertoire: 'uploads/avatars', Max: 2Mo (2097152 octets), Redimensionner à max 800px de large
$imageManager = new ImageManager(
    repertoire: __DIR__ . '/../uploads/avatars',
    maxSize: 2097152,
    sansAccent: true,
    redimensionner: true,
    width: 800
);

// 2. Encapsulation du fichier transmis via $_FILES
$inputFile = new InputFileImg($_FILES['avatar']);

// 3. Traitement et sauvegarde physique
if ($imageManager->ajouter($inputFile, rename: true)) {
    // Succès : le nom final nettoyé/renommé est récupérable
    $nomFichierSauvegarde = $inputFile->getValue();
    echo "Image enregistrée sous : " . $nomFichierSauvegarde;
} else {
    // Échec : récupération du message d'erreur de validation ou de traitement
    echo "Erreur : " . $inputFile->getValidationMessage();
}
```

### Cas 2 : Remplacement d'un document PDF existant

PHP

```
use ClasseTechnique\PdfManager;
use ClasseTechnique\InputFile;

$pdfManager = new PdfManager(
    repertoire: __DIR__ . '/../uploads/contrats',
    maxSize: 5242880 // 5 Mo
);

$inputFile = new InputFile($_FILES['document_pdf']);

// Remplace le fichier 'contrat_123.pdf' tout en conservant son nom sur le disque
if ($pdfManager->enregistrer('contrat_123.pdf', $inputFile)) {
    echo "Le contrat a été mis à jour avec succès.";
} else {
    echo "Échec du remplacement : " . $inputFile->getValidationMessage();
}
```

## 5. Synthèse des Méthodes Importantes

| Méthode                       | Rôle                                                                                                |
| ----------------------------- | --------------------------------------------------------------------------------------------------- |
| `ajouter($file, $rename)`     | Copie le fichier. Si `$rename` est `false` et que le fichier existe, l'opération échoue.            |
| `remplacer($ancien, $file)`   | Écrase de manière atomique un fichier existant sans changer son nom sur le disque.                  |
| `enregistrer($ancien, $file)` | **Méthode polyvalente** : appelle `ajouter()` si `$ancien` est vide, sinon appelle `remplacer()`.   |
| `getLesFichiers()`            | Liste les fichiers valides du dossier en ignorant les fichiers cachés (`.`, `..`) et non autorisés. |
| `supprimer($nom)`             | Efface le fichier du disque après vérification du nom.                                              |