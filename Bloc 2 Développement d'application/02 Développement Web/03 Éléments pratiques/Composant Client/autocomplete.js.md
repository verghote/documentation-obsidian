# 1. Principe

`autoComplete.js` permet d'ajouter une autocomplétion à un champ de saisie.

Le fonctionnement général est le suivant :

```text
1. L'utilisateur saisit du texte
              ↓
2. autoComplete.js recherche dans les données
              ↓
3. Les résultats sont affichés
              ↓
4. L'utilisateur sélectionne un résultat
              ↓
5. L'application récupère l'objet sélectionné
```

Il existe deux façons principales de fournir les données au composant :

- **un tableau JavaScript** déjà disponible dans la page ;
- **une fonction AJAX** permettant de rechercher les données sur le serveur.

Cette documentation présente d'abord la solution avec un tableau, puis la solution avec une requête AJAX.

# 2. Installation

## 2.1. Chargement du composant

Nous utilisons `autoComplete.js` à partir de son CDN :

```html
<script src="https://cdn.jsdelivr.net/npm/@tarekraafat/autocomplete.js@10.2.10/dist/autoComplete.min.js"></script>
```

Nous ne chargeons pas la feuille de style fournie par `autoComplete.js`.

Les styles proposés par la bibliothèque ne sont pas compatibles avec ceux utilisés dans notre application. La présentation du champ et des résultats est donc réalisée avec les feuilles de style de l'application.

**CDN** signifie _Content Delivery Network_. Il s'agit d'un réseau de serveurs permettant de distribuer des fichiers, comme des bibliothèques JavaScript ou des feuilles de style, aux utilisateurs.

Les principaux avantages d'un CDN sont :

- **Rapidité** : le fichier peut être servi depuis un serveur géographiquement proche de l'utilisateur.
- **Simplicité** : il n'est pas nécessaire de stocker la bibliothèque sur notre serveur.
- **Mise en cache** : le fichier peut déjà être présent dans le cache du navigateur.
- **Répartition de la charge** : la distribution des fichiers est assurée par le réseau de serveurs du CDN.
- **Disponibilité de nombreuses bibliothèques** : Bootstrap, jQuery, Vue, React, etc.

# 3. Cas 1 — Les données sont disponibles dans un tableau JavaScript

C'est la solution la plus simple.

Les données sont récupérées côté serveur puis transmises à la page.

Par exemple, le contrôleur PHP peut transmettre la liste des coureurs :

```php
$page = new Page();

$page->setTitre("Recherche sur le nom et prénom")
    ->avecJeton()
    ->addComposant("autocomplete")
    ->setDonnee('lesCoureurs', Coureur::getListe())
    ->afficher();
```

La liste peut ensuite être récupérée en JavaScript :

```js
const lesCoureurs = getData('lesCoureurs');
```

Nous disposons alors d'un tableau JavaScript.

Chaque élément du tableau peut par exemple avoir cette forme :

```js
{
    licence: "123456",
    nom: "DUPONT",
    prenom: "Jean",
    nomPrenom: "DUPONT Jean"
}
```

## 3.1. Exemple complet

Voici une configuration complète d'`autoComplete.js` utilisant directement le tableau `lesCoureurs` :

```js
const autoCompleteJS = new autoComplete({
    selector: "#search",

    threshold: 1,

    debounce: 300,

    data: {
        src: lesCoureurs,
        cache: true,
        keys: ["nomPrenom"],
    },

    searchEngine: "loose",

    resultsList: {
        maxResults: 10,
        noResults: true,

        element: (list, data) => {

            if (!data.results.length) {

                const message = document.createElement("li");

                message.textContent = "Aucun coureur trouvé.";
                message.setAttribute("class", "no_result");

                list.appendChild(message);
            }
        }
    },

    resultItem: {
        highlight: true,

        element: (item, data) =>
            item.innerHTML = `<span>${data.match}</span>`
    },

    events: {
        input: {
            selection: (event) => {

                const selection =
                    event.detail.selection.value;

                search.value = selection.nomPrenom;

                rechercher(selection.licence);
            }
        }
    }
});
```

