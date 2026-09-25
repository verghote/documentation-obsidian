# Présentation

`PdfManager` est un gestionnaire spécialisé dans la gestion des fichiers PDF.

Il hérite de `FileManager` et ne contient aucune logique de traitement particulière.

Son rôle est uniquement de définir les caractéristiques propres aux documents PDF :

- extensions autorisées ;
- types MIME autorisés.

Toute la gestion commune est assurée automatiquement par `FileManager` :

- configuration de `InputFile` ;
- validation du fichier ;
- contrôle du téléversement ;
- copie physique ;
- suppression ;
- consultation des fichiers présents.

---

# Architecture

```
Contrôleur
      │
      ▼
PdfManager
      │
      ▼
FileManager
      │
      ▼
InputFile
      │
      ▼
Répertoire PDF
```

Le contrôleur ne connaît pas les règles de validation.

Il fournit uniquement :

- un gestionnaire PDF ;
- un objet `InputFile` contenant le fichier reçu.

---

# Hiérarchie

```
FileManager
      │
      └── PdfManager
```

`PdfManager` est une classe concrète.

Elle peut être instanciée directement.

---

# Responsabilité de PdfManager

`PdfManager` a une responsabilité volontairement limitée :

- indiquer que les fichiers gérés sont des PDF ;
- fournir les règles minimales nécessaires à `FileManager`.

Elle définit :

```php
protected array $lesExtensions = [
    'pdf'
];

protected array $lesTypes = [
    'application/pdf'
];
```

Ces propriétés sont utilisées automatiquement par la classe mère.

---

# Pourquoi utiliser des propriétés protégées ?

Les propriétés :

```php
$lesExtensions

$lesTypes
```

sont déclarées avec le niveau `protected`.

Ce choix permet :

- à `PdfManager` de définir simplement sa configuration ;
- à `FileManager` de contrôler et utiliser ces valeurs ;
- d'empêcher une modification externe après création de l'objet.

Elles représentent la configuration interne du gestionnaire.

Elles ne doivent donc pas être publiques.

---

# Création d'un gestionnaire PDF

La création nécessite uniquement le répertoire de stockage.

Exemple :

```php
$pdfManager = new PdfManager('/data/pdf');
```

Lors de la construction, `FileManager` vérifie automatiquement :

- que le répertoire existe ;
- qu'une extension PDF est définie ;
- qu'un type MIME PDF est défini.

---

# Ajout d'un fichier PDF

L'ajout est réalisé avec :

```php
$pdfManager->ajouter($file);
```

Exemple complet :

```php
$pdfManager = new PdfManager(
    '/data/pdf'
);


$file = new InputFile(
    Requete::getFile('document')
);


if (!$pdfManager->ajouter($file)) {

    ReponseJson::envoyerLesErreurs([
        'global' => [
            $file->getValidationMessage()
        ]
    ]);
}
```

---

# Fonctionnement de ajouter()

La méthode `ajouter()` est héritée de `FileManager`.

Le traitement est :

```
PdfManager::ajouter()
          │
          ▼
Configuration automatique de InputFile
          │
          ▼
Validation du fichier
          │
          ▼
Copie physique
```

`PdfManager` transmet automatiquement :

```php
extensions :
[
    'pdf'
]


types :
[
    'application/pdf'
]
```

Puis `InputFile` réalise les contrôles.

---

# Validation d'un PDF

Les contrôles sont réalisés par `InputFile`.

Ils comprennent notamment :

- présence du fichier ;
- erreur de téléversement PHP ;
- taille maximale ;
- extension ;
- type MIME réel ;
- nom du fichier ;
- gestion des doublons.

Le type MIME n'est pas basé sur la valeur envoyée par le navigateur.

Il est déterminé à partir du fichier temporaire présent sur le serveur.

---

# Stockage du fichier

Après validation :

```
Fichier temporaire PHP
          │
          ▼
InputFile::getNomFinal()
          │
          ▼
FileManager::copier()
          │
          ▼
Répertoire PDF
```

La copie :

- vérifie l'existence du fichier temporaire ;
- protège contre les chemins invalides ;
- copie le fichier ;
- supprime le fichier temporaire PHP.

---

# Liste des fichiers PDF

La méthode :

```php
$pdfManager->getLesFichiers();
```

retourne uniquement les fichiers ayant l'extension :

```
.pdf
```

Exemple :

```php
[
    "facture.pdf",
    "rapport.pdf",
    "contrat.pdf"
]
```

Les autres fichiers présents dans le répertoire sont ignorés.

---

# Suppression d'un PDF

La suppression utilise :

```php
$pdfManager->supprimer($nomFichier);
```

Exemple :

```php
$nomFichier = Requete::postString('nomFichier');


if (!$pdfManager->supprimer($nomFichier)) {

    ReponseJson::envoyerLesErreurs([
        'global' => [
            'Le fichier PDF n\'a pas pu être supprimé.'
        ]
    ]);
}
```

Avant suppression, `FileManager` vérifie :

- que le nom ne contient pas de chemin ;
- que l'extension est autorisée ;
- que le fichier existe.

---

# Création d'un gestionnaire similaire

Pour gérer un nouveau type de document simple, il suffit de créer une classe dérivée.

Exemple :

```php
class WordManager extends FileManager
{
    protected array $lesExtensions = [
        'docx'
    ];

    protected array $lesTypes = [
        'application/vnd.openxmlformats-officedocument.wordprocessingml.document'
    ];
}
```

Aucune méthode supplémentaire n'est nécessaire.

---

# Répartition des responsabilités

|Classe|Responsabilité|
|-|-|
|`InputFile`|Validation du fichier téléversé|
|`FileManager`|Gestion physique et orchestration|
|`PdfManager`|Définition des règles PDF|
|Contrôleur|Réception du fichier et appel du gestionnaire|

---

# Conclusion

`PdfManager` illustre le principe général de l'architecture :

- une classe spécialisée minimale ;
- aucune duplication de code ;
- aucune logique de stockage dans la classe fille ;
- une configuration déclarative par propriétés protégées.

La classe décrit uniquement **ce qu'elle accepte**, tandis que `FileManager` décrit **comment le fichier est traité**.