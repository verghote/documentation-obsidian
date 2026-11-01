## 1. Objectif

`FileManager` est la classe technique responsable de la **gestion physique des fichiers sur disque**.

Elle intervient après la réception et la validation d'un fichier par l'application. Son rôle est de centraliser les opérations d'accès au système de fichiers :

- vérifier l'existence physique d'un fichier ;
- lister les fichiers présents dans un répertoire ;
- générer un nom disponible ;
- copier un fichier dans le répertoire de stockage ;
- remplacer un fichier existant ;
- supprimer un fichier.

`FileManager` ne prend pas en charge la réception HTTP d'un fichier et ne réalise pas la validation métier ou la validation du contenu du fichier.

# 2. Positionnement dans le traitement d'un fichier

Dans l'application, le traitement d'un fichier téléversé suit généralement cette chaîne :

```text
Requête HTTP
     │
     ▼
Requete
     │
     │ récupération du fichier
     ▼
InputFile
     │
     │ validation du fichier
     │ - taille
     │ - extension
     │ - type MIME
     │ - règles configurées
     ▼
FileManager
     │
     │ stockage physique
     ▼
Répertoire de fichiers
     │
     ▼
ReponseJson
```

Chaque classe a donc une responsabilité distincte.

| Classe        | Responsabilité                                          |
| ------------- | ------------------------------------------------------- |
| `Requete`     | Récupération et contrôle des données de la requête HTTP |
| `InputFile`   | Validation du fichier téléversé                         |
| `FileManager` | Stockage physique du fichier                            |
| `ReponseJson` | Transmission de la réponse au client                    |

Cette séparation permet d'éviter de mélanger la réception d'un fichier, sa validation et son stockage physique.

# 3. Configuration

`FileManager` peut recevoir le tableau de configuration utilisé pour le traitement du fichier.

Exemple de configuration `pdf` :

```php
return [
    'repertoire' => substr(DOSSIER_DOCUMENT, strlen(DOSSIER_WWW)),
    'maxSize' => 512 * 1024,
    'lesExtensions' => ['pdf'],
    'lesTypes' => ['application/pdf'],
    'renommerSiExiste' => false,
];
```

Les paramètres n'ont pas tous la même responsabilité.

## Paramètres utilisés par `InputFile`

Ces paramètres concernent principalement la validation du fichier :

```php
'maxSize' => 512 * 1024,
'lesExtensions' => ['pdf'],
'lesTypes' => ['application/pdf'],
```

Ils permettent notamment de vérifier :

- la taille maximale ;
- l'extension ;
- le type MIME.

## Paramètres utilisés par `FileManager`

`FileManager` utilise notamment :

```php
'renommerSiExiste' => false,
```

Ce paramètre détermine le comportement lorsqu'un fichier portant déjà le nom demandé existe dans le répertoire.

### Important

Le tableau de configuration peut donc être partagé entre `InputFile` et `FileManager`, mais chaque classe n'en exploite pas nécessairement les mêmes paramètres.


# 4. Création du FileManager

Le `FileManager` est créé avec le **répertoire physique de stockage**.

Exemple :

```php
$fileManager = new FileManager(DOSSIER_PDF, $lesParametres);
```

Dans cet exemple :

```php
$lesParametres = Config::chargerPhp('pdf');
```

Le même tableau est transmis à `InputFile` et à `FileManager`.

Le chemin physique du stockage est fourni par la constante : DOSSIER_PDF

Il est recommandé d'utiliser les constantes de répertoire définies par l'application plutôt que de reconstruire manuellement les chemins dans les contrôleurs.

# 5. Exemple complet : téléversement d'un PDF

Le cas classique est le téléversement d'un fichier PDF.

Le contrôleur réalise les étapes suivantes :

