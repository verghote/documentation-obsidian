## Objectif

`ImageManager` est un gestionnaire spécialisé dans le stockage des images.

Il hérite de `FileManager` et fournit automatiquement la configuration adaptée aux principaux formats d'images.

Aucune validation supplémentaire n'est nécessaire : toute la gestion est héritée de `FileManager`.

---

# Types de fichiers acceptés

Le gestionnaire accepte les formats suivants :

- JPG
- JPEG
- PNG
- GIF
- WEBP

La validation porte à la fois sur :

- l'extension du fichier ;
- son type MIME réel.

Un fichier dont l'extension ne correspond pas au contenu est donc refusé.

---

# Création d'un gestionnaire

Le gestionnaire est généralement obtenu à l'aide de la fabrique :

```
$imageManager = FileManagerFactory::image();
```

Le répertoire de stockage est défini lors de sa création.

---

# Ajouter une image

L'ajout valide automatiquement l'image avant sa copie.

```
$file = new InputFile($_FILES['image']);

if (!$imageManager->ajouter($file)) {
    echo $file->getValidationMessage();
}
```

Si la validation réussit, l'image est copiée dans le répertoire de stockage.

---

# Remplacer une image

Le remplacement conserve le nom de l'image existante.

Seul son contenu est remplacé.

```
$file = new InputFile($_FILES['image']);

$imageManager->remplacer('photo.jpg', $file);
```

Cette opération est généralement utilisée par un service métier.

---

# Supprimer une image

Une image peut être supprimée à partir de son nom.

```
$imageManager->supprimer('photo.jpg');
```

---

# Consulter les images

Le gestionnaire peut retourner la liste des images présentes dans le répertoire.

```
$liste = $imageManager->getLesFichiers();
```

Seules les images possédant une extension autorisée sont retournées.

---

# Héritage

`ImageManager` ne redéfinit aucune méthode.

Il se contente de définir les extensions et les types MIME autorisés.

Toute la logique de validation, de copie, de remplacement et de suppression est héritée de `FileManager`.