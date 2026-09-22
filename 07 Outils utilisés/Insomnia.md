# Rôle d'Insomnia

**Insomnia** est un outil de développement et de test d'API.

Il permet notamment de :

- envoyer des requêtes HTTP (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`…) ;
- tester des API REST ;
- transmettre des paramètres, des données et des fichiers ;
- définir les en-têtes HTTP ;
- gérer des variables et des environnements ;
- consulter les réponses du serveur ;
- organiser les requêtes ;
- tester des contrôleurs PHP sans passer par l'interface graphique de l'application.

Dans le cadre de cette application, **Insomnia peut notamment être utilisé pour tester directement les contrôleurs PHP Ajax**.

# Interface principale

L'interface d'Insomnia est organisée autour de plusieurs éléments.
## Requête

Une requête contient principalement :

- la méthode HTTP ;
- l'URL ;
- les paramètres ;
- les en-têtes ;
- le corps de la requête ;
- éventuellement l'authentification.

La requête est envoyée avec le bouton **Send**.

## Réponse

Après l'envoi, Insomnia affiche notamment :

- le code HTTP (`200`, `400`, `403`, `404`, `500`…) ;
- le contenu de la réponse ;
- les en-têtes HTTP ;
- le temps de réponse.

Pour les contrôleurs Ajax de l'application, la réponse est généralement au format **JSON**.

# Méthodes HTTP principales

|Méthode|Rôle|
|---|---|
|GET|Récupérer des données|
|POST|Envoyer des données / effectuer une opération|
|PUT|Remplacer une ressource|
|PATCH|Modifier partiellement une ressource|
|DELETE|Supprimer une ressource|

Le contrôleur PHP détermine la méthode qui doit être utilisée.

Par exemple :  Requete::exigerGet() indique que le contrôleur attend une requête `GET`.

# Créer un projet ou un espace de travail

Insomnia organise les requêtes à l'intérieur d'un **Project** ou d'un espace de travail.

Le principe est de regrouper dans un même espace les requêtes appartenant à une même application ou à une même API.

Exemple :

```text
Consultation
│
├── Projets
│   ├── Liste
│   ├── Création
│   └── Modification
│
├── Coureurs
│   ├── Recherche
│   └── Consultation
│
└── Compétences
    └── Liste
```

Cette organisation permet de conserver tous les tests liés à l'application au même endroit.
# Créer une requête HTTP

Une requête peut être créée à partir de l'interface de création d'une nouvelle requête.

Il faut ensuite :

1. donner un nom à la requête ;
2. choisir la méthode HTTP ;
3. saisir l'URL ;
4. configurer éventuellement les paramètres ;
5. configurer les Headers ;
6. configurer le Body si nécessaire ;
7. envoyer la requête avec **Send**.

Exemple :

```text
GET http://consultation/projet/ajax/getlescompetences.php
```

# Paramètres GET

Les paramètres `GET` sont transmis dans l'URL.

Exemple :

```text
http://consultation/projet/ajax/getlescompetences.php?idProjet=10
```

Le paramètre est :

|KEY|VALUE|
|---|---|
|idProjet|10|

Insomnia ajoute automatiquement le paramètre à l'URL.

La requête finale devient :

```text
GET /projet/ajax/getlescompetences.php?idProjet=10
```

En PHP, le paramètre est normalement disponible dans :

```php
$_GET['idProjet'];
```

Dans l'application, on privilégie cependant l'utilisation de la classe `Requete` pour récupérer et contrôler les paramètres.

# Body

Le **Body** contient les données transmises dans le corps de la requête HTTP.

Il est principalement utilisé avec :

- `POST` ;
- `PUT` ;
- `PATCH`.

Selon le besoin, Insomnia permet notamment d'envoyer :

- des données de formulaire ;
- des données URL-encoded ;
- du JSON ;
- des fichiers.

Le format choisi doit correspondre à ce que le contrôleur PHP attend.

# `application/x-www-form-urlencoded`

Ce format correspond au fonctionnement classique d'un formulaire HTML :

```html
<form method="post">
```

Les données sont transmises sous la forme de couples :

```text
clé=valeur
```

Exemple :

|Key|Value|
|---|---|
|nom|Projet Portfolio|
|lesCompetences[]|1|
|lesCompetences[]|5|
|lesCompetences[]|12|

Le corps de la requête sera de la forme :

```text
nom=Projet+Portfolio&lesCompetences%5B%5D=1&lesCompetences%5B%5D=5&lesCompetences%5B%5D=12
```

Le type MIME est :

```http
Content-Type: application/x-www-form-urlencoded
```

PHP récupère alors les données dans :

```php
$_POST['nom'];
$_POST['lesCompetences'];
```

Dans l'application, ces données peuvent être récupérées avec les méthodes appropriées de `Requete`.

# `multipart/form-data`

Le format `multipart/form-data` est principalement utilisé lorsqu'il faut transmettre des fichiers.

Il correspond notamment à :

```html
<form method="post" enctype="multipart/form-data">
```

Chaque donnée est envoyée dans une partie distincte du corps HTTP.

Exemple :

|Key|Value|
|---|---|
|nom|Projet Portfolio|
|fichier|document.pdf|

Ce format permet donc de transmettre simultanément :

- des données texte ;
- des tableaux ;
- des fichiers.

En PHP, les données de formulaire sont disponibles dans :

```php
$_POST
```

et les fichiers dans :

```php
$_FILES
```

# JSON

Insomnia permet également d'envoyer directement un document JSON.

Exemple :

```json
{
    "nom": "Projet Portfolio",
    "lesCompetences": [1, 5, 12]
}
```

Le type MIME doit être :

```http
Content-Type: application/json
```

Contrairement aux données `form-urlencoded` ou `multipart/form-data`, les données JSON ne sont pas placées automatiquement dans :

```php
$_POST
```

Elles doivent être lues à partir du flux :

```text
php://input
```

puis décodées.

La classe `Requete` permet de masquer cette différence au niveau du contrôleur.

---

# Comparaison des formats

|Critère|`form-data`|`x-www-form-urlencoded`|JSON|
|---|---|---|---|
|Formulaire HTML|Oui|Oui|Non|
|Fichiers|Oui|Non|Non|
|Données texte|Oui|Oui|Oui|
|Tableaux|Oui|Oui|Oui|
|Objets imbriqués|Peu pratique|Peu pratique|Très simple|
|`$_POST`|Oui|Oui|Non|
|`php://input`|Non|Non|Oui|
|Utilisation courante|Formulaire avec fichiers|Formulaire simple|API / échanges structurés|

# Quel format utiliser avec l'application ?

Le format dépend du contrôleur PHP testé.

Pour un contrôleur recevant des données provenant d'un formulaire classique :

```text
x-www-form-urlencoded
```

est généralement approprié.

Pour un formulaire comportant un fichier :

```text
multipart/form-data
```

est nécessaire.

Pour un contrôleur conçu pour recevoir du JSON :

```text
application/json
```

doit être utilisé.

Il est donc important de ne pas choisir le format uniquement en fonction des possibilités d'Insomnia : **le format doit correspondre à ce que le contrôleur attend**.

# Headers

Les **Headers** permettent de transmettre des informations supplémentaires au serveur.

Exemple :

```http
Content-Type: application/json
```

ou :

```http
X-Requested-With: XMLHttpRequest
```

ou encore :

```http
X-CSRF-Token: ...
```

Pour les contrôleurs Ajax de l'application, certains Headers sont nécessaires en fonction des protections mises en place.

# Header `Content-Type`

Le `Content-Type` indique au serveur le format du corps de la requête.

Exemples :

|Données envoyées|Content-Type|
|---|---|
|JSON|`application/json`|
|Formulaire classique|`application/x-www-form-urlencoded`|
|Formulaire avec fichier|`multipart/form-data`|

Avec `multipart/form-data`, Insomnia doit généralement gérer automatiquement la valeur complète contenant la `boundary`.

# Simuler un appel AJAX

Les scripts PHP du répertoire `ajax` sont normalement appelés par la fonction JavaScript :

```javascript
appelAjax()
```

Le navigateur transmet notamment l'en-tête :

```http
X-Requested-With: XMLHttpRequest
```

Pour tester directement un contrôleur Ajax avec Insomnia, il faut donc reproduire cet en-tête lorsque le serveur ou le contrôleur le vérifie.

Dans Insomnia :

| Header             | Value            |
| ------------------ | ---------------- |
| `X-Requested-With` | `XMLHttpRequest` |

**Attention :** cet en-tête peut être ajouté manuellement par n'importe quel client HTTP. Il ne constitue donc pas, à lui seul, une preuve de sécurité.

# Tester un contrôleur GET

Prenons un contrôleur :

```text
ajax/getcoureur.php
```

qui attend un paramètre :

```text
search
```

et qui exige une requête GET :

```php
Requete::exigerGet();
```

Dans Insomnia :

```text
Method : GET
URL    : http://consultation/coureur/ajax/getcoureur.php
```

Dans les paramètres :

|KEY|VALUE|
|---|---|
|search|Dupont|

Insomnia construit alors une URL similaire à :

```text
http://consultation/coureur/ajax/getcoureur.php?search=Dupont
```

Si le contrôleur exige également l'en-tête Ajax :

```text
X-Requested-With: XMLHttpRequest
```

il faut l'ajouter dans les Headers.

# Exemple de contrôleur GET

Le contrôleur peut contenir :

```php
Requete::exigerGet();

$search = Requete::getString('search');

if (trim($search) === '') {

    ReponseJson::envoyerLesErreurs([
        'nomR' => "Le paramètre 'search' est vide."
    ]);
}

if (!preg_match("/^[\p{L} '-]+$/u", $search)) {

    ReponseJson::envoyerLesErreurs([
        'nomR' => "Seules les lettres et les espaces sont autorisées"
    ]);
}

ReponseJson::envoyerLesDonnees(
    Coureur::getByNomPrenom($search)
);
```

Insomnia permet de tester chacune de ces situations.

# Tester une erreur de validation

Par exemple, envoyer :

```text
search=
```

permet de tester :

```php
if (trim($search) === '')
```

Le serveur doit alors retourner une réponse JSON d'erreur.

Le résultat peut être similaire à :

```json
{
    "erreurs": {
        "nomR": "Le paramètre 'search' est vide."
    }
}
```

La structure exacte dépend de l'implémentation de `ReponseJson`.

# Tester une erreur de format

On peut également envoyer :

```text
search=12345
```

Le contrôle :

```php
preg_match("/^[\p{L} '-]+$/u", $search)
```

doit refuser cette valeur si le contrôleur attend uniquement un nom.

Insomnia permet ainsi de tester non seulement les cas nominaux, mais également les données invalides.

# Tester un contrôleur POST

Pour un contrôleur qui utilise :

```php
Requete::exigerPost();
```

il faut sélectionner :

```text
POST
```

dans Insomnia.

Exemple :

```text
POST http://consultation/projet/ajax/enregistrer.php
```

Puis choisir dans le Body le format attendu par le contrôleur.

Pour un formulaire classique :

```text
application/x-www-form-urlencoded
```

Exemple :

|KEY|VALUE|
|---|---|
|idProjet|10|
|nom|Projet Portfolio|

# Protection CSRF

Certains contrôleurs Ajax sont protégés contre les attaques **CSRF** (_Cross-Site Request Forgery_).

Dans ce cas, une requête envoyée depuis Insomnia ne possède normalement pas automatiquement :

- la session PHP de l'utilisateur ;
- le cookie `PHPSESSID` ;
- le jeton CSRF ;
- le Header `X-CSRF-Token`.

La requête peut donc être refusée.

# Principe du mécanisme CSRF

Dans l'application, le serveur génère un jeton CSRF et l'intègre notamment dans la page HTML :

```html
<meta
    name="csrf-token"
    content="...">
```

La fonction :

```javascript
appelAjax()
```

récupère ce jeton et l'ajoute aux requêtes Ajax sous la forme :

```http
X-CSRF-Token: ...
```

Le serveur peut alors comparer :

```text
jeton reçu
     │
     ▼
X-CSRF-Token
     │
     │ comparaison
     ▼
jeton associé à la session PHP
```

La requête est acceptée si les deux correspondent.

# Tester un contrôleur protégé par CSRF

Pour tester avec Insomnia un contrôleur qui exige un jeton CSRF, il faut reproduire la session du navigateur.

Deux éléments sont notamment nécessaires :

```text
PHPSESSID
X-CSRF-Token
```

Le couple doit correspondre à la même session.

# Récupérer le `PHPSESSID`

Dans le navigateur :

1. ouvrir la page de l'application ;
2. ouvrir les outils de développement avec `F12` ;
3. ouvrir l'onglet **Application** ;
4. rechercher les **Cookies** ;
5. récupérer le cookie `PHPSESSID`.

Sa valeur doit être associée à la session ayant généré le jeton CSRF.

Dans Insomnia, cette session peut être reproduite en ajoutant le cookie correspondant à la requête.

# Récupérer le jeton CSRF

Dans la page HTML de l'application, rechercher :

```html
<meta name="csrf-token"
      content="...">
```

La valeur de l'attribut `content` constitue le jeton CSRF.

Elle doit être transmise dans Insomnia avec :

|Header|Value|
|---|---|
|`X-CSRF-Token`|valeur du jeton CSRF|

# Configuration d'une requête protégée

Une requête POST protégée nécessite au minimum :

## Headers

| Key              | Value           |
| ---------------- | --------------- |
| X-Requested-With | XMLHttpRequest  |
| X-CSRF-Token     | valeur du jeton |

## Cookie

```text
PHPSESSID = valeur de la session
```

## Body

Le format correspondant à ce qu'attend le contrôleur.

Par exemple :

|Key|Value|
|---|---|
|idProjet|10|

Le point essentiel est que le `PHPSESSID` et le `X-CSRF-Token` proviennent de la **même session**.

# Pourquoi le `PHPSESSID` est indispensable ?

Le jeton CSRF est associé à la session PHP.

Envoyer uniquement :

```http
X-CSRF-Token: ...
```

ne suffit donc pas.

Le serveur doit également retrouver la session correspondant au jeton.

Le mécanisme peut être représenté ainsi :

```text
Insomnia
   │
   ├── PHPSESSID ────────────┐
   │                          │
   └── X-CSRF-Token           │
                              ▼
                         Serveur PHP
                              │
                    ┌─────────┴─────────┐
                    │                   │
              session PHP         token reçu
                    │                   │
                    └────────┬──────────┘
                             │
                         comparaison
                             │
                       ┌─────┴─────┐
                       │           │
                    identiques   différents
                       │           │
                       ▼           ▼
                    accepté      refusé
```

# Réponse JSON

Les contrôleurs Ajax utilisent généralement :

```php
ReponseJson::envoyerLesDonnees(...)
```

pour transmettre une réponse positive.

En cas d'erreur :

```php
ReponseJson::envoyerLesErreurs(...)
```

est utilisée.

Insomnia permet de visualiser directement le JSON retourné par le serveur.

Cela permet notamment de vérifier :

- les données retournées ;
- la structure JSON ;
- les messages d'erreur ;
- les codes HTTP ;
- les éventuelles informations supplémentaires.

# Tester les différents cas

Insomnia est particulièrement utile pour tester séparément les différents scénarios.

|Test|Résultat attendu|
|---|---|
|Paramètre valide|Données JSON|
|Paramètre absent|Erreur JSON|
|Paramètre vide|Erreur JSON|
|Mauvais type|Erreur JSON|
|Mauvais format|Erreur JSON|
|Ressource inexistante|Erreur JSON|
|Mauvaise méthode HTTP|Refus|
|Jeton CSRF absent|Refus|
|Jeton CSRF incorrect|Refus|
|Session absente|Refus éventuel|

Cette approche permet de tester le contrôleur indépendamment de l'interface graphique.

---

# Insomnia et `appelAjax()`

Dans l'application réelle :

```text
Interface HTML
      │
      ▼
JavaScript
      │
      ▼
appelAjax()
      │
      ├── méthode HTTP
      ├── paramètres
      ├── Headers
      ├── CSRF
      └── session
      │
      ▼
Contrôleur Ajax
```

Avec Insomnia :

```text
Insomnia
   │
   ├── méthode HTTP
   ├── paramètres
   ├── Headers
   ├── CSRF
   └── session
   │
   ▼
Contrôleur Ajax
```

Insomnia permet donc de **reproduire manuellement la requête normalement construite par `appelAjax()`**.

# Insomnia comme outil de diagnostic

Insomnia ne sert pas uniquement à vérifier qu'un contrôleur fonctionne.

Il permet également d'identifier l'origine d'un problème.

Par exemple :

```text
Le contrôleur ne répond pas
        │
        ├── 403 ?
        │     └── vérifier les protections HTTP
        │
        ├── 400 ?
        │     └── vérifier les paramètres
        │
        ├── erreur CSRF ?
        │     └── vérifier session + token
        │
        ├── 500 ?
        │     └── vérifier le traitement PHP
        │
        └── 200 mais données incorrectes ?
              └── vérifier le traitement métier
```

Il devient ainsi possible de distinguer un problème :

- HTTP ;
- de sécurité ;
- de validation ;
- de traitement métier ;
- de données ;
- ou de réponse JSON.

---

# Comparaison Postman / Insomnia

|Fonction|Postman|Insomnia|
|---|---|---|
|Requêtes HTTP|Oui|Oui|
|GET / POST / PUT / DELETE|Oui|Oui|
|Paramètres|Oui|Oui|
|Headers|Oui|Oui|
|Body JSON|Oui|Oui|
|Formulaire|Oui|Oui|
|Fichiers|Oui|Oui|
|Cookies|Oui|Oui|
|Environnements|Oui|Oui|
|Tests d'API|Oui|Oui|
|Organisation des requêtes|Oui|Oui|
|Réponse JSON|Oui|Oui|

Les deux outils permettent donc de réaliser les mêmes tests fondamentaux pour les contrôleurs Ajax de l'application.

# Procédure recommandée pour tester un contrôleur Ajax

Pour tester un contrôleur PHP Ajax avec Insomnia :

1. Identifier la méthode HTTP attendue.
2. Identifier les paramètres attendus.
3. Identifier le format du Body attendu.
4. Identifier les Headers obligatoires.
5. Ajouter `X-Requested-With` si nécessaire.
6. Ajouter le jeton CSRF si le contrôleur le vérifie.
7. Ajouter le cookie `PHPSESSID` correspondant à ce jeton.
8. Envoyer la requête.
9. Vérifier le code HTTP.
10. Vérifier le contenu JSON.
11. Tester également les cas d'erreur.



> **Insomnia permet ainsi de tester le contrôleur indépendamment de l'interface utilisateur, tout en reproduisant les éléments nécessaires à son fonctionnement.**