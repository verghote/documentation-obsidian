#  Rôle de Postman

**Postman** est un outil de test et de développement d’API.

Il permet de :

- Envoyer des requêtes HTTP (GET, POST, PUT, DELETE…)
- Tester des API REST
- Vérifier les réponses (JSON, XML, HTML…)
- Automatiser des tests
- Organiser les requêtes en collections
- Documenter une API

##  Interface principale

Éléments clés :

- **Barre de requête**
    - Méthode HTTP (GET, POST, PUT…)
    - URL
    - Bouton `Send`
- **Onglets**
    - Params
    - Authorization
    - Headers
    - Body
    - Tests
- **Réponse**
    - Status code (200, 404, 500…)
    - Body
    - Headers
    - Temps de réponse

## Méthodes HTTP principales

| Méthode | Rôle                   |
| ------- | ---------------------- |
| GET     | Récupérer des données  |
| POST    | Créer une ressource    |
| PUT     | Modifier une ressource |
| PATCH   | Modifier partiellement |
| DELETE  | Supprimer              |

#  Créer une Collection

Une **collection** permet d’organiser plusieurs requêtes liées à une même API ou application.

Exemple : 

- Consultation
- Gestion
- Upload

## Étapes

1. Cliquer sur + 
2. Sélectionner **Collection**
3. Donner un nom


La collection apparaît dans le panneau gauche.

# Créer une requête

## Étapes

1. Cliquer sur **New**
2. Sélectionner **HTTP Request**
3. Donner un nom
4. Choisir une collection
5. Cliquer sur **Save**

## Configurer la requête

###  Choisir la méthode : GET POST PUT ou DELETE

###  Entrer l’URL

Exemple : http://consultation/projet/ajax/getlescompetences.php

# Onglet Params

Il permet d'ajouter des paramètres qui seront transmis en utilisant la méthode GET (paramètres dans l'url)

| KEY | VALUE |
| --- | ----- |
| id  | 10    |

# Onglet Body

L'onglet **Body** permet de définir le contenu envoyé dans le corps d'une requête HTTP (POST, PUT, PATCH...). Trois modes sont couramment utilisés : **form-data**, **x-www-form-urlencoded** et **raw (JSON)**. 
Bien qu'ils permettent tous de transmettre des données, ils correspondent à des formats HTTP différents.
## 1. form-data

Le mode **form-data** reproduit le fonctionnement d'un formulaire HTML utilisant l'encodage :

```
<form method="post" enctype="multipart/form-data">
```

Chaque donnée est transmise séparément sous la forme d'un couple **clé / valeur**.
## Caractéristiques

- permet d'envoyer du texte ;
- permet d'envoyer un ou plusieurs fichiers ;
- chaque champ est indépendant ;
- format utilisé pour les formulaires contenant des pièces jointes.
### Exemple

| Key              | Value            |
| ---------------- | ---------------- |
| nom              | Projet Portfolio |
| lesCompetences[] | 1                |
| lesCompetences[] | 5                |
| lesCompetences[] | 12               |

Postman construit cette fois un document MIME multipart. 

Les mêmes données sont envoyées sous forme de plusieurs parties indépendantes :

```
POST /api HTTP/1.1
Content-Type: multipart/form-data; boundary=----------------123456
```

Puis le corps :

```
----------------123456
Content-Disposition: form-data; name="nom"

Projet Portfolio
----------------123456
Content-Disposition: form-data; name="lesCompetences[]"

1
----------------123456
Content-Disposition: form-data; name="lesCompetences[]"

5
----------------123456
Content-Disposition: form-data; name="lesCompetences[]"

12
----------------123456--
```

Chaque champ est transmis séparément.

Ils sont récupérés en PHP à l'aide de la variable super globale $_POST :

```
$_POST['nom'];
$_POST['lesCompetences'];
```

# 2. x-www-form-urlencoded

Le mode **x-www-form-urlencoded** correspond au format utilisé par les formulaires HTML classiques :

```
<form method="post">
```

Les données sont toujours envoyées sous forme de couples **clé / valeur**, mais elles sont toutes regroupées dans une seule chaîne de caractères.

Les espaces deviennent `+` et les caractères spéciaux sont encodés (`%20`, `%26`, etc.).
## Caractéristiques

- uniquement des données texte ;
- ne permet pas l'envoi de fichiers ;
- format très répandu pour les formulaires simples et les authentifications.

