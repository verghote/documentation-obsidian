# Présentation

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
- une requête AJAX envoyant du JSON ;

le code du contrôleur reste exactement le même.

```php
$nom = Requete::postString('nom');

$lesCompetences = Requete::postArray('lesCompetences');
```

# Objectifs

La classe `Requete` poursuit plusieurs objectifs :

- centraliser l'accès aux paramètres HTTP ;
- masquer les différences entre formulaire HTML et requête JSON ;
- vérifier que la requête est conforme aux attentes ;
- convertir automatiquement les valeurs vers le type attendu ;
- produire des messages d'erreur homogènes ;
- simplifier les contrôleurs.

# Vérification de la requête

Avant de lire les paramètres, un contrôleur peut vérifier que la requête reçue est conforme.

## Vérifier la méthode HTTP

### Requête POST

```
Requete::exigerPost();
```

Le contrôleur refuse toute requête autre qu'un POST.

### Requête GET

```
Requete::exigerGet();
```

Le contrôleur refuse toute requête autre qu'un GET.

### Méthode HTTP personnalisée

```php
Requete::exigerMethode('PUT');
```

Permet d'imposer n'importe quelle méthode HTTP.

## Vérifier qu'il s'agit d'une requête AJAX

```php
Requete::exigerAjax();
```

Cette méthode vérifie la présence de l'en-tête :

```
X-Requested-With: XMLHttpRequest
```

Si la requête ne provient pas d'un appel AJAX, une réponse JSON d'erreur est immédiatement renvoyée.

## Tester sans provoquer d'erreur

Lorsque le contrôleur souhaite simplement connaître le type de requête :

```php
if (Requete::estAjax()) {

}
```

La méthode retourne simplement :

```
true
```

ou

```
false
```

# Intégration avec la classe `Ajax`

Dans un contrôleur AJAX, les premières lignes sont presque toujours identiques :

```php
Requete::exigerAjax();

Requete::exigerPost();

Jeton::verifier();
```

Cette séquence garantit que :

- la requête provient bien d'un appel AJAX ;
- la méthode HTTP est correcte ;
- le jeton CSRF est valide.

Comme cette combinaison est utilisée dans la majorité des contrôleurs, le mini-framework propose la classe `Ajax`.

Le code précédent devient :

```php
Ajax::exiger();
```

Par défaut, cette méthode impose :

- une requête AJAX ;
- une méthode POST ;
- un jeton CSRF valide.

Il est également possible de personnaliser son comportement.
### Requête GET

```
Ajax::exiger('GET');
```

### Sans contrôle du jeton CSRF

```
Ajax::exiger('POST', false);
```

La classe `Ajax` est un raccourci destiné à alléger les contrôleurs.

Les méthodes de la classe `Requete` restent bien entendu disponibles lorsqu'un contrôle plus spécifique est nécessaire.

## Les alias les plus utilisés

La classe `Ajax` propose plusieurs méthodes correspondant aux cas d'utilisation les plus courants. Ces méthodes sont simplement des raccourcis autour de `Ajax::exiger()`. Elles permettent d'exprimer immédiatement l'intention du contrôleur.

|Alias|Équivalent|Utilisation|
|---|---|---|
|`Ajax::exigerPost()`|`Ajax::exiger('POST', true)`|Requête AJAX `POST` avec vérification du jeton CSRF. C'est le cas le plus fréquent pour les opérations d'ajout, modification ou suppression.|
|`Ajax::exigerGet()`|`Ajax::exiger('GET', true)`|Requête AJAX `GET` avec vérification du jeton. Utilisée pour les consultations nécessitant un jeton.|
|`Ajax::exigerPostSansJeton()`|`Ajax::exiger('POST', false)`|Requête AJAX `POST` sans contrôle du jeton lorsque celui-ci n'est pas nécessaire.|
|`Ajax::exigerGetSansJeton()`|`Ajax::exiger('GET', false)`|Requête AJAX `GET` sans contrôle du jeton. Souvent utilisée pour des consultations publiques.|

### Exemple

Au lieu d'écrire :

```
Ajax::exiger('POST', true);
```

on préférera généralement :

```
Ajax::exigerPost();
```

De la même manière :

```
Ajax::exiger('GET', false);
```

s'écrit simplement :

```
Ajax::exigerGetSansJeton();
```

Ces alias n'ajoutent aucune fonctionnalité supplémentaire ; ils rendent simplement les contrôleurs plus lisibles en exprimant clairement les contraintes de la requête dès la première ligne du script. Dans la majorité des contrôleurs du mini-framework, ce sont ces alias qui seront utilisés plutôt que la méthode générique `Ajax::exiger()`.

# Tableau récapitulatif

| Méthode                          | Description                                     |
| -------------------------------- | ----------------------------------------------- |
| `methode()`                      | Retourne la méthode HTTP utilisée               |
| `exigerPost()`                   | Exige une requête POST                          |
| `exigerGet()`                    | Exige une requête GET                           |
| `exigerMethode()`                | Exige une méthode HTTP particulière             |
| `estAjax()`                      | Indique si la requête est AJAX                  |
| `exigerAjax()`                   | Refuse les appels non AJAX                      |
| `get()`                          | Lecture brute d'un paramètre GET                |
| `post()`                         | Lecture brute d'un paramètre POST ou JSON       |
| `getString()`, `getInt()`, ...   | Lecture typée GET                               |
| `postString()`, `postInt()`, ... | Lecture typée POST                              |
| `getNullable...()`               | Lecture GET acceptant `null`                    |
| `postNullable...()`              | Lecture POST acceptant `null`                   |
| `getFile()`                      | Retourne un fichier envoyé                      |
| `existeFichier()`                | Vérifie la présence d'un champ fichier          |
| `fichierEnvoye()`                | Vérifie qu'un fichier a réellement été transmis |

