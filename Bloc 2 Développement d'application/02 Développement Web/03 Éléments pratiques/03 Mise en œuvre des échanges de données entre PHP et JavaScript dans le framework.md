# Principe général

Le framework met en œuvre deux modes de communication entre le serveur (PHP) et le navigateur (JavaScript) :

1. **Transmission des données lors de la génération d'une page HTML**
2. **Transmission des données lors d'un appel Ajax**

Ces deux mécanismes utilisent le format **JSON**, mais leur mise en œuvre est différente.

# 1. Transmission des données lors du chargement d'une page

## Principe

Lorsqu'un contrôleur génère une page, il peut transmettre des données au JavaScript de la page.

Le principe est le suivant :

```text
Base de données
        │
        ▼
Classe métier
        │
        ▼
Contrôleur PHP
        │
        ▼
Classe Page
        │
        ▼
Template interface.php
        │
        ▼
Balises <script type="application/json">
        │
        ▼
Navigateur
        │
        ▼
JavaScript (getData)
```

Les données sont directement intégrées dans le document HTML.

## Le rôle de la classe Page

Le contrôleur ne construit jamais lui-même le code HTML.

Il décrit simplement la page à afficher.

Exemple :

```php
$page = new Page();

$page
    ->setTitre("Liste des catégories")
    ->setDonnee("lesCategories", Categorie::getAll())
    ->setDonnee("dateMin", Categorie::getDateNaissanceMin())
    ->setDonnee("dateMax", Categorie::getDateNaissanceMax())
    ->addScript("/js/categorie.js")
    ->afficher();
```

La méthode `setDonnee()` mémorise une information destinée au JavaScript.

## Encodage sécurisé des données

Les données sont automatiquement converties en JSON grâce à la classe `ReponseJson`.

Le framework utilise :

```php
ReponseJson::encoderPourHTML($valeur)
```

Cette méthode applique automatiquement les options de sécurité adaptées :

- JSON_UNESCAPED_UNICODE
- JSON_UNESCAPED_SLASHES
- JSON_HEX_TAG
- JSON_HEX_AMP
- JSON_HEX_APOS
- JSON_HEX_QUOT

Le contrôleur n'a donc jamais besoin d'appeler directement `json_encode()`.

## Génération de la balise HTML

Le template principal génère automatiquement une balise :

```html
<script type="application/json" id="lesCategories">
[
    {
        "id":"BE",
        "nom":"Benjamin"
    }
]
</script>
```

Cette balise :

- n'exécute aucun JavaScript ;
- contient uniquement des données JSON ;
- est facilement récupérable côté navigateur.

##  Récupération des données côté JavaScript

Le framework fournit la fonction la fonction **getData()** contenue dans le composant fonction/page.js

Exemple :

```javascript
import {getData} from "/composant/fonction/page.js";

const lesCategories = getData("lesCategories");
```

Cette fonction :

- recherche la balise `<script>` à partir de son 'id' ;
- vérifie son existence ;
- réalise automatiquement le `JSON.parse()`;
- génère une erreur explicite si les données sont absentes ou invalides.

L'utilisation est donc très simple.

## Fonctionnement interne

Le fonctionnement est équivalent au code suivant :

```javascript
const script = document.getElementById("lesCategories");

const lesCategories = JSON.parse(script.textContent);
```

Le framework ajoute simplement des contrôles de sécurité.
# 2. Transmission des données lors d'un appel Ajax

Le second mode de communication est utilisé lorsqu'une page déjà affichée souhaite dialoguer avec le serveur sans être rechargée.

Le principe est le suivant :

```text
JavaScript
      ▼
appelAjax()
      ▼
JSON
      ▼
Script PHP Ajax
      ▼
Classe métier
      ▼
Base de données
      ▼
Réponse JSON
      ▼
appelAjax()
      ▼
fonction der appel success()
```

# 9. Envoi des données

Le framework fournit la fonction **appelAjax()** contenue dans le composant **fonction/ajax.js**

Exemple :

```javascript
appelAjax({
    url: "/ajax/modifier.php",
    data: {
        tableName: "Categorie",
        primaryKey: "BE",
        columns: {
            nom: "Benjamin",
            ageMin: 11,
            ageMax: 13
        }
    }
});
```

Il n'est pas nécessaire d'utiliser `JSON.stringify()`.

Le framework se charge automatiquement :
- de convertir les données en JSON ;
- de définir les en-têtes HTTP appropriés (`Content-Type: application/json`) ;
- d'envoyer la requête au serveur.

Le développeur manipule uniquement des objets JavaScript.
## Traitement côté PHP

Exemple :

```php
declare(strict_types=1);

use ClasseMetier\Categorie;
use ClasseTechnique\Ajax;
use ClasseTechnique\ReponseJson;
use ClasseTechnique\Requete;

// Chargement automatique des classes
require $_SERVER['DOCUMENT_ROOT'] . '/../bootstrap/bootstrap.php';

// Vérification d'un appel AJAX par la méthode POST sécurisé par un jeton
Ajax::exigerPost();

// récupération des données transmises
$columns = Requete::postArray('columns');

// création de l'objet métier
$categorie = new Categorie();

// ajout de la catégorie
$resultat = $categorie->add($columns);

if ($resultat === true) {
    ReponseJson::envoyerMessage("Catégorie ajoutée");
}

// en cas d'erreur
ReponseJson::envoyerLesErreurs($categorie->getErrors());
```

Les scripts Ajax utilisent la classe `Requete`.
La classe :

- lit le corps JSON de la requête ;
- effectue automatiquement le `json_decode()` ;
- vérifie le type attendu ;
- lève une erreur en cas de données invalides.

Le script PHP ne manipule donc jamais directement `$_POST`.

# 11. Traitement métier

Les données sont ensuite transmises à la classe métier.

Exemple :

```php
$table = FabriqueTable::creer($tableName);

$table->modify($primaryKey, $columns);
```

Toute la validation est réalisée dans la classe métier.

# 12. Envoi de la réponse

Les scripts Ajax utilisent la classe `ReponseJson`.

Succès :

```php
ReponseJson::envoyerMessage("Enregistrement modifié");
```

Erreur :

```php
ReponseJson::envoyerLesErreurs($table->getErrors());
```

Le script PHP ne construit jamais lui-même le JSON.

# 13. Traitement de la réponse côté JavaScript

Lorsque le serveur répond avec un succès, la fonction passée dans la propriété `success` est automatiquement exécutée.

Exemple :

```javascript
appelAjax({

    ...

    success: () => {

        afficherToast("Catégorie modifiée");

    }

});
```

Si le serveur renvoie des erreurs de validation, le framework les affiche automatiquement sous les champs concernés.

Le développeur n'a donc généralement rien à écrire pour gérer les erreurs classiques de validation.
# 14. Résumé

Le Framework automatise complètement les échanges entre PHP et JavaScript.

| Communication | Classe / Fonction utilisée |
|---------------|----------------------------|
| PHP → JavaScript (chargement de page) | `Page::setDonnee()` |
| Encodage JSON | `ReponseJson::encoderPourHTML()` |
| Lecture côté JavaScript | `getData()` |
| JavaScript → PHP | `appelAjax()` |
| Lecture des paramètres | `Requete` |
| Traitement métier | classes dérivées de `Table` |
| Réponse JSON | `ReponseJson` |

Grâce à cette architecture, les contrôleurs, les scripts Ajax et le code JavaScript restent très simples. Les conversions JSON, les validations de type, la gestion des erreurs et la construction des réponses sont entièrement prises en charge par les classes techniques du framework.