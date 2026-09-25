Les méthodes **GET** et **POST** sont les deux méthodes HTTP les plus utilisées pour transmettre des données entre un navigateur et un serveur.

# 1. La méthode GET

La méthode **GET** transmet les données dans l'URL.

Exemple : https://www.monsite.fr/client.php?id=12&nom=Dupont

Les données sont visibles dans la barre d'adresse.
## Côté JavaScript

```
const params = new URLSearchParams({    id: 12,    nom: 'Dupont'});
fetch('/client.php?' + params.toString());
```
## Côté PHP

Les données sont récupérées dans :

```
$id = $_GET['id'];
$nom = $_GET['nom'];
```

ou :

```
$id = filter_input(INPUT_GET, 'id');
```
## Utilisation

La méthode GET est adaptée :

- aux recherches ;
- aux consultations ;
- aux filtres ;
- aux pages ne modifiant pas les données.

Exemple :

```
/article.php?id=15/produits.php?categorie=informatique
```

# 2. La méthode POST

La méthode **POST** transmet les données dans le corps de la requête HTTP (_body_).

Les données ne sont pas visibles dans l'URL.

Plusieurs formats d'encodage sont possibles.
##  POST avec application/x-www-form-urlencoded

C'est le format historique des formulaires HTML.

## Envoi JavaScript

```
const data = new URLSearchParams();
data.append('id', 12);
data.append('nom', 'Dupont');
fetch('/client.php', { method: 'POST',
                       headers: {'Content-Type': 'application/x-www-form-urlencoded'},
                       body: data});
```

## Requête HTTP

```
POST /client.phpid=12&nom=Dupont
```

## Réception PHP

```
$id = $_POST['id'];$nom = $_POST['nom'];
```

#### Avantages

- Simple.
- Compatible avec tous les serveurs.
- Alimente automatiquement `$_POST`.

#### Inconvénients

- Pas adapté aux fichiers.
- Peu pratique pour les structures complexes.

##  POST avec multipart/form-data

Format utilisé pour les formulaires contenant des fichiers.

### Envoi JavaScript

```
const data = new FormData();
data.append('id', 12);
data.append('nom', 'Dupont');
fetch('/client.php', {method: 'POST', body: data});
```

### Avec un fichier

```
const data = new FormData();
data.append('photo', fichier);
data.append('nom', 'Dupont');
```

Le navigateur génère automatiquement :

```
Content-Type: multipart/form-data; boundary=...
```

### Réception PHP

#### Champs texte

```
$nom = $_POST['nom'];
```

#### Fichiers

```
$fichier = $_FILES['photo'];
```

#### Avantages

- Gère les fichiers.
- Compatible avec les formulaires HTML.
- Alimente automatiquement `$_POST` et `$_FILES`.

#### Inconvénients

- Format plus volumineux.

## POST avec application/json

Très utilisé dans les API modernes.
### Envoi JavaScript

```
fetch('/client.php', {   
 method: 'POST',    
 headers: {'Content-Type': 'application/json'},    
 body: JSON.stringify({id: 12, nom: 'Dupont'})});
```

### Requête HTTP

```
POST /client.php
Content-Type: application/json
{ "id": 12,"nom": "Dupont"}
```

### Réception PHP

Avec JSON :

```
$contenu = file_get_contents('php://input');
$data = json_decode($contenu, true);
```

Puis :

```
$id = $data['id'];
$nom = $data['nom'];
```

## #Important

Dans ce cas :

```
$_POST
```

est généralement vide.
#### Avantages

- Très adapté aux API.
- Gère facilement les objets complexes et les tableaux.

#### Inconvénients

- Nécessite un décodage JSON côté PHP.

# 3. Comparatif des différents formats

| Méthode | Format                            | Envoi JS         | Réception PHP                   |
| ------- | --------------------------------- | ---------------- | ------------------------------- |
| GET     | URL                               | URLSearchParams  | `$_GET`                         |
| POST    | application/x-www-form-urlencoded | URLSearchParams  | `$_POST`                        |
| POST    | multipart/form-data               | FormData         | `$_POST` + `$_FILES`            |
| POST    | application/json                  | JSON.stringify() | `php://input` + `json_decode()` |

# 4. Quelle solution choisir ?

| Besoin                  | Solution recommandée   |
| ----------------------- | ---------------------- |
| Consultation de données | GET                    |
| Formulaire simple       | POST + URLSearchParams |
| Upload de fichiers      | POST + FormData        |
| API REST                | POST + JSON            |

# Lien
+ [[Postman#Onglet Body]]
