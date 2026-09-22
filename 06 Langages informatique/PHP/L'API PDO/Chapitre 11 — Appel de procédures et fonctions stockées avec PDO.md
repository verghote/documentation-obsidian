## Objectifs

À la fin de ce chapitre, vous serez capable de :

- comprendre le rôle des procédures et fonctions stockées ;
- appeler une procédure stockée depuis PHP avec PDO ;
- transmettre des paramètres à une procédure ;
- récupérer un résultat retourné par une procédure ou une fonction ;
- connaître les limites des paramètres de sortie avec PDO.

---

# 11.1 Introduction

Une procédure ou une fonction stockée est un ensemble d'instructions SQL enregistré directement dans le SGBDR.

Contrairement aux requêtes SQL écrites dans PHP, le code est exécuté côté serveur de base de données.

Exemple d'utilisation :

- calcul complexe ;
- traitement nécessitant plusieurs requêtes ;
- contrôle métier centralisé ;
- opérations utilisées par plusieurs applications.

---

# 11.2 Procédure stockée et fonction stockée

Il existe deux types principaux d'objets stockés.

## La procédure stockée

Une procédure réalise une action.

Exemples :

- ajouter un utilisateur ;
- modifier une commande ;
- générer un traitement.

Elle est appelée avec :

```
CALL nomProcedure()
```

---

## La fonction stockée

Une fonction retourne obligatoirement une valeur.

Exemples :

- calculer un montant ;
- retourner un état ;
- fournir une information calculée.

Elle est utilisée dans une requête SQL :

```
SELECT nomFonction();
```

---

# 11.3 Avantages des traitements stockés

## Performances

Le code SQL est :

- stocké sur le serveur ;
- analysé ;
- optimisé par le SGBDR.

---

## Centralisation des règles métier

Une règle peut être utilisée par plusieurs applications.

Exemple :

Une fonction de calcul de remise :

```
Application Web
Application mobile
Application interne

        |
        |
        v

Fonction SQL commune
```

---

## Réduction du code côté application

Une seule instruction peut remplacer plusieurs requêtes SQL.

---

# 11.4 Inconvénients

Les procédures stockées présentent aussi quelques limites :

- elles augmentent la charge du serveur de base de données ;
- elles dépendent du langage du SGBDR ;
- elles sont parfois moins faciles à maintenir qu'un code métier PHP.

Il faut donc les utiliser lorsque cela apporte un réel intérêt.

---

# 11.5 Appeler une procédure sans paramètre

Exemple de procédure MySQL :

```
CREATE PROCEDURE getUtilisateurs()
BEGIN

    SELECT id,
           nom,
           prenom
    FROM utilisateur
    ORDER BY nom;

END
```

Appel depuis PHP :

```
$db = Database::getInstance();


$stmt = $db->query(
    "CALL getUtilisateurs()"
);


$utilisateurs =
    $stmt->fetchAll(PDO::FETCH_ASSOC);


$stmt->closeCursor();
```

Résultat :

```
[
    [
        "id" => 1,
        "nom" => "Martin",
        "prenom" => "Paul"
    ]
]
```

---

# 11.6 Pourquoi utiliser `closeCursor()` ?

Lorsqu'une procédure retourne un jeu de résultats, le curseur doit être libéré.

Exemple :

```
$stmt->closeCursor();
```

C'est particulièrement important lorsque plusieurs appels SQL sont réalisés après l'appel d'une procédure.

---

# 11.7 Appeler une procédure avec des paramètres

Exemple :

Création d'une procédure :

```
CREATE PROCEDURE getUtilisateursClasse(
    idClasse INT
)
BEGIN

    SELECT nom,
           prenom
    FROM utilisateur
    WHERE idClasse = idClasse;

END
```

Appel PHP :

```
$sql = "
CALL getUtilisateursClasse(:idClasse)
";


$stmt = $db->prepare($sql);


$stmt->execute([
    "idClasse" => $idClasse
]);


$resultat =
    $stmt->fetchAll(PDO::FETCH_ASSOC);
```

---

# 11.8 Procédure réalisant une insertion

Exemple :

Procédure SQL :

```
CREATE PROCEDURE ajouterUtilisateur(
    nom VARCHAR(50),
    email VARCHAR(100)
)
BEGIN

    INSERT INTO utilisateur(
        nom,
        email
    )
    VALUES(
        nom,
        email
    );

END
```

---

Appel depuis PHP :

```
$sql = "
CALL ajouterUtilisateur(
    :nom,
    :email
)
";


$stmt = $db->prepare($sql);


$stmt->execute([
    "nom" => $nom,
    "email" => $email
]);
```

---

