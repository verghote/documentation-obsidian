## Présentation

La classe `Requete` centralise l'accès aux données transmises par une requête HTTP.

Elle constitue le point d'entrée unique vers les paramètres reçus par l'application, qu'ils proviennent :

- d'une requête GET ;
- d'un formulaire HTML (`application/x-www-form-urlencoded`) ;
- d'un formulaire `multipart/form-data` (upload de fichiers) ;
- d'une requête JSON (`application/json`).

Les autres classes de l'application ne doivent jamais accéder directement aux superglobales PHP (`$_GET`, `$_POST` ou `php://input`). Elles doivent utiliser exclusivement les méthodes de la classe `Requete`.

L'un des principaux intérêts de cette classe est que le contrôleur n'a pas à se préoccuper du mode de transmission des données.

Que le client envoie :

- un formulaire HTML ;
- une requête AJAX utilisant `FormData` ;
- une requête AJAX envoyant du JSON (y compris des tableaux structurés) ;

le code du contrôleur reste exactement le même :

```php
$nom = Requete::postString('nom');
$lesCompetences = Requete::postArray('lesCompetences');
$lesPieces = Requete::postJsonArray('pieces');
```
## Objectifs

La classe `Requete` poursuit plusieurs objectifs :

- centraliser l'accès aux paramètres HTTP ;
- masquer les différences entre formulaire HTML, `FormData` et JSON ;
- vérifier que la requête est conforme aux attentes ;
- convertir automatiquement les valeurs vers le type attendu (tableaux natifs ou décodés) ;
- produire des messages d'erreur homogènes ;
- simplifier les contrôleurs.

## Vérification de la requête

Avant de lire les paramètres, un contrôleur peut vérifier que la requête reçue est conforme grâce aux méthodes statiques de `Requete`.
### Vérifier la méthode HTTP et l'appel AJAX

- **Requête POST (Standard AJAX avec Jeton CSRF)** :
    
    ```php
    Requete::exigerPost();
    ```
    
    Cette méthode vérifie automatiquement :
    
    1. Que l'appel provient bien d'une requête AJAX (`exigerAjax()`).
    2. Que la méthode HTTP est bien POST (`exigerMethode('POST')`).
    3. Que le jeton CSRF est valide (`Jeton::verifier()`).

- **Requête POST sans vérification de jeton** :
   
    ```php
    Requete::exigerPostSansJeton();
    ```
    
- **Requête GET** :

    ```php
    Requete::exigerGet();
    ```
    
    Le contrôleur refuse toute requête autre qu'un GET.
    
- **Méthode HTTP personnalisée** :

    ```php
    Requete::exigerMethode('PUT');
    ```
    
    Permet d'imposer n'importe quelle méthode HTTP brute.
    

### Vérifier qu'il s'agit d'une requête AJAX seule


```php
Requete::exigerAjax();
```

Cette méthode vérifie la présence de l'en-tête `X-Requested-With: XMLHttpRequest`. Si la requête ne provient pas d'un appel AJAX, une réponse JSON d'erreur est immédiatement renvoyée.

### Tester sans provoquer d'erreur

Lorsque le contrôleur souhaite simplement connaître le type de requête :


```php
if (Requete::estAjax()) {
    // ...
}
```

La méthode retourne simplement `true` ou `false`.

## Principe général

Un contrôleur ne doit jamais accéder directement aux superglobales PHP.

- Au lieu d'écrire :
    
    
    ```php
    $id = $_GET['id'];
    $nom = $_POST['nom'];
    ```
    
- On utilisera :
    
    PHP
    
    ```php
    $id = Requete::getInt('id');
    $nom = Requete::postString('nom');
    ```
    

La classe se charge automatiquement :

- de vérifier la présence du paramètre ;
- de lire les données au bon endroit ;
- de gérer les requêtes JSON ;
- de convertir les valeurs vers le type attendu ;
- de produire un message d'erreur explicite.

## Gestion transparente des requêtes POST

### Formulaire HTML

Que le navigateur envoie `application/x-www-form-urlencoded` ou `multipart/form-data`, PHP remplit automatiquement `$_POST`.
### Requête JSON ou Données complexes

Lorsque le navigateur envoie `Content-Type: application/json` ou que le JavaScript sérialise un tableau complexe d'objets via `JSON.stringify()`, les données ne sont plus directement disponibles sous forme de tableau natif dans `$_POST`.

La classe `Requete` :

- lit automatiquement `php://input` ou les chaînes reçues ;
- décode le JSON via `json_decode(..., true)` ;
- transforme les structures JSON en tableaux PHP exploitables ;
- mémorise le résultat dans un cache interne pour éviter des lectures multiples du flux.

Le reste de l'application ne voit aucune différence.

## Méthodes de lecture

### Méthodes GET typées

|**Méthode**|**Type de retour**|
|---|---|
|`getString()`|`string`|
|`getInt()`|`int`|
|`getFloat()`|`float`|
|`getBool()`|`bool`|
|`getArray()`|`array`|
|`getDate()`|`string`|
|`getEmail()`|`string`|
|`getUrl()`|`string`|
|`getScalar()`|`string`|

### Méthodes POST typées

