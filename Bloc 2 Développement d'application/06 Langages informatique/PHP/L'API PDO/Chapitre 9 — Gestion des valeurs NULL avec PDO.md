## Objectifs

À la fin de ce chapitre, vous serez capable de :

- comprendre la différence entre une valeur vide et une valeur `NULL` ;
- enregistrer correctement une valeur `NULL` dans une base de données ;
- utiliser `PDO::PARAM_NULL` ;
- gérer les champs facultatifs lors d'un ajout ou d'une modification ;
- éviter les erreurs liées aux conversions automatiques de PHP et du SGBDR.

---

# 9.1 Introduction

Dans une base de données, une colonne peut accepter une valeur particulière appelée :

```
NULL
```

`NULL` signifie :

> aucune valeur connue ou aucune information renseignée.

Il ne signifie pas :

- une chaîne vide ;
- zéro ;
- une valeur par défaut.

---

# 9.2 Différence entre vide et NULL

Ces trois valeurs sont différentes :

|Valeur|Signification|
|---|---|
|`NULL`|aucune valeur|
|`''`|chaîne vide|
|`0`|valeur numérique zéro|

Exemple :

Table `utilisateur` :

|id|nom|téléphone|
|---|---|---|
|1|Martin|0600000000|
|2|Dupont|NULL|
|3|Durand|""|

Ces trois lignes ne représentent pas la même information.

---

# 9.3 Les champs facultatifs

Une colonne autorisant `NULL` possède généralement une définition SQL comme :

```
telephone VARCHAR(20) NULL
```

Cela signifie que l'utilisateur peut ne pas renseigner cette information.

Exemple :

Formulaire :

```
<input name="telephone">
```

Si le champ est laissé vide, PHP reçoit :

```
$_POST["telephone"]
```

avec comme valeur :

```
""
```

et non :

```
NULL
```

---

# 9.4 Transformer une valeur vide en NULL

Avant d'envoyer la donnée à PDO, il faut souvent effectuer une conversion.

Exemple :

```
$telephone = $_POST["telephone"] ?? null;

if ($telephone === "") {
    $telephone = null;
}
```

Ainsi :

|Saisie utilisateur|Valeur PHP|
|---|---|
|Champ rempli|`"0600000000"`|
|Champ vide|`null`|

---

# 9.5 Insérer une valeur NULL

Exemple :

Table :

```
CREATE TABLE utilisateur
(
    id INT AUTO_INCREMENT PRIMARY KEY,
    nom VARCHAR(50),
    telephone VARCHAR(20) NULL
);
```

Insertion :

```
$sql = <<<SQL
INSERT INTO utilisateur(
    nom,
    telephone
)
VALUES(
    :nom,
    :telephone
)
SQL;


$stmt = $db->prepare($sql);


$stmt->bindValue(
    "nom",
    $nom
);


$stmt->bindValue(
    "telephone",
    $telephone,
    $telephone === null
        ? PDO::PARAM_NULL
        : PDO::PARAM_STR
);


$stmt->execute();
```

---

# 9.6 Pourquoi préciser `PDO::PARAM_NULL` ?

Si on transmet simplement :

```
$stmt->bindValue(
    "telephone",
    null
);
```

PDO peut parfois ne pas interpréter correctement le type attendu.

Il est préférable d'indiquer explicitement :

```
PDO::PARAM_NULL
```

Ainsi PDO transmet réellement :

```
NULL
```

et non une chaîne vide.

---

# 9.7 Modifier une valeur existante en NULL

Exemple :

Utilisateur :

|id|téléphone|
|---|---|
|15|0600000000|

L'utilisateur supprime son numéro.

On veut obtenir :

|id|téléphone|
|---|---|
|15|NULL|

Code :

```
$sql = <<<SQL
UPDATE utilisateur
SET telephone = :telephone
WHERE id = :id
SQL;


$stmt = $db->prepare($sql);


$stmt->bindValue(
    "id",
    $id,
    PDO::PARAM_INT
);


$stmt->bindValue(
    "telephone",
    $telephone,
    $telephone === null
        ? PDO::PARAM_NULL
        : PDO::PARAM_STR
);


$stmt->execute();
```

---

# 9.8 Exemple avec plusieurs champs facultatifs

Supposons une table :

```
compte

id
nom
prenom
email
telephone
adresse
```

Les colonnes :

```
nom VARCHAR(50) NULL,
prenom VARCHAR(50) NULL
```

peuvent ne pas être renseignées.

Code :

```
$sql = <<<SQL
UPDATE compte
SET nom = :nom,
    prenom = :prenom
WHERE email = :email
SQL;


$stmt = $db->prepare($sql);


$stmt->bindValue(
    "email",
    $email
);


$stmt->bindValue(
    "nom",
    $nom,
    $nom === null
        ? PDO::PARAM_NULL
        : PDO::PARAM_STR
);


$stmt->bindValue(
    "prenom",
    $prenom,
    $prenom === null
        ? PDO::PARAM_NULL
        : PDO::PARAM_STR
);


$stmt->execute();
```

