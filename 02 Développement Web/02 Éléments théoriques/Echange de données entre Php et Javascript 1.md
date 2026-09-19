## Principe général

Une application web est composée de deux environnements distincts :

- **le serveur**, qui exécute les scripts PHP ;
- **le navigateur**, qui exécute le HTML, le CSS et le JavaScript.

Ces deux environnements ne partagent pas directement leurs variables. Lorsqu'une information doit être transmise du serveur vers le navigateur, ou inversement, elle doit être convertie dans un format compris par les deux langages.

Le format **JSON** (*JavaScript Object Notation*) est aujourd'hui le standard utilisé pour ces échanges.

Il permet de représenter simplement :

- des valeurs simples (chaîne de caractères, nombre, booléen, valeur `null`) ;
- des objets ;
- des tableaux ;
- des structures de données complexes composées de plusieurs niveaux.

Le schéma suivant résume le principe.

```text
                Serveur (PHP)
                      │
          Variables PHP (tableaux, objets…)
                      │
               json_encode()
                      │
                      ▼
                 Données JSON
                      │
================= HTTP =================
                      │
                 Données JSON
                      ▲
                JSON.parse()
                      │
          Objets et tableaux JavaScript
                      │
             Navigateur (JavaScript)
```

Le même principe est utilisé dans le sens inverse :

```text
            JavaScript
                  │
        JSON.stringify()
                  │
                  ▼
             Données JSON
                  │
================ HTTP =================
                  │
             Données JSON
                  ▲
             json_decode()
                  │
            Variables PHP
```

# Le format JSON

JSON est un format textuel indépendant des langages de programmation.

Par exemple, un tableau PHP :

```php
$personnes = [
    [
        'nom' => 'Martin',
        'prenom' => 'Paul',
        'age' => 18
    ],
    [
        'nom' => 'Durand',
        'prenom' => 'Julie',
        'age' => 21
    ]
];
```

est converti en JSON sous la forme :

```json
[
    {
        "nom": "Martin",
        "prenom": "Paul",
        "age": 18
    },
    {
        "nom": "Durand",
        "prenom": "Julie",
        "age": 21
    }
]
```

Cette représentation est comprise aussi bien par PHP que par JavaScript.

# Conversion des données côté PHP

PHP fournit la fonction **json_encode()** permettant de convertir une variable PHP en chaîne JSON.

Syntaxe :

```php
$json = json_encode($variable);
```

Cette fonction accepte pratiquement tous les types de données PHP :

- chaînes de caractères ;
- nombres ;
- booléens ;
- tableaux ;
- objets.

Par exemple :

```php
$personnes = [
    [
        'nom' => 'Martin',
        'age' => 18
    ],
    [
        'nom' => 'Durand',
        'age' => 21
    ]
];

$json = json_encode($personnes);
```

La variable `$json` contient alors une chaîne de caractères au format JSON pouvant être envoyée au navigateur.

# Pourquoi utiliser json_encode() sur une chaîne ?

Une chaîne PHP ressemble déjà à une chaîne JSON.

Par exemple :

```php
$message = "Bonjour";
```

pourrait sembler directement exploitable.

Cependant, `json_encode()` présente plusieurs avantages :

- il ajoute automatiquement les guillemets nécessaires ;
- il échappe correctement les caractères spéciaux ;
- il garantit la validité du document JSON.

Par exemple :

```php
$message = 'Il a répondu : "Bonjour"';
```

devient :

```json
"Il a répondu : \"Bonjour\""
```

Les caractères suivants sont notamment correctement échappés :

- les guillemets (`"`) ;
- les antislashs (`\`) ;
- les retours à la ligne (`\n`) ;
- les tabulations (`\t`).

L'utilisation de `json_encode()` est donc recommandée même lorsqu'une simple chaîne doit être transmise.

# Les options de json_encode()

La fonction `json_encode()` accepte un second paramètre permettant de modifier le comportement de l'encodage.

Exemple :

```php
$json = json_encode($donnees, JSON_UNESCAPED_UNICODE);
```

Plusieurs constantes peuvent être combinées grâce à l'opérateur `|`.

Exemple :

```php
$json = json_encode(
    $donnees,
    JSON_UNESCAPED_UNICODE
    | JSON_UNESCAPED_SLASHES
);
```

Les principales options sont les suivantes.

| Constante | Rôle |
|-----------|------|
| `JSON_UNESCAPED_UNICODE` | Conserve les caractères Unicode (accents, emojis...) sans les convertir en séquences `\uXXXX`. |
| `JSON_UNESCAPED_SLASHES` | Empêche l'échappement des caractères `/`. |
| `JSON_HEX_TAG` | Convertit `<` et `>` en séquences Unicode afin d'éviter certaines injections HTML. |
| `JSON_HEX_AMP` | Convertit le caractère `&`. |
| `JSON_HEX_APOS` | Convertit l'apostrophe `'`. |
| `JSON_HEX_QUOT` | Convertit les guillemets `"`. |

Ces options permettent :

- d'améliorer la lisibilité du JSON produit ;
- de limiter certains risques d'injection lorsque les données sont intégrées dans une page HTML ;
- de faciliter l'inspection du JSON lors du développement.

# Conversion côté JavaScript

Lorsque le navigateur reçoit une chaîne JSON, celle-ci doit être reconvertie en objet JavaScript.

Cette opération est réalisée à l'aide de la méthode :

```javascript
const personnes = JSON.parse(chaineJson);
```

Le résultat est directement exploitable :

```javascript
console.log(personnes[0].nom);

for (const personne of personnes) {
    console.log(personne.nom);
}
```

Les objets obtenus se manipulent exactement comme des objets JavaScript classiques.

# Conversion de JavaScript vers PHP

Lorsqu'un navigateur envoie des données au serveur (par exemple lors d'un appel Ajax), un objet JavaScript doit être converti en JSON.

