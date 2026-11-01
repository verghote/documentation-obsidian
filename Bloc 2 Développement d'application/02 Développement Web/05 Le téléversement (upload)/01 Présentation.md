 
## 1. Principe général  
  
Le téléversement de fichiers (*upload*) permet à un utilisateur d'envoyer un fichier depuis son poste client vers un serveur web.  
  
En HTML, l'envoi d'un fichier repose sur l'utilisation d'un champ `<input>` de type `file`.  
  
Le processus général est le suivant :  
  
1. L'utilisateur sélectionne un fichier dans son navigateur.  
2. Le navigateur transmet le fichier au serveur.  
3. PHP place temporairement le fichier dans le répertoire temporaire configuré par `upload_tmp_dir` (ou, à défaut, dans le répertoire temporaire système).
4. L'application contrôle le fichier reçu.  
5. Le fichier est accepté puis déplacé dans son emplacement définitif.  
  
> ⚠️ **Ne jamais faire confiance aux données transmises par le client.**  
> Tous les contrôles de sécurité doivent obligatoirement être réalisés côté serveur.  
  
# 2. Le champ HTML `input` de type `file`  
  
Un champ de type `file` permet à l'utilisateur de sélectionner un fichier :  
  
```html  
<input type="file" id="fichier" accept=".pdf"/>  
```  
  
L'attribut `accept` permet de filtrer les fichiers proposés dans la fenêtre de sélection.  
  
Exemples :  
  
```html  
<input type="file" accept=".jpg,.png,.pdf">  
```  
  
```html  
<input type="file" accept="application/pdf">  
```  
  
```html  
<input type="file" accept="image/*">  
```  
  