```php
  
use ClasseTechnique\ReponseJson;  
use ClasseTechnique\Requete;  
use ClasseTechnique\InputFile;  
use ClasseTechnique\FileManager;  
use ClasseTechnique\Config;  
  
// Chargement automatique des classes et des constantes (DOSSIER_PDF)  
require $_SERVER['DOCUMENT_ROOT'] . '/../bootstrap/bootstrap.php';  
  
// 1. Vérification d'un appel AJAX par la méthode POST  
Requete::exigerPost();  
  
// 2. Récupération du fichier téléversé  
$fichier = Requete::getFile('fichier');  
  
if (!$fichier) {  
    ReponseJson::envoyerErreur("Aucun fichier n'a été transmis.");  
}  
  
// Récupération des paramètres de configuration pour les fichiers PDF  
$lesParametres = Config::chargerPhp('pdf');  
  
// 3. Instanciation et paramétrage de l'objet InputFile  
$inputFile = new InputFile($fichier, $lesParametres);  
  
// 4. Validation du fichier PDF et alimentation de la propriété value contenant le nom du fichier  
if (!$inputFile->checkValidity()) {  
    ReponseJson::envoyerErreur($inputFile->getValidationMessage());  
}  
  
// 5. Instanciation du FileManager avec le répertoire défini par la constante globale  
$fileManager = new FileManager(DOSSIER_PDF, $lesParametres);  
  
// 6. Enregistrement du fichier (renommé si nécessaire selon la configuration)  
$fileManager->copier($fichier['tmp_name'], $inputFile->getValue());  
  
// 7. Envoi de la liste à jour des fichiers PDF  
ReponseJson::envoyerLesDonnees($fileManager->getLesFichiers());
```

La responsabilité de `FileManager` commence donc seulement à ce stade.

Le fichier a déjà :

1. été reçu par la requête HTTP ;
2. été extrait par `Requete` ;
3. été soumis à `InputFile` ;
4. été validé ;
5. reçu un nom exploitable via `getValue()`.

`FileManager` peut alors effectuer le stockage physique.

# 6. Rôle de `InputFile` avant `FileManager`

Il est important de ne pas utiliser `FileManager` directement pour traiter un fichier provenant de `$_FILES`.

Le fichier doit d'abord être validé.

Exemple :

```php
$inputFile = new InputFile($fichier, $lesParametres);

if (!$inputFile->checkValidity()) {
    ReponseJson::envoyerErreur($inputFile->getValidationMessage());  
}
```

Cette étape permet notamment d'appliquer la configuration :

```php
'maxSize' => 512 * 1024,
'lesExtensions' => ['pdf'],
'lesTypes' => ['application/pdf'],
```

### Principe

`InputFile` répond à la question :

> « Le fichier reçu est-il acceptable pour l'application ? »

`FileManager` répond à la question :

> « Comment stocker physiquement ce fichier ? »

Cette distinction doit être conservée dans les contrôleurs.

# 7. Stockage du fichier

Une fois le fichier validé :

```php
$fileManager->copier($fichier['tmp_name'], $inputFile->getValue());
```

Le premier paramètre est le chemin du fichier temporaire fourni par PHP.
Le second paramètre est le nom sous lequel le fichier doit être stocké.

Exemple :

```text
/tmp/phpA8F32
        │
        │ copier()
        ▼
DOSSIER_PDF/rapport.pdf
```

Après une copie réussie, la source temporaire est supprimée par `FileManager`.

Le contrôleur n'a donc pas à effectuer lui-même un `unlink()` de la source.

# 8. Nom du fichier utilisé pour le stockage

Le nom transmis à `copier()` provient ici de :

```php
$inputFile->getValue()
```

Cela permet de séparer :

- le nom reçu du client ;
- le nom validé et éventuellement préparé par `InputFile` ;
- le nom réellement utilisé par `FileManager`.

Exemple :

```php
$nom = $inputFile->getValue();

$fileManager->copier($fichier['tmp_name'], $nom);
```

Cette approche est préférable à l'utilisation directe d'une valeur provenant de la requête.

# 9. Gestion des collisions de noms

Le comportement est contrôlé par :

```php
'renommerSiExiste' => false,
```

Dans la configuration PDF fournie, cette valeur est `false`.

Cela signifie que si :

```text
rapport.pdf
```

existe déjà, l'appel :

```php
$fileManager->copier($source, rapport.pdf');
```

provoque une `UserException`.

Le fichier existant n'est donc pas écrasé.

## 9.1 Autoriser le renommage automatique

Si la configuration contient :

```php
'renommerSiExiste' => true,
```

`FileManager` génère automatiquement un nom disponible.

Exemple :

```text
rapport.pdf
rapport(1).pdf
rapport(2).pdf
```

Dans ce cas, le nom retourné par `copier()` doit être récupéré :

```php
$nomStocke = $fileManager->copier($source,'rapport.pdf');
```

`$nomStocke` peut alors contenir : rapport(1).pdf

Le développeur doit utiliser cette valeur s'il doit mémoriser le nom du fichier.

# 10. Consulter les fichiers après un téléversement

Dans le contrôleur de téléversement, après la copie :