Cette conversion est réalisée avec :

```javascript
const json = JSON.stringify(personnes);
```

Le serveur reçoit alors une chaîne JSON.

Pour retrouver les données PHP correspondantes, on utilise :

```php
$personnes = json_decode($json, true);
```

Le second paramètre (`true`) demande à PHP de produire un tableau associatif.

Sans ce paramètre :

```php
$personnes = json_decode($json);
```

PHP retourne des objets (`stdClass`).

# Résumé

Les échanges entre PHP et JavaScript reposent systématiquement sur le format JSON.

Les principales fonctions utilisées sont :

| Sens de conversion | Fonction |
|--------------------|----------|
| PHP → JSON | `json_encode()` |
| JSON → PHP | `json_decode()` |
| JSON → JavaScript | `JSON.parse()` |
| JavaScript → JSON | `JSON.stringify()` |

Ces quatre fonctions constituent le socle des échanges de données entre le serveur et le navigateur dans une application web moderne.

---

# Mise en œuvre des échanges de données entre PHP et JavaScript dans le framework

## Principe général

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

Aucune requête Ajax n'est nécessaire.

Les données sont directement intégrées dans le document HTML.

# 2. Le rôle de la classe Page

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

# 3. Encodage sécurisé des données

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

# 4. Génération de la balise HTML

Le template principal génère automatiquement une balise :

```html
<script
    type="application/json"
    id="lesCategories">
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

# 5. Récupération des données côté JavaScript

Le framework fournit la fonction :

```javascript
getData()
```

Exemple :

```javascript
import {getData} from "/composant/fonction/page.js";

const lesCategories = getData("lesCategories");
```

Cette fonction :

- recherche la balise `<script>` ;
- vérifie son existence ;
- réalise automatiquement le `JSON.parse()`;
- génère une erreur explicite si les données sont absentes ou invalides.

L'utilisation est donc très simple.

# 6. Fonctionnement interne

Le fonctionnement est équivalent au code suivant :

```javascript
const script = document.getElementById("lesCategories");

const lesCategories = JSON.parse(script.textContent);
```

Le framework ajoute simplement des contrôles de sécurité.

# 7. Exemple complet

## Contrôleur

```php
$page = new Page();

$page
    ->setDonnee("lesCategories", Categorie::getAll())
    ->afficher();
```

↓
## HTML généré

```html
<script
    type="application/json"
    id="lesCategories">
[
    {
        "id":"BE",
        "nom":"Benjamin"
    }
]
</script>
```

↓
## JavaScript

```javascript
const lesCategories = getData("lesCategories");
```

↓
## Résultat

```javascript
[
    {
        id: "BE",
        nom: "Benjamin"
    }
]
```

# 8. Transmission des données lors d'un appel Ajax

Le second mode de communication est utilisé lorsqu'une page déjà affichée souhaite dialoguer avec le serveur sans être rechargée.

Le principe est le suivant :

```text
JavaScript
      │
      ▼
appelAjax()
      │
      ▼
JSON
      │
      ▼
Script PHP Ajax
      │
      ▼
Classe métier
      │
      ▼
Base de données
      │
      ▼
Réponse JSON
      │
      ▼
appelAjax()
      │
      ▼
fonction success()
```

# 9. Envoi des données

Le framework fournit la fonction :

```javascript
appelAjax()
```

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

# 10. Réception des données côté PHP

Les scripts Ajax utilisent la classe `Requete`.

Exemple :

```php
$tableName = Requete::postString('tableName');

$primaryKey = Requete::postScalar('primaryKey');

$columns = Requete::postArray('columns');
```

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