#  Présentation 

Ce fichier contient deux fonctions principales :

- `appelAjax()` → fonction principale utilisée dans l’application
- `appelAjaxSimple()` → fonction simplifiée pour des appels externes (API publiques)

#  Rôle de appelAjax()

La fonction `appelAjax()` sert à :

. envoyer des requêtes HTTP vers le serveur PHP  
. récupérer des données sans recharger la page  
. centraliser toute la logique AJAX de l’application  
.  gérer automatiquement :

- les erreurs réseau
- les erreurs serveur
- les erreurs métier (PHP)
- la sécurité CSRF (requêtes POST)

# Principe général

Chargement de la fonction appelAjax

```javascript
import {appelAjax} from "/composant/fonction/ajax.js";
```

Appel

```javascript
appelAjax({
	url,  
	data = null,  
	method = 'POST',  
	success = null,  
	error = null,  
	dataType = 'json' })
```


**url** : script PHP à interroger (toujours placé dans le sous répertoire ajax du module)

**data** : objet Json ou objet formData contenant les données passées en paramètre

**method** : **POST** ou GET

**success** : fonction de rappel qui sera automatiquement exécuté en cas de réussite

**error** : fonction de rappel qui sera automatiquement exécuté en cas d’erreur (en plus de l’erreur déjà affichée par la fonction elle-même)

**dataType** : le type de réponse attendu : **JSON**, TEXT ou blob

#  Les différents types d’appels possibles

## 1. GET simple (lecture de données)

utilisé pour récupérer des informations

```javascript
appelAjax({    
    url: '/ajax/liste.php',    
    method: 'GET'
});
```

PHP récupère les données transmises dans : **$_GET**
## 2. POST avec objet JavaScript (cas le plus courant)

C’est le mode recommandé dans l’application

```javascript
appelAjax({  
    url: '/ajax/modifier.php',  
    data: {  
        tableName : 'Categorie',  
        primaryKey: categorie.id,  
        columns: {  
            nom: nom.value,  
            ageMin: ageMin.value,  
            ageMax: ageMax.value  
        }  
    },  
    success: data => {  
     messageBox(data.message)  
     // mettre à jour le tableau lesCategories  
      categorie.nom = nom.value;  
      categorie.ageMin = ageMin.value;  
      categorie.ageMax = ageMax.value;  
    }  
});
```

Ce que fait automatiquement la fonction :

- Utilise la méthode POST (méthode définie par défaut)
- transforme l’objet en `FormData`
- envoie les données au serveur

PHP récupère les données transmises dans **$_POST**

Il est possible de passer un tableau Javascript, la fonction appelAjax va le convertir automatiquement en JSON

```javascript
function enregistrer() {  
    msg.innerHTML = '';  
    appelAjax({  
        url: 'ajax/enregistrer.php',  
        data: {  
            nom: nom.value,  
            lesCompetences: lesCompetencesDuProjet  
        },  
        success: () => retournerVers('Projet enregistré', '/projet/liste')  
    });  
}
```

Pour récupérer les données côté serveur :
 ```php
 $nom = Requete::postString('nom');  
 $lesCompetences = Requete::postArray('lesCompetences');
 ```
## 3. POST avec FormData (cas avancé)

utilisé surtout pour les fichiers

```
const formData = new FormData();
formData.append('id', 12);
formData.append('image', fileInput.files[0]);
appelAjax({ 
   url: '/ajax/upload.php',
   data: formData 
});
```

Avantage :

- permet l’envoi de fichiers (`$_FILES`)
- recommandé pour le téléversement de fichier (upload)

PHP récupère avec les données et le ou les fichiers transmis respectivement dans  **$_POST et $_FILES**

## 4. POST avec URLSearchParams (cas rare et déconseillé)

utilisé pour des requêtes encodées “à l’ancienne”

```javascript
const params = new URLSearchParams();
params.append('id', 12);
params.append('nom', 'test');
appelAjax({ 
     url: '/ajax/modifier.php',
     data: params
});
```

Particularités :

- envoie les données en `application/x-www-form-urlencoded`
- format proche des formulaires HTML classiques

Mais dans notre architecture :

- **ce cas est rare**
- **déconseillé**

# Sécurité intégrée (important)

Pour les requêtes POST :

un token CSRF est automatiquement ajouté

- invisible pour le développeur
- vérifié côté PHP
- protège contre les attaques externes

# Résumé simple

| Type d’appel           | Usage             | Recommandation               |
| ---------------------- | ----------------- | ---------------------------- |
| GET                    | lecture           | ✔️ courant                   |
| POST + objet JS        | envoi de données  | ✔️ recommandé                |
| POST + FormData        | fichiers / upload | ✔️ obligatoire pour fichiers |
| POST + URLSearchParams | format ancien     | ⚠️ déconseillé               |


# Lien

- [[Méthode Get et méthode POST]]