```php
ReponseJson::envoyerLesDonnees($fileManager->getLesFichiers());
```

`getLesFichiers()` permet de récupérer la liste actuelle des fichiers présents dans le répertoire.

Exemple :

```php
[
    'document.pdf',
    'rapport.pdf',
    'facture.pdf'
]
```

Cela permet au contrôleur de renvoyer directement au client l'état actualisé du répertoire.

# 11. Suppression d'un fichier

La suppression ne nécessite pas `InputFile`.

Le fichier existe déjà sur le serveur : il suffit de transmettre son nom au `FileManager`.

Exemple :

```php
$nomFichier = Requete::postString('nomFichier');

$fileManager = new FileManager(DOSSIER_PDF);

if (!$fileManager->supprimer($nomFichier)) {
    ReponseJson::envoyerLesErreurs([
        'global' => [
            "Le fichier n'a pas pu être supprimé."
        ]
    ]);
}
```

Puis le contrôleur peut retourner la liste actualisée :

```php
ReponseJson::envoyerLesDonnees($fileManager->getLesFichiers());
```

# 12. Exemple complet de suppression

Le traitement est donc :

```text
Requête HTTP
     │
     ▼
Requete::postString()
     │
     │ nom du fichier
     ▼
FileManager
     │
     │ supprimer()
     ▼
Fichier physique
     │
     ▼
getLesFichiers()
     │
     ▼
ReponseJson
```

Exemple complet :

```php
declare(strict_types=1);

use ClasseTechnique\FileManager;
use ClasseTechnique\ReponseJson;
use ClasseTechnique\Requete;

require $_SERVER['DOCUMENT_ROOT']. '/../bootstrap/bootstrap.php';

Requete::exigerPost();

$nomFichier = Requete::postString('nomFichier');

$fileManager = new FileManager(DOSSIER_PDF);

if (!$fileManager->supprimer($nomFichier)) {
    ReponseJson::envoyerErreur("Le fichier n'a pas pu être supprimé.");
}

ReponseJson::envoyerLesDonnees($fileManager->getLesFichiers());
```


# 13. `supprimer()` et fichier déjà absent

`supprimer()` retourne `true` lorsque le fichier n'existe déjà plus.

Cette caractéristique permet d'effectuer une suppression sans avoir à tester préalablement :

```php
if ($fileManager->existe($nomFichier)) {
    $fileManager->supprimer($nomFichier);
}
```

Le test préalable n'est donc généralement pas nécessaire.

Il est possible de faire directement :

```php
if (!$fileManager->supprimer($nomFichier)) {
    // Échec réel de la suppression
}
```

Cette approche évite une double vérification du système de fichiers.

# 14. Remplacement d'un fichier

Le remplacement est utilisé lorsqu'un fichier métier existe déjà et doit conserver son nom.

Exemple :

```php
$fileManager->remplacer($fichier['tmp_name'], $nomFichier);
```

Le fichier cible doit obligatoirement exister.

Contrairement à `copier()`, `remplacer()` ne cherche pas un autre nom.

```text
Ancien fichier
rapport.pdf
     │
     │ remplacer()
     ▼
Nouveau contenu
rapport.pdf
```

Le nom du fichier reste donc inchangé.

# 15. Quand utiliser `copier()` ou `remplacer()` ?

|Situation|Méthode|
|---|---|
|Nouveau fichier|`copier()`|
|Ajout avec nom disponible|`copier()`|
|Ajout avec renommage automatique|`copier()` + `renommerSiExiste = true`|
|Fichier déjà existant à remplacer|`remplacer()`|
|Suppression|`supprimer()`|
|Vérification d'existence|`existe()`|
|Liste des fichiers|`getLesFichiers()`|

### Exemple

Pour créer un nouveau document :

```php
$nomStocke = $fileManager->copier($source, $nom);
```

Pour modifier le document existant :

```php
$fileManager->remplacer($source,$nom);
```

Pour supprimer le document :

```php
$fileManager->supprimer($nom);
```

# 16. Vérification de l'existence

La méthode :

```php
$fileManager->existe($nom);
```

vérifie l'existence physique du fichier dans le répertoire géré.

Elle peut notamment être utilisée avant une opération métier qui dépend de la présence du fichier.

Exemple :

```php
if (!$fileManager->existe($nomFichier)) {
    throw new UserException(
        "Le fichier demandé n'existe pas."
    );
}
```

