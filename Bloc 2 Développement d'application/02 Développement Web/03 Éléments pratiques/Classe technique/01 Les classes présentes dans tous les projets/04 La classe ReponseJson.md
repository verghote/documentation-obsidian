## 1. Présentation

La classe `ReponseJson` centralise la production des réponses JSON de l'application.

Elle est principalement utilisée par les contrôleurs appelés en AJAX ou par les API REST.

Son rôle est de prendre en charge :

- la définition du code HTTP ;
- la définition du type MIME ;
- l'encodage des données au format JSON ;
- l'envoi de la réponse ;
- l'arrêt du script après l'envoi de la réponse.

La classe fournit **deux modes d'utilisation** :

1. une méthode générique `envoyer()` permettant de construire librement la réponse ;
2. des méthodes spécialisées correspondant aux cas d'utilisation les plus courants.

Les deux approches peuvent être utilisées indifféremment dans l'application.

# 2. Utilisation : deux possibilités

## 2.1. Utiliser la méthode générique `envoyer()`

La méthode `envoyer()` constitue le point d'entrée générique de la classe.

Elle permet au développeur de définir lui-même complètement le contenu de la réponse et son code HTTP.

Exemple :

```php
ReponseJson::envoyer([
    'message' => 'Opération réalisée avec succès.'
]);
```

Réponse :

```json
{
    "message": "Opération réalisée avec succès."
}
```

Il est également possible de préciser le code HTTP :

```php
ReponseJson::envoyer(
    ['message' => 'Ressource créée.'],
    201
);
```

Réponse :

```json
{
    "message": "Ressource créée."
}
```

avec :

```text
HTTP 201 Created
```

Cette forme est particulièrement utile lorsque la réponse ne correspond pas exactement à l'un des cas prévus par les méthodes spécialisées.


## 2.2. Utiliser les méthodes spécialisées

Pour les situations courantes, la classe propose des méthodes simplifiées.

Par exemple :

```php
ReponseJson::envoyerMessage('Modification effectuée.');
```

est un raccourci pour :

```php
ReponseJson::envoyer([
    'message' => 'Modification effectuée.'
]);
```

Les méthodes spécialisées rendent donc le code des contrôleurs plus explicite et plus lisible.

### Les méthodes disponibles

| Méthode               | Code HTTP par défaut | Utilisation              |
| --------------------- | -------------------: | ------------------------ |
| `envoyer()`           |                `200` | Réponse générique        |
| `envoyerMessage()`    |                `200` | Message de confirmation  |
| `envoyerLesDonnees()` |                `200` | Transmission de données  |
| `envoyerCreation()`   |                `201` | Création d'une ressource |
| `envoyerLesErreurs()` |                `422` | Erreurs de validation    |
| `envoyerErreur()`     |                `400` | Erreur générale          |

Les méthodes spécialisées ne constituent pas des mécanismes différents : elles utilisent toutes la méthode générique `envoyer()`.

# 3. Choisir entre les deux approches

En règle générale :

### Utiliser une méthode spécialisée lorsque le cas correspond

```php
ReponseJson::envoyerMessage('Modification effectuée.');
```

ou :

```php
ReponseJson::envoyerLesDonnees($lesCategories);
```

Cette forme est à privilégier pour les cas courants car elle rend immédiatement compréhensible l'intention du contrôleur.

### Utiliser `envoyer()` lorsque la réponse est particulière

```php
ReponseJson::envoyer([
    'message' => 'Traitement terminé.',
    'total' => $total,
    'resultats' => $resultats
]);
```

La méthode générique évite ainsi de multiplier les méthodes spécialisées pour des besoins occasionnels.

# 4. Pourquoi une classe dédiée ?

Sans cette classe, chaque contrôleur devrait répéter le même traitement :

```php
http_response_code(200);
header('Content-Type: application/json; charset=utf-8');
echo json_encode($donnees);
exit;
```

En centralisant ce traitement dans `ReponseJson` :

