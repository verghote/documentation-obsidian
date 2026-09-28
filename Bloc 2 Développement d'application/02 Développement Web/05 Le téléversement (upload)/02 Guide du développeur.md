  
Nous pouvons distinguer 4 cas pratiques nécessitant la mise en place d'un téléversement

 
- gérer une bibliothèque de documents PDF (uploaddocument) ;
- gérer une bibliothèque d'images (uploadimage) ;  
- gérer les enregistrements une table en liaison avec un fichier PDF (document) ;  
- gérer les enregistrements d'une table en liaison avec un fichier image (club).  
  
## 1. Vue d'ensemble  
  
Un téléversement traverse plusieurs couches. Le navigateur aide l'utilisateur à  
sélectionner un fichier, mais la validation déterminante et l'écriture sont  
effectuées côté serveur.  
  
```text  
Navigateur  
  -> requête multipart/form-data (champ "fichier")  
  -> Requete::getFile()  ==> récupération de $_Files["fichier"]
  -> InputFile ou InputFileImg  
  -> Service métier (si le fichier est lié à un enregistrement)  
  -> FileManagerFactory  -> PdfManager ou ImageManager  -> FileManager : validation, nommage et stockage  -> répertoire configuré et, au besoin, table métier  
```  
  
Le circuit court (`uploadclassement`, `uploadimage`) appelle directement un  gestionnaire de fichiers. Le circuit métier (`document`, `club`) passe par un  service qui maintient la cohérence entre le fichier physique et l'enregistrement  en base de données.  
  
## 2. Présenter les classes dans l'ordre du flux  
  
L'ordre ci-dessous suit la façon dont une donnée entre dans le système et est  traitée. Il permet de comprendre les dépendances avant de lire les exemples de  modules.  
  
### 2.1 `Input` : socle commun des valeurs contrôlées  
  
`src\ClasseTechnique\Input.php` est une classe abstraite. Elle fournit les  
propriétés et accesseurs communs aux entrées contrôlées :  
  
- la valeur (`value`) ;  
- le caractère obligatoire (`required`) ;  
- le message de validation (`validationMessage`).  
  
`InputFile` hérite de ce socle. Il redéfinit toutefois `checkValidity()` pour valider un téléversement complet plutôt qu'une simple valeur de formulaire.  
  
### 2.2 `InputFile` : représentation et validation du fichier reçu  
  
`src\ClasseTechnique\InputFile.php` reçoit le tableau PHP d'un fichier,  normalement obtenu depuis `$_FILES` par `Requete::getFile('fichier')`.  
L'objet conserve les informations PHP du téléversement et utilise `value` pour  le nom qui sera enregistré.  
  
Pendant `checkValidity()`, il vérifie, dans cet ordre :  
  
1. l'état du téléversement PHP et la présence du fichier temporaire ;  
2. la taille maximale configurée ;  
3. l'extension autorisée ;  
4. le type MIME déterminé côté serveur avec FileInfo ;  
5. la validation spécialisée éventuelle ;  
6. le nettoyage du nom (casse et accents selon la configuration).  
  
Les extensions, types MIME et limites sont fournis par le gestionnaire de  fichiers avant cette validation. L'objet expose notamment  `getTmpName()`, `getName()`, `getSize()`, `getError()`, `getValue()` et  `getValidationMessage()`.  
  
> Le type MIME transmis par le navigateur n'est pas une preuve fiable. La  
> validation de `InputFile` calcule le type à partir du fichier temporaire.  
  
### 2.3 `InputFileImg` : contrôles propres aux images  
  
`src\ClasseTechnique\InputFileImg.php` hérite de `InputFile`. Il ajoute :  
  
- la validation du contenu et des dimensions par `getimagesize()` ;  
- `setDimensions()` pour fixer des dimensions minimales ou maximales ;  
- `getDimensions()` pour lire la largeur et la hauteur du fichier temporaire.  
  