Pour une suppression simple, ce test n'est généralement pas nécessaire puisque `supprimer()` considère déjà l'absence du fichier comme une réussite.

# 17. Génération d'un nom unique

La méthode :

```php
$fileManager->genererNomUnique($nom);
```

permet de rechercher un nom disponible.

Exemple :

```php
$nom = $fileManager->genererNomUnique('document.pdf');
```

Si les fichiers suivants existent :

```text
document.pdf
document(1).pdf
document(2).pdf
```

le résultat sera :

```text
document(3).pdf
```

Cette fonctionnalité est utilisée automatiquement par `copier()` lorsque :

```php
'renommerSiExiste' => true
```

Il n'est donc normalement pas nécessaire de l'appeler directement dans un contrôleur.

# 18. `getLesFichiers()` et filtrage par extension

Lorsque la configuration contient :

```php
'lesExtensions' => ['pdf'],
```

`getLesFichiers()` ne retourne que les fichiers ayant une extension correspondant à cette configuration.

Exemple :

```text
Répertoire physique :

rapport.pdf
contrat.pdf
image.jpg
notes.txt
```

Avec :

```php
'lesExtensions' => ['pdf']
```

le résultat sera :

```php
[
    'rapport.pdf',
    'contrat.pdf'
]
```

### Attention

Ce filtrage concerne **la liste retournée par `getLesFichiers()`**.

Il ne remplace pas la validation effectuée par `InputFile`.

La configuration :

```php
'lesExtensions' => ['pdf']
```

est donc utilisée à deux endroits différents selon le composant :

```text
InputFile
    └── contrôle que le fichier reçu est autorisé

FileManager
    └── filtre les fichiers retournés par getLesFichiers()
```

# 19. Contrat d'utilisation

Lorsqu'un développeur utilise `FileManager`, il doit respecter les principes suivants.

### Avant `copier()`

Le fichier doit avoir été :

- récupéré correctement ;
- validé ;
- contrôlé selon les règles métier ;
- associé à un nom utilisable.

Exemple :

```php
$fichier = Requete::getFile('fichier');

$inputFile = new InputFile($fichier, $lesParametres);

if (!$inputFile->checkValidity()) {
    ReponseJson::envoyerErreur($inputFile->getValidationMessage());
}
```

Puis seulement :

```php
$fileManager->copier($fichier['tmp_name'], $inputFile->getValue());
```

### Pour une suppression

Le développeur transmet uniquement le nom du fichier :

```php
$fileManager->supprimer($nomFichier);
```

Il n'a pas à construire lui-même le chemin physique.

### Pour un remplacement

Le nom cible doit correspondre à un fichier déjà présent :

```php
$fileManager->remplacer($sourceTmp, $nomFichier);
```

# 20. Exemple de cycle de vie d'un PDF

Pour un fichier PDF téléversé :

```text
1. Requete
   │
   └── reçoit $_FILES['fichier']

2. Config
   │
   └── charge la configuration "pdf"

3. InputFile
   │
   ├── vérifie la taille
   ├── vérifie l'extension
   ├── vérifie le type MIME
   └── prépare le nom du fichier

4. FileManager
   │
   └── copie le fichier temporaire
       vers DOSSIER_PDF

5. getLesFichiers()
   │
   └── récupère la liste actualisée

6. ReponseJson
   │
   └── retourne les données au client
```

Pour une suppression :

```text
1. Requete
   │
   └── récupère le nom du fichier

2. FileManager
   │
   └── supprime le fichier physique

3. getLesFichiers()
   │
   └── récupère la liste actualisée

4. ReponseJson
   │
   └── retourne les données au client
```

---

# 24. Résumé pour le développeur

## Synthèse des responsabilités

```text
┌─────────────────────────────────────────────┐
│                 CONTRÔLEUR                  │
│                                             │
│  Orchestre le traitement                    │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│                  Requete                    │
│                                             │
│  Récupère les données HTTP                  │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│                 InputFile                   │
│                                             │
│  Valide le fichier reçu                     │
│  Taille / extension / MIME / règles métier  │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│                FileManager                  │
│                                             │
│  Gère le stockage physique                  │
│  copie / remplacement / suppression         │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│             Système de fichiers             │
│                                             │
│              DOSSIER_PDF                    │
└─────────────────────────────────────────────┘
```

**FileManager ne décide pas si un fichier est acceptable : il sait uniquement comment le stocker, le remplacer, le rechercher ou le supprimer physiquement.**