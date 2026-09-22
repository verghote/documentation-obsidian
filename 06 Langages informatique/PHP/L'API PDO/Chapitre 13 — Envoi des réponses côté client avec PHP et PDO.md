## Objectifs

À la fin de ce chapitre, vous serez capable de :

- transmettre les résultats obtenus avec PDO vers l'interface utilisateur ;
- convertir des données PHP en JSON ;
- préparer des données avant affichage ;
- comprendre le fonctionnement d'un échange entre PHP et JavaScript ;
- gérer le cas particulier des appels Ajax.

---

# 13.1 Introduction

Dans une application Web moderne, PHP joue généralement le rôle de serveur.

Son fonctionnement est souvent le suivant :

```
Navigateur (client)
        |
        | Requête HTTP
        v
Script PHP
        |
        | Requête PDO
        v
Base de données
        |
        | Résultat SQL
        v
Script PHP
        |
        | Réponse JSON ou HTML
        v
Navigateur
```

PHP récupère les données avec PDO puis doit les transmettre au navigateur.

Le format le plus utilisé pour les échanges entre serveur et client est :

```
JSON
```

---

# 13.2 Conversion d'un tableau PHP en JSON

PDO retourne généralement des tableaux PHP :

```
[
    [
        "id" => 1,
        "nom" => "Martin"
    ],
    [
        "id" => 2,
        "nom" => "Dupont"
    ]
]
```

Pour envoyer ces données au navigateur :

```
echo json_encode($donnees);
```

Résultat envoyé :

```
[
    {
        "id":1,
        "nom":"Martin"
    },
    {
        "id":2,
        "nom":"Dupont"
    }
]
```

JavaScript peut directement exploiter cette structure.

---

# 13.3 Préparer des données avant envoi

Le serveur peut enrichir les données avant transmission.

Exemple :

Table :

```
etudiant

id
nom
photo
```

La base contient :

```
photo = "martin.jpg"
```

Avant envoi, on ajoute une information indiquant si le fichier existe.

```
$sql = "
SELECT id,
       nom,
       photo
FROM etudiant
";


$db = Database::getInstance();

$stmt = $db->query($sql);


$etudiants =
    $stmt->fetchAll(PDO::FETCH_ASSOC);



$rep = RACINE . "/data/photos";


foreach ($etudiants as &$etudiant) {

    $etudiant["present"] =
        isset($etudiant["photo"])
        &&
        file_exists(
            "$rep/{$etudiant["photo"]}"
        );

}


echo json_encode($etudiants);
```

Résultat :

```
[
    {
        "id":1,
        "nom":"Martin",
        "photo":"martin.jpg",
        "present":true
    }
]
```

Le client n'a donc pas besoin de vérifier lui-même l'existence du fichier.

---

# 13.4 Transmission dans une page PHP classique

Dans une application générant directement une page HTML, on peut transmettre des données JavaScript depuis PHP.

Exemple :

```
$data = json_encode(
    Etudiant::getAll()
);


$head = <<<HTML
<script>

let etudiants = $data;

</script>
HTML;


require RACINE .
"/include/interface.php";
```

---

La variable :

```
$head
```

est ensuite utilisée dans le modèle HTML :

```
<head>

...

<?php

if (isset($head)) {

    echo $head;

}

?>

...

</head>
```

Le navigateur reçoit alors :

```
<script>

let etudiants = [
    {
        "nom":"Martin"
    }
];

</script>
```

---

# 13.5 Utilisation côté JavaScript

Le tableau PHP devient un objet JavaScript.

Exemple :

```
console.log(etudiants);
```

Résultat :

```
[
 {
   nom:"Martin"
 }
]
```

On peut ensuite l'utiliser :

```
etudiants.forEach(
    etudiant => {

        console.log(
            etudiant.nom
        );

    }
);
```

---

# 13.6 Pourquoi utiliser JSON ?

JSON présente plusieurs avantages :

- format léger ;
- reconnu directement par JavaScript ;
- indépendant du langage utilisé ;
- adapté aux échanges Web.

