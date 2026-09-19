Dans tous nos développement WEB, les requêtes AJAX doivent être réalisées avec la fonction :

```javascript
appelAjax()
```

Elle se trouve dans :

```javascript
/composant/fonction/ajax.js
```

Il faut donc commencer par l'importer :

```javascript
import { appelAjax } from "/composant/fonction/ajax.js";
```

## 1. À quoi sert `appelAjax()` ?

`appelAjax()` permet à JavaScript de communiquer avec un script PHP **sans recharger la page**.

Par exemple :

```text
Page affichée
     ↓
JavaScript
     ↓
appelAjax()
     ↓
script PHP
     ↓
réponse
     ↓
JavaScript
     ↓
mise à jour de la page
```

La fonction prend en charge automatiquement plusieurs opérations :

- l'envoi de la requête HTTP ;
- la transmission des données ;
- l'envoi du jeton de sécurité ;
- la récupération de la réponse ;
- la gestion des erreurs HTTP ;
- la gestion des erreurs retournées par PHP ;
- la conversion de la réponse JSON.

Le développeur doit donc principalement s'occuper du **traitement de la réponse en cas de succès**.

> **Règle à retenir :** pour appeler un script PHP situé dans un répertoire `ajax/`, utilisez `appelAjax()` et non directement `fetch()`.

# 2. La syntaxe générale

La fonction peut être appelée ainsi :

```javascript
appelAjax({
    url: "...",
    data: ...,
    method: "...",
    success: ...,
    error: ...,
    dataType: "..."
});
```

Les paramètres sont les suivants :

|Paramètre|Rôle|Valeur par défaut|
|---|---|---|
|`url`|Script PHP à appeler|—|
|`data`|Données à transmettre au serveur|`null`|
|`method`|Méthode HTTP utilisée|`"POST"`|
|`success`|Fonction exécutée si la requête réussit|`null`|
|`error`|Fonction supplémentaire de gestion des erreurs|`null`|
|`dataType`|Format de la réponse attendue|`"json"`|

Dans la plupart des cas, **`method` et `dataType` n'ont pas besoin d'être précisés**.

# 3. Le cas le plus simple : appeler un script PHP

Supposons que le module possède le script :

```text
ajax/liste.php
```

On peut simplement écrire :

```javascript
appelAjax({
    url: "ajax/liste.php",
    success: data => {
        afficher(data);
    }
});
```

Le script PHP est appelé et sa réponse est récupérée dans :

```javascript
data
```

Par défaut, `appelAjax()` attend une réponse au format JSON.

Le PHP peut donc retourner par exemple :

```php
echo json_encode($lesCoureurs);
```

et JavaScript récupérera directement le tableau :

```javascript
success: lesCoureurs => {
    afficher(lesCoureurs);
}
```

# 4. Envoyer des données au PHP

Le cas le plus fréquent consiste à transmettre des données au serveur.

Par exemple, pour rechercher un coureur à partir de sa licence :

```javascript
appelAjax({
    url: "ajax/getbylicence.php",

    data: {
        licence: 123456
    },

    success: data => {
        afficher(data);
    }
});
```

Côté PHP, les données sont récupérées avec les outils habituels de l'application :

```php
$licence = Requete::postString('licence');
```

Le principe est donc :

```text
JavaScript
     ↓
data: { licence: 123456 }
     ↓
appelAjax()
     ↓
PHP
     ↓
Requete::postString('licence')
```

## 4.1. Plusieurs données

On peut naturellement transmettre plusieurs valeurs :

```javascript
appelAjax({
    url: "ajax/rechercher.php",

    data: {
        nom: "DUPONT",
        prenom: "Jean",
        age: 25
    },

    success: data => {
        afficher(data);
    }
});
```

Côté PHP :

```php
$nom = Requete::postString('nom');
$prenom = Requete::postString('prenom');
$age = Requete::postInt('age');
```

# 5. Le paramètre `success`

`success` contient la fonction qui sera exécutée lorsque la requête s'est correctement déroulée.

Par exemple :

```javascript
appelAjax({
    url: "ajax/getbylicence.php",

    data: {
        licence: 123456
    },

    success: data => {
        afficher(data);
    }
});
```

`data` contient directement la réponse du serveur.

Il n'est donc pas nécessaire de faire :

```javascript
response.json()
```

ni de tester manuellement le code HTTP.

`appelAjax()` s'en charge.

## 5.1. Si la réponse contient plusieurs données

