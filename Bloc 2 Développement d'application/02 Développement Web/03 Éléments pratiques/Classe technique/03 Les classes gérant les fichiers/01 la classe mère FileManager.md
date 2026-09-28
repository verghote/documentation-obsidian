## Objectif

`FileManager` est la classe de base permettant de gérer le stockage physique de fichiers téléversés.

Elle centralise toutes les opérations communes :

- ajout d'un fichier ;
- remplacement d'un fichier existant ;
- suppression d'un fichier ;
- consultation du contenu d'un répertoire.

Les classes spécialisées PdfManager et  ImageManager héritent de cette classe et définissent uniquement les types de fichiers qu'elles acceptent.

# Principe

Un `FileManager` ne reçoit jamais directement les informations issues de `$_FILES`.

Le fichier est représenté par un objet `InputFile` qui réalise :

- les contrôles de téléversement ;
- la validation du fichier ;
- la préparation du nom de stockage.

Le gestionnaire intervient ensuite pour réaliser l'opération physique sur le système de fichiers.

```
Navigateur
      ▼
InputFile (validation)
      ▼
FileManager (stockage)
```

# Création d'un gestionnaire

Chaque type de fichier possède son propre gestionnaire.

Exemple :

```php
$pdfManager = FileManagerFactory::document();
```

Le constructeur vérifie automatiquement :

- que le répertoire de stockage existe ;
- que les extensions autorisées sont définies ;
- que les types MIME autorisés sont définis.

Une mauvaise configuration provoque immédiatement une exception.

# Ajouter un fichier

L'ajout consiste à :

1. configurer automatiquement `InputFile` ;
2. vérifier le fichier ;
3. copier le fichier dans le répertoire de stockage.

```php
$file = new InputFile($_FILES['fichier']);

if (!$pdfManager->ajouter($file)) {
    echo $file->getValidationMessage();
}
```

En cas de succès, le fichier est présent dans le répertoire de stockage.


# Remplacer un fichier

Le remplacement conserve le nom du fichier existant.

Seul son contenu est remplacé.

Le nouveau fichier est validé avant toute copie.

```php
$file = new InputFile($_FILES['fichier']);

$pdfManager->remplacer('contrat.pdf', $file);
```

Cette opération ne modifie jamais le nom du fichier.

Elle est généralement utilisée par un service métier qui connaît le fichier à remplacer.

# Supprimer un fichier

La suppression retire un fichier du répertoire de stockage.

```
$pdfManager->supprimer('contrat.pdf');
```

Le nom fourni doit être un simple nom de fichier appartenant au répertoire géré.

# Consulter les fichiers

Le gestionnaire peut retourner la liste des fichiers présents.

```
$liste = $pdfManager->getLesFichiers();
```

Seuls les fichiers possédant une extension autorisée sont retournés.

# Personnalisation

Les gestionnaires spécialisés définissent uniquement leurs caractéristiques.

Exemple :

```
class PdfManager extends FileManager
{
    protected array $lesExtensions = ['pdf'];
    protected array $lesTypes = ['application/pdf'];
}
```

Si une configuration supplémentaire est nécessaire, la classe dérivée peut redéfinir :

```
protected function configurerInputFile(InputFile $inputFile): void
```

Cette méthode permet de modifier certains paramètres de `InputFile` avant sa validation.

---

# Répartition des responsabilités

`InputFile`

- représente un fichier reçu ;
- valide le téléversement ;
- prépare le nom de stockage.

`FileManager`

- configure `InputFile` ;
- réalise les opérations physiques sur le système de fichiers ;
- garantit que les opérations respectent la configuration du gestionnaire.

Les services applicatifs (par exemple `ServiceDocument`) utilisent ensuite `FileManager` pour maintenir la cohérence entre les données métier et le stockage physique des fichiers.