- tous les contrôleurs produisent des réponses homogènes ;
- le code des contrôleurs est plus lisible ;
- l'encodage JSON est centralisé ;
- les en-têtes HTTP sont centralisés ;
- les erreurs d'encodage sont gérées de manière uniforme ;
- une évolution de la politique de réponse ne nécessite qu'une seule modification.
    
# 5. Une classe utilitaire statique

La classe est déclarée `final` :

```php
final class ReponseJson
```

Elle possède également un constructeur privé :

```php
private function __construct(){}
```

Il est donc impossible de créer une instance :

```php
$reponse = new ReponseJson();
```

Les méthodes sont appelées directement sur la classe :

```php
ReponseJson::envoyerMessage('Opération réalisée.');
```

La classe ne possède aucun état métier.

# 6. La méthode `envoyer()`

```php
ReponseJson::envoyer(
    mixed $contenu = null,
    int $codeHttp = 200
);
```

C'est la méthode générique de la classe.

Elle permet de transmettre n'importe quelle donnée pouvant être encodée en JSON et de choisir le code HTTP.

### Exemple simple

```php
ReponseJson::envoyer([
    'message' => 'Opération réalisée avec succès.'
]);
```

Réponse :

```json
{
    "message": "Opération réalisée avec succès."
}
```

### Exemple avec des données

```php
ReponseJson::envoyer([
    'nom' => 'Dupont',
    'prenom' => 'Jean',
    'age' => 42
]);
```

### Exemple avec un code HTTP spécifique

```php
ReponseJson::envoyer(
    ['message' => 'Ressource créée.'],
    201
);
```

Cette méthode constitue la solution à utiliser lorsqu'aucune méthode spécialisée ne correspond exactement au besoin.

# 7. La méthode `envoyerMessage()`

```php
ReponseJson::envoyerMessage(string $message);
```

Cette méthode est destinée aux réponses contenant simplement un message de confirmation.

Exemple :

```php
ReponseJson::envoyerMessage('Modification effectuée.');
```

Réponse :

```json
{
    "message": "Modification effectuée."
}
```

Code HTTP :

```text
200 OK
```

Elle est notamment adaptée après :

- une modification ;
- une suppression ;
- une validation ;
- une opération réussie.
# 8. La méthode `envoyerLesDonnees()`

```php
ReponseJson::envoyerLesDonnees(array $lesDonnees);
```

Cette méthode est destinée aux réponses contenant des données.

Exemple :

```php
ReponseJson::envoyerLesDonnees($lesCategories);
```

Réponse possible :

```json
[
    {
        "id": 1,
        "nom": "Senior"
    },
    {
        "id": 2,
        "nom": "Jeune"
    }
]
```

Code HTTP :

```text
200 OK
```

Elle constitue un raccourci pratique lorsque la réponse est simplement constituée d'un tableau de données.

# 9. La méthode `envoyerCreation()`

```php
ReponseJson::envoyerCreation(int|string $id);
```

Cette méthode est destinée à signaler la création d'une nouvelle ressource.

Exemple :

```php
ReponseJson::envoyerCreation($id);
```

Réponse :

```json
{
    "id": 125
}
```

Code HTTP :

```text
201 Created
```

Le code HTTP `201` indique que la requête a entraîné la création d'une nouvelle ressource.

# 10. La méthode `envoyerLesErreurs()`

```php
ReponseJson::envoyerLesErreurs(array $lesErreurs, int $codeHttp = 422);
```

Cette méthode permet de retourner plusieurs erreurs associées à des champs ou à des éléments fonctionnels.

Exemple :

```php
ReponseJson::envoyerLesErreurs(['nom' => 'Le nom est obligatoire.', 'email' => 'Adresse électronique invalide.']);
```

Réponse :

```json
{
    "errors": {
        "nom": "Le nom est obligatoire.",
        "email": "Adresse électronique invalide."
    }
}
```

Code HTTP par défaut :

```text
422 Unprocessable Content
```

Le code HTTP peut être modifié si nécessaire :

```php
ReponseJson::envoyerLesErreurs($lesErreurs, 400);
```

# 11. La méthode `envoyerErreur()`

```php
ReponseJson::envoyerErreur(string $message, int $codeHttp = 400);
```

Cette méthode permet de retourner une erreur générale.