### Exemple

|Key|Value|
|---|---|
|nom|Projet Portfolio|
|lesCompetences[]|1|
|lesCompetences[]|5|
|lesCompetences[]|12|
On saisit exactement les données de la même façon.

Postman se charge d'alimenter le corps de la requête comme une unique chaîne de caractères

```
nom=Projet+Portfolio&lesCompetences%5B%5D=1&lesCompetences%5B%5D=5&lesCompetences%5B%5D=12
```

Le navigateur encode automatiquement :

- les espaces (`+`) ;
- les caractères spéciaux (`[` devient `%5B`, `]` devient `%5D`).

La requête ressemble donc à :

```
POST /api HTTP/1.1
Content-Type: application/x-www-form-urlencoded

nom=Projet+Portfolio&lesCompetences%5B%5D=1&lesCompetences%5B%5D=5&lesCompetences%5B%5D=12
```

En PHP :

```
$_POST['nom'];
$_POST['lesCompetences'];
```

Le résultat est exactement le même qu'avec **form-data**.

# Différence entre form-data et x-www-form-urlencoded

Du point de vue de PHP, les deux formats alimentent **$_POST**.
Le code PHP est donc **strictement identique**, même si les données ont circulé différemment sur le réseau.

La différence se situe au niveau du protocole HTTP.

|form-data|x-www-form-urlencoded|
|---|---|
|chaque champ est envoyé séparément|toutes les données sont regroupées dans une seule chaîne|
|autorise les fichiers|n'autorise pas les fichiers|
|un peu plus volumineux|plus compact|

En pratique :

- **form-data** est utilisé lorsqu'un formulaire contient un fichier ;
- **x-www-form-urlencoded** est utilisé pour les formulaires ne contenant que du texte.

# 3. raw (JSON)

Le mode **raw** permet d'envoyer directement le contenu saisi dans Postman.

En choisissant **JSON**, le corps de la requête contient un document JSON.

Exemple :

```
{
    "nom": "Projet Portfolio",
    "lesCompetences": [1, 5, 12]
}
```

Le type MIME envoyé est :

```
Content-Type: application/json
```

Contrairement aux deux autres formats, PHP **ne remplit pas** la variable `$_POST`.

Les données doivent être lues dans le flux :

```
php://input
```

puis décodées avec :

```
json_decode(...)
```

C'est précisément ce que fait la classe `Requete`, ce qui permet d'écrire :

```
$nom = Requete::postString('nom');
$lesCompetences = Requete::postArray('lesCompetences');
```

sans se préoccuper du mode de transmission.

# Comparaison

|Critère|form-data|x-www-form-urlencoded|raw JSON|
|---|---|---|---|
|Formulaire HTML|Oui|Oui|Non|
|Envoi de fichiers|Oui|Non|Non|
|Données texte|Oui|Oui|Oui|
|Tableaux|Oui|Oui|Oui|
|Objets imbriqués|Peu pratique|Peu pratique|Très simple|
|Remplit `$_POST`|Oui|Oui|Non|
|Lecture de `php://input`|Non|Non|Oui|
|Utilisation principale|Formulaires avec fichiers|Formulaires simples|API REST|

# Quel format utiliser ?

Pour une application Web classique utilisant des formulaires HTML, on privilégie :

- **x-www-form-urlencoded** pour les formulaires simples ;
- **form-data** lorsqu'il faut transmettre des fichiers.

Pour une API REST, le format recommandé est **raw JSON**, car il permet de représenter naturellement des structures complexes (tableaux, objets imbriqués) et il est devenu le standard des échanges entre applications.

Grâce à la classe **Requete**, les contrôleurs peuvent traiter indifféremment des données provenant d'un formulaire HTML ou d'une requête JSON, sans modification du code métier.
# Ajouter des Headers

L'onglet **Headers** permet d'ajouter des en-têtes HTTP à la requête.

Selon les contrôles réalisés par l'application, certains en-têtes peuvent être indispensables.

## Content-Type

Il indique le format des données envoyées.

Exemples :

|Format|Header|
|---|---|
|JSON|`Content-Type: application/json`|
|Formulaire classique|`application/x-www-form-urlencoded`|
|Formulaire avec fichier|`multipart/form-data` (géré automatiquement par Postman)|
## Appel AJAX