Exemple :

PHP :

```
[
    "nom" => "Martin"
]
```

JSON :

```
{
    "nom":"Martin"
}
```

JavaScript :

```
objet.nom
```

---

# 13.7 Cas particulier : les appels Ajax

Ajax permet au navigateur d'appeler un script PHP sans recharger toute la page.

Le fonctionnement devient :

```
Utilisateur
     |
     |
JavaScript
     |
     | requête Ajax
     v
PHP
     |
     |
PDO
     |
     |
Base de données
     |
     |
JSON
     |
     v
JavaScript
```

---

# 13.8 Exemple d'appel Ajax

JavaScript :

```
import {
    appelAjax
}
from "/composant/fonction/ajax.js";



function chargerEtudiants()
{

    appelAjax({

        url:
        "ajax/getEtudiants.php",


        success:
        data => {

            afficherEtudiants(data);

        }

    });

}
```

---

# 13.9 Le script PHP appelé par Ajax

Fichier :

```
ajax/getEtudiants.php
```

Contenu :

```
<?php

$etudiants =
    Etudiant::getAll();


echo json_encode($etudiants);
```

La réponse HTTP contient :

```
[
 {
   "nom":"Martin",
   "prenom":"Paul"
 }
]
```

---

# 13.10 Réception automatique côté JavaScript

Avec une fonction Ajax correctement configurée, la conversion JSON peut être automatique.

Le callback reçoit directement un objet JavaScript :

```
success:
data => {

    console.log(
        data[0].nom
    );

}
```

Résultat :

```
Martin
```

---

# 13.11 Envoyer des paramètres à PHP

Exemple :

JavaScript :

```
appelAjax({

    url:
    "ajax/getCompetences.php",


    data:
    {
        idProjet: 15
    },


    success:
    data => {

        afficher(data);

    }

});
```

---

PHP :

```
<?php

$idProjet =
    $_POST["idProjet"];


$resultat =
    Projet::getCompetences(
        $idProjet
    );


echo json_encode($resultat);
```

---

# 13.12 Vérification des paramètres reçus

Même dans un appel Ajax, les données reçues doivent être contrôlées.

Exemple :

```
if (!isset($_POST["idProjet"])) {

    echo json_encode(
        [
            "erreur" =>
            "Identifiant absent"
        ]
    );

    exit();

}
```

---

# 13.13 Format des réponses Ajax

Une bonne pratique consiste à toujours retourner du JSON.

Exemple succès :

```
{
    "success":true,
    "data":[
        {
            "nom":"Martin"
        }
    ]
}
```

Exemple erreur :

```
{
    "success":false,
    "message":
    "Projet inexistant"
}
```

Cela permet au JavaScript de traiter uniformément les réponses.

---

# 13.14 Relation avec les classes métiers

Une organisation propre sépare les responsabilités.

Exemple :

```
ajax/getEtudiants.php

        |
        v

Classe Etudiant

        |
        v

Database

        |
        v

PDO

        |
        v

Base de données
```

Le script Ajax ne contient pas de SQL.

Il demande simplement :

```
Etudiant::getAll();
```

---

# 13.15 Bonnes pratiques

- Envoyer les données côté client sous forme JSON.
- Utiliser `json_encode()` pour convertir les tableaux PHP.
- Préparer les données côté serveur avant affichage.
- Ne jamais envoyer directement des informations sensibles.
- Séparer les scripts Ajax, les classes métiers et l'accès PDO.
- Vérifier les paramètres reçus par les scripts Ajax.
- Utiliser un format de réponse cohérent.

---

# À retenir

- PDO récupère les données depuis la base.
- PHP prépare et transforme les données.
- JSON permet l'échange entre PHP et JavaScript.
- Les appels Ajax permettent de mettre à jour une interface sans recharger la page.
- Les classes métiers doivent rester responsables des accès aux données.
- Les scripts Ajax doivent uniquement gérer la communication entre client et serveur.