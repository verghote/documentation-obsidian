## Objectifs

À la fin de ce chapitre, vous serez capable de :

- réaliser des opérations de modification avec PDO ;
- utiliser correctement `prepare()` et `execute()` pour les requêtes d'écriture ;
- insérer de nouveaux enregistrements ;
- récupérer l'identifiant généré après une insertion ;
- modifier et supprimer des données ;
- connaître les particularités des requêtes qui ne retournent pas de résultats.

---

# 8.1 Introduction

Les requêtes de modification permettent de changer le contenu d'une base de données.

Elles correspondent aux opérations **CRUD** :

|Opération|SQL|Rôle|
|---|---|---|
|Create|`INSERT`|Ajouter une ligne|
|Read|`SELECT`|Lire des données|
|Update|`UPDATE`|Modifier une ligne|
|Delete|`DELETE`|Supprimer une ligne|

Dans ce chapitre, nous étudions les opérations qui modifient les données :

- `INSERT`
- `UPDATE`
- `DELETE`

---

# 8.2 Différence entre consultation et modification

Une requête `SELECT` retourne un jeu de résultats.

Exemple :

```
SELECT *
FROM utilisateur;
```

Elle nécessite généralement :

```
fetch()
```

ou :

```
fetchAll()
```

---

Une requête de modification ne retourne généralement aucune ligne.

Exemple :

```
UPDATE utilisateur
SET actif = 1
WHERE id = 10;
```

Après :

```
execute()
```

le travail est terminé.

---

# 8.3 Pourquoi utiliser `prepare()` ?

Les requêtes de modification utilisent presque toujours des données provenant :

- d'un formulaire ;
- d'un utilisateur ;
- d'une autre partie de l'application.

Il faut donc utiliser des paramètres.

Exemple incorrect :

```
$sql = "
INSERT INTO utilisateur
VALUES('$nom','$email')
";
```

Cette écriture présente les mêmes risques que pour les requêtes `SELECT`.

La bonne pratique :

```
$sql = <<<SQL
INSERT INTO utilisateur(
    nom,
    email
)
VALUES(
    :nom,
    :email
)
SQL;
```

---

# 8.4 Ajouter un enregistrement (`INSERT`)

## Exemple simple

Ajout d'un utilisateur :

```
$db = Database::getInstance();


$sql = <<<SQL
INSERT INTO utilisateur(
    nom,
    prenom,
    email
)
VALUES(
    :nom,
    :prenom,
    :email
)
SQL;


$stmt = $db->prepare($sql);


$stmt->execute([
    "nom" => $nom,
    "prenom" => $prenom,
    "email" => $email
]);
```

L'enregistrement est maintenant présent dans la table.

---

# 8.5 Récupérer l'identifiant généré

Les tables utilisent souvent une clé primaire auto-incrémentée :

```
id INT AUTO_INCREMENT PRIMARY KEY
```

Après une insertion :

```
$id = $db->lastInsertId();
```

permet de récupérer l'identifiant attribué.

Exemple complet :

```
$sql = <<<SQL
INSERT INTO projet(
    nom
)
VALUES(
    :nom
)
SQL;


$stmt = $db->prepare($sql);

$stmt->execute([
    "nom" => $nom
]);


$idProjet = $db->lastInsertId();
```

---

# 8.6 Exemple : création d'un objet métier

Dans une architecture organisée, l'ajout est généralement placé dans une classe métier.

Exemple :

```
class Projet
{

    public static function ajouter(string $nom): int
    {

        $db = Database::getInstance();


        $sql = <<<SQL
        INSERT INTO projet(nom)
        VALUES(:nom)
        SQL;


        $stmt = $db->prepare($sql);


        $stmt->execute([
            "nom" => $nom
        ]);


        return $db->lastInsertId();

    }

}
```

Utilisation :

```
$id = Projet::ajouter("Application PDO");
```

---

# 8.7 Modifier un enregistrement (`UPDATE`)

Exemple :

Modifier l'adresse mail d'un utilisateur.

```
$sql = <<<SQL
UPDATE utilisateur
SET email = :email
WHERE id = :id
SQL;


$stmt = $db->prepare($sql);


$stmt->execute([
    "id" => $id,
    "email" => $email
]);
```

---

# 8.8 L'importance de la clause `WHERE`

Une erreur fréquente consiste à oublier `WHERE`.

Exemple dangereux :

```
UPDATE utilisateur
SET actif = 0;
```

Cette requête modifie **toutes les lignes**.

La plupart des modifications doivent cibler un enregistrement précis :

```
UPDATE utilisateur
SET actif = 0
WHERE id = :id;
```

---

# 8.9 Connaître le nombre de lignes modifiées