|**Méthode**|**Type de retour**|**Description**|
|---|---|---|
|`postString()`|`string`|Chaîne de caractères simple.|
|`postInt()`|`int`|Valeur entière.|
|`postFloat()`|`float`|Nombre décimal.|
|`postBool()`|`bool`|Booléen.|
|`postArray()`|`array`|Tableau HTML natif (champs multiples `name="items[]"`).|
|`postJsonArray()`|`array`|**Nouveau** : Décode automatiquement une chaîne JSON postée (ex: `JSON.stringify()`) en tableau PHP associatif.|
|`postDate()`|`string`|Date au format SQL (`AAAA-MM-JJ`).|
|`postEmail()`|`string`|Adresse email valide.|
|`postUrl()`|`string`|URL valide.|
|`postScalar()`|`string`|Valeur scalaire générique (idéal pour les clés primaires).|

_Exemple d'utilisation avec `postJsonArray` :_


```
// Récupération d'un tableau d'objets JSON transmis par le client (ex: pièces manquantes)
$lesPieces = Requete::postJsonArray('pieces');
```

### Valeurs acceptant `null`

Lorsque la colonne SQL accepte la valeur `NULL`, la classe propose des variantes dédiées :

- `postNullableString(string $nom)`
- `postNullableInt(string $nom)`
- `postNullableFloat(string $nom)`
- `postNullableBool(string $nom)`
- `postNullableDate(string $nom)`
    

Ces méthodes distinguent correctement une chaîne vide `""` d'une valeur `null`.

## Gestion des fichiers

Pour les formulaires comportant un champ `<input type="file">` :

- **Récupérer un fichier** :
    
    ```php
    $fichier = Requete::getFile('photo');
    ```
    
- **Vérifier l'existence du champ** :
    
    ```php
    if (Requete::existeFichier('photo')) { /* ... */ }
    ```
    
- **Vérifier qu'un fichier a réellement été transmis** (tient compte de `UPLOAD_ERR_NO_FILE`) :
    
    ```php
    if (Requete::fichierEnvoye('photo')) { /* ... */ }
    ```
    

## Tableau récapitulatif

|**Méthode**|**Description**|
|---|---|
|`methode()`|Retourne la méthode HTTP utilisée.|
|`exigerPost()`|Exige une requête AJAX POST avec vérification du jeton CSRF.|
|`exigerPostSansJeton()`|Exige une requête AJAX POST sans contrôle de jeton.|
|`exigerGet()`|Exige une requête GET.|
|`exigerMethode()`|Exige une méthode HTTP particulière.|
|`estAjax()`|Indique si la requête est AJAX.|
|`exigerAjax()`|Refuse les appels non AJAX.|
|`get()`|Lecture brute d'un paramètre GET.|
|`post()`|Lecture brute d'un paramètre POST ou JSON.|
|`getString()`, `getInt()`, etc.|Lecture typée GET.|
|`postString()`, `postInt()`, etc.|Lecture typée POST (inclus `postArray()` et `postJsonArray()`).|
|`getNullable...()`|Lecture GET acceptant `null`.|
|`postNullable...()`|Lecture POST acceptant `null`.|
|`getFile()`|Retourne un fichier envoyé.|
|`existeFichier()`|Vérifie la présence d'un champ fichier.|
|`fichierEnvoye()`|Vérifie qu'un fichier a réellement été transmis.|

## Exemple d'un contrôleur AJAX complet

Grâce à la classe `Requete`, un contrôleur reste très court et se concentre exclusivement sur l'orchestration :

```php
<?php
declare(strict_types=1);

require $_SERVER['DOCUMENT_ROOT'].'/../bootstrap/autoload.php';

use ClasseTechnique\Requete;
use ClasseTechnique\ReponseJson;
use Service\CoureurService;

Requete::exigerPost();

$licence = Requete::postString('licence');

ReponseJson::envoyerLesDonnees(CoureurService::getByLicence($licence));
```

Le contrôleur ne fait que quatre choses :

1. vérifier la requête ;
2. lire les paramètres ;
3. appeler le service métier ;
4. renvoyer la réponse JSON.

## Sécurité

La classe `Requete` ne protège pas directement contre les injections SQL ou les attaques XSS. 

Son rôle est de récupérer proprement les données, vérifier leur présence et contrôler leur type.

- **Injections SQL** : Repose sur l'utilisation de requêtes préparées (ex: PDO).
- **Attaques XSS** : Repose sur l'échappement des données avant affichage (`htmlspecialchars`).
- **Nettoyage optionnel** : Il est possible de nettoyer les chaînes avant validation métier :
    
    ```php
    $nom = strip_tags(trim(Requete::postString('nom')));
    ```
    

## Philosophie

La classe `Requete` s'inscrit dans la philosophie générale du mini-framework : **chaque classe a une responsabilité unique**.

- `Requete` gère à la fois le contexte de la requête (AJAX, méthode HTTP, jeton) et la lecture/décodage/conversion des paramètres.
- `Service` applique les règles métier.
- `ReponseJson` construit la réponse HTTP.

Chaque contrôleur devient ainsi un simple orchestrateur, généralement limité à quelques lignes, ce qui améliore sa lisibilité, facilite sa maintenance et garantit un comportement homogène dans toute l'application.