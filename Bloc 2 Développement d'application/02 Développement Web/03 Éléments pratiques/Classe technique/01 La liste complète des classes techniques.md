
| Classe             | Objectif général                                                                                                                      |     |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------- | --- |
| `Column`           | Classe de base pour valider et normaliser une valeur de colonne métier.                                                               |     |
| `ColumnBool`       | Valider une valeur booléenne provenant du formulaire ou du code.                                                                      |     |
| `ColumnDate`       | Valider une date SQL au format `AAAA-MM-JJ`.                                                                                          |     |
| `ColumnEmail`      | Valider une adresse e-mail avec contrôle DNS optionnel.                                                                               |     |
| `ColumnInt`        | Valider un entier avec bornes optionnelles.                                                                                           |     |
| `ColumnList`       | Valider une valeur appartenant à une liste autorisée.                                                                                 |     |
| `ColumnText`       | Valider et normaliser un texte simple.                                                                                                |     |
| `ColumnTextarea`   | Valider un texte multiligne avec gestion HTML.                                                                                        |     |
| `ColumnUrl`        | Valider une URL avec contrôle d'accessibilité optionnel.                                                                              |     |
| `Config`           | Charger les fichiers de configuration PHP ou JSON.                                                                                    |     |
| `ContrainteUnique` | Définir une règle d'unicité pour une ou plusieurs colonnes.                                                                           |     |
| `Database`         | Fournir l'unique connexion PDO de l'application.                                                                                      |     |
| `Erreur`           | Installer le gestionnaire global d'exceptions applicatives et SQL.                                                                    |     |
| `FileManager`      | Gérer le stockage, le remplacement et la suppression de fichiers.                                                                     |     |
| `ImageManager`     | Gérer le stockage de fichiers image, avec redimensionnement possible.                                                                 |     |
| `Input`            | Classe de base pour contrôler une valeur saisie.                                                                                      |     |
| `InputFile`        | Valider un fichier téléversé.                                                                                                         |     |
| `InputFileImg`     | Valider un fichier image téléversé avec contraintes de dimensions.                                                                    |     |
| `InterfaceHtml`    | Générer les fragments HTML standard d'une page. Utilisée uniquement par le script viw/interfaceHTMl qui construit tous les pages HTML |     |
| `InterfaceSystem`  | Afficher une page système hors cycle HTML standard. Concerne uniquement les pages d'erreur du dossier public/erreur                   |     |
| `Ip`               | Consulter et gérer les IP bloquées.                                                                                                   |     |
| `Jeton`            | Créer, vérifier et supprimer le jeton CSRF.                                                                                           |     |
| `Journal`          | Écrire et consulter des journaux applicatifs.                                                                                         |     |
| `Page`             | Décrire une page HTML à rendre.                                                                                                       |     |
| `PdfManager`       | Gérer le stockage de fichiers PDF.                                                                                                    |     |
| `ReponseJson`      | Produire des réponses JSON standardisées.                                                                                             |     |
| `Requete`          | Lire et typer les paramètres HTTP et les fichiers envoyés.                                                                            |     |
| `Select`           | Exécuter simplement des requêtes SQL de lecture.                                                                                      |     |
| `Std`              | Fournir des helpers techniques de validation et de conversion.                                                                        |     |
| `Table`            | Fournir le socle CRUD et la validation des classes métier.                                                                            |     |
| `UserException`    | Porter un message utilisateur avec code HTTP associé.                                                                                 |     |
  
## `Column`  
  
**Méthodes à utiliser**  
  
- `getValidationMessage()` : récupérer le message d'erreur après `checkValidity()`.  
- `checkValidity()` : valider la valeur portée par la colonne.  
  
```php  
$colonne = new ColumnText(required: true, maxLength: 50);  
$colonne->Value = 'Jean';  
  
if (!$colonne->checkValidity()) {  
    echo $colonne->getValidationMessage();}  
```  
  
## `ColumnBool`  
  