Exemple :

```php
ReponseJson::envoyerErreur('La requête est invalide.');
```

Réponse :

```json
{
    "error": "La requête est invalide."
}
```

Code HTTP :

```text
400 Bad Request
```

Le code HTTP peut être précisé :

```php
ReponseJson::envoyerErreur('Accès interdit.', 403);
```

Réponse :

```json
{
    "error": "Accès interdit."
}
```

avec :

```text
403 Forbidden
```

# 12. Les réponses d'erreur et `appelAjax()`

Les réponses produites par `ReponseJson` sont conçues pour être utilisées avec la fonction JavaScript `appelAjax()`.

Le JavaScript reconnaît notamment deux structures :

### Erreur générale

```json
{
    "error": "Une erreur est survenue."
}
```

Elle est affichée automatiquement comme erreur globale.

### Erreurs associées à des champs

```json
{
    "errors": {
        "login": "Login obligatoire.",
        "email": "Adresse électronique invalide."
    }
}
```

`appelAjax()` peut alors afficher automatiquement chaque erreur au niveau du champ concerné.

Le développeur n'a donc normalement **pas besoin de reproduire ce traitement dans chaque appel JavaScript**.

# 13. Les codes HTTP

Les méthodes spécialisées utilisent un code HTTP adapté à leur utilisation.

| Situation            |                        Code |
| -------------------- | --------------------------: |
| Réponse normale      |                    `200 OK` |
| Ressource créée      |               `201 Created` |
| Erreur générale      |           `400 Bad Request` |
| Erreur de validation | `422 Unprocessable Content` |

La méthode générique `envoyer()` permet toutefois d'utiliser n'importe quel code HTTP approprié :

```php
ReponseJson::envoyer(['error' => 'Accès interdit.'], 403);
```

Le code HTTP décrit le résultat de la requête HTTP tandis que le contenu JSON fournit les informations nécessaires au client.

# 14. Les en-têtes HTTP

Toutes les réponses HTTP utilisent :

```text
Content-Type: application/json; charset=utf-8
```

Le développeur n'a donc pas besoin de définir lui-même cet en-tête.

La classe s'en charge automatiquement dans :

```php
ReponseJson::envoyer()
```

# 15. Encodage JSON

Toutes les réponses HTTP utilisent les options :

```php
JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES | JSON_THROW_ON_ERROR
```

### `JSON_UNESCAPED_UNICODE`

Les caractères accentués restent lisibles.

Par exemple :

```json
{
    "nom": "Éric"
}
```

### `JSON_UNESCAPED_SLASHES`

Les `/` ne sont pas inutilement échappés.

### `JSON_THROW_ON_ERROR`

Une erreur d'encodage provoque une `JsonException`.

L'application peut ainsi traiter cette erreur avec son mécanisme global de gestion des exceptions plutôt que de produire silencieusement une réponse JSON incorrecte.

# 16. Fin systématique du script

Les méthodes de réponse HTTP terminent le script après avoir envoyé la réponse.

La méthode générique possède le type de retour :

```php
never
```

Cela indique qu'elle ne revient jamais à son appelant.

Par conséquent :

```php
ReponseJson::envoyerMessage('Opération réalisée.');

// Ce code ne sera jamais exécuté.
faireQuelqueChose();
```

Il n'est donc pas nécessaire d'ajouter :

```php
exit;
```

après l'appel.

# 17. Transmission de données vers une page HTML

`ReponseJson` possède également :

```php
ReponseJson::encoderPourHtml()
```

Cette méthode est différente de `envoyer()`.

Elle **n'envoie pas de réponse HTTP** et ne termine pas le script.

Elle sert à encoder des données PHP destinées à être intégrées dans une page HTML, par exemple dans une balise :

```html
<script type="application/json">
```

Elle utilise des options d'encodage supplémentaires :

```php
JSON_HEX_TAG
JSON_HEX_AMP
JSON_HEX_APOS
JSON_HEX_QUOT
```

afin de neutraliser les caractères pouvant être interprétés comme du HTML ou du JavaScript.

# 18. Utilisation avec `Page`