`InputFile` appelle la méthode spécialisée pendant sa validation. Le contrôle  des dimensions ne s'applique donc que si le contrôleur ou le service construit  un `InputFileImg`. 
Les points d'entrée présentés plus bas ne configurent pas de  limites dimensionnelles avec `setDimensions()` ; ils s'appuient sur le type  image et, pour certains parcours, sur le redimensionnement serveur.  
  
### 2.4 `FileManager` : opérations communes de stockage  
  
`src\ClasseTechnique\FileManager.php` est la classe abstraite commune à tous les gestionnaires. Elle reçoit un répertoire de stockage et définit les opérations  communes :  
  
| Méthode                           | Rôle                                                                             |     |
| --------------------------------- | -------------------------------------------------------------------------------- | --- |
| getLesFichiers()                  | Liste les fichiers dont l'extension est autorisée.                               |     |
| existe($nom)                      | Vérifie l'existence d'un fichier au nom autorisé.                                |     |
| ajouter($inputFile, $rename)      | Configure puis valide l'entrée, gère le doublon et copie le fichier.             |     |
| remplacer($ancienNom, $inputFile) | Valide un nouveau fichier et remplace le fichier existant en conservant son nom. |     |
| supprimer($nom)                   | Supprime le fichier du répertoire géré.                                          |     |
  
Avant l'ajout ou le remplacement, `FileManager` transmet à `InputFile` les  extensions et types MIME autorisés, puis appelle `configurerInputFile()` et  `checkValidity()`. 
Le stockage ne doit donc pas être réalisé avant l'appel au  gestionnaire.  
  
Pour un ajout, le paramètre `$rename` décide du traitement d'un nom déjà  présent :  
  
- `false` : l'ajout échoue avec un message de doublon ;  
- `true` : le gestionnaire cherche un nom libre, par exemple  en ajoutant un numéro :  `classement(1).pdf`.  
  
Le remplacement conserve le nom déjà associé à l'enregistrement et remplace  physiquement le fichier à cette adresse.  
  
### 2.5 `PdfManager` et `ImageManager` : règles spécialisées  
  
`src\ClasseTechnique\PdfManager.php` hérite de `FileManager`. Il limite les  fichiers à l'extension `pdf` et au type MIME `application/pdf`, puis transmet à  `InputFile` la taille maximale et la règle de suppression des accents.  
  
`src\ClasseTechnique\ImageManager.php` hérite aussi de `FileManager`. Il  autorise les formats image définis dans la classe, configure la taille et le  nom, et peut redimensionner l'image avant son stockage. 
Le redimensionnement  n'est utilisé que si :  
  
1. l'instance reçue est un `InputFileImg` ;  
2. `redimensionner` est activé dans la configuration ;  
3. au moins une dimension cible est supérieure à zéro.  
  
Avec un `InputFile` ordinaire, `ImageManager` utilise la copie commune de  `FileManager` : le contrôle spécifique du contenu et des dimensions de  `InputFileImg` n'est alors pas exécuté.  
  
### 2.6 `FileManagerFactory` et configuration  
  
`src\Service\FileManagerFactory.php` associe un contexte à son gestionnaire,  son fichier de configuration et son répertoire :  
  
| Fabrique | Gestionnaire | Configuration | Stockage |  
|---|---|---|---|  
| `FileManagerFactory::pdf()` | `PdfManager` | `config\pdf.php` | `public\data\pdf` |  
| `FileManagerFactory::document()` | `PdfManager` | `config\document.php` | `public\data\document` |  
| `FileManagerFactory::image()` | `ImageManager` | `config\image.php` | `public\data\image` |  
| `FileManagerFactory::club()` | `ImageManager` | `config\club.php` | `public\data\club` |  
  
Les constantes de répertoire sont définies dans `bootstrap\bootstrap.php`.  
`PdfManager` utilise notamment `maxSize` et `sansAccent`. 
`ImageManager` utilise  ces paramètres ainsi que `redimensionner`, `width` et `height`.  
  
Le choix d'ajouter avec ou sans renommage est fait à l'appel de  `ajouter($file, true|false)` ; 
la fabrique ne lit pas de paramètre `rename`.  
Une limite de taille égale à zéro signifie qu'aucune limite applicative n'est  imposée par `InputFile` (les limites PHP restent applicables).  
  