Si le contrôleur vérifie que la requête provient d'un appel AJAX par exemple avec Requete::exigerPost(), il faut ajouter l'en-tête suivant :

| Key              | Value          |
| ---------------- | -------------- |
| X-Requested-With | XMLHttpRequest |

Sans cet en-tête, le contrôleur considérera que la requête ne provient pas d'un appel AJAX et pourra la refuser.

## Authentification

Certaines API utilisent également des en-têtes tels que :

```
Authorization: Bearer <token>
```

ou une clé API spécifique.

# Tester un contrôleur protégé par un jeton CSRF 

De nombreuses applications web protègent les formulaires contre les attaques **CSRF** (Cross-Site Request Forgery).

Dans ce cas, chaque formulaire HTML contient un jeton généré par le serveur et vérifié lors de la soumission.

Lorsqu'une requête est envoyée depuis **Postman**, ce jeton n'est généralement pas présent. Si le contrôleur vérifie sa présence (par exemple avec une méthode comme `Ajax::exigerPost()`), la requête sera rejetée.

```
{
    "error": "Les informations de sécurité ne sont plus disponibles. Veuillez recharger la page."
}
```

Deux solutions sont alors possibles :

- désactiver temporairement le contrôle CSRF pendant les tests ;

```
// Vérification d'un appel AJAX par la méthode POST sécurisé par un jeton  
// Requete::exigerPost();
```

Pour des tests unitaires de contrôleurs, cette solution est généralement la plus simple.

- ou reproduire complètement le fonctionnement de l'application (session utilisateur, récupération puis transmission du jeton CSRF).

Pour reproduire le fonctionnement il faut connaitre comment le jeton est mis en place

Le jeton est :

- généré côté serveur ;
- conservé en session PHP ;
- injecté dans chaque page HTML sous la forme :

```
<meta name="csrf-token"
      content="477d48e4bfb6ef0355af8fcf0c5ff5ca0a3b3e88be3fc4a436cfe9a5d802bdd5">
```

Le JavaScript de l'application lit cette valeur et ajoute automatiquement l'en-tête HTTP : (e travail est réaliser automatiquement par la fonction appelAjax()

```
X-CSRF-Token: 477d48e4bfb6ef0355af8fcf0c5ff5ca0a3b3e88be3fc4a436cfe9a5d802bdd5
```

à chaque requête AJAX.

Si l'on envoie directement une requête POST avec Postman, le serveur ne retrouve :

- ni la session PHP ayant généré le jeton ;
- ni le header `X-CSRF-Token`.

La vérification échoue donc immédiatement.

Pour tester un contrôleur protégé, il faut reproduire les opérations réalisées automatiquement par le navigateur en  :

+ ajoutant dans Headers la clé X-CSRF-Token avec la valeur récupérée dans la balise <meta name="csrf-token" value = "….">  (F12 > Onglet Sources)

+ ajoutant le cookie PHPSESSID (lien Cookies en dessous du bouton Send) avec la valeur récupérée dans la console du navigateur (F12 > Onglet Application > Storage >  Cookies)

Pour récupérer ses deux valeurs, il suffit d'ouvrir dans le navigateur la page du contrôleur principal, par exemple http://consultation/projet/liste/

Résumé de la configuration

| Header           | Valeur                                   |
| ---------------- | ---------------------------------------- |
| X-Requested-With | XMLHttpRequest                           |
| X-CSRF-Token     | valeur récupérée dans la balise `<meta>` |
Cookies > Manage Cookies > PHPSESSID 

Le serveur pourra alors comparer :

- le jeton présent dans la session ;
- le jeton reçu dans l'en-tête `X-CSRF-Token`.

La requête sera acceptée si les deux valeurs correspondent.

**Remarque :** le jeton CSRF est conservé pendant toute la durée de la session (`Jeton::creer()` réutilise le jeton existant). Il n'est donc pas nécessaire de récupérer un nouveau jeton avant chaque requête. Tant que la session reste active, le même couple `PHPSESSID` / `X-CSRF-Token` peut être utilisé pour tester plusieurs contrôleurs. 

Postman peut récupérer automatiquement le cookie de session à l'aide de l'extension Postman Interceptor
https://learning.postman.com/docs/use/capturing-request-data/interceptor/

Sous Chrome : https://chromewebstore.google.com/detail/postman-interceptor/aicmkgpgakddgnaphhhpliifpcfhicfo