## 3.2. Les propriétés utilisées

Les principales propriétés utilisées dans cet exemple sont :

|Propriété|Rôle|
|---|---|
|`selector`|Indique le champ HTML sur lequel l'autocomplétion est activée.|
|`threshold`|Nombre minimal de caractères à saisir avant de lancer la recherche.|
|`debounce`|Délai d'attente, en millisecondes, avant d'effectuer la recherche après une saisie.|
|`data.src`|Source des données utilisées pour l'autocomplétion. Ici, il s'agit du tableau `lesCoureurs`.|
|`data.cache`|Active ou désactive le mécanisme de cache d'`autoComplete.js`.|
|`data.keys`|Indique la ou les propriétés des objets dans lesquelles effectuer la recherche.|
|`searchEngine`|Définit le moteur utilisé pour rechercher les correspondances. Ici, `"loose"` permet une recherche souple.|
|`resultsList.maxResults`|Nombre maximal de résultats affichés.|
|`resultsList.noResults`|Active la gestion du cas où aucun résultat n'est trouvé.|
|`resultsList.element`|Permet de personnaliser le contenu affiché lorsqu'aucun résultat n'est trouvé.|
|`resultItem.highlight`|Met en évidence la partie du résultat correspondant à la recherche.|
|`resultItem.element`|Permet de personnaliser l'affichage d'un résultat.|
|`events.input.selection`|Fonction exécutée lorsqu'un utilisateur sélectionne un résultat.|

### Les quatre propriétés essentielles

Pour une première utilisation, il faut principalement retenir :

|Propriété|Question à laquelle elle répond|
|---|---|
|`selector`|**Sur quel champ ?**|
|`data.src`|**Dans quelles données ?**|
|`data.keys`|**Dans quelle propriété rechercher ?**|
|`events.input.selection`|**Que faire après la sélection ?**|

Les autres propriétés servent principalement à régler le comportement et l'affichage du composant.

## 3.3. Comprendre `selector`

```js
selector: "#search"
```

Cette propriété indique le champ HTML sur lequel l'autocomplétion doit être activée.

Le HTML contient donc un champ correspondant :

```html
<input id="search"
       type="text"
       autocomplete="off">
```

Le lien est :

```text
selector: "#search"
        ↓
<input id="search">
```

## 3.4. Comprendre `data.src` et `data.keys`

Dans notre exemple :

```js
data: {
    src: lesCoureurs,
    cache: true,
    keys: ["nomPrenom"]
}
```

Les propriétés de `data` sont :

|Propriété|Rôle|
|---|---|
|`src`|Source des données utilisées par l'autocomplétion.|
|`cache`|Indique si le mécanisme de cache est activé.|
|`keys`|Indique les propriétés dans lesquelles la recherche doit être effectuée.|

### `src`

```js
src: lesCoureurs
```

`src` indique que les données sont contenues dans le tableau :

```js
lesCoureurs
```

`autoComplete.js` va donc rechercher directement dans ce tableau.

### `keys`

```js
keys: ["nomPrenom"]
```

Les éléments du tableau sont des objets.

Par exemple :

```js
{
    licence: "123456",
    nom: "DUPONT",
    prenom: "Jean",
    nomPrenom: "DUPONT Jean"
}
```

`keys` indique à `autoComplete.js` dans quelle propriété rechercher.

Ici :

```js
keys: ["nomPrenom"]
```

signifie que la recherche est effectuée dans :

```js
coureur.nomPrenom
```

Ainsi, si l'utilisateur saisit :

```text
dup
```

`autoComplete.js` recherche les correspondances dans la propriété `nomPrenom`.

## 3.5. Comprendre `threshold` et `debounce`

### `threshold`

```js
threshold: 1
```

Indique le nombre minimal de caractères nécessaires avant de lancer la recherche.

Ici, la recherche commence dès que l'utilisateur saisit :

```text
1 caractère
```

### `debounce`

```js
debounce: 300
```

Indique un délai de 300 millisecondes avant d'effectuer la recherche.