Le serveur peut retourner un objet :

```php
echo json_encode([
    'nom' => 'DUPONT',
    'prenom' => 'Jean',
    'licence' => '123456'
]);
```

JavaScript reçoit directement cet objet :

```javascript
success: data => {

    console.log(data.nom);
    console.log(data.prenom);
    console.log(data.licence);
}
```

# 6. Exemple courant : enregistrer des données

Voici un exemple typique :

```javascript
appelAjax({
    url: "ajax/enregistrer.php",

    data: {
        nom: nom.value,
        prenom: prenom.value,
        age: age.value
    },

    success: data => {
        messageBox(data.message);
    }
});
```

Le PHP récupère les données :

```php
$nom = Requete::postString('nom');
$prenom = Requete::postString('prenom');
$age = Requete::postInt('age');
```

Le serveur peut ensuite retourner :

```php
echo json_encode([
    'message' => 'Personne enregistrée.'
]);
```

JavaScript reçoit alors :

```javascript
data.message
```

# 7. Envoyer un tableau ou un objet complexe

`appelAjax()` sait également transmettre des données plus complexes.

Par exemple :

```javascript
appelAjax({
    url: "ajax/enregistrer.php",

    data: {
        nom: "Dupont",

        competences: [
            {
                id: 1,
                niveau: 3
            },
            {
                id: 2,
                niveau: 4
            }
        ]
    },

    success: data => {
        console.log(data);
    }
});
```

Il est donc possible de transmettre directement des structures JavaScript.

Côté PHP, on récupère le tableau avec les méthodes prévues par l'application :

```php
$nom = Requete::postString('nom');
$competences = Requete::postArray('competences');
```

# 8. Utiliser `GET`

Par défaut, `appelAjax()` utilise `POST`.

Pour effectuer une requête `GET`, il faut préciser :

```javascript
method: "GET"
```

Par exemple :

```javascript
appelAjax({
    url: "ajax/rechercher.php",

    method: "GET",

    data: {
        search: "DUP"
    },

    success: data => {
        console.log(data);
    }
});
```

La requête correspond alors à une URL de type :

```text
ajax/rechercher.php?search=DUP
```

Côté PHP, les données sont récupérées dans les paramètres `GET`.

Dans notre architecture, on utilisera principalement :

- `GET` pour récupérer des informations ;
- `POST` pour envoyer ou modifier des données.

# 9. `GET` ou `POST` ?

|Méthode|Utilisation habituelle|
|---|---|
|`GET`|Lire ou rechercher des données|
|`POST`|Envoyer, créer, modifier ou supprimer des données|

Exemple de lecture :

```javascript
appelAjax({
    url: "ajax/getbylicence.php",
    method: "GET",
    data: {
        licence: 123456
    },
    success: afficher
});
```

Exemple de modification :

```javascript
appelAjax({
    url: "ajax/modifier.php",
    data: {
        id: 12,
        nom: "DUPONT"
    },
    success: afficher
});
```

Comme `POST` est la valeur par défaut, il n'est généralement pas nécessaire d'écrire :

```javascript
method: "POST"
```

# 10. Envoyer un fichier avec `FormData`

Lorsqu'un fichier doit être envoyé, on utilise `FormData`.

Par exemple :

```javascript
const formData = new FormData();

formData.append("id", 12);
formData.append("photo", photo.files[0]);

appelAjax({
    url: "ajax/upload.php",
    data: formData,

    success: data => {
        console.log(data);
    }
});
```

Côté PHP :

```php
$id = Requete::postInt('id');
```

Le fichier est récupéré dans :

```php
$_FILES
```

`FormData` est donc principalement utilisé pour les **uploads de fichiers**.

# 11. Modifier le type de réponse

Par défaut, `appelAjax()` attend une réponse JSON :

```javascript
dataType: "json"
```

Dans la majorité des cas, il n'est donc pas nécessaire de préciser cette propriété.

## Réponse JSON

```javascript
appelAjax({
    url: "ajax/liste.php",

    success: data => {
        console.log(data);
    }
});
```

C'est le cas standard.

## Réponse texte

Si le serveur renvoie du texte :

```javascript
appelAjax({
    url: "ajax/message.php",

    dataType: "text",

    success: data => {
        console.log(data);
    }
});
```

## Réponse fichier

Pour récupérer une donnée binaire, on peut utiliser :

```javascript
appelAjax({
    url: "ajax/document.php",

    dataType: "blob",

    success: data => {
        // traitement du fichier
    }
});
```

