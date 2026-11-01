> **Objectif :** comprendre comment deux applications communiquent,  
> distinguer API, Web Service et API REST, lire une requête HTTP,  
> interpréter une réponse et savoir interroger une API depuis PHP.

## 1. Pourquoi faire communiquer des applications ?

Une application n'a pas toujours intérêt à tout développer elle-même.  
Elle peut s'appuyer sur les fonctionnalités ou les données d'un autre  logiciel.

Exemples : 
- un site de course à pied récupère les prévisions  météorologiques auprès d'un service spécialisé ; 
- une boutique en ligne  confie le paiement à un prestataire ; 
- une application mobile récupère  les données d'un serveur ; 
- un site utilise un service de cartographie  pour afficher un itinéraire ; 
- une application métier vérifie une  licence auprès d'un service distant.

Pour communiquer, les applications doivent convenir d'une **interface**,  de règles d'échange et d'un format de données. C'est le rôle d'une API.

## 2. API, Web Service et API REST : les distinctions

### 2.1 API

Une **API** (_Application Programming Interface_, ou interface de programmation d'applications) définit comment un logiciel peut utiliser  les fonctionnalités ou les données exposées par un autre logiciel. 
Elle  précise notamment les opérations disponibles, les paramètres attendus,  les résultats possibles et les erreurs.

Une API n'implique pas nécessairement Internet. 
Par exemple, une bibliothèque PHP expose des classes et des méthodes que le code de  l'application peut appeler. PDO fournit une interface permettant à PHP  d'accéder à des bases de données au moyen de pilotes.
### 2.2 Web Service

Un **Web Service** est un service logiciel accessible à distance par un  réseau, généralement au moyen de protocoles Web. 
Il permet à des  applications distinctes d'échanger des requêtes et des réponses.

Les échanges peuvent utiliser HTTP et des formats comme JSON ou XML.  
Certains services s'appuient sur SOAP ; d'autres exposent une API HTTP  de style REST.
### 2.3 API REST

Une **API REST** est une API Web conçue selon le style architectural  REST (_Representational State Transfer_). 
Elle modélise les données ou  les objets métier sous forme de ressources identifiées par des URI et  s'appuie généralement sur HTTP pour les lire ou les modifier.
### 2.4 À retenir

- **API** : interface permettant à un logiciel d'utiliser un autre   logiciel.
- **Web Service** : service accessible à distance par un réseau.
- **API REST** : API Web qui applique les principes du style  architectural REST.

Dans l'usage courant, les termes _API Web_ et _Web Service_ se  recouvrent souvent, mais ils ne sont pas strictement synonymes dans  toutes les définitions. 
Une API n'est pas nécessairement un Web Service.
## 3. Architecture client-serveur

Une communication typique comporte deux rôles :

- **Client** : application qui formule une requête. Cela peut être un  navigateur, une application mobile, un serveur PHP ou un autre  programme.
- **Serveur** : application qui reçoit la requête, vérifie les droits  et les données, exécute le traitement puis renvoie une réponse.

```
Client                                  Serveur API
  |                                          |
  |---- Requête HTTP ----------------------->|
  |     méthode, URI, en-têtes, corps        |
  |                                          | Vérifications
  |                                          | Traitement métier
  |                                          | Accès aux données
  |<--- Réponse HTTP ------------------------|
  |     statut, en-têtes, corps              |
```

Le client n'a pas nécessairement un accès direct à la base de données du  serveur. 
L'API forme une **couche d'abstraction** : elle expose des opérations maîtrisées sans révéler les détails internes du stockage.
## 4. HTTP : le protocole des échanges Web

HTTP (_Hypertext Transfer Protocol_) définit la structure des requêtes  et des réponses. HTTPS correspond à HTTP transporté dans une connexion  TLS, qui protège la confidentialité et l'intégrité des échanges et  
permet l'authentification du serveur.

Une requête HTTP comprend principalement : 
1. une méthode ; 
2. une cible  (chemin et, éventuellement, paramètres de requête) ; 3. des en-têtes ;  
3. éventuellement un corps (_body_).

Une réponse comprend : 
1. un code de statut ; 
2. des en-têtes ; 
3. éventuellement un corps contenant les données ou une description de  l'erreur.
### 4.1 URI et URL

Une **URI** identifie une ressource. 
Une **URL** est un type d'URI qui  indique comment localiser cette ressource.

Exemple :

```
https://api.exemple.fr/courses/42?langue=fr
```

- `https` : schéma ;
- `api.exemple.fr` : hôte ;
- `/courses/42` : chemin de la ressource ;
- `?langue=fr` : paramètre de requête.

Les paramètres de requête sont souvent utilisés pour filtrer,  rechercher, trier ou paginer une collection. Ils ne remplacent pas les  contrôles de validation côté serveur.
### 4.2 Méthodes HTTP courantes

`GET` Lire une ressource 

`POST` Créer une ressource ou déclencher un  traitement

`PUT` Remplacer la représentation d'une  ressource

`PATCH` Modifier partiellement une ressource

 `DELETE` Supprimer une ressource 
 
Ces conventions sont courantes mais doivent être précisées par la  documentation de l'API. En particulier, `PUT` et `PATCH` ne sont pas  strictement équivalents : le premier correspond généralement au  
remplacement complet, le second à une modification partielle.
### 4.3 CRUD et HTTP

CRUD désigne les quatre opérations élémentaires sur les données :

- **Create** : créer ;
- **Read** : lire ;
- **Update** : modifier ;
- **Delete** : supprimer.

Exemple avec une ressource `courses` :

```
GET    /courses
GET    /courses/42
POST   /courses
PUT    /courses/42
PATCH  /courses/42
DELETE /courses/42
```

`GET /courses` désigne la collection ; 
`GET /courses/42` désigne une  course précise. 
La ressource est identifiée par son URI, tandis que la  méthode exprime l'action demandée.
### 4.4 En-têtes HTTP importants

`Accept` Formats de réponse que le client sait traiter  
`Content-Type` Format du corps envoyé  
`Authorization` Informations d'authentification ou d'autorisation  
`User-Agent` Identification du logiciel client, lorsque nécessaire

Exemple :

```
Content-Type: application/json
Accept: application/json
Authorization: Bearer <jeton>
```

`Content-Type` décrit le contenu envoyé ; `Accept` indique les formats de réponse acceptés. Ces deux en-têtes ont donc des rôles différents.
## 5. Codes de statut HTTP

Le code HTTP est essentiel : il indique le résultat général de la  requête. Il ne faut pas supposer que la requête a réussi simplement  parce que le serveur a renvoyé du texte ou du JSON.

`200 OK` Requête réussie Lecture ou modification  avec réponse

`201 Created` Ressource créée Création réussie

`202 Accepted` Traitement accepté, pas Traitement différé  nécessairement terminé

`204 No Content` Succès sans corps de Suppression réussie  réponse

`400 Bad Request` Requête mal formée ou JSON incorrect ou  invalide paramètres mal  construits

`401 Unauthorized` Authentification Jeton manquant ou  absente ou invalide expiré

`403 Forbidden` Accès refusé Utilisateur authentifié  mais non autorisé

`404 Not Found` Ressource introuvable Identifiant inexistant

`405 Method Not Allowed` Méthode non prise en `DELETE` non autorisé  charge pour cette  ressource

`409 Conflict` Conflit avec l'état Doublon ou conflit de  courant modification

`415 Unsupported Media Type` Format du corps non JSON attendu, pris en charge formulaire envoyé

`422 Unprocessable Content` Contenu compris mais Date incohérente données non conformes  aux règles métier

`429 Too Many Requests` Limite de requêtes Trop de requêtes en peu dépassée de temps

`500 Internal Server Error` Erreur interne du Exception non gérée serveur

`502 Bad Gateway` Réponse invalide d'un Défaillance d'un serveur amont service intermédiaire

`503 Service Unavailable` Service temporairement Maintenance ou indisponible surcharge

`504 Gateway Timeout` Délai dépassé auprès Service distant trop d'un service amont lent

Les catégories sont : 
- `1xx` : information ; 
- `2xx` : succès ; 
- `3xx`: redirection ; 
- `4xx` : problème associé à la requête ou aux droits du client ; 
- `5xx` : problème côté serveur ou infrastructure.

Le détail exact dépend du contrat de l'API. Un code de statut et un  message explicatif sont complémentaires.
## 6. Les formats de données : JSON et XML

### 6.1 JSON

JSON (_JavaScript Object Notation_) est un format textuel largement utilisé pour échanger des données. 
Il permet de représenter des objets, des tableaux, des chaînes, des nombres, des booléens et la valeur  `null`.

```
{
  "id": 42,
  "nom": "Course des hortillonnages",
  "date": "2026-10-25",
  "distance": 10,
  "inscriptionsOuvertes": true
}
```

JSON est un format de données, pas un protocole de communication. 
HTTP  transporte le message ; JSON en décrit le contenu.

En PHP, on peut encoder une structure PHP avec `json_encode()` et  décoder du JSON avec `json_decode()`.

```php
$json = json_encode($donnees, JSON_UNESCAPED_UNICODE);

$donnees = json_decode($json, true);
if (json_last_error() !== JSON_ERROR_NONE) {
    // Le JSON reçu est invalide.
}
```

Le second argument `true` de `json_decode()` demande une représentation  sous forme de tableaux associatifs PHP.
### 6.2 XML et SOAP

XML est un format structuré à balises. 
SOAP est un protocole de messagerie dont les messages sont structurés en XML. SOAP peut être transporté par HTTP et d'autres mécanismes.

REST n'impose pas JSON : une API REST peut utiliser plusieurs formats de  représentation. JSON est toutefois très fréquent dans les API Web  modernes.

## 7. REST : principes et bonnes pratiques

REST est un **style architectural**, et non un protocole ou un format de  données.

Ses principes fondamentaux incluent : 
- **Séparation client-serveur** :  interface utilisateur et gestion des données peuvent évoluer  indépendamment. 
- **Stateless** : chaque requête contient les informations nécessaires à son traitement ; le serveur ne dépend pas  d'un contexte de session conservé entre les requêtes pour comprendre la  requête courante. 
- **Cache** : certaines réponses peuvent être mises en cache lorsque les règles HTTP l'autorisent. 
- **Interface uniforme** :  les ressources et leurs représentations sont manipulées selon des règles  cohérentes. 
- **Système en couches** : le client n'a pas besoin de  connaître tous les intermédiaires entre lui et le service. 
- **Code à la  demande** : principe facultatif, rarement central dans les API de  données.

Une API est dite **RESTful** lorsqu'elle respecte suffisamment ces contraintes. 
Utiliser HTTP et JSON ne suffit pas, à lui seul, à garantir  une conception REST.
### 7.1 Nommer les ressources

Privilégier des noms représentant des ressources plutôt que des verbes d'action dans les chemins :

```
GET    /courses
GET    /courses/42
POST   /courses
DELETE /courses/42
```

Éviter, dans une conception orientée ressources, de multiplier des  chemins comme `/getCourses`, `/createCourse` ou `/deleteCourse/42`. 
Les  méthodes HTTP portent déjà une partie de l'intention.

Cette convention reste une pratique de conception : certains traitements  métier ne se représentent pas naturellement comme une simple opération  CRUD.
### 7.2 Stateless ne signifie pas sans authentification

Une API REST peut exiger un jeton à chaque requête. _Stateless_ signifie  notamment que le serveur ne doit pas avoir besoin d'une conversation  implicite conservée entre deux appels pour comprendre le suivant. 
Le  client transmet les informations nécessaires à chaque requête.
## 8. REST, RPC et SOAP

### REST

Le client manipule des ressources à l'aide de méthodes HTTP. C'est une  approche courante pour les API Web.
### RPC

Le client demande l'exécution d'une opération distante, avec des  paramètres. L'interface ressemble davantage à un appel de fonction.
### SOAP

SOAP définit la structure des messages XML et des règles de  communication. Dans l'écosystème SOAP, un document WSDL peut décrire les  opérations et les messages du service. 
En PHP, l'extension SOAP fournit  notamment `SoapClient`, lorsque cette extension est installée et  activée.

Exemple simplifié :

```php
$client = new SoapClient('https://exemple.fr/service?wsdl');
$resultat = $client->__soapCall('RechercherCourse', [$parametres]);
```

Le code réel dépend du WSDL et du contrat du service. 
SOAP n'est pas  synonyme de Web Service : un Web Service peut être fondé sur REST ou sur  une autre approche.

## 9. Authentification, autorisation et sécurité

Ces notions sont proches, mais différentes :

- **Authentification** : vérifier l'identité du client ou de l'utilisateur.
- **Autorisation** : déterminer ce que cette identité a le droit de  faire.
- **Jeton d'accès** (_access token_) : valeur transmise par le client pour présenter ses droits ou son identité selon le mécanisme  utilisé.

Exemple d'en-tête :

```
Authorization: Bearer <jeton>
```

Bonnes pratiques : 
1. utiliser HTTPS ; 
2. ne jamais publier un jeton dans un dépôt Git ou dans le code envoyé au navigateur ; 
3. stocker les secrets dans une configuration protégée ou des variables d'environnement ; 
4. limiter les droits du jeton au strict nécessaire ; 
5. prévoir sa rotation et sa révocation ; 
6. valider les données reçues côté serveur ;  
7. vérifier les autorisations sur chaque opération et chaque ressource ;  
8. limiter le débit des requêtes lorsque nécessaire ; 
9. éviter d'exposer les détails internes, les secrets ou les traces techniques  dans les messages d'erreur ; 
10. journaliser les événements utiles sans  enregistrer les jetons ni les données sensibles.

Il ne faut pas transmettre un jeton dans l'URL : les URL peuvent être conservées dans des journaux, l'historique ou des systèmes  intermédiaires. L'en-tête `Authorization` est généralement préférable.

CORS est un mécanisme de sécurité appliqué par les navigateurs pour contrôler les requêtes inter-origines. 
Ce n'est **ni une  authentification ni un contrôle d'accès côté serveur** ; une API doit  toujours vérifier elle-même l'identité et les droits.
## 10. Interroger une API REST depuis PHP avec cURL

cURL permet à PHP d'effectuer une requête HTTP vers un service distant.  
Le déroulement général est le suivant :

1. initialiser la requête ;
2. configurer l'URL, la méthode, les en-têtes et éventuellement le  corps ;
3. exécuter la requête ;
4. vérifier les erreurs de transport ;
5. lire le code HTTP ;
6. fermer la ressource cURL ;
7. analyser la réponse et traiter les erreurs applicatives.
### 10.1 Lire une ressource avec GET

```php
<?php
declare(strict_types=1);

$url = 'https://api.exemple.fr/courses/42';

$ch = curl_init();

curl_setopt_array($ch, [
    CURLOPT_URL => $url,
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_CONNECTTIMEOUT => 5,
    CURLOPT_TIMEOUT => 20,
    CURLOPT_HTTPHEADER => [
        'Accept: application/json',
    ],
]);

$corps = curl_exec($ch);

if ($corps === false) {
    $erreur = curl_error($ch);
    curl_close($ch);
    throw new RuntimeException('Erreur de transport cURL : ' . $erreur);
}

$codeHttp = curl_getinfo($ch, CURLINFO_HTTP_CODE);
curl_close($ch);

if ($codeHttp < 200 || $codeHttp >= 300) {
    throw new RuntimeException('L’API a renvoyé le statut HTTP ' . $codeHttp);
}

$donnees = json_decode($corps, true);

if (json_last_error() !== JSON_ERROR_NONE) {
    throw new RuntimeException('La réponse de l’API ne contient pas un JSON valide.');
}

// Les données peuvent maintenant être exploitées après validation de leur structure.
```

À comprendre : 
- `CURLOPT_RETURNTRANSFER` permet de récupérer le corps  de la réponse dans une variable ; 
- `CURLOPT_CONNECTTIMEOUT` limite le  temps consacré à l'établissement de la connexion ; 
- `CURLOPT_TIMEOUT` limite la durée totale de la requête ; 
- `curl_exec()` qui renvoie  `false` indique une erreur cURL, mais pas une erreur HTTP ; 
- un statut  HTTP `404` ou `500` ne signifie pas nécessairement que `curl_exec()` renverra `false` ; 
- il faut donc vérifier séparément le transport et le  statut HTTP.

Dans une application de production, on remplacera les exceptions  génériques par une gestion adaptée à l'architecture, à la journalisation  et à l'affichage des erreurs.
### 10.2 Créer une ressource avec POST et JSON

```php
<?php
declare(strict_types=1);

$url = 'https://api.exemple.fr/courses';

$donnees = [
    'nom' => 'Course des hortillonnages',
    'date' => '2026-10-25',
    'distance' => 10,
];

$json = json_encode($donnees, JSON_UNESCAPED_UNICODE);

if ($json === false) {
    throw new RuntimeException('Impossible d’encoder les données en JSON.');
}

$token = getenv('API_TOKEN');

if ($token === false || $token === '') {
    throw new RuntimeException('Le jeton API est absent de la configuration.');
}

$ch = curl_init();

curl_setopt_array($ch, [
    CURLOPT_URL => $url,
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_CONNECTTIMEOUT => 5,
    CURLOPT_TIMEOUT => 20,
    CURLOPT_POST => true,
    CURLOPT_POSTFIELDS => $json,
    CURLOPT_HTTPHEADER => [
        'Content-Type: application/json',
        'Accept: application/json',
        'Authorization: Bearer ' . $token,
    ],
]);

$corps = curl_exec($ch);

if ($corps === false) {
    $erreur = curl_error($ch);
    curl_close($ch);
    throw new RuntimeException('Erreur de transport cURL : ' . $erreur);
}

$codeHttp = curl_getinfo($ch, CURLINFO_HTTP_CODE);
curl_close($ch);

// Une création réussie renvoie souvent 201, mais il faut suivre le contrat de l’API.
if ($codeHttp !== 201) {
    throw new RuntimeException('Échec de la création : statut HTTP ' . $codeHttp);
}

// Certaines API renvoient un corps JSON ; d’autres peuvent renvoyer un corps vide.
```

Pour `PUT`, `PATCH` ou `DELETE`, la méthode peut être précisée avec  `CURLOPT_CUSTOMREQUEST`. La structure exacte du corps et les codes  attendus sont ceux de la documentation de l'API.
### 10.3 Envoyer des données : les principaux formats

`application/json` `Content-Type: application/json` Échange de données  structurées

`application/x-www-form-urlencoded` `Content-Type: application/x-www-form-urlencoded` Formulaire de champs simples

`multipart/form-data` En-tête généré avec sa frontière (_boundary_) Formulaire avec  fichiers et champs

Côté PHP, `$_POST` reçoit généralement les champs des formulaires  `application/x-www-form-urlencoded` et `multipart/form-data`. 
Pour un  corps JSON, on lit généralement le flux d'entrée :

```php
$corps = file_get_contents('php://input');
$donnees = json_decode($corps, true);

if (json_last_error() !== JSON_ERROR_NONE || !is_array($donnees)) {
    http_response_code(400);
    // Renvoyer une erreur structurée adaptée au contrat de l’API.
    exit;
}
```

Il faut aussi vérifier le `Content-Type`, la présence des champs, leurs  types, leurs formats et les règles métier. 
Le décodage JSON ne valide  pas à lui seul les données.

## 11. Concevoir une réponse d'API cohérente

Une API doit documenter les réponses de succès et d'erreur. Une réponse  
JSON peut par exemple être :

```
{
  "message": "Course créée",
  "data": {
    "id": 42,
    "nom": "Course des hortillonnages"
  }
}
```

Ou, en cas de validation invalide :

```
{
  "message": "Les données transmises sont invalides",
  "errors": {
    "date": "La date doit être au format AAAA-MM-JJ."
  }
}
```

Ces structures sont des conventions possibles, pas une norme  universelle. 
L'important est de garder un contrat stable et documenté.  
Le code HTTP doit correspondre au résultat : par exemple `201` pour une  création réussie, `400` ou `422` pour des données invalides selon la  convention choisie, et `204` lorsqu'aucun corps n'est renvoyé.

Une API ne doit pas renvoyer systématiquement `200` pour toutes les  situations : cela empêche les clients de distinguer correctement les  succès des erreurs HTTP.
## 12. Documentation d'une API : que doit-elle préciser ?

Pour utiliser une API sans avoir à deviner son fonctionnement, sa  documentation devrait indiquer :

- l'URL de base et les environnements disponibles ;
- les ressources et les chemins ;
- les méthodes HTTP acceptées ;
- les paramètres de chemin et de requête ;
- les en-têtes requis ;
- le mécanisme d'authentification et les droits nécessaires ;
- le format du corps attendu ;
- les champs obligatoires, leurs types et leurs contraintes ;
- les codes HTTP possibles ;
- la structure des réponses de succès et d'erreur ;
- les limites de débit et les règles de pagination ;
- les éventuelles versions et règles de compatibilité.

OpenAPI est une spécification couramment utilisée pour décrire les API  HTTP, leurs opérations, paramètres, schémas de données et réponses.  
Swagger est le nom d'un ensemble d'outils souvent associés à cette spécification.

## 13. Limitation du nombre de requêtes (_rate limiting_)

Un service public ou partagé doit parfois limiter le nombre de requêtes  afin de prévenir les abus et de protéger ses ressources.

- **Fixed Window Counter** : compte les requêtes dans des fenêtres  fixes (par exemple, une minute). Simple, mais des pointes peuvent  apparaître au changement de fenêtre.
- **Sliding Window** : tient compte d'une période glissante pour lisser la limite.
- **Token Bucket** : des jetons sont ajoutés à un rythme défini ;  chaque requête consomme un jeton. Il permet souvent des pointes  limitées.
- **Leaky Bucket** : régule le débit de sortie ou la capacité d'acceptation selon une logique de seau qui se vide à rythme  contrôlé.

Une API peut signaler le dépassement de limite par  `429 Too Many Requests` et fournir des informations de temporisation,  par exemple `Retry-After`. Le mécanisme choisi doit être cohérent avec  le besoin et fonctionner correctement lorsque plusieurs processus ou serveurs traitent les requêtes.
## 14. Communication synchrone et asynchrone

### Synchrone

Le code appelant attend le résultat avant de poursuivre le traitement  dépendant de ce résultat. 
C'est le fonctionnement courant d'un appel  cURL PHP.
### Asynchrone

Le programme peut poursuivre d'autres opérations pendant qu'une tâche  attend une réponse ou un traitement. 
Dans un navigateur, `fetch()`  renvoie une promesse ; on peut utiliser `await` ou `.then()` pour  traiter la réponse. Dans certains systèmes, le serveur accepte un  travail et le termine plus tard : le client peut alors interroger son  état ou recevoir une notification.

L'asynchronisme ne garantit pas, à lui seul, une meilleure performance.  
Il évite surtout de bloquer inutilement certaines opérations ; il ajoute  aussi de la complexité dans la gestion des erreurs, des délais et de  l'état du traitement.
## 15. Avantages, limites et points de vigilance

### Avantages

- faire communiquer des applications écrites dans des langages  différents ;
- réutiliser des données et des fonctionnalités existantes ;
- séparer l'interface, la logique métier et les données ;
- favoriser l'intégration et l'automatisation ;
- faire évoluer certains composants indépendamment grâce à un contrat  stable.
### Limites

- latence et indisponibilité du réseau ou du fournisseur ;
- dépendance à la disponibilité et aux changements de l'API distante ;
- quotas et coûts éventuels ;
- risques de sécurité en cas de contrôle d'accès ou de validation  insuffisants ;
- besoin de documentation, de tests, de surveillance et de gestion des versions.

Un client robuste prévoit les délais d'attente, traite les codes d'erreur, valide les réponses, évite les nouvelles tentatives aveugles  et respecte les limites de débit. Les opérations qui modifient des  données ne doivent pas être répétées sans réfléchir : une nouvelle  tentative peut provoquer des doublons si l'API ne prévoit pas de  mécanisme adapté.
## 16. Méthode d'analyse d'une requête API

Pour analyser une requête ou déboguer un échange, répondre dans cet  ordre :

1. **Quel est le serveur cible ?** Vérifier le schéma, l'hôte et le chemin.
2. **Quelle ressource est visée ?** Lire le chemin et les paramètres.
3. **Quelle méthode est utilisée ?** `GET`, `POST`, `PUT`, `PATCH` ou `DELETE` ?
4. **Quels en-têtes sont envoyés ?** `Accept`, `Content-Type`,  `Authorization` ?
5. **Y a-t-il un corps ?** Est-il conforme au format annoncé ?
6. **Le transport a-t-il réussi ?** DNS, connexion, TLS, délai  d'attente ?
7. **Quel est le statut HTTP ?** Succès, erreur client ou erreur  serveur ?
8. **Le corps de réponse est-il valide ?** JSON décodable et structure   attendue ?
9. **Les données sont-elles conformes au contrat métier ?**
10. **Que faut-il journaliser ?** Le nécessaire pour diagnostiquer, sans secrets ni données sensibles.
## 17. Exemple complet de lecture d'une requête

Considérons :

```
POST /api/courses HTTP/1.1
Host: api.exemple.fr
Content-Type: application/json
Accept: application/json
Authorization: Bearer <jeton>

{
  "nom": "Course des hortillonnages",
  "date": "2026-10-25",
  "distance": 10
}
```

Interprétation : 
- `POST` demande la création d'une ressource ou  l'exécution d'un traitement défini par l'API ; 
- `/api/courses` est la  cible ; 
- `Host` indique l'hôte ; 
- `Content-Type` annonce que le corps envoyé est du JSON ; 
- `Accept` indique que le client attend du JSON en réponse ; 
- `Authorization` présente un jeton, qui doit être vérifié par le serveur ; 
- le corps contient les données proposées pour la création.

Une réponse possible :

```
HTTP/1.1 201 Created
Content-Type: application/json

{
  "message": "Course créée",
  "data": {
    "id": 42,
    "nom": "Course des hortillonnages",
    "date": "2026-10-25",
    "distance": 10
  }
}
```

Le statut `201` indique que la création a réussi. 
Le corps fournit ici  l'identifiant et la représentation de la ressource créée. 
D'autres API  peuvent renvoyer une structure différente : c'est le contrat documenté  qui fait foi.
## 18. Synthèse à mémoriser

- Une **API** définit une interface logicielle.
- Un **Web Service** expose un service logiciel accessible à distance.
- Une **API REST** utilise le style architectural REST, généralement  au-dessus de HTTP.
- HTTP définit les méthodes, les en-têtes et les codes de statut ; JSON est un format de données.
- Une URI identifie une ressource ; une URL est un type d'URI  permettant de la localiser.
- Les méthodes HTTP courantes sont `GET`, `POST`, `PUT`, `PATCH` et  `DELETE`.
- Le statut HTTP et le corps de réponse doivent être examinés  séparément.
- L'authentification vérifie l'identité ; l'autorisation vérifie les  droits.
- HTTPS, validation, contrôle d'accès, gestion des erreurs et  limitation de débit sont des éléments essentiels.
- La documentation de l'API définit le contrat que le client doit respecter.