Cela évite de recalculer les résultats à chaque frappe lorsque l'utilisateur saisit rapidement plusieurs caractères.

Ce paramètre est particulièrement intéressant lorsque la source des données est obtenue par AJAX.

## 3.6. Limiter et personnaliser les résultats

La configuration :

```js
resultsList: {
    maxResults: 10,
    noResults: true
}
```

permet de contrôler la liste des résultats.

|Propriété|Rôle|
|---|---|
|`maxResults`|Limite le nombre de résultats affichés.|
|`noResults`|Permet de gérer le cas où aucun résultat n'est trouvé.|
|`element`|Permet de personnaliser la liste affichée.|

Dans notre exemple :

```js
maxResults: 10
```

signifie qu'au maximum dix résultats seront affichés.

## 3.7. Personnaliser l'affichage d'un résultat

La configuration :

```js
resultItem: {
    highlight: true,

    element: (item, data) =>
        item.innerHTML = `<span>${data.match}</span>`
}
```

permet de personnaliser l'affichage des résultats.

|Propriété|Rôle|
|---|---|
|`highlight`|Met en évidence la partie correspondant à la recherche.|
|`element`|Permet de personnaliser le HTML d'un résultat.|

Cette partie concerne uniquement **l'affichage** des résultats.

Elle n'intervient pas dans la récupération des données.

## 3.8. Récupérer le résultat sélectionné

Lorsqu'un utilisateur sélectionne un résultat, l'événement :

```js
events: {
    input: {
        selection: (event) => {
            ...
        }
    }
}
```

est déclenché.

L'objet sélectionné est récupéré avec :

```js
const selection = event.detail.selection.value;
```

`selection` contient **l'objet complet** provenant du tableau.

Par exemple :

```js
{
    licence: "123456",
    nom: "DUPONT",
    prenom: "Jean",
    nomPrenom: "DUPONT Jean"
}
```

Nous pouvons donc accéder à toutes ses propriétés :

```js
selection.nom
selection.prenom
selection.licence
selection.nomPrenom
```

## 3.9. Effectuer un traitement après la sélection

Dans notre exemple :

```js
selection: (event) => {

    const selection =
        event.detail.selection.value;

    search.value =
        selection.nomPrenom;

    rechercher(selection.licence);
}
```

Deux opérations sont réalisées.

### Afficher le choix dans le champ

```js
search.value = selection.nomPrenom;
```

Le champ contient alors :

```text
DUPONT Jean
```

### Utiliser une donnée de l'objet sélectionné

```js
rechercher(selection.licence);
```

La licence peut par exemple être utilisée pour récupérer des informations complémentaires :

```js
function rechercher(licence) {

    appelAjax({
        url: 'ajax/getbylicence.php',

        data: {
            licence: licence
        },

        success: afficher
    });
}
```

Le fonctionnement est donc :

```text
lesCoureurs
     ↓
autoComplete.js
     ↓
saisie de l'utilisateur
     ↓
recherche dans le tableau
     ↓
sélection d'un coureur
     ↓
event.detail.selection.value
     ↓
récupération de la licence
     ↓
appelAjax()
     ↓
getbylicence.php
     ↓
affichage des informations
```

**Important :** dans cet exemple, AJAX n'est pas utilisé pour l'autocomplétion.

La recherche est effectuée directement dans le tableau JavaScript.

La requête AJAX intervient uniquement **après la sélection du coureur**.

# 4. Cas 2 — Les données sont récupérées par AJAX

Lorsque le nombre de données est important, il peut être préférable de ne pas charger toute la liste dans le navigateur.

Dans ce cas, `data.src` ne reçoit plus directement un tableau.

Il reçoit une fonction :

```js
src: async (query) => {
    ...
}
```

`autoComplete.js` appelle cette fonction lorsqu'une recherche est effectuée.

Le serveur peut alors rechercher les données dans la base et retourner uniquement les résultats nécessaires.

## 4.1. Exemple complet

