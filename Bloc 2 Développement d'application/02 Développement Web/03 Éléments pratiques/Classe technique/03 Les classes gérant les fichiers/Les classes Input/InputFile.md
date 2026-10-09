### Présentation

`InputFile` (qui hérite de la classe abstraite `Input`) est spécialisée dans la validation et la préparation d'un fichier téléversé via un formulaire HTTP POST (`$_FILES`).

Sa responsabilité est **exclusivement de valider** le fichier et d'en normaliser le nom. Elle ne réalise **aucune opération de stockage physique ou de déplacement sur le disque**, ces tâches étant déléguées aux classes dérivées de `FileManager`.

### Architecture et héritage

Plaintext

```
                  Input
                    │
                    ▼
                InputFile
                    │
        ┌───────────┴───────────┐
        │                       │
  InputFileImg        Autres spécialisations
```

### Instanciation et paramétrage

`InputFile` prend en paramètre le tableau associatif issu de `$_FILES` (contenant obligatoirement les clés `name`, `tmp_name`, `error` et `size`) ainsi qu'un tableau de configuration `$lesParametres` :

```php
use ClasseTechnique\InputFile;

$parametres = [
    'maxSize' => 5_000_000,                                    // Taille max en octets (0 = illimité)
    'lesExtensions' => ['pdf', 'docx'],                        // Extensions autorisées
    'lesTypes' => ['application/pdf', 'application/msword'],  // Types MIME autorisés
    'renommerSiExiste' => true                                 // Indique si la gestion des doublons est activée
];

$inputFile = new InputFile($_FILES['document'], $parametres);
```

### Cycle de validation : `checkValidity()`

La méthode `checkValidity()` exécute la séquence de vérification de manière séquentielle. Le traitement s'interrompt dès qu'une erreur est détectée :

Plaintext

```
InputFile::checkValidity()
        │
        ▼
verifierTeleversement()  ──▶ [Erreurs PHP, absence de fichier, is_uploaded_file()]
        │
        ▼
verifierTaille()         ──▶ [Comparaison avec maxSize]
        │
        ▼
verifierExtension()      ──▶ [Comparaison de l'extension extraite avec lesExtensions]
        │
        ▼
verifierTypeMime()       ──▶ [Détection fileinfo (finfo) du contenu réel]
        │
        ▼
validationSpecifique()   ──▶ [Extensible par héritage (retourne true par défaut)]
        │
        ▼
preparerNomFichier()     ──▶ [Passe en minuscules, supprime accents/caractères spéciaux]
        │
        ▼
     SUCCÈS
```

#### Contrôles effectués :

1. **Téléversement PHP** :
    
    - Vérification des codes d'erreur PHP (`UPLOAD_ERR_OK`, `UPLOAD_ERR_NO_FILE`, etc.).
        
    - Contrôle du champ obligatoire (`isRequired()`).
        
    - Validation de sécurité via `is_uploaded_file()` (garantit que le fichier provient d'une requête HTTP POST valide).
        
2. **Taille** : Contrôle du nombre d'octets.
    
3. **Extension** : Validation de l'extension extraite en minuscules.
    
4. **Type MIME réel** : Analyse du contenu binaire via l'extension PHP `fileinfo` (`finfo_open`), sans se fier à l'en-tête transmis par le navigateur.
    
5. **Préparation et nettoyage du nom** :
    
    - Passage en minuscules.
        
    - Suppressions des accents via `Transliterator` ou `iconv`.
        
    - Suppression des caractères non autorisés (caractères conservés : `a-z`, `0-9`,  `_`, `-`, ).
        
    - Affectation de la valeur nettoyée accessible via `getValue()`.
### Accesseurs principaux

```php
$inputFile->getName();                 // Nom d'origine du fichier
$inputFile->getTmpName();              // Chemin temporaire PHP (ex: /tmp/phpYtR6e)
$inputFile->getSize();                 // Taille en octets
$inputFile->getError();                // Code d'erreur PHP
$inputFile->getExtension();            // Extension en minuscules
$inputFile->getMimeType();             // Type MIME réel détecté par finfo
$inputFile->getValue();                // Nom du fichier nettoyé et normalisé
$inputFile->getRenommerSiExiste();     // Booleen indiquant la stratégie de renommage
$inputFile->getValidationMessage();    // Message d'erreur explicite en cas d'échec
```