# Principe général

Un contrôleur ne doit jamais accéder directement aux superglobales PHP.

Au lieu d'écrire :

```php
$id = $_GET['id'];

$nom = $_POST['nom'];
```

on utilisera :

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

# Gestion transparente des requêtes POST

Une requête POST peut être envoyée de plusieurs façons.

## Formulaire HTML

Le navigateur envoie :

```
application/x-www-form-urlencoded
```

ou

```
multipart/form-data
```

PHP remplit automatiquement `$_POST`.

## Requête JSON

Lorsque le navigateur envoie :

```
Content-Type: application/json
```

les données ne sont plus présentes dans `$_POST`.

La classe `Requete` :

- lit automatiquement `php://input` ;
- décode le JSON ;
- transforme les objets JSON en tableaux PHP ;
- mémorise le résultat afin d'éviter plusieurs lectures du flux.

Le reste de l'application ne voit aucune différence.

# Méthodes de lecture brute

## GET

```php
$page = Requete::get('page');
```

Retourne la valeur sans effectuer de conversion.

## POST

```php
$table = Requete::post('table');
```

Fonctionne aussi bien avec :

- un formulaire HTML ;
- un objet `FormData` ;
- une requête JSON.

# Méthodes GET typées

|Méthode|Retour|
|---|---|
|`getString()`|string|
|`getInt()`|int|
|`getFloat()`|float|
|`getBool()`|bool|
|`getArray()`|array|
|`getDate()`|string|
|`getEmail()`|string|
|`getUrl()`|string|
|`getScalar()`|string|

Exemple :

```php
$id = Requete::getInt('id');

$date = Requete::getDate('date');
```

# Méthodes POST typées

|Méthode|Retour|
|---|---|
|`postString()`|string|
|`postInt()`|int|
|`postFloat()`|float|
|`postBool()`|bool|
|`postArray()`|array|
|`postDate()`|string|
|`postEmail()`|string|
|`postUrl()`|string|
|`postScalar()`|string|

Exemple :

```php
$nom = Requete::postString('nom');

$age = Requete::postInt('age');

$colonnes = Requete::postArray('colonnes');
```

# Valeurs acceptant `null`

Lorsque la colonne SQL accepte la valeur `NULL`, la classe propose des variantes dédiées.

Exemples :

```php
$description = Requete::postNullableString('description');

$dateFin = Requete::postNullableDate('dateFin');

$age = Requete::postNullableInt('age');
```

Ces méthodes distinguent correctement :

- une chaîne vide (`""`) ;
- une valeur `null`.

# Gestion des fichiers

Pour les formulaires comportant un champ `<input type="file">`, la classe fournit plusieurs méthodes.

## Récupérer un fichier

```php
$fichier = Requete::getFile('photo');
```

## Vérifier l'existence d'un champ fichier

```php
if (Requete::existeFichier('photo')) {

}
```

---

## Vérifier qu'un fichier a réellement été envoyé

```php
if (Requete::fichierEnvoye('photo')) {

}
```

Cette méthode tient compte du code d'erreur `UPLOAD_ERR_NO_FILE`.

---

# Exemple d'un contrôleur AJAX

Grâce aux différentes classes du mini-framework, un contrôleur reste très court.

```php
<?php

require $_SERVER['DOCUMENT_ROOT'].'/../bootstrap/autoload.php';

use ClasseTechnique\Ajax;
use ClasseTechnique\Requete;
use ClasseTechnique\ReponseJson;
use Service\CoureurService;

Ajax::exigerPost();

$licence = Requete::postString('licence');

ReponseJson::envoyerLesDonnees(
    CoureurService::getByLicence($licence)
);
```

Le contrôleur ne fait que quatre choses :

1. vérifier la requête ;
2. lire les paramètres ;
3. appeler le service métier ;
4. renvoyer la réponse JSON.

Toute la logique technique est déléguée au mini-framework.


# Avantages

- Un point d'accès unique aux données HTTP.
- Gestion transparente des formulaires HTML et des requêtes JSON.
- Contrôle centralisé des méthodes HTTP.
- Vérification des appels AJAX.
- Conversion automatique vers les types PHP.
- Messages d'erreur homogènes.
- Contrôleurs beaucoup plus courts.
- Séparation claire entre logique technique et logique métier.


# Sécurité

La classe `Requete` ne protège pas directement contre les injections SQL ou les attaques XSS.

Son rôle est de :

- récupérer proprement les données ;
- vérifier leur présence ;
- contrôler leur type.

La protection contre les injections SQL repose sur les requêtes préparées :

```sql
INSERT INTO projet(nom)
VALUES(:nom)
```

La protection contre les attaques XSS repose sur l'échappement des données avant affichage :

```php
<?= htmlspecialchars($projet['nom']) ?>
```

Il est également recommandé de nettoyer les chaînes avant validation métier lorsque cela est pertinent.

```php
$nom = strip_tags(trim(Requete::postString('nom')));
```

Cette opération supprime les balises HTML et élimine les espaces inutiles avant le traitement métier.

# Philosophie

La classe `Requete` s'inscrit dans la philosophie générale du mini-framework : **chaque classe a une responsabilité unique**.

- `Ajax` vérifie le contexte de la requête (AJAX, méthode HTTP, jeton CSRF).
- `Requete` lit et convertit les paramètres.
- `Service` applique les règles métier.
- `ReponseJson` construit la réponse HTTP.

Chaque contrôleur devient ainsi un simple orchestrateur, généralement limité à quelques lignes, ce qui améliore sa lisibilité, facilite sa maintenance et garantit un comportement homogène dans toute l'application.