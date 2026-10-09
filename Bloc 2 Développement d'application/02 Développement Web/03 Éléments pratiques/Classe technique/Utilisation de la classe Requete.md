# Présentation

La classe `Requete` est le point d'accès unique aux informations transmises par le navigateur.

Elle permet aux scripts PHP de :

- connaître le type de requête HTTP reçue ;
- vérifier que la requête utilise la méthode attendue ;
- contrôler qu'un appel provient d'Ajax ;
- récupérer les paramètres envoyés par le navigateur ;
- convertir automatiquement les valeurs dans le type attendu.

# Principe général

La classe `Requete` poursuit un objectif simple :

- ne plus accéder directement à `$_GET`, `$_POST`, `$_SERVER` ou `php://input`;
- centraliser toutes les vérifications liées aux requêtes HTTP ;
- fournir directement des valeurs déjà contrôlées et converties.

Ainsi, les contrôleurs peuvent se concentrer uniquement sur leur traitement métier.

# Utilisation générale

Avant toute utilisation, les classes doivent être chargées :

```php
require $_SERVER['DOCUMENT_ROOT'] . '/../bootstrap/autoload.php';
```

Ensuite la classe peut être utilisée directement :

```php
use ClasseTechnique\Requete;
```

# Vérification de la méthode HTTP

Chaque script doit connaître le type de requête qu'il attend.

## Vérifier une requête POST

Exemple pour un traitement Ajax d'ajout ou de modification :

```php
Requete::verifierPost();
```

La requête doit obligatoirement être envoyée en POST.

Cette méthode évite de répéter dans chaque contrôleur le code suivant :

```php
if ($_SERVER['REQUEST_METHOD'] !== 'POST') {  
    throw new Exception("Méthode POST obligatoire");  
}
```
## Vérifier une requête GET

Exemple pour une recherche ou une récupération d'information :

```php
Requete::verifierGet();
```

## Vérification générique

Il est possible de vérifier n'importe quelle méthode autorisée :

```php
Requete::verifierMethode('POST');
```

Méthodes reconnues : GET, POST, PUT, PATCH, DELETE
    
# Récupérer des paramètres POST

Les requêtes Ajax de l'application utilisent principalement POST.

Exemple JSON envoyé :

```json
{
    "tableName": "Coureur",
    "columns": {
        "nom": "DUPONT"
    }
}
```

Lecture :

```php
$tableName = Requete::postString('tableName');

$columns = Requete::postArray('columns');
```

Les méthodes typées POST disponibles vérifient :

- que le paramètre demandé existe ;
- que sa valeur correspond au type attendu.

Le traitement suit techniquement la chaîne suivante :

```
Requete::postString('tableName')
        ↓
post('tableName')
        ↓
postData()
        |
        → Content-Type classique  → Lecture $_POST
        |
        → ou application/json     → lecture JSON php://input → décodage JSON
        ↓
readValue(postData(), 'tableName')
        |
        → absent ? → UserException
        |
        → présent
            ↓
        asString($valeur,'tableName')
                 |
                 → pas une string ? → UserException
                 |
                 → string
                     ↓
                   retour
```

Cette décomposition montre que chaque méthode possède une responsabilité unique :

|Méthode|Responsabilité|
|---|---|
|`postString()`|Exprimer le besoin de l'appelant (« je veux une chaîne »).|
|`postData()`|Unifier les différentes sources de données HTTP.|
|`post()`|Extraire la valeur brute d'un paramètre.|
|`readValue()`|Vérifier que le paramètre existe.|
|`asString()`|Vérifier le type demandé.|

# Exemple complet d'un script Ajax

```php
require $_SERVER['DOCUMENT_ROOT'] . '/../bootstrap/autoload.php';

use ClasseTechnique\Requete;
use ClasseTechnique\Jeton;
use ClasseTechnique\ReponseJson;

// Vérifier la méthode HTTP
Requete::verifierPost();

// Vérifier le token CSRF
Jeton::verifier();

// récupérer la valeur des paramètres transmis en vérifiant leur présence et leur type
$tableName = Requete::postString('tableName');
$columns = Requete::postArray('columns');

// traitement métier
```

## Méthodes POST  typées disponibles

Les méthodes typées disponibles permettent de récupérer un paramètre HTTP en vérifiant sa présence et son type.

### Texte

```php
$nom = Requete::postString('nom');
```
### Entier

```php
$id = Requete::postInt('id');
```
### Nombre

```php
$taille = Requete::postFloat('taille');
```
### Booléen

```php
$visible = Requete::postBool('visible');
```
### Tableau