**Méthodes à utiliser**  
  
- `checkValidity()` : convertir et valider une valeur booléenne.  
  
```php  
$colonne = new ColumnBool();  
$colonne->Value = '1';  
$colonne->checkValidity();  
  
$actif = $colonne->Value; // true  
```  
  
## `ColumnDate`  
  
**Méthodes à utiliser**  
  
- `checkValidity()` : valider une date SQL, avec bornes si besoin.  
  
```php  
$colonne = new ColumnDate(min: '2025-01-01', max: '2026-12-31');  
$colonne->Value = '2026-09-03';  
  
if (!$colonne->checkValidity()) {  
    echo $colonne->getValidationMessage();}  
```  
  
## `ColumnEmail`  
  
**Méthodes à utiliser**  
  
- `checkValidity()` : valider une adresse e-mail.  
  
```php  
$colonne = new ColumnEmail(maxLength: 100, verifierDomaine: true);  
$colonne->Value = 'contact@example.com';  
  
if (!$colonne->checkValidity()) {  
    echo $colonne->getValidationMessage();}  
```  
  
## `ColumnInt`  
  
**Méthodes à utiliser**  
  
- `checkValidity()` : valider un entier et le convertir en `int`.  
  
```php  
$colonne = new ColumnInt(min: 1, max: 120);  
$colonne->Value = '42';  
$colonne->checkValidity();  
  
$age = $colonne->Value; // 42  
```  
  
## `ColumnList`  
  
**Méthodes à utiliser**  
  
- `getValues()` : récupérer la liste autorisée.  
- `checkValidity()` : vérifier l'appartenance à cette liste.  
  
```php  
$colonne = new ColumnList(values: ['draft', 'published', 'archived']);  
$colonne->Value = 'published';  
  
if ($colonne->checkValidity()) {  
    $statut = $colonne->Value;}  
```  
  
## `ColumnText`  
  
**Méthodes à utiliser**  
  
- `checkValidity()` : valider et normaliser un texte.  
  
```php  
$colonne = new ColumnText(  
    minLength: 2,    maxLength: 80,    supprimerEspaceSuperflu: true);  
$colonne->Value = '  Jean   Dupont  ';  
$colonne->checkValidity();  
  
$nom = $colonne->Value; // Jean Dupont  
```  
  
## `ColumnTextarea`  
  
**Méthodes à utiliser**  
  
- `checkValidity()` : valider et éventuellement filtrer le HTML.  
  
```php  
$colonne = new ColumnTextarea(acceptHtml: false);  
$colonne->Value = '<script>x</script><b>Texte</b>';  
$colonne->checkValidity();  
  
$contenu = $colonne->Value;  
```  
  
## `ColumnUrl`  
  
**Méthodes à utiliser**  
  
- `checkValidity()` : valider une URL.  
  
```php  
$colonne = new ColumnUrl(verifierExistence: true);  
$colonne->Value = 'https://example.com';  
  
if (!$colonne->checkValidity()) {  
    echo $colonne->getValidationMessage();}  
```  
  
## `Config`  
  
**Méthodes à utiliser**  
  
- `chargerPhp()` : charger une configuration PHP.  
- `chargerJson()` : charger une configuration JSON.  
  
```php  
$db = Config::chargerPhp('database');  
$menu = Config::chargerJson('menuvertical');  
```  
  
## `ContrainteUnique`  
  
**Méthodes à utiliser**  
  
- aucune méthode publique hors constructeur ; la classe sert à déclarer une contrainte.  
  
```php  
$contrainte = new ContrainteUnique(  
    ['email'],    "Cette adresse e-mail existe déjà.");  
```  
  
## `Database`  
  
**Méthodes à utiliser**  
  
- `getInstance()` : récupérer la connexion PDO.  
- `getLesParametres()` : lire la configuration DB courante.  
  
```php  
$db = Database::getInstance();  
$cmd = $db->query('select now()');  
```  
  