Après une modification :

```
$stmt->rowCount();
```

retourne le nombre de lignes affectées.

Exemple :

```
$stmt->execute([
    "id" => $id,
    "email" => $email
]);


if ($stmt->rowCount() > 0) {

    echo "Modification effectuée";

}
```

---

## Attention

Selon les SGBDR, une modification avec la même valeur peut retourner :

```
0 ligne modifiée
```

même si la requête a réussi.

Exemple :

Valeur actuelle :

```
email = test@test.fr
```

Nouvelle valeur :

```
email = test@test.fr
```

Le SGBDR peut considérer qu'aucune modification réelle n'a eu lieu.

---

# 8.10 Supprimer un enregistrement (`DELETE`)

Exemple :

```php
 $db = Database::getInstance();  
 $sql = <<<SQL  
            delete from projet where id = :id;
 SQL;  
 $cmd = $db->prepare($sql);  
 $cmd->execute(['id' => $id]);
```

```php
$db = Database::getInstance();  
$sql = <<<SQL  
            delete from projet where id = :id;
SQL;  
$cmd = $db->prepare($sql);  
$cmd->bindValue(':id', $id, PDO::PARAM_INT);  
$cmd->execute();
```

`execute(array)` est la forme privilégiée pour les requêtes classiques et `bindValue()` pour les besoins spécifiques.

---

# 8.11 Suppression logique

Dans de nombreuses applications, on évite de supprimer réellement les données.

On utilise une suppression logique.

Exemple :

Table :

```
utilisateur

id
nom
actif
```

Au lieu de :

```
DELETE FROM utilisateur
WHERE id = :id;
```

on fait :

```
UPDATE utilisateur
SET actif = 0
WHERE id = :id;
```

Avantages :

- conservation de l'historique ;
- possibilité de restauration ;
- meilleure traçabilité.

---

# 8.12 Exécuter une modification sans paramètre avec `exec()`

Lorsque la requête ne contient aucune variable, il est possible d'utiliser :

```
$db->exec();
```

Exemple :

```php
$sql = "
UPDATE parametre
SET valeur = 'oui'
WHERE nom = 'maintenance'
";


$nb = $db->exec($sql);
```

La méthode retourne :

```
nombre de lignes affectées
```

---

# 8.13 Différence entre `exec()` et `execute()`

||exec()|execute()|
|---|---|---|
|Paramètres|Non|Oui|
|Préparation|Non|Oui|
|Sécurité|limitée|meilleure|
|Retour|nombre de lignes|booléen|

Dans une application professionnelle :

- utiliser `execute()` dans la majorité des cas ;
- réserver `exec()` aux requêtes internes sans paramètres.

---

# 8.14 Gestion des valeurs NULL

Une valeur vide provenant d'un formulaire :

```
""
```

n'est pas équivalente à :

```
NULL
```

Exemple :

```
$stmt->bindValue(
    "telephone",
    $telephone,
    $telephone === null
        ? PDO::PARAM_NULL
        : PDO::PARAM_STR
);
```

Cette problématique sera étudiée dans un chapitre spécifique.

---

# 8.15 Exemple complet : modification d'un compte

```
class Compte
{

    public static function modifierEmail(
        int $id,
        string $email
    ): void
    {

        $db = Database::getInstance();


        $sql = <<<SQL
        UPDATE compte
        SET email = :email
        WHERE id = :id
        SQL;


        $stmt = $db->prepare($sql);


        $stmt->execute([
            "id" => $id,
            "email" => $email
        ]);

    }

}
```

---

# 8.16 Bonnes pratiques

- Toujours utiliser des requêtes préparées pour les données dynamiques.
- Toujours vérifier la présence d'une clause `WHERE` dans `UPDATE` et `DELETE`.
- Utiliser `lastInsertId()` après un `INSERT` lorsque l'identifiant généré est nécessaire.
- Utiliser `rowCount()` uniquement pour les opérations de modification.
- Préférer une suppression logique lorsque l'historique des données est important.
- Regrouper les opérations d'accès aux données dans des classes métiers.
- Utiliser une transaction lorsque plusieurs modifications doivent être cohérentes.

---

# À retenir

- `INSERT`, `UPDATE` et `DELETE` sont les principales requêtes de modification.
- Elles utilisent généralement `prepare()` puis `execute()`.
- Après un `INSERT`, `lastInsertId()` permet de récupérer l'identifiant créé.
- `rowCount()` permet de connaître le nombre de lignes affectées par une modification.
- Une clause `WHERE` est indispensable pour éviter les modifications ou suppressions globales accidentelles.
- Les modifications complexes impliquant plusieurs requêtes doivent être réalisées dans une transaction.