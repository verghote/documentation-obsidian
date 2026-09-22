## Objectifs

À la fin de ce chapitre, vous serez capable de :

- comprendre le fonctionnement des requêtes préparées ;
- connaître les avantages des requêtes préparées par rapport aux requêtes classiques ;
- utiliser les paramètres nommés et anonymes ;
- choisir entre `execute()`, `bindValue()` et `bindParam()` ;
- réutiliser une requête préparée plusieurs fois ;
- adopter les bonnes pratiques pour écrire des requêtes sécurisées et performantes.

---

# 5.1 Pourquoi utiliser des requêtes préparées ?

Une requête préparée est une requête SQL dont la structure est **analysée une seule fois** par le SGBDR. Les valeurs des paramètres sont ensuite transmises séparément lors de l'exécution.

Contrairement à une requête construite par concaténation, la partie SQL reste fixe et seules les données changent.

Exemple :

```
SELECT id, nom
FROM utilisateur
WHERE id = :id
```

La valeur de `:id` sera fournie au moment de l'exécution.

---

# 5.2 Fonctionnement général

Une requête préparée se déroule en trois étapes.

En PHP :

```
$sql = <<<SQL
SELECT *
FROM utilisateur
WHERE id = :id
SQL;

$stmt = $db->prepare($sql);

$stmt->execute([
    "id" => 12
]);
```

---

# 5.3 Pourquoi les requêtes préparées sont-elles plus sûres ?

Prenons l'exemple d'une authentification.

Un développeur débutant pourrait écrire :

```
$sql = "
SELECT *
FROM utilisateur
WHERE login = '$login'
AND password = '$password'
";
```

Si l'utilisateur saisit :

```
' OR 1=1 --
```

la requête devient :

```
SELECT *
FROM utilisateur
WHERE login = '' OR 1=1 --'
```

Le prédicat `OR 1=1` étant toujours vrai, l'authentification est contournée : c'est une **injection SQL**.

Avec une requête préparée :

```
$sql = <<<SQL
SELECT *
FROM utilisateur
WHERE login = :login
AND password = :password
SQL;

$stmt = $db->prepare($sql);

$stmt->execute([
    "login" => $login,
    "password" => $password
]);
```

Les valeurs sont transmises séparément de la requête SQL. Elles sont interprétées comme des données et **jamais comme du code SQL**.

---

# 5.4 Les avantages des requêtes préparées

Les requêtes préparées offrent plusieurs avantages :

- protection contre les injections SQL ;
- meilleure lisibilité du code ;
- séparation entre le code SQL et les données ;
- gestion automatique des caractères spéciaux (apostrophes, guillemets, etc.) ;
- possibilité de réutiliser la même requête plusieurs fois ;
- optimisation des performances lorsque la même requête est exécutée à plusieurs reprises.

---

# 5.5 Les paramètres nommés

Les paramètres nommés commencent par `:`.

```
SELECT *
FROM utilisateur
WHERE id = :id
```

Ils sont associés à des valeurs grâce à un tableau :

```
$stmt->execute([
    "id" => 12
]);
```

Ou avec `bindValue()` :

```
$stmt->bindValue("id", 12);

$stmt->execute();
```

Les paramètres nommés sont les plus lisibles et sont généralement recommandés.

---

# 5.6 Les paramètres anonymes

Les paramètres anonymes sont représentés par `?`.

```
SELECT *
FROM utilisateur
WHERE nom = ?
```

Les valeurs sont fournies dans l'ordre d'apparition :

```
$stmt->execute([
    $nom
]);
```

Avec plusieurs paramètres :

```
SELECT *
FROM utilisateur
WHERE nom = ?
AND prenom = ?
```

```
$stmt->execute([
    $nom,
    $prenom
]);
```

---

# 5.7 Paramètres nommés ou anonymes ?

Les deux syntaxes sont équivalentes.

Cependant, les paramètres nommés présentent plusieurs avantages :

- ils rendent le code plus lisible ;
- l'ordre des paramètres n'a pas d'importance ;
- ils réduisent les erreurs lors de l'ajout ou de la suppression de paramètres.

Par conséquent, ils sont généralement privilégiés dans les applications professionnelles.

---

# 5.8 Les trois façons de transmettre les paramètres

PDO offre trois approches.

## 1. Avec `execute(array)`

```
$stmt->execute([
    "id" => $id,
    "nom" => $nom
]);
```

C'est la solution la plus simple lorsque les types peuvent être déduits automatiquement.

---

## 2. Avec `bindValue()`

```
$stmt->bindValue("id", $id);
$stmt->bindValue("nom", $nom);

$stmt->execute();
```

Cette méthode est utile lorsqu'il est nécessaire de préciser le type (`PDO::PARAM_INT`, `PDO::PARAM_NULL`, etc.).

---

## 3. Avec `bindParam()`

```
$stmt->bindParam("id", $id);

$stmt->execute();
```