## 3. Où placer les responsabilités  
  
| Couche                       | Responsabilité                                                                                                                                        |     |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | --- |
| JavaScript / formulaire      | Sélection, pré-contrôle ergonomique et envoi du fichier. Ce n'est pas une barrière de sécurité.                                                       |     |
| Contrôleur AJAX              | Vérifier la requête, extraire les paramètres et le fichier, instancier l'entrée adaptée, appeler le gestionnaire ou le service, renvoyer une réponse. |     |
| `InputFile` / `InputFileImg` | Validation du fichier reçu et préparation de son nom.                                                                                                 |     |
| Gestionnaire                 | Règles de format, validation déclenchée, ajout/remplacement/suppression et stockage physique.                                                         |     |
| Service métier               | Coordonner le gestionnaire avec l'enregistrement associé et exposer les erreurs métier.                                                               |     |
| Classe métier / table        | Valider et enregistrer les données de l'entité en base ; ne gère pas directement le contenu physique du fichier.                                      |     |
  
## 4. Exemple A : classement PDF dans `uploadclassement`  
  
### À la consultation  le script public\uploadclassement\index.php 

+ appelle  `FileManagerFactory::pdf()`, 
+ demande la liste avec `getLesFichiers()` 
+ la  transmet à la page. 

Le navigateur affiche les fichiers du répertoire  `data\pdf`.  
  
### À l'ajout  le script public\uploadclassement\ajax\ajouter.php suit ce circuit :  
  
```php  
Requete::exigerPost();  
  
$pdfManager = FileManagerFactory::pdf();  
$file = new InputFile(Requete::getFile('fichier'));  
$file->setRequired(true);  
  
if (!$pdfManager->ajouter($file, true)) {  
    ReponseJson::envoyerLesErreurs(['global' => [$file->getValidationMessage()]]);
}  
  
ReponseJson::envoyerLesDonnees($pdfManager->getLesFichiers());  
```  
  
`InputFile` convient ici parce que le fichier attendu est un PDF, pas une image.  
Le gestionnaire applique les règles PDF de `config\pdf.php`. 
Le paramètre `true`  autorise le renommage si le nom existe déjà. 
La réponse contient la liste actualisée. 

La suppression, dans `ajax\supprimer.php`, appelle le même  gestionnaire puis renvoie également la liste.  
  
## 5. Exemple B : image de bibliothèque dans `uploadimage`  
  
`public\uploadimage\index.php` utilise `FileManagerFactory::image()` pour  obtenir la liste initiale. 

Côté client, Son JavaScript pré-contrôle la taille et l'extension,  
puis envoie l'objet `File` sous le champ `fichier` avec `FormData`.  
  
`public\uploadimage\ajax\ajouter.php` reçoit ce champ et construit un  
`InputFileImg`. Le gestionnaire est créé par `FileManagerFactory::image()` ;  
il valide l'image et applique la configuration de `config\image.php` (limite  
actuelle de 300 Ko et redimensionnement jusqu'à 350 px de largeur). L'ajout est  
appelé avec `$rename = false` : un doublon est refusé. Après succès, le  
contrôleur renvoie la liste des images.  
  
Le contrôleur appelle actuellement `$file->setRequired(false)`. Cela ne suffit  
pas, dans l'implémentation de `InputFile::checkValidity()`, à rendre l'absence  
de fichier valide : après le contrôle de `UPLOAD_ERR_NO_FILE`, la validation  
continue vers l'extension et le type MIME, qui échouent pour un fichier absent.  
Le parcours d'ajout doit donc être traité comme nécessitant effectivement un  
fichier.  
  
La suppression suit le même modèle que celle des classements, avec  
`FileManagerFactory::image()` et `supprimer()`.  
  
## 6. Exemple C : PDF associé à un document  
  
Ici, il ne suffit pas d'écrire un PDF dans un répertoire : son nom doit être  
enregistré dans la table `document`. Le contrôleur  
`public\document\ajout\ajax\ajouter.php` :  
  