## `Erreur`  
  
**Méthodes à utiliser**  
  
- `definirLesContraintes()` : associer un nom de contrainte SQL à un message métier.  
- `installerGestionnaire()` : installer le handler global des exceptions.  
  
```php  
Erreur::definirLesContraintes([  
    'uq_utilisateur_email' => "Cette adresse e-mail existe déjà."]);  
  
Erreur::installerGestionnaire();  
```  
  
## `FileManager`  
  
**Méthodes à utiliser**  
  
- `getRepertoire()` : récupérer le répertoire physique.  
- `getLesFichiers()` : lister les fichiers autorisés.  
- `existe()` : vérifier la présence d'un fichier.  
- `ajouter()` : stocker un nouveau fichier.  
- `remplacer()` : remplacer un fichier existant.  
- `supprimer()` : supprimer un fichier.  
- `enregistrer()` : ajouter ou remplacer selon le contexte.  
  
```php  
$manager = new PdfManager(DOSSIER_RACINE . '/document');  
$fichier = new InputFile(Requete::getFile('pieceJointe'));  
  
if (!$manager->ajouter($fichier)) {  
    echo $fichier->getValidationMessage();}  
```  
  
## `ImageManager`  
  
**Méthodes à utiliser**  
  
- utiliser les méthodes héritées de `FileManager` : `ajouter()`, `remplacer()`, `supprimer()`, `getLesFichiers()`.  
  
```php  
$manager = new ImageManager(  
    DOSSIER_RACINE . '/image',    maxSize: 2_000_000,    redimensionner: true,    width: 1200);  
  
$image = new InputFileImg(Requete::getFile('photo'));  
$manager->ajouter($image, true);  
```  
  
## `Input`  
  
**Méthodes à utiliser**  
  
- `getValue()` / `setValue()` : lire ou définir la valeur.  
- `isRequired()` / `setRequired()` : gérer l'obligation.  
- `getValidationMessage()` / `setValidationMessage()` : gérer le message d'erreur.  
- `checkValidity()` : appliquer le contrôle de base.  

La classe Input est une classe abstraite qui donc ne peut pas être instanciée directement
`Input` représente un **concept générique de champ contrôlé**, et non un type de donnée concret.
Elle fournit des éléments communs aux classes dérivées
```
Input
  │
  └── InputFile
        │
        └── InputFileImg
```
  
## `InputFile`  
  
**Méthodes à utiliser**  
  
- `setLesExtensions()`, `setLesTypes()`, `setMaxSize()`, `setSansAccent()`, `setCasse()` : configurer la validation.  
- `getName()`, `getTmpName()`, `getSize()`, `getExtension()`, `getMimeType()` : lire les infos du fichier.  
- `isValide()` : savoir si la validation est passée.  
- `checkValidity()` : valider le fichier.  
  
```php  
$fichier = new InputFile(Requete::getFile('document'));  
$fichier->setLesExtensions(['pdf']);  
$fichier->setLesTypes(['application/pdf']);  
$fichier->setMaxSize(1_000_000);  
  
if (!$fichier->checkValidity()) {  
    echo $fichier->getValidationMessage();}  
```  
  
## `InputFileImg`  
  
**Méthodes à utiliser**  
  
- `setDimensions()` : définir les contraintes de taille.  
- `getDimensions()` : lire largeur et hauteur.  
- `checkValidity()` : héritée de `InputFile`.  
  
```php  
$image = new InputFileImg(Requete::getFile('photo'));  
$image->setLesExtensions(['jpg', 'png']);  
$image->setLesTypes(['image/jpeg', 'image/png']);  
$image->setDimensions(minWidth: 300, maxWidth: 2000);  
$image->checkValidity();  
```  
  
## `InterfaceHtml`  
  
**Méthodes à utiliser**  
  
- `head()` : produire les ressources du `<head>`.  
- `header()` : produire l'entête commun.  
- `contenu()` : produire le contenu de la page.  
- `footer()` : produire le pied de page.  
  