La variable est évaluée au moment de l'exécution.

Cette méthode est moins fréquente dans les applications modernes.

---

# 5.9 Réutiliser une requête préparée

L'un des intérêts majeurs des requêtes préparées est leur réutilisation.

```
$sql = <<<SQL
INSERT INTO journal(message)
VALUES(:message)
SQL;

$stmt = $db->prepare($sql);

foreach ($messages as $message) {

    $stmt->execute([
        "message" => $message
    ]);

}
```

La requête n'est préparée qu'une seule fois.

---

# 5.10 Exemple : recherche par identifiant

```
$sql = <<<SQL
SELECT id,
       nom,
       prenom
FROM utilisateur
WHERE id = :id
SQL;

$stmt = $db->prepare($sql);

$stmt->execute([
    "id" => $id
]);

return $stmt->fetch();
```

---

# 5.11 Exemple : recherche avec plusieurs critères

```
$sql = <<<SQL
SELECT id,
       nom,
       prenom
FROM utilisateur
WHERE nom = :nom
AND prenom = :prenom
SQL;

$stmt = $db->prepare($sql);

$stmt->execute([
    "nom" => $nom,
    "prenom" => $prenom
]);

return $stmt->fetchAll();
```

---

# 5.12 Exemple : insertion

```
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

---

# 5.13 Exemple : mise à jour

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

# 5.14 Exemple : suppression

```
$sql = <<<SQL
DELETE
FROM utilisateur
WHERE id = :id
SQL;

$stmt = $db->prepare($sql);

$stmt->execute([
    "id" => $id
]);
```

---

# 5.15 Les paramètres `NULL`

Lorsque la colonne accepte la valeur `NULL`, il peut être nécessaire de préciser explicitement le type.

```
$stmt->bindValue(
    "telephone",
    $telephone,
    $telephone === null
        ? PDO::PARAM_NULL
        : PDO::PARAM_STR
);

$stmt->execute();
```

Un chapitre entier sera consacré à la gestion des valeurs `NULL`.

---

# 5.16 Les paramètres ne remplacent que les valeurs

Un paramètre ne peut remplacer **qu'une valeur**, jamais un élément de la syntaxe SQL.

Correct :

```
SELECT *
FROM utilisateur
WHERE id = :id
```

Incorrect :

```
SELECT *
FROM :table
```

Ou :

```
ORDER BY :colonne
```

Les noms de tables, de colonnes, les mots-clés SQL et les opérateurs doivent être écrits directement dans la requête.

---

# 5.17 Les erreurs courantes

### Concaténer des variables

❌

```
$sql = "SELECT * FROM utilisateur WHERE id = $id";
```

✔

```
$sql = "SELECT * FROM utilisateur WHERE id = :id";
```

---

### Mélanger paramètres nommés et anonymes

❌

```
SELECT *
FROM utilisateur
WHERE id = ?
AND nom = :nom
```

Une requête doit utiliser **un seul type de paramètre**.

---

### Oublier un paramètre

```
$stmt->execute([
    "nom" => $nom
]);
```

alors que la requête contient également `:prenom`.

Tous les paramètres doivent recevoir une valeur avant l'exécution.

---

# 5.18 Comparaison des différentes approches

|Méthode|Lisibilité|Types explicites|Réutilisation|
|---|---|---|---|
|`execute(array)`|⭐⭐⭐|Non|Oui|
|`bindValue()`|⭐⭐|Oui|Oui|
|`bindParam()`|⭐|Oui|Oui|

Dans la majorité des cas :

- **`execute(array)`** est la solution la plus concise ;
- **`bindValue()`** est recommandé lorsque le type doit être précisé ;
- **`bindParam()`** est réservé à des besoins spécifiques.

---

# Bonnes pratiques

- Préparer systématiquement les requêtes contenant des paramètres.
- Privilégier les **paramètres nommés** pour améliorer la lisibilité.
- Utiliser `execute(array)` lorsque les types sont simples.
- Employer `bindValue()` pour les valeurs `NULL` ou lorsqu'un type PDO doit être indiqué.
- Éviter toute concaténation de données utilisateur dans une requête SQL.
- Réutiliser une même requête préparée lorsqu'elle est exécutée plusieurs fois avec des valeurs différentes.

---

# À retenir

- Une requête préparée sépare la structure SQL des données fournies par l'application.
- Les paramètres peuvent être **nommés** (`:id`) ou **anonymes** (`?`), mais il ne faut jamais mélanger les deux dans une même requête.
- `execute(array)` est la méthode la plus simple pour transmettre des paramètres.
- `bindValue()` et `bindParam()` permettent un contrôle plus fin, notamment sur le type des valeurs.
- Les requêtes préparées améliorent la sécurité, la lisibilité et les performances des applications.
- Elles constituent la méthode recommandée pour toute requête intégrant des données provenant de l'utilisateur ou de l'application.