```js
const autoCompleteJS = new autoComplete({
    selector: "#search",

    threshold: 1,

    debounce: 300,

    data: {

        src: async (query) => {

            const reponse = await appelAjax({
                url: "ajax/getbyname.php",
                method: "GET",
                data: {
                    search: query
                },
                dataType: "json"
            });

            return reponse ?? [];
        },

        keys: ["nomPrenom"],

        cache: false
    },

    searchEngine: "loose",

    resultsList: {
        maxResults: 10,
        noResults: true,

        element: (list, data) => {

            if (!data.results.length) {

                const message =
                    document.createElement("li");

                message.textContent =
                    "Aucun étudiant trouvé."
                    + (
                        data.query
                            ? ` pour "${data.query}"`
                            : ""
                    );

                message.setAttribute(
                    "class",
                    "no_result"
                );

                list.appendChild(message);
            }
        }
    },

    resultItem: {
        highlight: true,

        element: (item, data) =>
            item.innerHTML = `<span>${data.match}</span>`
    },

    events: {
        input: {
            selection: (event) => {

                const selection =
                    event.detail.selection.value;

                search.value =
                    selection.nomPrenom;

                afficher(selection);
            }
        }
    }
});
```

## 4.2. Les propriétés utilisées avec une source AJAX

La configuration reste pratiquement identique à celle utilisée avec un tableau.

La principale différence concerne `data.src`.

|Propriété|Rôle|
|---|---|
|`selector`|Indique le champ HTML sur lequel l'autocomplétion est activée.|
|`threshold`|Nombre minimal de caractères avant de lancer une recherche.|
|`debounce`|Délai d'attente avant d'effectuer la recherche.|
|`data.src`|Fonction appelée pour récupérer les données auprès du serveur.|
|`data.keys`|Indique la ou les propriétés dans lesquelles rechercher.|
|`data.cache`|Active ou désactive le cache.|
|`searchEngine`|Définit le moteur de recherche utilisé.|
|`resultsList.maxResults`|Nombre maximal de résultats affichés.|
|`resultsList.noResults`|Permet de gérer l'absence de résultat.|
|`resultsList.element`|Permet de personnaliser l'affichage lorsqu'il n'y a aucun résultat.|
|`resultItem.highlight`|Met en évidence la correspondance dans le résultat.|
|`resultItem.element`|Permet de personnaliser l'affichage d'un résultat.|
|`events.input.selection`|Fonction exécutée lorsqu'un résultat est sélectionné.|

La plupart des propriétés sont donc communes aux deux solutions.

C'est principalement **la source des données** qui change.

## 4.3. Comprendre `query`

Avec une source AJAX :

```js
src: async (query) => {
    ...
}
```

`query` représente le texte actuellement saisi par l'utilisateur.

Cette valeur est fournie automatiquement par `autoComplete.js`.

Par exemple, si l'utilisateur saisit :

```text
dup
```

alors :

```js
query === "dup"
```

Cette valeur peut être envoyée au serveur :

```js
data: {
    search: query
}
```

Le serveur reçoit alors :

```text
search=dup
```

Il peut utiliser cette valeur pour effectuer sa recherche dans la base de données.

Le fonctionnement est donc :

```text
Utilisateur saisit "dup"
          ↓
autoComplete.js
          ↓
query = "dup"
          ↓
appelAjax()
          ↓
getbyname.php
          ↓
Recherche en base de données
          ↓
Retour des résultats
          ↓
autoComplete.js
```

## 4.4. Comprendre `async` et `await`

Une requête AJAX n'est pas instantanée.

Le navigateur doit :

1. envoyer une requête au serveur ;
2. attendre son traitement ;
3. recevoir la réponse.

La fonction `src` est donc déclarée avec :

```js
async
```

```js
src: async (query) => {
    ...
}
```

`async` indique que cette fonction est **asynchrone**.

Elle peut ainsi utiliser :

```js
await
```

pour attendre le résultat d'une opération asynchrone.

Dans notre exemple :

```js
const reponse = await appelAjax({
    ...
});
```

Le programme attend que `appelAjax()` ait obtenu la réponse avant de continuer.

