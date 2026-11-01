## Objectifs

À la fin de ce chapitre, vous serez capable de :

- comprendre le rôle de la classe `PDO` ;
- distinguer les méthodes de consultation et de mise à jour ;
- utiliser les principales méthodes de la classe ;
- choisir la méthode la plus adaptée selon le contexte ;
- comprendre le cycle de vie d'une connexion PDO.

---

# 3.1 Rôle de la classe `PDO`

La classe `PDO` représente **la connexion entre une application PHP et une base de données**.

Une fois cette connexion établie, elle permet :

- d'exécuter des requêtes SQL ;
- de préparer des requêtes paramétrées ;
- de gérer les transactions ;
- de récupérer le dernier identifiant inséré ;
- de configurer certains paramètres de la connexion.

Dans une application correctement structurée, **un seul objet `PDO` est créé par requête HTTP**.

---

# 3.2 Cycle de vie d'un objet PDO

L'utilisation d'un objet `PDO` suit généralement les étapes suivantes :

La connexion est automatiquement fermée lorsque le script PHP se termine.

Il est néanmoins possible de la fermer explicitement :

```
$pdo = null;
```

Dans la pratique, cette opération est rarement nécessaire.

---

# 3.3 Les principales méthodes

La classe `PDO` comporte plusieurs dizaines de méthodes, mais seules quelques-unes sont utilisées quotidiennement.

|Méthode|Retour|Utilisation|
|---|---|---|
|`prepare()`|`PDOStatement`|Préparer une requête paramétrée|
|`query()`|`PDOStatement`|Exécuter un `SELECT` sans paramètre|
|`exec()`|`int`|Exécuter une requête de mise à jour sans paramètre|
|`lastInsertId()`|`string`|Dernier identifiant généré|
|`beginTransaction()`|`bool`|Démarrer une transaction|
|`commit()`|`bool`|Valider une transaction|
|`rollBack()`|`bool`|Annuler une transaction|
|`setAttribute()`|`bool`|Modifier un attribut PDO|
|`getAttribute()`|`mixed`|Lire un attribut PDO|
|`quote()`|`string`|Échapper une chaîne (usage limité)|

Les chapitres suivants détailleront certaines de ces méthodes plus en profondeur.

---

# 3.4 `query()`

## Principe

`query()` exécute immédiatement une requête SQL.

Elle est principalement utilisée pour les requêtes **SELECT ne comportant aucun paramètre**.

```
$stmt = $pdo->query(
    "SELECT id, nom
     FROM utilisateur"
);
```

La méthode retourne un objet `PDOStatement`.

On peut ensuite récupérer les données.

```
$utilisateurs = $stmt->fetchAll();
```

---

## Quand utiliser `query()` ?

Lorsque la requête :

- ne contient aucun paramètre ;
- est exécutée une seule fois.

Exemple :

```
$sql = <<<SQL
SELECT id,
       nom,
       prenom
FROM utilisateur
ORDER BY nom
SQL;

$stmt = $pdo->query($sql);

return $stmt->fetchAll();
```

---

## Quand éviter `query()` ?

Dès qu'une valeur provient :

- d'un formulaire ;
- d'une URL ;
- d'une session ;
- d'un cookie ;
- d'une API.

Dans ce cas il faut utiliser une **requête préparée**.

---

# 3.5 `prepare()`

`prepare()` prépare une requête SQL contenant un ou plusieurs paramètres.

```
$sql = <<<SQL
SELECT *
FROM utilisateur
WHERE id = :id
SQL;

$stmt = $pdo->prepare($sql);
```

À ce stade, **la requête n'est pas encore exécutée**.

Elle sera exécutée plus tard grâce à :

```
$stmt->execute();
```

ou

```
$stmt->execute([
    "id" => 12
]);
```

La préparation d'une requête apporte plusieurs avantages :

- protection contre les injections SQL ;
- meilleure lisibilité ;
- possibilité d'exécuter plusieurs fois la même requête.

---

# 3.6 `exec()`

Contrairement à `query()`, la méthode `exec()` **ne retourne aucun jeu de résultats**.

Elle est destinée aux requêtes de mise à jour sans paramètres.

Exemple :

```
$sql = <<<SQL
UPDATE utilisateur
SET actif = 0
SQL;

$nb = $pdo->exec($sql);
```

La valeur retournée correspond au nombre de lignes modifiées.

```
echo $nb;
```

---

## Cas d'utilisation

- UPDATE
- DELETE
- INSERT
- CREATE TABLE
- DROP TABLE

sans paramètres.

Dès qu'un paramètre est nécessaire, on préférera `prepare()`.

---

# 3.7 `lastInsertId()`

Cette méthode retourne l'identifiant généré lors du dernier `INSERT`.

Exemple :

