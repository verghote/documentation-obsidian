## 1. Introduction

Il existe plusieurs façons d'interroger une API REST. Un simple appel Ajax peut suffire si aucune donnée sensible ou donnée d'authentification n'est nécessaire.

Cependant, la solution la plus couramment utilisée en PHP repose sur la mise en place d'une session cURL qui suit le principe suivant :

- **Initialisation** : `$ch = curl_init();`
- **Configuration des options**
- **Exécution de la requête** : `$reponse = curl_exec($ch);`
- **Vérification des erreurs cURL**
- **Vérification du code de statut HTTP**
- **Fermeture de la session** : `curl_close($ch);`
- **Analyse de la réponse** si besoin
## 2. Configuration des options

Les options permettent d'indiquer :
+ l'URL de l'API, la méthode (`POST`, `GET`, `DELETE`, etc.), 
+ les données transmises dans le corps de la requête
+ les en-têtes (headers) HTTP (`Content-Type`, `Authorization`).

Nous pouvons transmettre chaque option individuellement à l'aide de l'instruction `curl_setopt` :

```php
curl_setopt($ch, CURLOPT_URL, $url);
```

Ou transmettre l'ensemble des options d'un coup par l'instruction `curl_setopt_array` :

```php
$options = [
    CURLOPT_URL => $url,
    CURLOPT_RETURNTRANSFER => true,
];
curl_setopt_array($ch, $options);
```

### Détail des options principales :

- **`CURLOPT_URL`** : Définit l'URL de la requête.
- **`CURLOPT_RETURNTRANSFER`** : Si défini à `true`, la réponse sera retournée sous forme de chaîne de caractères au lieu d'être affichée directement.
- **`CURLOPT_HTTPHEADER`** : Permet d'inclure des en-têtes HTTP personnalisés :
    
    ```php
    CURLOPT_HTTPHEADER => [
        'Content-Type: application/json',
        'Authorization: Bearer ' . $token,
        'User-Agent: verghote' 
    ]
    ```
    
