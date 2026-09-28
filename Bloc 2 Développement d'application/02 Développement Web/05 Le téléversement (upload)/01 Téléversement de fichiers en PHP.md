 
## 1. Principe général  
  
Le téléversement de fichiers (*upload*) permet à un utilisateur d'envoyer un fichier depuis son poste client vers un serveur web.  
  
En HTML, l'envoi d'un fichier repose sur l'utilisation d'un champ `<input>` de type `file`.  
  
Le processus général est le suivant :  
  
1. L'utilisateur sélectionne un fichier dans son navigateur.  
2. Le navigateur transmet le fichier au serveur.  
3. PHP place temporairement le fichier sur le serveur (dossier c:/wamps.  
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
<button  
    class="btn btn-danger w-100"    id="btnAjouter">    Ajouter un fichier PDF de 2 Mo maximum</button>  
  
<input  
    type="file"    id="fichier"    accept=".pdf"    style="display:none">  
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
  
| Clé        | Description                                 |     |
| ---------- | ------------------------------------------- | --- |
| `tmp_name` | Chemin du fichier temporaire sur le serveur |     |
| `name`     | Nom original du fichier côté client         |     |
| `size`     | Taille du fichier en octets                 |     |
| `type`     | Type MIME transmis par le navigateur        |     |
| `error`    | Code d'erreur du transfert                  |     |
# 5. Les erreurs de téléversement  
  
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
  
# 6. Contrôles de sécurité côté serveur  
  
Avant toute utilisation d'un fichier reçu, plusieurs contrôles doivent être effectués :  
  
- présence d'un fichier ;  
- absence d'erreur de transfert ;  
- taille maximale ;  
- extension autorisée ;  
- véritable type MIME ;  
- nom du fichier ;  
- emplacement de stockage.  
  
## 6.1 Vérifier qu'un fichier existe  
  
```php  
if (!isset($_FILES['fichier'])) {  
    echo json_encode(['error' => "Aucun fichier transmis"]);    exit;}  
```  
  
## 6.2 Vérifier les erreurs de transfert  
  
```php  
if ($_FILES['fichier']['error'] !== UPLOAD_ERR_OK) {  
    echo json_encode(['error' => "Erreur lors du transfert"]);    exit;}  
```  
  
## 6.3 Récupérer les informations du fichier  
  
```php  
$tmp = $_FILES['fichier']['tmp_name'];  
  
$nomFichier = $_FILES['fichier']['name'];  
  
$taille = $_FILES['fichier']['size'];  
```  
  
# 7. Vérification de la taille du fichier  
  
La taille maximale autorisée doit être définie en fonction des besoins de l'application.  
  
Exemple : limitation à 2 Mo.  
  
```php  
$tailleMax = 2 * 1024 * 1024;  
  
if ($taille > $tailleMax) {  
    echo json_encode(['error' => "La taille du fichier dépasse la taille autorisée"]);    exit;}  
```  
  
La taille est exprimée en octets :  
  
| Taille | Valeur             |
| ------ | :----------------- |
| 1 Ko   | 1024 octets        |
| 1 Mo   | 1024 × 1024 octets |
  
# 8. Vérification de l'extension  
  
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
  
# 9. Vérification du type MIME  
  
Le type MIME indique le format supposé du fichier.  
  
Exemples :  
  
| Extension | Type MIME         |  
| --------- | ----------------- |  
| `.pdf`    | `application/pdf` |  
| `.jpg`    | `image/jpeg`      |  
| `.png`    | `image/png`       |  
| `.txt`    | `text/plain`      |  
  
## 9.1 Limite de la valeur fournie par le navigateur  
  
La valeur $_FILES['fichier']['type'] provient du client.  
  
Elle peut être falsifiée.  
  
Il ne faut donc pas l'utiliser pour effectuer un contrôle de sécurité.  
  
## 9.2 Utilisation de FileInfo  
  
PHP fournit l'extension **FileInfo** permettant d'analyser le contenu réel du fichier.  
  
Exemple :  
  
```php  
$finfo = finfo_open(FILEINFO_MIME_TYPE);  
$type = finfo_file($finfo, $tmp);  
finfo_close($finfo);  
```  
  
Le type MIME obtenu est ensuite comparé aux valeurs autorisées :  
  
```php  
$typesAutorises = ["application/pdf"];  
if (!in_array($type, $typesAutorises)) {  
    echo json_encode(['error' => "Type de fichier non accepté"]);    exit;}  
```  
  
# 10. Les principaux types MIME  
  
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
  
# 11. Notion de signature de fichier (*magic number*)  
  
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
  
# 12. Copie du fichier sur le serveur  
  
Lorsque tous les contrôles sont validés, le fichier temporaire peut être déplacé vers son emplacement définitif.  
  
Exemple :  
  
```php  
copy($tmp, '../document/' . $nomFichier);  
```  
  
## 12.1 Les différentes méthodes de déplacement  
  
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
  
Elle est généralement recommandée lorsque le stockage est effectué sur le même système de fichiers.  
  
> Le fichier temporaire est supprimé automatiquement à la fin de l'exécution du script.  
  
# 13. Gestion des doublons  
  
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
  
# 14. Classe `InputFile`  
  
Afin d'éviter de reproduire les mêmes contrôles dans chaque application, une classe dédiée peut encapsuler toute la logique de téléversement.  
  
La classe `InputFile` prend en charge :  
  
- la récupération du fichier envoyé ;  
- les contrôles de validité ;  
- la vérification de taille ;  
- la vérification d'extension ;  
- la vérification MIME ;  
- la copie du fichier.  
  
# 15. Configuration d'un téléversement  
  
Le comportement du téléversement est défini par un tableau de configuration.  
  
Exemple :  
  
```php  
[  
    'repertoire' => '/data/document',  
    'extensions' => [        'pdf'    ],  
    'types' => [        'application/pdf'    ],  
    'maxSize' => 1024 * 1024,  
    'require' => true,  
    'rename' => false,  
    'sansAccent' => false,  
    'accept' => '.pdf',  
    'label' => 'Fichier PDF (1 Mo maximum)']  
```  
  
## Signification des paramètres  
  
| Paramètre    | Description                         |  
| ------------ | ----------------------------------- |  
| `repertoire` | Répertoire de stockage              |  
| `extensions` | Extensions autorisées               |  
| `types`      | Types MIME autorisés                |  
| `maxSize`    | Taille maximale en octets           |  
| `require`    | Fichier obligatoire ou facultatif   |  
| `rename`     | Renommer automatiquement le fichier |  
| `sansAccent` | Suppression des accents dans le nom |  
| `accept`     | Valeur utilisée dans le champ HTML  |  
| `label`      | Libellé associé au champ            |  
  
# 16. Utilisation de la classe `InputFile`  
  
Exemple :  
  
```php  
$file = new InputFile($configuration);  
```  
  
La validation est réalisée avec :  
  
```php  
if ($file->checkValidity()) {  
    $file->copy();}  
```  
  
En cas d'erreur :  
  
```php  
$message = $file->getValidationMessage();  
```  
  
  
> Important : le champ HTML doit obligatoirement utiliser le nom `fichier`.  
  
Exemple :  
  
```html  
<input type="file" name="fichier">  
```  
  
Le fichier sera alors accessible avec :  
  
```php  
$_FILES['fichier']  
```  
  
# 17. Exemple d'une classe spécialisée  
  
Une classe spécifique peut encapsuler la configuration d'un type de fichier.  
  
Exemple : gestion des fichiers PDF.  
  
```php  
class FichierPDF  
{  
  
    private const CONFIG = [  
        'repertoire' => '/data/pdf',  
        'extensions' => [            'pdf'        ],  
        'types' => [            'application/pdf'        ],  
        'maxSize' => 512 * 1024,  
        'require' => true,  
        'rename' => false,  
        'sansAccent' => true,  
        'accept' => '.pdf',  
        'label' =>            'Fichier PDF à téléverser (512 Ko max)'    ];  
  
    public static function ajouter(): array    {  
        $file = new InputFile(            self::CONFIG        );  
  
        if ($file->checkValidity()) {  
            if ($file->copy()) {  
                return [                    'success' => true,                    'message' =>                        "Le fichier a été téléversé avec succès"                ];  
            }  
            return [                'success' => false,                'message' =>                    "Le fichier n'a pas pu être téléversé"            ];        }  
  
        return [            'success' => false,            'message' =>                $file->getValidationMessage()        ];    }}  
```  
  
# 18. Traitements côté client avec JavaScript  
  
Avant l'envoi au serveur, il est possible d'effectuer des contrôles côté client.  
  
Ces contrôles permettent :  
  
- d'améliorer l'expérience utilisateur ;  
- d'éviter un envoi inutile au serveur ;  
- d'afficher rapidement un message d'erreur.  
  
Cependant, ils ne remplacent jamais les contrôles côté serveur.  
  
> ⚠️ Le code JavaScript exécuté dans le navigateur peut être modifié par l'utilisateur.  
> La validation finale doit toujours être réalisée en PHP.  
  
# 19. L'objet `File`  
  
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
  
## 19.1 L'objet `FileList`  
  
`FileList` est un tableau contenant les fichiers sélectionnés.  
  
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
  
## 19.2 Propriétés d'un objet `File`  
  
Un objet `File` possède notamment :  
  
| Propriété | Description                         |  
| --------- | ----------------------------------- |  
| `name`    | Nom du fichier avec son extension   |  
| `size`    | Taille du fichier en octets         |  
| `type`    | Type MIME déclaré par le navigateur |  
  
Exemple :  
  
```javascript  
let file = fichier.files[0];  
  
console.log(file.name);  
console.log(file.size);  
console.log(file.type);  
```  
  
# 20. Stockage du fichier sélectionné  
  
Une variable globale peut conserver le fichier validé avant son envoi.  
  
```javascript  
let leFichier = null;  
```  
  
Elle contiendra l'objet `File` uniquement après validation.  
  
# 21. Paramètres transmis au JavaScript  
  
Les paramètres de téléversement peuvent être transmis depuis PHP.  
  
Exemple :  
  
```php  
$lesParametres = json_encode(  
    FichierPDF::getConfig(),    JSON_UNESCAPED_SLASHES |    JSON_UNESCAPED_UNICODE);  
  
  
echo <<<HTML  
  
<script>  
  
const lesParametres = $lesParametres;  
  
</script>  
  
HTML;  
```  
  
Le JavaScript dispose alors d'un objet contenant :  
  
```javascript  
{  
    extensions: ["pdf"],    maxSize: 524288,    accept: ".pdf"}  
```  
  
# 22. Initialisation du champ fichier  
  
L'attribut `accept` peut être alimenté automatiquement :  
  
```javascript  
const fichier = document.getElementById('fichier');  
  
  
fichier.accept =  lesParametres.accept;  
```  
  
Cela évite de dupliquer les paramètres entre PHP et HTML.  
  
# 23. Contrôle du fichier côté client  
  
Une fonction de contrôle peut vérifier :  
  
- l'extension ;  
- la taille maximale ;  
- éventuellement le type MIME.  
  
Exemple :  
  
```javascript  
function controlerFichier(file) {  
    effacerLesErreurs();    if (fichierValide(file, lesParametres)) {        nomFichier.value = file.name;        leFichier = file;    } else {        leFichier = null;        nomFichier.value = '';    }}  
```  
  
# 24. Sélection d'un fichier  
  
Lorsque l'utilisateur sélectionne un fichier :  
  
```javascript  
fichier.onchange = () => {  
    if (fichier.files.length > 0) {        controlerFichier(fichier.files[0]);    }  
};  
```  
  
# 25. Déclencher l'ouverture du sélecteur  
  
Si le champ `file` est masqué, un bouton peut déclencher son ouverture.  
  
```javascript  
btnFichier.onclick = () => { fichier.click();};  
```  
  
# 26. Validation avant envoi  
  
Le bouton d'envoi vérifie qu'un fichier valide existe.  
  
```javascript  
btnAjouter.onclick = () => {  
    if (leFichier === null) {        afficherErreurSaisie('fichier', 'Veuillez sélectionner un fichier');    } else {        ajouter();    }};  
```  
  
# 27. Glisser-déposer d'un fichier  
  
Il est possible d'utiliser une zone graphique comme zone de dépôt.  
  
Exemple :  
  
```javascript  
nomFichier.ondragover = (e) => {    e.preventDefault();  
};  
  
nomFichier.ondrop = (e) => {    e.preventDefault();  
    controlerFichier(e.dataTransfer.files[0]);  
};  
```  
  
# 28. Envoi du fichier avec `FormData`  
  
Un objet `File` ne peut pas être envoyé directement avec une requête HTTP classique.  
  
Il faut utiliser un objet `FormData`.  
  
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
  
# 29. Fonction d'envoi Ajax  
  
Exemple :  
  
```javascript  
function ajouter() {  
    const formData = new FormData();    formData.append( 'fichier', leFichier);    appelAjax({        url: 'ajax/ajouter.php',        data: formData,        success: (data) => {            afficher(data);        }    });}  
```  
  
# 30. Traitement spécifique des images  
  
Le téléversement d'image reprend les principes généraux :  
  
- contrôle de présence ;  
- contrôle de taille ;  
- contrôle d'extension ;  
- contrôle MIME ;  
- déplacement du fichier.  
  
Cependant, une image permet des contrôles supplémentaires :  
  
- largeur ;  
- hauteur ;  
- format ;  
- redimensionnement.  
  
# 31. Interface de sélection d'image  
  
Une interface courante consiste à créer une zone :  
  
- cliquable ;  
- compatible glisser-déposer ;  
- permettant une prévisualisation.  
  
Exemple :  
  
```html  
<div  
    id="cible"    class="upload"    style="width:300px;height:300px">  
  
    Cliquez ou glissez-déposez une image.  
    <div id="label"></div>  
</div>  
  
  
<input  
    type="file"    id="fichier"    style="display:none">  
```  
  
# 32. Classe `InputFileImg`  
  
Pour gérer les images, une classe spécialisée peut dériver de `InputFile`.  
  
Exemple :  
  
```php  
class InputFileImg extends InputFile  
{  
  
    private int $width;  
    private int $height;  
    private bool $redimensionner;  
}  
```  
  
Elle ajoute notamment :  
  
| Paramètre | Rôle |  
|-|-|  
| `width` | Largeur maximale |  
| `height` | Hauteur maximale |  
| `redimensionner` | Active le redimensionnement |  
  
# 33. Configuration d'un téléversement d'image  
  
Exemple :  
  
```php  
[  
    'repertoire' => '/data/image',  
    'extensions' => [        'jpg',        'png',        'webp',        'avif'    ],  
    'types' => [        'image/jpeg',        'image/png',        'image/webp',        'image/avif'    ],  
    'maxSize' => 150 * 1024,  
    'require' => true,  
    'rename' => true,  
    'sansAccent' => true,  
    'redimensionner' => true,  
    'width' => 350,  
    'height' => 150]  
```  
  
# 34. Vérification des dimensions d'une image  
  
Si aucun redimensionnement n'est prévu, les dimensions peuvent être contrôlées côté client.  
  
Exemple :  
  
```javascript  
verifierDimensionsImage(  
    file,    lesParametres,    () => ajouter(file));  
```  
  
# 35. Contrôle complet d'une image côté client  
  
Exemple :  
  
```javascript  
function controlerFichier(file) {  
    effacerLesErreurs();    if (!fichierValide(file, lesParametres)) {        return;    }  
    if (lesParametres.redimensionner) {        ajouter(file);    } else {        verifierDimensionsImage(file, lesParametres,   () => ajouter(file));    }}  
```  
  
# 36. Redimensionnement serveur  
  
Le redimensionnement doit idéalement être réalisé côté serveur.  
  
Une bibliothèque PHP peut être utilisée.  
  
Installation avec Composer :  
  
```bash  
composer require gumlet/image-resize```  
  
Exemple de traitement :  
  
```php  
$image = new ImageResize($fichier);  
  
$image->resizeToWidth(350);  
  
$image->save($destination);  
```  
  
# 37. Bonnes pratiques générales  
  
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
  
La mise en place d'une classe comme `InputFile` permet de factoriser les contrôles et d'obtenir un système de téléversement fiable, réutilisable et facilement configurable.