Une fois la réponse reçue :

```js
return reponse ?? [];
```

retourne les données à `autoComplete.js`.

On peut résumer :

```text
src(query)
    ↓
appelAjax()
    ↓
attente de la réponse
    ↓
réponse reçue
    ↓
return
    ↓
autoComplete.js
```

## 4.5. Pourquoi utiliser `appelAjax()` plutôt que `fetch()` ?

Dans notre application, les appels vers les scripts AJAX doivent passer par :

```js
appelAjax()
```

Cette fonction est utilisée pour effectuer les requêtes AJAX de l'application et prend notamment en charge les **mécanismes de protection et de contrôle d'accès** nécessaires à nos scripts.

Il ne faut donc pas remplacer :

```js
const reponse = await appelAjax({
    url: "ajax/getbyname.php",
    method: "GET",
    data: {
        search: query
    },
    dataType: "json"
});
```

par un simple :

```js
fetch("ajax/getbyname.php?search=" + query);
```

`fetch()` est une API JavaScript standard permettant d'effectuer une requête HTTP.

Cependant, `fetch()` ne connaît pas les mécanismes de protection propres à notre application.

**Dans notre architecture, les appels vers les scripts AJAX doivent donc passer par `appelAjax()`.**

## 4.6. Le rôle de `dataType`

Dans notre exemple :

```js
dataType: "json"
```

indique que la réponse attendue du serveur est au format JSON.

Le script PHP doit donc retourner des données JSON correspondant aux résultats de la recherche.

Par exemple :

```json
[
    {
        "licence": "123456",
        "nom": "DUPONT",
        "prenom": "Jean",
        "nomPrenom": "DUPONT Jean"
    },
    {
        "licence": "789012",
        "nom": "DUPUIS",
        "prenom": "Paul",
        "nomPrenom": "DUPUIS Paul"
    }
]
```

## 4.7. Le rôle de `return`

La fonction `src` doit retourner les données à `autoComplete.js`.

Dans notre exemple :

```js
return reponse ?? [];
```

Si la requête retourne un tableau :

```js
reponse
```

ce tableau est transmis à `autoComplete.js`.

Le composant peut alors effectuer son traitement et afficher les résultats.

## 4.8. Pourquoi utiliser `?? []` ?

L'opérateur :

```js
??
```

est appelé **opérateur de coalescence nulle**.

Dans :

```js
return reponse ?? [];
```

cela signifie :

> Retourner `reponse` si elle existe, sinon retourner un tableau vide.

Ainsi :

```js
reponse
```

est retournée lorsqu'elle contient une valeur.

Si :

```js
reponse === null
```

alors :

```js
[]
```

est retourné.

Cela garantit que `autoComplete.js` reçoit toujours un tableau de données.

## 4.9. Format des données retournées

Le serveur doit retourner un tableau d'objets compatible avec la configuration de `keys`.

Par exemple :

```js
[
    {
        licence: "123456",
        nom: "DUPONT",
        prenom: "Jean",
        nomPrenom: "DUPONT Jean"
    },
    {
        licence: "789012",
        nom: "DUPUIS",
        prenom: "Paul",
        nomPrenom: "DUPUIS Paul"
    }
]
```

Avec :

```js
keys: ["nomPrenom"]
```

`autoComplete.js` recherche dans :

```js
nomPrenom
```

La structure des objets retournés par le serveur doit donc être cohérente avec les propriétés utilisées par le composant.

## 4.10. Récupérer la sélection

La récupération du résultat sélectionné fonctionne de la même manière qu'avec une source sous forme de tableau :

```js
const selection =
    event.detail.selection.value;
```

`selection` contient l'objet retourné par le serveur.

Par exemple :

```js
{
    licence: "123456",
    nom: "DUPONT",
    prenom: "Jean",
    nomPrenom: "DUPONT Jean"
}
```

On peut donc utiliser :

```js
selection.nom
selection.prenom
selection.licence
selection.nomPrenom
```

Le changement de source — tableau ou AJAX — ne modifie donc pas le fonctionnement de la sélection.