L'attribut `accept` peut contenir :  
+ Une extension de fichier : .jpg, .png, .pdf, .doc  
+ Un type MIME précis : application/pdf, application/msword, image/jpeg  
+ Un type MIME générique : image/*, audio/*, video/*  
  
L'attribut `accept` **n'effectue aucun contrôle de sécurité**.  
  
Il sert uniquement à filtrer les fichiers affichés dans la fenêtre de sélection du navigateur.  
  
Un utilisateur peut facilement contourner cette limitation :  
  
- en modifiant l'extension d'un fichier ;  
- en utilisant un autre outil d'envoi HTTP ;  
- en envoyant directement une requête vers le serveur.  
  
Les contrôles réels doivent donc être réalisés côté serveur.  
  
# 3. Mise en forme du champ fichier  
  
L'apparence du champ `file` dépend du navigateur utilisé.  
  
Son rendu est généralement peu personnalisable :  
  
- bouton différent selon le navigateur ;  
- texte variable ;  
- intégration difficile dans une interface graphique.  
  
Une solution courante consiste à masquer le champ original et à déclencher son ouverture depuis un bouton personnalisé.  
  
Exemple :  
  
```html  
<input type="file" id="fichier" accept=".pdf" style="display:none"/>  
```  
  
Le bouton visible déclenchera ensuite l'ouverture du sélecteur de fichiers.  
  
```html  
<button class="btn btn-danger w-100" id="btnAjouter">
       Ajouter un fichier PDF de 2 Mo maximum
 </button>  
  
<input type="file" id="fichier" accept=".pdf" style="display:none">  
```  
  
Le JavaScript permet ensuite de déclencher le clic :  
  
```javascript  
btnAjouter.onclick = () => {  
    fichier.click();};  
```  
  
Une autre approche consiste à associer :  
  
- un bouton « Parcourir » ;  
- un champ texte affichant le nom du fichier.  
  
Exemple :  
  
```html  
<label for="nomFichier" class='obligatoire'>Ficher PDF (taille limitée à 512 Mo)</label>  
<div class="groupe-fichier">  
    <button id="btnFichier" class="btn btn-sm btn-outline-secondary">📁 Choisir un fichier</button>  
    <span id="nomFichier" class="texte-muted ms-2"></span>  
</div>  
<input type="file" id="fichier" accept=".pdf" style='display: none '>  
<button id="btnAjouter" class="btn btn-danger">Ajouter</button>  
<table>
```  
  
Avantages :  
  
- interface plus claire ;  
- possibilité d'ajouter du glisser-déposer ;  
- contrôle du fichier avant envoi.  
# 4. Traitement côté serveur avec PHP  
  
## 4.1 Réception du fichier  
  
Lorsqu'un fichier est envoyé, PHP le place temporairement dans un dossier défini par la configuration :  
  
```ini  
; Temporary directory for HTTP uploaded files (will use system default if not
; specified).
; https://php.net/upload-tmp-dir
upload_tmp_dir ="c:/wamp64/tmp"
```  
  
Le fichier reçoit un nom temporaire généré automatiquement.  
  
Les informations concernant le fichier sont accessibles dans la variable superglobale :  
  
```php  
$_FILES  
```  
  
## 4.2 Structure de `$_FILES`  
  
Le nom utilisé dans `$_FILES` correspond à l'attribut `name` du champ HTML.  
  
Exemple :  
  
```html  
<input type="file" name="fichier">  
```  
  
Les informations sont accessibles avec :  
  
```php  
$_FILES['fichier']  
```  
  
Cette variable contient un tableau associatif :  
  
| Clé      | Description                                 |     |
| -------- | ------------------------------------------- | --- |
| tmp_name | Chemin du fichier temporaire sur le serveur |     |
| name     | Nom original du fichier côté client         |     |
| size     | Taille du fichier en octets                 |     |
| type     | Type MIME transmis par le navigateur        |     |
| error    | Code d'erreur du transfert                  |     |
## 4.3. Les erreurs de téléversement  
  
La clé :  
  
```php  
$_FILES['fichier']['error']  
```  
  
contient le résultat du transfert.  
  
Les principales valeurs sont :  
  
| Constante | Valeur | Signification |  
|-|-|-|  
| `UPLOAD_ERR_OK` | 0 | Téléversement réussi |  
| `UPLOAD_ERR_INI_SIZE` | 1 | Taille supérieure à `upload_max_filesize` |  
| `UPLOAD_ERR_FORM_SIZE` | 2 | Taille supérieure à `MAX_FILE_SIZE` |  
| `UPLOAD_ERR_PARTIAL` | 3 | Téléversement partiel |  
| `UPLOAD_ERR_NO_FILE` | 4 | Aucun fichier transmis |  
| `UPLOAD_ERR_NO_TMP_DIR` | 6 | Dossier temporaire absent |  
| `UPLOAD_ERR_CANT_WRITE` | 7 | Échec d'écriture disque |  
  
## 4.4 Contrôles de sécurité côté serveur  
  
Avant toute utilisation d'un fichier reçu, plusieurs contrôles doivent être effectués :  
  
- présence d'un fichier ;  
- absence d'erreur de transfert ;  
- taille maximale ;  
- extension autorisée ;  
- véritable type MIME ;  
- nom du fichier ;  
- emplacement de stockage.  
  
### 4.4.1 Vérifier qu'un fichier existe  
  
```php  
if (!isset($_FILES['fichier'])) {  
    echo json_encode(['error' => "Aucun fichier transmis"]);    exit;}  
```  
  
### 4.4.2 Vérifier les erreurs de transfert  
  
```php  
if ($_FILES['fichier']['error'] !== UPLOAD_ERR_OK) {  
    echo json_encode(['error' => "Erreur lors du transfert"]);    exit;}  
```  
  
### 4.4.3 Récupérer les informations du fichier  
  
```php  
$tmp = $_FILES['fichier']['tmp_name'];  
  
$nomFichier = $_FILES['fichier']['name'];  
  
$taille = $_FILES['fichier']['size'];  
```  
  
### 4.4.4. Vérification de la taille du fichier  
  
La taille maximale autorisée doit être définie en fonction des besoins de l'application.  
  
Exemple : limitation à 2 Mo.  
  
```php  
$tailleMax = 2 * 1024 * 1024;  
  
if ($taille > $tailleMax) {  
    echo json_encode(['error' => "La taille du fichier dépasse la taille autorisée"]);    
    exit;
}  
```  
  
La taille est exprimée en octets :  
  
| Taille | Valeur             |
| ------ | :----------------- |
| 1 Ko   | 1024 octets        |
| 1 Mo   | 1024 × 1024 octets |
Remarque : La limite applicative intervient **après certaines limites de PHP**.

Par exemple on trouve dans le fichier php.ini :

```ini
upload_max_filesize = 2M
post_max_size = 8M
```

Dans ce cas, un fichier de 3 Mo sera rejeté **avant même que ton code PHP puisse effectuer ton contrôle `$taille > $tailleMax`**.

### 4.4.5. Vérification de l'extension  
  
L'extension du fichier peut être récupérée grâce à la fonction `pathinfo()`.  
  
Exemple :  
  
```php  
$extension = strtolower(pathinfo($nomFichier, PATHINFO_EXTENSION));  
```  
  
On peut ensuite comparer cette extension avec une liste autorisée.  
  
```php  
$extensionsAutorisees = ["pdf"];  
  
if (!in_array($extension, $extensionsAutorisees)) {  
    echo json_encode(['error' => "Extension du fichier non acceptée"]);    exit;}  
```  
  
> ⚠️ Une extension seule ne constitue pas un contrôle de sécurité suffisant.  
> Un utilisateur peut simplement renommer un fichier.  
  
Exemple :  
  
```  
virus.exe → document.pdf  
```  
  
Le fichier possède alors une extension `.pdf` mais reste un exécutable.  
  
### 4.4.6. Vérification du type MIME  
  
Le type MIME indique le format supposé du fichier.  
  
Exemples :  
  
| Extension | Type MIME         |  
| --------- | ----------------- |  
| `.pdf`    | `application/pdf` |  
| `.jpg`    | `image/jpeg`      |  
| `.png`    | `image/png`       |  
| `.txt`    | `text/plain`      |  
  
  
La valeur $_FILES['fichier']['type']  provient du client.  Elle peut donc être facilement falsifiée.  
Il ne faut donc pas l'utiliser pour effectuer un contrôle de sécurité sur le type.  
    
PHP fournit l'extension **FileInfo** permettant d'analyser le contenu réel du fichier.  
  
Exemple :  
  
```php  
//  création une ressource `finfo` qui va permettre à PHP d'analyser des fichiers. `FILEINFO_MIME_TYPE` indique qu'on veut obtenir **le type MIME** du fichier.
$finfo = finfo_open(FILEINFO_MIME_TYPE);  
// retourne le type MIME du fichier $tmp qui correspond généralement au **chemin temporaire d'un fichier uploadé**.
$type = finfo_file($finfo, $tmp);  
// ferme la ressource `finfo`pour libérer les ressources utilisées.
finfo_close($finfo);  
```  
  
Le type MIME obtenu est ensuite comparé aux valeurs autorisées :  
  
```php  
$typesAutorises = ["application/pdf"];  
if (!in_array($type, $typesAutorises)) {  
    echo json_encode(['error' => "Type de fichier non accepté"]);    exit;}  
```  
  
  
Quelques types MIME couramment utilisés :  

| Type MIME | Utilisation |  
|-|-|  
| `application/pdf` | Document PDF |  
| `application/msword` | Document Word ancien format |  
| `application/vnd.openxmlformats-officedocument.wordprocessingml.document` | Document Word récent |  
| `application/vnd.ms-excel` | Fichier Excel ancien format |  
| `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` | Fichier Excel récent |  
| `text/plain` | Fichier texte |  
| `text/html` | Page HTML |  
| `image/jpeg` | Image JPEG |  
| `image/png` | Image PNG |  
| `application/octet-stream` | Type générique |  
  
  
La plupart des fichiers ne contiennent pas explicitement leur type MIME.  
  
Le type réel peut être détecté grâce à leur contenu binaire.  
  
On parle alors de **signature de fichier** ou *magic number*.  
  
Exemple :  
  
Un fichier JPEG commence généralement par :  
  
```  
FF D8 FF  
```  
  
Un fichier PNG commence par :  
  
```  
89 50 4E 47  
```  
  
Cette analyse est réalisée automatiquement par l'outil PHP `FileInfo`.  

La signature permet d'identifier ou d'orienter l'identification d'un format, mais elle ne constitue pas à elle seule une validation complète de la structure du fichier.
  
## 4.5 Copie du fichier sur le serveur  
  
Lorsque tous les contrôles sont validés, le fichier temporaire peut être déplacé vers son emplacement définitif.  
  
Exemple :  
  
```php  
copy($tmp, '../document/' . $nomFichier);  
```  
  
Plusieurs fonctions PHP peuvent être utilisées.  
  
### `copy()`  
  
Copie le fichier vers une nouvelle destination.  
  
```php  
copy($source, $destination);  
```  
  
Cette méthode est utile lorsque le serveur web et l'application se trouvent sur des unités de stockage différentes.  
  
### `rename()`  
  
Déplace et renomme le fichier.  
  
```php  
rename($source, $destination);  
```  
  
Le déplacement échoue si le fichier destination existe déjà.  
  
### `move_uploaded_file()`  
  
Fonction spécialisée pour les fichiers envoyés par HTTP.  
  
```php  
move_uploaded_file($source, $destination);  
```  
  
`move_uploaded_file()` est spécialement conçue pour déplacer un fichier ayant été téléversé via HTTP. Elle vérifie notamment que `$source` correspond bien à un fichier uploadé par PHP.
  
> Le fichier temporaire d'un upload PHP est normalement supprimé automatiquement à la fin de la requête s'il n'a pas été déplacé ou conservé.
  
## 4.6 Gestion des doublons  
  
Il est souvent nécessaire de garantir l'unicité du nom du fichier.  
  
Une solution consiste à ajouter un suffixe numérique.  
  
Exemple :  
  
```  
document.pdf  
document(1).pdf  
document(2).pdf  
```  
  
Code :  
  
```php  
$nom = pathinfo($nomFichier, PATHINFO_FILENAME);  
  
$i = 1;  
while (file_exists(REP_DOCUMENT . $nomFichier)) {  
    $nomFichier = $nom . "(" . $i++ . ")." . $extension;}  
```  
  

# 4.7. Redimensionnement de l'image
  
Il est possible de redimensionner l'image afin d'en réduire sa taille, car de nombreux utilisateurs sont incapables de redimensionner l'image avant de la téléverser.
Pourtant il existe de nombreuse façon de le faire et notamment l'utilisation du site https://resizeyourimage.com/

Dans nos projet, le redimensionnement est réalisé par la bibliothèque PHP `gumlet/php-image-resize` (contrainte `^3.0` dans `composer.json`). 
Elle fournit notamment les méthodes qui calculent une nouvelle taille en conservant les proportions. 
  
Installation avec Composer :  
  
```bash  
composer require gumlet/image-resize  
```

Exemple de traitement :  
  
```php  
$image = new ImageResize($fichier);  
  
$image->resizeToWidth(350);  
  
$image->save($destination);  
```  

 Le projet s'appuie sur cette bibliothèque dans `ImageManager` ; le code métier et les points d'entrée AJAX n'ont pas à l'utiliser directement.  
  
Dans `src/ClasseTechnique/ImageManager.php`, la bibliothèque est encapsulée dans `copierImage()`. Lorsque le gestionnaire doit copier un `InputFileImg` et que le redimensionnement est activé, cette méthode :  
  
1. ouvre le fichier temporaire avec `Gumlet\ImageResize` ;  
2. applique `resizeToWidth()`, `resizeToHeight()` ou `resizeToBestFit()` selon les dimensions configurées ;  
3. enregistre directement l'image traitée dans le répertoire de destination.  
  
La surcharge de `copierFichier()` achemine automatiquement les images vers ce traitement. L'ajout ou le remplacement d'une photo passe donc par le redimensionnement sans que `ServiceEtudiant` ni le développeur qui utilise le service aient à appeler la bibliothèque. En cas d'erreur de redimensionnement, `ImageManager` place le message dans l'objet fichier et signale l'échec au service.  
  
Pour le développeur, l'activation et les dimensions se règlent dans le fichier de configuration `config/etudiant.php` :  
  
```php  
'redimensionner' => true,  
'width' => 150,  
'height' => 150,  
```  
  
- `redimensionner => false` désactive le traitement et conserve la copie standard du fichier.  
- Si `width` et `height` sont tous deux positifs, l'image est ajustée pour tenir dans ce cadre, proportions conservées, sans agrandissement.  
- Si seule `width` est positive, la largeur est réduite à cette valeur au maximum ; si seule `height` est positive, la hauteur est réduite de la même façon. Les images déjà plus petites ne sont pas agrandies.  
- Si les deux dimensions valent `0`, aucune transformation n'est effectuée, même si `redimensionner` vaut `true`.  
  
La bibliothèque dépend de l'extension PHP GD ; cette extension doit donc être disponible dans l'environnement PHP qui exécute l'application. Hormis cette dépendance d'environnement, aucun code d'intégration n'est nécessaire en dehors de `ImageManager` : pour le cas d'usage existant, le développeur n'a qu'à modifier la configuration.
# 5. Traitements côté client avec JavaScript  
  
Avant l'envoi au serveur, il est possible d'effectuer des contrôles côté client.  
  
Ces contrôles permettent :  
  
- d'améliorer l'expérience utilisateur ;  
- d'éviter un envoi inutile au serveur ;  
- d'afficher rapidement un message d'erreur.  
  
Cependant, ils ne remplacent jamais les contrôles côté serveur.  
  
> ⚠️ Le code JavaScript exécuté dans le navigateur peut être modifié par l'utilisateur.  
> La validation finale doit toujours être réalisée en PHP.  
  
## 5.1 L'objet `File`  
  
Lorsqu'un utilisateur sélectionne un fichier, le navigateur crée un objet JavaScript de type `File`.  
  
Un champ :  
  
```html  
<input type="file" id="fichier">  
```  
  
possède une propriété :  
  
```javascript  
fichier.files  
```  
  
Cette propriété contient un objet `FileList`.  
  
`FileList` est un objet ressemblant à une collection indexée contenant les objets `File` sélectionnés.
  
Exemple :  
  
```javascript  
const fichiers = fichier.files;  
  
console.log(fichiers[0]);  
```  
  
Si plusieurs fichiers sont autorisés :  
  
```html  
<input type="file" multiple>  
```  
  
la collection peut contenir plusieurs objets `File`.  
  
Un objet `File` possède notamment :  
  
| Propriété | Description                         |     |
| --------- | ----------------------------------- | --- |
| `name`    | Nom du fichier avec son extension   |     |
| `size`    | Taille du fichier en octets         |     |
| type`     | Type MIME déclaré par le navigateur |     |
|           |                                     |     |

> **`File.type` fournit un type MIME déterminé par le navigateur. Cette valeur est indicative et ne doit pas être considérée comme une preuve du type réel du fichier. Le navigateur peut notamment déterminer ce type à partir du nom et de l'extension du fichier, et peut également fournir une chaîne vide lorsqu'il ne peut pas déterminer le type. Il ne faut donc jamais utiliser `File.type` comme mécanisme de sécurité côté client ou côté serveur.**

Exemple :  
  
```javascript  
let file = fichier.files[0];  
  
console.log(file.name); // photo.jpg 
console.log(file.size); // taille en octets 
console.log(file.type); // généralement "image/jpeg"
```  

Si un utilisateur renomme virus.exe → photo.jpg le type renvoyé sera "image/jpeg"

## 5.3 Paramètres à transmettre au JavaScript  
  
Il faut transmettre les éléments suivants pour un document PDF par exemple : 

```javascript
const repertoire =  '/data/pdf';  
const maxSize = 512 * 1024;  
const lesExtensions = ['pdf'];
```

Pour une image :
```javascript
// Répertoire de stockage  
const repertoire = '/data/photo';  
  
// les extensions autorisées  
const lesExtensions = ['jpg', 'jpeg', 'png', 'webp', 'avif'];  
  
// Taille maximale (300 Ko)  
const maxSize = 150 * 1024;  
  
// Redimensionnement automatique  
const redimensionner = true;  
  
// Dimensions maximales  
const width = 150;  
const height = 150;
```

La variable redimensionner influence le contrôle sur les dimensions qui ne sera pas réalisé si la valeur est true.

Si l'image ne doit pas être redimensionnée mais qu'il n'y a pas de limite sur ces dimensions , width et height prennent la valeur 0.

Il est possible de placer une limite uniquement sur la largeur ou sur la hauteur/

Remarque : Les paramètres de téléversement pourrait être transmis depuis PHP afin d'éviter les éventuelles incohérences.
  
## 5.4. Initialisation du champ fichier  
  
L'attribut `accept` de la balise file permet de filtrer les fichiers affichés dans la boîte de dialogue:  
  
```html
<input type="file" id="fichier" accept=".jpg, .jpeg, .png, .webp, .avif" hidden>
```  
  
Ce n'est cependant qu'un filtre logique qui peut être enlevé par l'utilisateur 
  
## 5.5 Contrôle du fichier 

Lorsque l'utilisateur sélectionne un fichier :  
  
```javascript  
fichier.onchange = () => {  
    if (fichier.files.length > 0) { 
        controlerFichier(fichier.files[0]);
    }  
};  
```

La fonction controlerFichier fait appel à deux fonctions de la bibliothèque /composant/fonction/fichier.js pour contrôler le fichier

La fonction fichierValide  permet de contrôler l'extension et la taille maximale et s'applique sur tous les fichiers/.
La fonction verifierImage permet de vérifier les dimensions si le fichier est une image. 
C'est une fonction asynchrone  dont le dernier paramètre pointe une fonction de rappel qui reçoit implicitement l'image chargé en mémoire.

La fonction verifier Image permet de façon détournée de vérifier le type Mime car elle essaye de charger l'image en mémoire ce qui déclenche une erreur si le fichier n'est pas une image.
La fonction récupère l'erreur par l'intermédiaire de son gestionnaire d'événement onError et en déduit alors qu'il ne s'agit pas d'une image

Exemple :  
  
```javascript  
import {fichierValide, verifierImage,} from "/composant/fonction/fichier.js";

function controlerFichier(file) {  
    // Efface les erreurs précédentes  
    effacerLesErreurs();  
    // Vérification de taille et d'extension  
    if (!fichierValide(file, maxSize, lesExtensions)) {  
        return;  
    }  
    // Vérifications spécifiques pour un fichier image  
    // la fonction de rappel reçoit le fichier et l'image éventuellement redimensionnée si le redimensionnement est demandé    
    verifierImage(file, redimensionner, width, height, majPhoto);  
}


function majPhoto(file) {  
    // transfert du fichier vers le serveur dans le répertoire sélectionné  
    const formData = new FormData();  
    formData.append('fichier', file);  
    formData.append('id', idEtudiant);  
    appelAjax({  
        url: 'ajax/majphoto.php',  
        data: formData,  
        success: (data) => {  
            afficherToast("La photo a été mise à jour");  
            const cible = document.getElementById('cible' + idEtudiant);  
            const image = cible.querySelector('.carte-img');  
            image.src = `${repertoire}/${encodeURIComponent(data.photo)}?t=${Date.now()}`;  
            cible.closest('.carte-etudiant').querySelector('.btn-supprimer-photo').hidden = false;  
        }  
    });  
}
```  
  
Si le champ `file` est masqué, un bouton peut déclencher son ouverture.  
  
```javascript  
btnFichier.onclick = () => { fichier.click();};  
```  
  
## 5.6. Mise en place du Glisser-déposer   
  
Il est possible d'utiliser une zone graphique comme zone de dépôt pour y faire glisser un fichier en provenance de l'explorateur par exemple.  

Exemple :  
  
```html  
<div id="cible" class="upload" style="height: 300px;">  
    Cliquez ou glissez-déposez vos images dans ce cadre pour les ajouter à la galerie.  
    <br>  
    Formats acceptés : JPG, JPEG, PNG, WEBP ou AVIF  
    <br>  
    Taille maximale : 350 Ko  
    <br>  
    L'image sera redimensionnée afin de ne pas dépasser 350 px en largeur  
</div>  
<input type="file" id="fichier" accept=".jpg,.jpeg,.png,.webp,.avif" style='display:none'>
```  

Le glisser déposer dans ce cas simple se gère  uniquement à l'aide de deux gestionnaires :
+ ondragover : qui désactive le traitement par défaut (si l'on fait glisser un fichier dans une fenêtre du navigateur, ce dernier ouvre par défaut un nouvel onglet permettant de visualiser le fichier)
+ ondrop : qui se déclenche quand le fichier est déposé dans la zone
  
Exemple :  
  
```javascript  
nomFichier.ondragover = (e) => {    
    e.preventDefault();  
};  
  
nomFichier.ondrop = (e) => {    
    e.preventDefault();  
    controlerFichier(e.dataTransfer.files[0]);  
};  
```  
  
## 5.7 Envoi du fichier avec `FormData`  
  
`FormData` permet de construire facilement une requête `multipart/form-data`, adaptée à l'envoi de fichiers et d'autres données de formulaire.
  
Création :  
  
```javascript  
const formData = new FormData();  
```  
  
Ajout du fichier :  
  
```javascript  
formData.append('fichier', leFichier);  
```  
  
Le nom utilisé dans `append()` doit correspondre au nom attendu par PHP :  
  
```php  
$_FILES['fichier']  
```  

  Le fichier est alors envoyé côte serveur accompagné éventuellement d'autres données à l'aide de la fonction appelAjax
  
```javascript  
function ajouter() {  
    const formData = new FormData();    
    formData.append( 'fichier', leFichier);    
    appelAjax({
            url: 'ajax/ajouter.php',
            data: formData,
            success: (data) => afficher(data)
    });
}  
```  
  
  


# 6. Bonnes pratiques générales  
  
Pour sécuriser un téléversement de fichier :  
  
✅ Toujours contrôler côté serveur.  
✅ Ne jamais utiliser uniquement `accept`.  
✅ Ne jamais faire confiance à `$_FILES['type']`.  
✅ Vérifier l'extension et le véritable type MIME.  
✅ Limiter la taille maximale.  
✅ Stocker les fichiers hors du répertoire public si possible.  
✅ Générer des noms uniques.  
✅ Éviter l'exécution de fichiers déposés.  
✅ Contrôler la structure des fichiers importés (CSV, Excel, XML...).  
  
---  
  
# Conclusion  
  
Un téléversement sécurisé repose sur une séparation claire des responsabilités :  
  
| Partie        | Rôle                                       |  
| ------------- | ------------------------------------------ |  
| HTML          | Sélection du fichier                       |  
| JavaScript    | Confort utilisateur et contrôles rapides   |  
| PHP           | Validation de sécurité et stockage         |  
| Classe métier | Réutilisation et centralisation des règles |  
### Niveau 1 — Interface

```text
accept
```

But :

> aider l'utilisateur à sélectionner le bon fichier.

**Aucune sécurité.**

### Niveau 2 — JavaScript

```text
file.size
file.name
file.type
dimensions de l'image
décodage de l'image
```

But :

> améliorer l'expérience utilisateur et éviter des transferts inutiles.

**Aucune sécurité.**

### Niveau 3 — PHP

```text
$_FILES['fichier']['error']
taille
extension
FileInfo
structure du fichier
nom de stockage
emplacement
```

But :

> **validation réelle et sécurité.**


La mise en place des classes techniques suivantes permettent de factoriser les contrôles, de centraliser les règles de téléversement et d'obtenir un système réutilisable et facilement configurable.

La gestion est répartie entre plusieurs classes, chacune avec une responsabilité distincte :

|Classe|Responsabilité|
|---|---|
|`Input`|Fournir la base commune aux objets de saisie : valeur, caractère obligatoire et message de validation.|
|`InputFile`|Représenter et valider un fichier téléversé : erreur d'envoi, taille, extension, type MIME et nom.|
|`InputFileImg`|Spécialiser `InputFile` pour contrôler qu'il s'agit d'une image et, si demandé, ses dimensions.|
|`FileManager`|Fournir les opérations génériques de stockage des fichiers : vérifier un nom, ajouter, remplacer, supprimer et vérifier l'existence d'un fichier.|
|`ImageManager`|Spécialiser `FileManager` pour les images : formats et types MIME autorisés, taille maximale, traitement du nom et redimensionnement éventuel.|
|`FileManagerFactory`|Construire et configurer le gestionnaire adapté au besoin, sans répéter cette configuration dans les services.|
|`ServiceEtudiant`|Coordonner le gestionnaire de fichiers avec les règles concernant l'étudiant et la référence de la photo en base de données.|

Les classes de saisie forment une chaîne d'héritage distincte de celle des gestionnaires :

## `Input`, `InputFile` et `InputFileImg` : représenter et valider la saisie

Ces classes ne stockent pas elles-mêmes le fichier dans le répertoire final. Elles représentent la saisie reçue et vérifient qu'elle peut être traitée avant que le gestionnaire ne la copie.

### `Input`

`src/ClasseTechnique/Input.php` est la classe abstraite de base des objets de saisie. Elle porte les propriétés et comportements partagés, notamment la valeur, l'indication « obligatoire » et le message d'erreur de validation.

### `InputFile`

`src/ClasseTechnique/InputFile.php` hérite de `Input` et représente un fichier téléversé par PHP. Son contrôle `checkValidity()` vérifie le bon déroulement du téléversement, la taille maximale, l'extension, le type MIME et la préparation du nom du fichier. Les extensions, types MIME et limites sont fournis par le gestionnaire qui l'utilise.

### `InputFileImg`

`src/ClasseTechnique/InputFileImg.php` hérite de `InputFile`. En plus des contrôles de fichier, il vérifie que le fichier temporaire contient une image valide. Il peut aussi vérifier des dimensions minimales ou maximales si elles sont définies.

Dans le parcours photo étudiant, `ServiceEtudiant::enregistrerPhoto()` construit un `InputFileImg` à partir du fichier reçu. L'objet est ensuite transmis à `ImageManager`, qui lui applique les règles configurées avant l'ajout ou le remplacement.

## `FileManager` : les opérations génériques

`src/ClasseTechnique/FileManager.php` est une classe abstraite. Elle factorise les opérations communes à différents types de fichiers et laisse aux classes filles le soin de définir les extensions et types MIME autorisés.

Elle vérifie notamment les noms de fichier afin qu'ils ne contiennent pas de chemin, puis fournit les opérations suivantes :

- `ajouter()` valide le fichier reçu et le copie dans le répertoire de stockage ;
- `remplacer()` valide le nouveau fichier et remplace le fichier existant en conservant son nom ;
- `supprimer()` retire le fichier du stockage ;
- `existe()` vérifie la présence d'un fichier autorisé ;
- `enregistrer()` choisit entre ajout et remplacement selon le nom existant fourni.

La validation du téléversement s'appuie sur `InputFile` (code d'erreur PHP, taille, extension, type MIME et nom). Quand l'objet fourni est un `InputFileImg`, sa validation spécifique vérifie également la validité de l'image et ses dimensions éventuelles.

## `ImageManager` : la spécialisation pour les images

`src/ClasseTechnique/ImageManager.php` hérite de `FileManager`. Il déclare les extensions et les types MIME admis pour les images, puis configure l'objet `InputFile` avec la taille maximale et le traitement du nom.

Il redéfinit la copie du fichier pour pouvoir, si cette option est activée, redimensionner une image avant de l'enregistrer. Les opérations générales (contrôle du nom, ajout, remplacement ou suppression) restent celles héritées de `FileManager`.

Cette séparation évite de mélanger les règles propres aux images avec les opérations de stockage communes à tout type de fichier.