- **`CURLOPT_USERAGENT`** : Permet de spécifier un en-tête `User-Agent` de manière plus simple et directe (obligatoire avec certaines API comme l'API GitHub) :
    
    ```php
    CURLOPT_USERAGENT => "verghote",
    ```
    
- **`CURLOPT_POST`** : La méthode par défaut est `GET`. Pour utiliser la méthode `POST`, il faut définir cette option à `true` et fournir les données à envoyer avec `CURLOPT_POSTFIELDS`.
- **`CURLOPT_CUSTOMREQUEST`** : Indispensable pour effectuer une requête de type `DELETE`, `PATCH` ou `PUT` :
    
    ```php
    CURLOPT_CUSTOMREQUEST => 'DELETE',
    ```
    
- **`CURLOPT_TIMEOUT`** : Le délai d'attente par défaut de cURL est généralement de 300 secondes (5 minutes). Il est conseillé de le réduire :
    
    ```php
    CURLOPT_TIMEOUT => 30,
    ```
    
- **`CURLOPT_CAINFO`** : Si l'API nécessite une validation du certificat SSL (obligatoire avec l'API GitHub) :
    
    ```php
    CURLOPT_CAINFO => 'i:/cacert.pem',
    ```
    
    _(Vous pouvez obtenir un certificat SSL de confiance depuis [curl.se/docs/caextract.html](https://curl.se/docs/caextract.html))._
## 3. La transmission des données par la méthode POST

Il existe différentes façons d'envoyer des données via POST selon le format attendu par l'API :

### A. `form-data` (`multipart/form-data`)

- Principalement utilisé pour l'upload de fichiers et de données de formulaire complexes.
- Permet l'envoi de fichiers binaires.
- Format similaire à celui d'un formulaire HTML classique.
- Côté API, les données se récupèrent via la superglobale `$_POST[]` et `$_FILES[]`.

### B. `x-www-form-urlencoded`

- Format par défaut des formulaires HTML simples.
- Les données sont encodées sous forme de chaîne de requête (`cle1=valeur1&cle2=valeur2`).
- À utiliser pour transmettre des données textuelles simples.
- Côté API, les données se récupèrent via `$_POST[]`.
### C. `raw` (Données brutes / JSON)

- Permet d'envoyer des données dans n'importe quel format (JSON, XML, texte brut).
- **Couramment utilisé pour envoyer du JSON dans les API REST modernes.**
- Côté API, la récupération s'effectue ainsi :
    
    ```php
    $donnees = json_decode(file_get_contents('php://input'), true);
    $nom = $donnees['nom'];
    ```
    
### Résumé des en-têtes `Content-Type` associés :

| **Méthode d'envoi**       | **Valeur de l'en-tête Content-Type** |
| ------------------------- | ------------------------------------ |
| **Raw (JSON)**            | `application/json`                   |
| **x-www-form-urlencoded** | `application/x-www-form-urlencoded`  |
| **form-data**             | `multipart/form-data`                |

_(C'est la documentation de l'API cible qui impose le format à employer)._
## 4. La réponse de l'API

La réponse d'une API REST est le plus souvent retournée au format JSON. Une structure standardisée courante ressemble à ceci :

```
{
   "message": "Description de la réponse ou de l'erreur",
   "data": {
       // Données de la réponse, si applicable
   },
   "errors": [
       {
           "id": "Description de l'erreur concernant ce champ",
           "global": "Description d'une erreur globale"
       }
   ]
}
```

Pour récupérer la réponse brute en PHP :

```php
$reponseJson = curl_exec($ch);
```
## 5. Le code de statut HTTP

La réponse s'accompagne d'un code de statut HTTP (`200`, `201`, `204`, `400`, `415`, `422`, etc.). Il permet d'identifier immédiatement s'il s'agit d'un succès (codes `2xx`) ou d'une erreur (`4xx` / `5xx`).

- Côté **client (PHP/cURL)**, on le récupère via `curl_getinfo` :
    
    ```php
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    ```
    
- Côté **API**, on l'émet à l'aide de la fonction :
    
    ```php
    http_response_code(201);
    ```

## 6. Les vérifications et la gestion des erreurs

Le processus de vérification doit suivre deux étapes :

1. Vérifier si la requête réseau a réussi (`curl_errno`).
2. Analyser le code HTTP et le contenu JSON retourné.
### Exemple de traitement pour une requête de suppression :

```php
// 1. Vérification des erreurs cURL
if (curl_errno($ch)) {
    $erreurCurl = curl_error($ch);
    curl_close($ch);
    Erreur::envoyerReponse("Erreur cURL: " . $erreurCurl, 'global');
}

// 2. Récupération du code HTTP et de la réponse
$httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
$reponse = json_decode($reponseJson, true);
$msg = $reponse['message'] ?? '';

// 3. Fermeture de la connexion cURL
curl_close($ch);

// 4. Analyse du code de retour
if ($httpCode === 200) {
    echo json_encode(['success' => $msg]);
} elseif ($httpCode === 400) {
    $msg .= " : " . ($reponse['details'] ?? '');
    echo json_encode(['error' => $msg]);
} elseif ($httpCode === 401) {
    echo json_encode(['error' => $msg]);
} else {
    Erreur::envoyerReponse(Erreur::getErreurHttp($httpCode), 'global');
}
```

## 7. Exemple complet : Création d'un enregistrement via cURL

```php
<?php
declare(strict_types=1);

// Activation du chargement dynamique des ressources
require $_SERVER['DOCUMENT_ROOT'] . "/include/autoload.php";

// Vérifier les paramètres POST locaux
if (!Std::existe('id', 'nom', 'date', 'distance')) {
    Erreur::envoyerReponse("Paramètres manquants", 'global');
}

// Récupération des paramètres transmis
$id = $_POST['id'];
$nom = $_POST['nom'];
$date = $_POST['date'];
$distance = $_POST['distance'];

// Récupérer le token d'autorisation depuis la variable d'environnement
$token = getenv('CALENDRIER_TOKEN');

// Initialisation de cURL
$url = "http://api/calendrier/";
$ch = curl_init();

// Données à envoyer au format tableau associatif
$data = [
    'id' => $id,
    'nom' => $nom,
    'date' => $date,
    'distance' => $distance
];

// Configuration des options cURL
$options = [
    CURLOPT_URL => $url,
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_TIMEOUT => 30,
    CURLOPT_HTTPHEADER => [
        'Content-Type: application/json',
        'Authorization: Bearer ' . $token
    ],
    CURLOPT_POST => true,
    CURLOPT_POSTFIELDS => json_encode($data)
];
curl_setopt_array($ch, $options);

// Exécution de la requête
$reponseJson = curl_exec($ch);

// Vérification des erreurs cURL
if (curl_errno($ch)) {
    $err = curl_error($ch);
    curl_close($ch);
    Erreur::envoyerReponse("Erreur cURL: " . $err, 'global');
}

// Récupération du code HTTP
$httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);

// Fermeture de la connexion cURL
curl_close($ch);

// Décodage de la réponse JSON
$reponse = json_decode($reponseJson, true);
$msg = $reponse['message'] ?? 'Réponse inconnue';

// Vérification et analyse du code HTTP
if ($httpCode === 201) {
    echo json_encode(['success' => $msg]);
} elseif ($httpCode === 400) {
    if (isset($reponse['details'])) {
        echo json_encode(['error' => $reponse['details']]);
    } else {
        Erreur::envoyerReponse($msg, 'global');
    }
} elseif ($httpCode === 401) {
    Erreur::envoyerReponse($msg, 'global');
} else {
    Erreur::envoyerReponse(Erreur::getErreurHttp($httpCode), 'global');
}
```