|`dataType`|Utilisation|
|---|---|
|`json`|Cas général|
|`text`|Réponse texte|
|`blob`|Fichier ou donnée binaire|

# 12. Gestion des erreurs

Dans la plupart des cas, **il n'est pas nécessaire de gérer les erreurs dans le code appelant**.

Par exemple :

```javascript
appelAjax({
    url: "ajax/enregistrer.php",

    data: {
        nom: nom.value
    },

    success: data => {
        afficher(data);
    }
});
```

Si le serveur renvoie une erreur, `appelAjax()` la détecte et affiche automatiquement le message approprié.

Elle prend notamment en charge :

- les erreurs réseau ;
- les erreurs HTTP ;
- les erreurs `403` ;
- les erreurs `404` ;
- les erreurs serveur ;
- les erreurs métier retournées par PHP ;
- les réponses JSON invalides.

Le code du développeur peut donc rester concentré sur le **cas où tout s'est bien passé**.

# 13. Le paramètre `error`

Il est possible de fournir une fonction `error` :

```javascript
appelAjax({
    url: "ajax/enregistrer.php",

    data: {
        nom: nom.value
    },

    success: data => {
        afficher(data);
    },

    error: erreur => {
        console.log(erreur);
    }
});
```

Cette fonction est exécutée **en complément** de la gestion automatique des erreurs réalisée par `appelAjax()`.

Elle est donc rarement nécessaire.

Elle peut être utile lorsque le script appelant doit effectuer une opération supplémentaire en cas d'erreur.

Par exemple :

```javascript
appelAjax({
    url: "ajax/enregistrer.php",

    data: {
        nom: nom.value
    },

    success: data => {
        afficher(data);
    },

    error: () => {
        bouton.disabled = false;
    }
});
```

> **Conseil :** ne pas utiliser `error` simplement pour afficher une erreur. `appelAjax()` s'en charge déjà.

# 14. La sécurité est automatique

Les appels réalisés avec `appelAjax()` bénéficient automatiquement du mécanisme de sécurité de l'application.

Le jeton de sécurité est ajouté à la requête par `appelAjax()`.

Le développeur n'a donc pas à écrire :

```javascript
headers: {
    ...
}
```

et ne doit pas gérer lui-même le jeton.

Il suffit d'utiliser :

```javascript
appelAjax({
    url: "ajax/monScript.php",
    ...
});
```

Cette règle est particulièrement importante pour les scripts situés dans les répertoires `ajax/`.

# 15. Exemple complet : recherche

Un exemple très courant est une recherche effectuée à partir de la saisie de l'utilisateur :

```javascript
function rechercher(texte) {

    appelAjax({

        url: "ajax/rechercher.php",

        data: {
            recherche: texte
        },

        success: data => {
            afficherResultats(data);
        }
    });
}
```

Le PHP reçoit :

```php
$recherche = Requete::postString('recherche');
```

Puis retourne les résultats :

```php
echo json_encode($resultats);
```

Le JavaScript reçoit directement :

```javascript
data
```

et peut les afficher :

```javascript
success: data => {
    afficherResultats(data);
}
```

# 16. Exemple complet : suppression

Pour supprimer un élément :

```javascript
function supprimer(id) {

    appelAjax({

        url: "ajax/supprimer.php",

        data: {
            id: id
        },

        success: data => {
            messageBox(data.message);
            actualiserListe();
        }
    });
}
```

Le PHP récupère :

```php
$id = Requete::postInt('id');
```

et effectue la suppression.

Le serveur peut retourner :

```php
echo json_encode([
    'message' => 'Élément supprimé.'
]);
```

Le JavaScript traite uniquement le succès :

```javascript
success: data => {
    messageBox(data.message);
    actualiserListe();
}
```

# 17. Exemple avec une sélection `autoComplete.js`

`appelAjax()` peut naturellement être utilisé avec un composant d'autocomplétion.

Par exemple :

```javascript
const autoCompleteJS = new autoComplete({

    selector: "#search",

    data: {

        src: async (query) => {

            const reponse = await appelAjax({

                url: "ajax/getbyname.php",

                data: {
                    search: query
                },

                success: null
            });

            return reponse ?? [];
        },

        keys: ["nomPrenom"]
    }
});
```

Ici :

```javascript
query
```

contient le texte saisi par l'utilisateur.

`appelAjax()` envoie ce texte au serveur :

```javascript
data: {
    search: query
}
```