# 5. Choisir entre tableau et AJAX

Le choix dépend principalement de la quantité de données et de l'endroit où la recherche doit être effectuée.

|Situation|Solution recommandée|
|---|---|
|Peu de données|Tableau JavaScript|
|Liste raisonnablement petite|Tableau JavaScript|
|Données déjà disponibles au chargement de la page|Tableau JavaScript|
|Beaucoup de données|AJAX|
|Données provenant d'une grande table en base de données|AJAX|
|Recherche devant être effectuée côté serveur|AJAX|
|Données susceptibles de changer fréquemment|AJAX|

## Source sous forme de tableau

```js
data: {
    src: lesCoureurs,
    keys: ["nomPrenom"]
}
```

Toutes les données sont déjà présentes dans le navigateur.

```text
Page
 ↓
lesCoureurs
 ↓
autoComplete.js
 ↓
Recherche locale
```

### Avantages

- Mise en œuvre simple.
- Recherche rapide.
- Aucune requête réseau lors de la saisie.
- Très adaptée aux petites listes.

### Inconvénients

- Toutes les données sont envoyées au navigateur.
- La page peut devenir lourde avec une grande quantité de données.
- Les données peuvent devenir obsolètes après le chargement de la page.

## Source AJAX

```js
data: {
    src: async (query) => {
        // récupération des données
    }
}
```

Les données sont récupérées uniquement lorsque l'utilisateur effectue une recherche.

```text
Saisie
 ↓
query
 ↓
appelAjax()
 ↓
Serveur
 ↓
Base de données
 ↓
Résultats
 ↓
autoComplete.js
```

### Avantages

- Ne nécessite pas de charger toute la liste.
- Adaptée aux grandes quantités de données.
- La recherche peut être effectuée directement dans la base de données.
- Les données sont récupérées au moment de la recherche.

### Inconvénients

- Nécessite des requêtes réseau.
- Le serveur doit traiter les recherches.
- Le code est légèrement plus complexe.
- La vitesse dépend notamment du réseau et du serveur.

# 6. Configuration minimale à retenir

Pour utiliser `autoComplete.js`, quatre éléments sont essentiels.

## 6.1. Indiquer le champ

```js
selector: "#search"
```

**Sur quel champ l'autocomplétion doit-elle fonctionner ?**

## 6.2. Indiquer la source

### Tableau

```js
data: {
    src: lesCoureurs
}
```

### AJAX

```js
data: {
    src: async (query) => {
        // récupération des données
    }
}
```

**Où trouver les données ?**

## 6.3. Indiquer la propriété recherchée

```js
keys: ["nomPrenom"]
```

**Dans quelle propriété des objets effectuer la recherche ?**

## 6.4. Traiter la sélection

```js
events: {
    input: {
        selection: (event) => {

            const selection =
                event.detail.selection.value;

            // traitement de l'objet sélectionné
        }
    }
}
```

**Que faire lorsque l'utilisateur sélectionne un résultat ?**

# 7. Synthèse

Le fonctionnement d'`autoComplete.js` peut être résumé ainsi :

```text
                       ┌── Tableau JavaScript
                       │
                       │
Champ de saisie ───────┤
                       │
                       └── Fonction AJAX
                              ↓
                       autoComplete.js
                              ↓
                         Recherche
                              ↓
                          Résultats
                              ↓
                          Sélection
                              ↓
                 event.detail.selection.value
                              ↓
                     Objet sélectionné
                              ↓
                      Traitement métier
```

Le point essentiel est que **la source des données peut changer sans modifier le principe de sélection**.

Avec un tableau :

```js
data: {
    src: lesCoureurs
}
```

Avec AJAX :

```js
data: {
    src: async (query) => {
        // récupération des données
    }
}
```

Dans les deux cas, l'objet sélectionné est récupéré de la même manière :

```js
const selection =
    event.detail.selection.value;
```

`autoComplete.js` se charge donc principalement de **la recherche et de l'affichage des résultats**.

L'application reste responsable du **traitement métier effectué après la sélection**.