```php  
$interface = new InterfaceHtml($page);  
echo $interface->head();  
```  
  
## `InterfaceSystem`  
  
**Méthodes à utiliser**  
  
- `afficher()` : afficher immédiatement la page système.  
  
```php  
(new InterfaceSystem(  
    'Maintenance',    "Le service est temporairement indisponible.",    'maintenance'))->afficher();  
```  
  
## `Ip`  
  
**Méthodes à utiliser**  
  
- `getAll()` : récupérer l'historique des IP bloquées.  
- `getIp()` : obtenir l'IP courante.  
- `getBlocage()` : connaître le blocage actif d'une IP.  
- `bloquer()` : enregistrer un blocage.  
- `bloquerEtArreter()` : bloquer puis interrompre le traitement.  
  
```php  
$ip = Ip::getIp();  
  
if (Ip::getBlocage($ip) !== null) {  
    throw new UserException("Adresse IP temporairement bloquée.");}  
```  
  
## `Jeton`  
  
**Méthodes **  
  
- `creer()` : créer ou récupérer le jeton CSRF : utilisée par la méthode Requete::exisgerPost().  
- `verifier()` : vérifier le jeton reçu.  :  utilisée indirectement (via InterfaceHtml) par l'appel de la méthode $page->afficher() si elle est précédée par $page->avecJeton();   
- `supprimer()` : supprimer le jeton en session : pas actuellement utilisé et pourrait être supprimée.  
  
```php  
public static function exigerPost(): void  
{  
    self::exigerAjax();  
    self::exigerMethode('POST');  
    Jeton::verifier();  
}
```  
  
## `Journal`  
  
**Méthodes à utiliser**  
  
- `ecrire()` : écrire un texte brut dans un journal.  
- `enregistrer()` : écrire une ligne journalisée avec date, script et IP.  
- `getLesEvenements()` : lire un journal.  
- `supprimer()` : supprimer un journal.  
- `getListe()` : lister les journaux disponibles.  
  
```php  
Journal::enregistrer('Connexion réussie', 'auth');  
$evenements = Journal::getLesEvenements('auth');  
```  
  
## `Page`  
  
**Méthodes à utiliser**  
  
- `setTitre()` / `getTitre()` : gérer le titre.  
- `addScript()` / `getScripts()` : gérer les scripts.  
- `addStyle()` / `getStyles()` : gérer les styles.  
- `addComposant()` : ajouter un composant standard.  
- `setDonnee()` / `getDonnees()` : transmettre des données au JavaScript.  
- `avecJeton()` / `necessiteUnJeton()` : gérer le besoin CSRF.  
- `afficher()` : rendre la page finale.  
  
```php  
$page = new Page();

$page->setTitre('Liste des coureurs')
    ->addComposant('datatable')
    ->setDonnee('coureurs', $lesCoureurs)    
    ->avecJeton();  
	->afficher();  
```  
  
## `PdfManager`  
  
**Méthodes à utiliser**  
  
- utiliser les méthodes héritées de `FileManager` : `ajouter()`, `remplacer()`, `supprimer()`, `getLesFichiers()`.  
  
```php  
$manager = new PdfManager(DOSSIER_RACINE . '/pdf', maxSize: 5_000_000);  
$pdf = new InputFile(Requete::getFile('brochure'));  
$manager->enregistrer(null, $pdf);  
```  
  
## `ReponseJson`  
  
**Méthodes à utiliser**  
  
- `envoyer()` : réponse JSON générique.  
- `envoyerMessage()` : message simple de succès.  
- `envoyerLesDonnees()` : réponse de données.  
- `envoyerCreation()` : réponse HTTP 201.  
- `envoyerLesErreurs()` : réponse de validation.  
- `envoyerErreur()` : erreur générale.  
- `encoderPourHtml()` : encoder du JSON injecté dans une page HTML.  : utilisé uniquement en interne par la méthode donnees() de la classe InterfaceHtml 