1. vérifie la requête POST ;  
2. récupère le titre et le fichier ;  
3. construit un `InputFile` ;  
4. appelle `ServiceDocument::ajouter($titre, $file)` ;  
5. renvoie les erreurs du service ou la liste des documents.  
  
`ServiceDocument` crée `FileManagerFactory::document()` (un `PdfManager`  
configuré pour `data\document`) et coordonne le stockage avec la classe métier  
`Document` :  
  
```text  
Contrôleur -> ServiceDocument -> PdfManager -> validation et stockage PDF  
                          \-> Document -> insertion SQL avec le nom du PDF  
```  
  
Le service rend le fichier obligatoire, stocke d'abord le PDF puis insère son  
nom avec le titre dans la base. Si l'insertion échoue, il supprime le fichier  
qu'il vient d'ajouter afin d'éviter un fichier orphelin.  
  
Pour remplacer un PDF, `public\document\maj\ajax\remplacer.php` appelle  
`ServiceDocument::remplacerPdf()`. Le service retrouve le document et demande  
au gestionnaire de remplacer le fichier associé, en conservant son nom. La  
suppression d'un document doit aussi passer par le service pour traiter  
l'enregistrement et le fichier ensemble.  
  
## 7. Exemple D : logo associé à un club  
  
Le logo est géré par `ServiceClub`, et non directement par la classe métier  
`Club`. Le service utilise `FileManagerFactory::club()` et un  
`ImageManager` associé au répertoire `data\club` et à `config\club.php`.  
Cette configuration limite le fichier à 150 Ko et n'active pas le  
redimensionnement.  
  
### Création d'un club avec logo  
  
`public\club\ajout\ajax\ajouter.php` construit un `InputFileImg` si un fichier a  
été transmis et le passe à `ServiceClub::ajouter()`. Le service rend ce logo  
obligatoire lorsqu'il existe, l'ajoute avec renommage activé, puis enregistre  
son nom dans `Club`. Si l'insertion en base échoue, le service supprime le logo  
ajouté.  
  
### Ajout ou remplacement d'un logo  
  
`public\club\logo\ajax\remplacer.php` transmet l'identifiant du club et le  
fichier à `ServiceClub::enregistrerLogo()`. Le service vérifie que le club  
existe, puis :  
  
- si le club n'a pas encore de logo, ajoute l'image et enregistre son nom dans  
  la table ;  
- si un logo existe, remplace le fichier au même nom, sans modifier la  
  référence stockée dans la table.  
  
`Club` reste responsable des données et règles métier du club. `ServiceClub`  
coordonne `Club` et `ImageManager` pour assurer la cohérence du logo physique  
et de sa référence.  
  
## 8. Patron à suivre pour un nouveau téléversement  
  
1. Déterminer si le fichier est autonome ou lié à une entité métier.  
2. Pour une ressource autonome, choisir `PdfManager` ou `ImageManager` via  
   `FileManagerFactory`. Pour une ressource liée, passer par le service métier  
   correspondant.  
3. Construire `InputFile` pour un PDF ou `InputFileImg` pour une image à  
   contrôler comme telle.  
4. Récupérer le fichier avec `Requete::getFile()` et vérifier la requête avec  
   `Requete::exigerPost()`.  
5. Appeler `ajouter()`, `remplacer()` ou `supprimer()` sur la couche  
   responsable ; ne pas recopier directement le fichier dans le contrôleur.  
6. En cas d'échec, renvoyer les erreurs fournies par l'entrée ou le service ;  
   en cas de succès, renvoyer les données actualisées utiles à l'interface.  
7. Si un fichier et un enregistrement sont liés, définir explicitement le  
   comportement de compensation en cas d'échec d'une des deux écritures.  
  
Les contrôles JavaScript (attribut HTML `accept`, taille, extension et aperçu)  
améliorent l'expérience utilisateur, mais peuvent être contournés. Le serveur  
doit toujours appliquer les contrôles et règles décrits dans ce guide.