```
$sql = <<<SQL
INSERT INTO utilisateur(nom)
VALUES ('Martin')
SQL;

$pdo->exec($sql);

$id = $pdo->lastInsertId();
```

Si la clé primaire est auto-incrémentée :

```
id = 128
```

Cette méthode est très utile lorsqu'il faut créer immédiatement des enregistrements enfants.

---

# 3.8 `beginTransaction()`

Cette méthode démarre une transaction.

```
$pdo->beginTransaction();
```

Toutes les requêtes suivantes feront partie de cette transaction.

Les transactions seront étudiées en détail dans un chapitre dédié.

---

# 3.9 `commit()`

Valide définitivement les modifications.

```
$pdo->commit();
```

Les données deviennent alors visibles pour les autres utilisateurs.

---

# 3.10 `rollBack()`

Annule toutes les modifications effectuées depuis le début de la transaction.

```
$pdo->rollBack();
```

La base retrouve son état initial.

---

# 3.11 `getAttribute()`

Cette méthode permet de lire la configuration actuelle de PDO.

Exemple :

```
echo $pdo->getAttribute(PDO::ATTR_DRIVER_NAME);
```

Résultat :

```
mysql
```

Autres exemples :

```
echo $pdo->getAttribute(PDO::ATTR_SERVER_VERSION);

echo $pdo->getAttribute(PDO::ATTR_CLIENT_VERSION);
```

---

# 3.12 `setAttribute()`

Permet de modifier un attribut après la création de la connexion.

Exemple :

```
$pdo->setAttribute(
    PDO::ATTR_DEFAULT_FETCH_MODE,
    PDO::FETCH_OBJ
);
```

Toutes les requêtes suivantes retourneront alors des objets.

---

# 3.13 `quote()`

`quote()` ajoute automatiquement les guillemets et échappe les caractères spéciaux d'une chaîne.

```
$nom = "O'Reilly";

echo $pdo->quote($nom);
```

Résultat :

```
'O''Reilly'
```

Cette méthode est aujourd'hui **peu utilisée**, car les requêtes préparées offrent une solution plus sûre et plus lisible.

---

# 3.14 Résumé des méthodes

|Méthode|Requête concernée|Paramètres SQL|Retour|
|---|---|---|---|
|`query()`|SELECT|❌|`PDOStatement`|
|`prepare()`|Toutes|✔|`PDOStatement`|
|`exec()`|INSERT, UPDATE, DELETE, DDL|❌|Nombre de lignes|
|`lastInsertId()`|INSERT|—|Identifiant|
|`beginTransaction()`|Transactions|—|bool|
|`commit()`|Transactions|—|bool|
|`rollBack()`|Transactions|—|bool|
|`setAttribute()`|Configuration|—|bool|
|`getAttribute()`|Configuration|—|Valeur|

---

# 3.15 Quelle méthode utiliser ?

Ce schéma constitue une règle simple à retenir :

- **SELECT sans paramètre** → `query()`
- **Toute requête avec paramètres** → `prepare()`
- **INSERT / UPDATE / DELETE sans paramètres** → `exec()`

---

# 3.16 Exemple complet

```
$db = Database::getInstance();

$sql = <<<SQL
SELECT id,
       nom,
       prenom
FROM utilisateur
ORDER BY nom
SQL;

$stmt = $db->query($sql);

$utilisateurs = $stmt->fetchAll();

$stmt->closeCursor();

return $utilisateurs;
```

Cet exemple illustre le cycle classique :

1. récupération de la connexion ;
2. exécution de la requête ;
3. récupération des résultats ;
4. fermeture du curseur.

---

# Bonnes pratiques

- Utiliser **`prepare()`** dès qu'une requête contient des paramètres.
- Réserver **`query()`** aux requêtes de consultation sans paramètres.
- Utiliser **`exec()`** uniquement pour les requêtes de mise à jour sans paramètres.
- Fermer explicitement les curseurs (`closeCursor()`) lorsqu'une même connexion exécute plusieurs requêtes successives.
- Centraliser la création de l'objet `PDO` dans une classe `Database`.
- Ne jamais instancier directement `PDO` dans les contrôleurs ou les vues.

---

# À retenir

- La classe **`PDO`** représente la connexion à la base de données.
- Les trois méthodes essentielles sont **`query()`**, **`prepare()`** et **`exec()`**.
- `prepare()` est la méthode à privilégier dès qu'une requête contient des paramètres.
- `query()` est réservée aux requêtes de consultation simples.
- `exec()` est destinée aux requêtes de mise à jour ne retournant pas de résultats.
- Les méthodes `beginTransaction()`, `commit()` et `rollBack()` permettent de garantir la cohérence des données lors d'opérations complexes.
- Les objets `PDOStatement` retournés par `query()` et `prepare()` seront étudiés en détail dans le chapitre suivant.