```php  
private function donnees(): string  
{  
    $html = '';  
    foreach ($this->page->getDonnees() as $id => $valeur) {  
        $json = ReponseJson::encoderPourHtml($valeur);  
        $html .= sprintf('<script type="application/json" id="%s">%s</script>', $id, $json);  
    }  
    return $html;  
}
```  
  
## `Requete`  
  
**Méthodes à utiliser**  
  
- Contrôle de requête : `methode()`, `exigerMethode()`, `exigerPost()`, `exigerPostSansJeton()`, `exigerGet()`, `estAjax()`.  
- Lecture brute : `get()`, `post()`, `getFile()`.  
- Lecture typée POST : `postString()`, `postInt()`, `postFloat()`, `postBool()`, `postArray()`, `postDate()`, `postEmail()`, `postUrl()`, `postScalar()`, `postNullableString()`, `postNullableInt()`, `postNullableFloat()`, `postNullableBool()`, `postNullableDate()`.  
- Lecture typée GET : `getString()`, `getInt()`, `getFloat()`, `getBool()`, `getArray()`, `getDate()`, `getEmail()`, `getUrl()`, `getScalar()`, `getNullableString()`, `getNullableInt()`, `getNullableFloat()`, `getNullableBool()`, `getNullableDate()`.  
- Fichiers : `existeFichier()`, `fichierEnvoye()`.  
  
```php  
Requete::exigerPost();  
  
$id = Requete::postInt('id');  
$email = Requete::postEmail('email');  
$photo = Requete::fichierEnvoye('photo')  
    ? Requete::getFile('photo')  
    : null;  
```  
  
## `Select`  
  
**Méthodes à utiliser**  
  
- `getRows()` : récupérer plusieurs lignes.  
- `getRow()` : récupérer une ligne.  
- `getValue()` : récupérer une valeur.  
  
```php  
$select = new Select();  
$coureurs = $select->getRows(  
    'select id, nom from coureur where actif = :actif',    ['actif' => 1]);  
```  
  
## `Std`  
  
**Méthodes à utiliser**  
  
- Présence/chaînes : `existe()`, `supprimerEspace()`, `supprimerAccent()`.  
- Dates : `encoderDate()`, `decoderDate()`, `dateFrValide()`, `dateMysqlValide()`.  
- Web : `urlValide()`, `urlAccessible()`, `emailValide()`, `domaineEmailExiste()`.  
- Formats métier : `passwordValide()`, `codePostalValide()`, `mobileValide()`, `fixeValide()`, `tempsValide()`, `nomValide()`, `nomAvecAccentValide()`.  
- Nombres/fichiers : `nombreEntierValide()`, `nombreReelValide()`, `getLesFichiers()`.  
  
```php  
if (!Std::emailValide($email)) {  
    throw new UserException("Adresse e-mail invalide.");}  
  
$dateSql = Std::encoderDate('03/09/2026');  
```  
  
## `Table`  
  
**Méthodes à utiliser**  
  
- `getErrors()` : récupérer les erreurs métier ou de validation.  
- `getLastInsertId()` : récupérer le dernier identifiant créé.  
- `getPrimaryKeyName()` : récupérer le nom de la clé primaire.  
- `getColumn()` : accéder à un objet `Column` défini dans la classe métier.  
- `addServiceError()` : ajouter une erreur depuis un service.  
- `add()` : insérer un enregistrement.  
- `modify()` : modifier un enregistrement.  
- `delete()` : supprimer un enregistrement.  
  
```php  
$categorie = new Categorie();  
  
if (!$categorie->add(['libelle' => 'Junior'])) {  
    $erreurs = $categorie->getErrors();}  
```  
  
## `UserException`  
  
**Méthodes à utiliser**  
  
- `getCodeHttp()` : récupérer le code HTTP associé.  
  
```php  
throw new UserException("Accès refusé.", 403);  
```