```php
$options = Requete::postArray('options');
```
### Date

```php
$date = Requete::postDate('date');
```
### Email

```php
$email = Requete::postEmail('email');
```
### URL

```php
$url = Requete::postUrl('url');
```
### Valeur simple

```php
$id = Requete::postScalar('id');
```

# Vérifier plusieurs paramètres

Avant un traitement, il est possible de vérifier uniquement la présence des paramètres nécessaires.

Exemple :

```php
Requete::verifierParametresPost('tableName', 'columns');
```

Pour GET :

```php
Requete::verifierParametresGet('id', 'nom');
```

# Paramètres pouvant être absents

Par défaut, un paramètre absent provoque une erreur.

Exemple :

```php
$nom = Requete::postString('nom');
```

Si `nom` n'existe pas dans la requête, une exception est générée.


Pour les valeurs facultatives, utiliser les méthodes `Nullable` : 
```php
postNullableString()
postNullableInt()
postNullableFloat()
postNullableBool()
postNullableDate()
```

Exemple :

```php
$prenom = Requete::postNullableString('prenom');
```

Résultats possibles : "Jean" ou null


# Récupérer des paramètres GET

Les paramètres GET sont récupérés avec les méthodes `get`.

Exemple :

URL :  recherche.php?nom=DUPONT

Code :

```php
$nom = Requete::getString('nom');
```

## Méthodes GET disponibles

Les méthodes GET suivent exactement le même principe que les méthodes POST.

```php
$nom = Requete::getString('nom');
$id = Requete::getInt('id');
$prix = Requete::getFloat('prix');
$actif = Requete::getBool('actif');
$liste = Requete::getArray('liste');
$date = Requete::getDate('date');
$email = Requete::getEmail('email');
$url = Requete::getUrl('url');
$cle = Requete::getScalar('id');
```
# Règles d'utilisation recommandées

## À faire

Utiliser :

```php
Requete::postString()
Requete::postInt()
Requete::getString()
Requete::getInt()
```

pour obtenir directement une donnée contrôlée.

## À éviter

Au lieu de

```
$id = $_POST['id'];
```

préférer

```
$id = Requete::postInt('id');
```

et au lieu de

```
$nom = $_GET['nom'];
```

préférer

```
$nom = Requete::getString('nom');
```

La classe `Requete` doit rester l'unique intermédiaire entre le navigateur et l'application.

# Connaître la méthode utilisée

Ces méthodes retournent simplement `true` ou `false`.

Exemples :

```php
if (Requete::estPost()) {
    // traitement POST
}
```

Méthodes disponibles :

```php
Requete::estGet();

Requete::estPost();

Requete::estPut();

Requete::estPatch();

Requete::estDelete();
```

# Vérifier un appel Ajax

Un script destiné uniquement à Ajax peut contrôler son origine :

```php
Requete::verifierAjax();
```

Pour tester simplement :

```php
if (Requete::estAjax()) {
    // appel Ajax
}
```

Rappel : Dans nos applications, la protection de ces scripts qui se trouvent toujours dans un répertoire ajax est assurée directement depuis le serveur Apache à partir du fichier .htaccess stockée dans le répertoire public

```apache
# Si l'URI contient "/ajax/" et que l'en-tête X-Requested-With n'est pas égal à XMLHttpRequest, alors refuser l'accès (403 Forbidden)  
RewriteCond %{REQUEST_URI} /ajax/  
RewriteCond %{HTTP:X-Requested-With} !^XMLHttpRequest$ [NC]  
RewriteRule ^ - [F,L]
```

# Résumé rapide

|Besoin|Méthode|
|---|---|
|Vérifier POST|`Requete::verifierPost()`|
|Vérifier GET|`Requete::verifierGet()`|
|Savoir si Ajax|`Requete::estAjax()`|
|Lire texte POST|`Requete::postString()`|
|Lire entier POST|`Requete::postInt()`|
|Lire tableau POST|`Requete::postArray()`|
|Lire texte GET|`Requete::getString()`|
|Lire entier GET|`Requete::getInt()`|
|Paramètre obligatoire|méthode classique|
|Paramètre facultatif|méthode `Nullable`|

La classe `Requete` constitue l'unique point d'entrée des données HTTP dans l'application. 
En centralisant leur lecture, leur validation et leur conversion, elle permet aux contrôleurs de manipuler directement des données fiables et typées.


Remarque : Cette classe n'est pas compatible avec PSR-7 afin de conserver ses avantages :
- la simplicité ;
- le typage automatique ;
- les messages d'erreur homogènes.