Dans l'architecture de l'application, les données spécifiques à une page sont généralement transmises à l'objet `Page`.

Exemple :

```php
use ClasseMetier\Categorie;
use ClasseTechnique\Page;

require $_SERVER['DOCUMENT_ROOT'] . '/../bootstrap/autoload.php';

$page = new Page();

$page
    ->setTitre('Liste des catégories')
    ->setDonnee('lesCategories', Categorie::getAll())
    ->addScript('/composant/html2pdf/html2pdf.bundle.min.js')
    ->afficher();
```

Le contrôleur ne construit donc pas lui-même le document HTML.

Il transmet les données nécessaires à `Page`, qui participe à la génération de l'interface.

Lors de la génération de la page, `interface.php` peut utiliser les mécanismes de `ReponseJson` pour transmettre les données au JavaScript sous forme de JSON sécurisé.

# 19. Exemple complet avec AJAX

Un contrôleur AJAX peut rester extrêmement simple.

```php
use ClasseTechnique\ReponseJson;

$lesCategories = Categorie::getAll();

ReponseJson::envoyerLesDonnees($lesCategories);
```

Pour une opération réussie :

```php
ReponseJson::envoyerMessage('La catégorie a été supprimée.');
```

Pour une erreur de validation :

```php
ReponseJson::envoyerLesErreurs(['nom' => 'Le nom est obligatoire.']);
```

Pour une erreur générale :

```php
ReponseJson::envoyerErreur('La catégorie demandée est introuvable.');
```

Et lorsqu'une réponse particulière est nécessaire :

```php
ReponseJson::envoyer([
    'message' => 'Traitement terminé.',
    'total' => $total,
    'resultats' => $resultats
]);
```

# 20. Recommandation pour les développeurs

Lorsqu'un contrôleur doit retourner une réponse JSON, il est recommandé de procéder dans cet ordre :
### 1. Une méthode spécialisée correspond-elle exactement au besoin ?

Utiliser la méthode spécialisée.

```php
ReponseJson::envoyerMessage('Opération réalisée.');
```

### 2. La réponse est-elle constituée de données ?

Utiliser :

```php
ReponseJson::envoyerLesDonnees($donnees);
```

### 3. La réponse est-elle une création de ressource ?

Utiliser :

```php
ReponseJson::envoyerCreation($id);
```

### 4. S'agit-il d'erreurs de validation ?

Utiliser :

```php
ReponseJson::envoyerLesErreurs($lesErreurs);
```

### 5. S'agit-il d'une erreur générale ?

Utiliser :

```php
ReponseJson::envoyerErreur($message);
```

### 6. Aucun cas spécialisé ne convient ?

Utiliser directement :

```php
ReponseJson::envoyer($contenu, $codeHttp);
```

Cette organisation permet de conserver une API simple tout en laissant au développeur toute la souplesse nécessaire.

# 21. Résumé

`ReponseJson` est le point central de production des réponses JSON de l'application.

Elle propose **deux niveaux d'utilisation** :

```php
ReponseJson::envoyer(...);
```

pour une utilisation générique et totalement libre,

ou :

```php
ReponseJson::envoyerMessage(...);
ReponseJson::envoyerLesDonnees(...);
ReponseJson::envoyerCreation(...);
ReponseJson::envoyerLesErreurs(...);
ReponseJson::envoyerErreur(...);
```

pour les cas d'utilisation courants.

Les méthodes spécialisées sont de simples façades autour de `envoyer()`. Elles permettent d'obtenir un code plus explicite sans imposer de contrainte au développeur.

La classe garantit ainsi :

- une production homogène des réponses JSON ;
- une gestion centralisée des codes HTTP ;
- une définition uniforme des en-têtes ;
- un encodage JSON cohérent ;
- une détection des erreurs d'encodage ;
- une gestion uniforme des erreurs côté client avec `appelAjax()` ;
- une transmission sécurisée des données vers les pages HTML ;
- une terminaison systématique des contrôleurs après l'envoi d'une réponse.
    

**Règle pratique : utiliser la méthode spécialisée lorsqu'elle correspond au besoin ; utiliser `envoyer()` dans tous les autres cas.**