# 11.9 Récupérer une valeur retournée par une fonction

Une fonction SQL retourne une valeur.

Exemple :

```
CREATE FUNCTION compterUtilisateurs()
RETURNS INT
BEGIN

    RETURN (
        SELECT COUNT(*)
        FROM utilisateur
    );

END
```

Appel :

```
$sql = "
SELECT compterUtilisateurs()
";


$stmt = $db->query($sql);


$total = $stmt->fetchColumn();
```

Résultat :

```
125
```

---

# 11.10 Fonction avec paramètre

Exemple :

Fonction SQL :

```
CREATE FUNCTION nombreCommandes(
    idClient INT
)
RETURNS INT
BEGIN

    RETURN (
        SELECT COUNT(*)
        FROM commande
        WHERE idClient = idClient
    );

END
```

PHP :

```
$sql = "
SELECT nombreCommandes(:idClient)
";


$stmt = $db->prepare($sql);


$stmt->execute([
    "idClient" => $idClient
]);


$total =
    $stmt->fetchColumn();
```

---

# 11.11 Procédure avec paramètre de sortie

Certaines procédures possèdent un paramètre `OUT`.

Exemple :

```
CREATE PROCEDURE ajouterCompte(
    login VARCHAR(50),
    email VARCHAR(100),
    OUT idCompte INT
)
BEGIN

    INSERT INTO compte(
        login,
        email
    )
    VALUES(
        login,
        email
    );


    SET idCompte = LAST_INSERT_ID();

END
```

L'objectif est de récupérer l'identifiant créé.

---

# 11.12 Limite des paramètres OUT avec PDO

En théorie, on pourrait écrire :

```
$stmt->bindValue(
    "idCompte",
    $id,
    PDO::PARAM_INT
);
```

Cependant, avec certains pilotes PDO, notamment PDO MySQL, les paramètres `OUT` des procédures stockées peuvent provoquer des problèmes.

Exemple d'erreur possible :

```
OUT or INOUT argument is not a variable
```

---

# 11.13 Solution recommandée : retourner une valeur par SELECT

Une solution plus fiable consiste à modifier la procédure.

Au lieu de :

```
OUT idCompte INT
```

on réalise :

```
CREATE PROCEDURE ajouterCompte(
    login VARCHAR(50),
    email VARCHAR(100)
)
BEGIN

    INSERT INTO compte(
        login,
        email
    )
    VALUES(
        login,
        email
    );


    SELECT LAST_INSERT_ID();

END
```

---

PHP :

```
$sql = "
CALL ajouterCompte(
    :login,
    :email
)
";


$stmt = $db->prepare($sql);


$stmt->execute([
    "login" => $login,
    "email" => $email
]);


$idCompte =
    $stmt->fetchColumn();
```

Cette méthode fonctionne avec tous les pilotes PDO courants.

---

# 11.14 Procédure et transaction

Une procédure peut être appelée dans une transaction PHP.

Exemple :

```
$db->beginTransaction();


try {

    $stmt = $db->prepare(
        "CALL ajouterCommande(:id)"
    );


    $stmt->execute([
        "id" => $id
    ]);


    $db->commit();

}
catch(Exception $e)
{

    $db->rollBack();

}
```

---

# 11.15 Procédure ou requêtes PHP classiques ?

Le choix dépend du contexte.

## Préférer PHP + PDO lorsque :

- la logique appartient à l'application ;
- le traitement est spécifique au site ;
- le code doit évoluer fréquemment.

---

## Préférer une procédure stockée lorsque :

- plusieurs applications utilisent le même traitement ;
- le traitement est très proche des données ;
- il nécessite beaucoup d'opérations SQL ;
- la performance du SGBDR est importante.

---

# 11.16 Bonnes pratiques

- Utiliser des paramètres dans les appels de procédures.
- Éviter de construire dynamiquement une commande SQL `CALL`.
- Fermer les curseurs après récupération des résultats.
- Préférer un retour par `SELECT` plutôt qu'un paramètre `OUT` avec certains pilotes PDO.
- Documenter les procédures stockées comme du code applicatif.
- Ne pas déplacer toute la logique métier dans la base de données sans justification.

---

# À retenir

- PDO permet d'appeler des procédures et fonctions stockées.
- Une procédure réalise une action ; une fonction retourne une valeur.
- Les appels utilisent `CALL` pour les procédures.
- Les fonctions sont utilisées dans une requête `SELECT`.
- Les paramètres d'entrée fonctionnent comme pour les requêtes préparées classiques.
- Les paramètres `OUT` peuvent être problématiques avec certains pilotes PDO.
- Retourner une valeur avec un `SELECT` est souvent la solution la plus portable.