---

# 9.9 Cas particulier : les valeurs numériques

Les valeurs numériques doivent également être traitées.

Exemple :

Champ :

```
age INT NULL
```

PHP :

```
$age = $_POST["age"] ?? null;
```

Conversion :

```
if ($age === "") {
    $age = null;
}
```

Puis :

```
$stmt->bindValue(
    "age",
    $age,
    $age === null
        ? PDO::PARAM_NULL
        : PDO::PARAM_INT
);
```

---

# 9.10 Tester une valeur NULL en SQL

En SQL, on ne teste jamais `NULL` avec :

```
=
```

Incorrect :

```
WHERE telephone = NULL
```

Cette condition ne fonctionne pas.

Il faut utiliser :

```
IS NULL
```

Exemple :

```
SELECT *
FROM utilisateur
WHERE telephone IS NULL;
```

---

Pour rechercher les valeurs non nulles :

```
SELECT *
FROM utilisateur
WHERE telephone IS NOT NULL;
```

---

# 9.11 Les valeurs NULL dans les recherches paramétrées

Un piège fréquent :

```
SELECT *
FROM utilisateur
WHERE telephone = :telephone
```

avec :

```
$telephone = null;
```

La requête devient équivalente à :

```
WHERE telephone = NULL
```

ce qui ne retourne aucun résultat.

---

Il faut adapter la requête :

```
if ($telephone === null) {

    $sql = "
    SELECT *
    FROM utilisateur
    WHERE telephone IS NULL
    ";

}
else {

    $sql = "
    SELECT *
    FROM utilisateur
    WHERE telephone = :telephone
    ";

}
```

---

# 9.12 Valeur par défaut et NULL

Une colonne peut posséder une valeur par défaut.

Exemple :

```
actif BOOLEAN DEFAULT 1
```

Lors d'un insert :

```
INSERT INTO utilisateur(nom)
VALUES('Martin');
```

le SGBDR utilise :

```
actif = 1
```

---

Mais :

```
INSERT INTO utilisateur(
    nom,
    actif
)
VALUES(
    'Martin',
    NULL
);
```

force la valeur :

```
actif = NULL
```

La valeur par défaut n'est donc pas utilisée.

---

# 9.13 NULL et JSON

Lorsqu'une donnée est envoyée au client :

PHP :

```
$data = [
    "nom" => "Martin",
    "telephone" => null
];


echo json_encode($data);
```

Résultat :

```
{
    "nom":"Martin",
    "telephone":null
}
```

JavaScript conserve cette information :

```
objet.telephone === null
```

---

# 9.14 Bonnes pratiques

- Toujours distinguer une valeur vide d'une valeur `NULL`.
- Convertir les champs de formulaire vides avant l'enregistrement.
- Utiliser `PDO::PARAM_NULL` lorsque la valeur doit être réellement NULL.
- Utiliser `IS NULL` et `IS NOT NULL` dans les requêtes SQL.
- Ne jamais tester `NULL` avec l'opérateur `=`.
- Prévoir les colonnes facultatives dans la conception de la base de données.
- Harmoniser la gestion des valeurs NULL dans les classes métiers.

---

# Exemple complet : modification d'un profil

```
public static function modifierProfil(
    int $id,
    ?string $nom,
    ?string $telephone
): void
{

    $db = Database::getInstance();


    $sql = <<<SQL
    UPDATE utilisateur
    SET nom = :nom,
        telephone = :telephone
    WHERE id = :id
    SQL;


    $stmt = $db->prepare($sql);


    $stmt->bindValue(
        "id",
        $id,
        PDO::PARAM_INT
    );


    $stmt->bindValue(
        "nom",
        $nom,
        $nom === null
            ? PDO::PARAM_NULL
            : PDO::PARAM_STR
    );


    $stmt->bindValue(
        "telephone",
        $telephone,
        $telephone === null
            ? PDO::PARAM_NULL
            : PDO::PARAM_STR
    );


    $stmt->execute();

}
```

---

# À retenir

- `NULL` représente l'absence de valeur dans une base de données.
- Une chaîne vide (`""`) et `NULL` sont deux informations différentes.
- Les champs facultatifs nécessitent souvent une conversion avant enregistrement.
- `PDO::PARAM_NULL` permet d'envoyer explicitement une valeur SQL `NULL`.
- Les tests SQL utilisent `IS NULL` et `IS NOT NULL`.
- Une bonne gestion des valeurs NULL évite de nombreuses erreurs lors des ajouts et modifications de données.