et retourne directement les résultats.

# 18. Les erreurs métier

Le serveur peut également retourner une erreur liée aux données fournies par l'utilisateur.

Par exemple :

```text
Le nom est obligatoire.
```

ou :

```text
Cette catégorie existe déjà.
```

Ces erreurs sont également prises en charge par `appelAjax()`.

Le développeur n'a donc normalement pas besoin de faire :

```javascript
if (response.error) {
    ...
}
```

ou :

```javascript
if (response.errors) {
    ...
}
```

La fonction analyse la réponse du serveur et affiche les messages au bon endroit.

Le code du développeur reste donc simple :

```javascript
appelAjax({

    url: "ajax/enregistrer.php",

    data: {
        nom: nom.value,
        age: age.value
    },

    success: data => {
        afficher(data);
    }
});
```

# 19. Bonnes pratiques

## Toujours utiliser `appelAjax()`

Pour les scripts AJAX de l'application :

```javascript
appelAjax({
    url: "ajax/monScript.php"
});
```

et non :

```javascript
fetch("ajax/monScript.php");
```

## Ne pas préciser inutilement les paramètres par défaut

Éviter :

```javascript
appelAjax({
    url: "ajax/liste.php",
    method: "POST",
    dataType: "json",
    data: null
});
```

Préférer :

```javascript
appelAjax({
    url: "ajax/liste.php"
});
```

## Utiliser `POST` pour transmettre des données

Par exemple :

```javascript
appelAjax({
    url: "ajax/enregistrer.php",

    data: {
        nom: nom.value,
        prenom: prenom.value
    },

    success: enregistrerSucces
});
```

## Ne traiter que le succès dans le JavaScript

Privilégier :

```javascript
success: data => {
    afficher(data);
}
```

plutôt que de reproduire dans chaque script la gestion des erreurs HTTP et serveur.

# 20. Les quatre cas à connaître

Pour la majorité des développements, ces quatre exemples suffisent.

### Lire des données

```javascript
appelAjax({
    url: "ajax/liste.php",

    success: data => {
        afficher(data);
    }
});
```

### Envoyer des données

```javascript
appelAjax({
    url: "ajax/enregistrer.php",

    data: {
        nom: nom.value,
        age: age.value
    },

    success: data => {
        afficher(data);
    }
});
```

### Effectuer une recherche en `GET`

```javascript
appelAjax({
    url: "ajax/rechercher.php",

    method: "GET",

    data: {
        recherche: texte
    },

    success: data => {
        afficher(data);
    }
});
```

### Envoyer un fichier

```javascript
const formData = new FormData();

formData.append("fichier", input.files[0]);

appelAjax({
    url: "ajax/upload.php",

    data: formData,

    success: data => {
        afficher(data);
    }
});
```

# 21. À retenir

`appelAjax()` est la fonction AJAX standard de notre application.

Dans la majorité des cas, un appel se résume à :

```javascript
appelAjax({

    url: "ajax/monScript.php",

    data: {
        // données à transmettre
    },

    success: data => {

        // traitement de la réponse
    }
});
```

Les règles essentielles sont :

|Règle|À retenir|
|---|---|
|Fonction à utiliser|`appelAjax()`|
|Script appelé|Toujours dans le répertoire `ajax/` du module|
|Méthode par défaut|`POST`|
|Format de réponse par défaut|`JSON`|
|Données simples|Objet JavaScript dans `data`|
|Fichiers|`FormData`|
|Succès|Traité dans `success`|
|Erreurs|Gérées automatiquement|
|Sécurité|Jeton transmis automatiquement|
|`fetch()`|Ne pas utiliser directement pour les appels AJAX de l'application|

### Le principe à retenir

```text
             JavaScript
                 │
                 │ appelAjax()
                 ▼
          ┌───────────────┐
          │ Script PHP    │
          │    ajax/      │
          └───────────────┘
                 │
                 ▼
             Réponse
                 │
                 ▼
          success(data)
```

**Le développeur décrit ce qu'il veut envoyer et ce qu'il veut faire en cas de succès. `appelAjax()` s'occupe du reste.**

Je pense que cette organisation est plus adaptée à une **documentation de référence pour tes développeurs** : ils peuvent commencer directement par les exemples, puis revenir aux paramètres lorsqu'ils en ont besoin. Elle évite aussi de leur faire apprendre le fonctionnement interne de `fetch()`, qui n'est pas utile pour utiliser correctement votre abstraction `appelAjax()`.