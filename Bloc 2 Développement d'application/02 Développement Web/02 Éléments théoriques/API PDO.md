# 1. Qu'est-ce que PDO ?

PDO (**PHP Data Objects**) est l'interface officielle de PHP permettant de communiquer avec une base de données.

Elle fournit une API unique pour accéder à différents systèmes de gestion de bases de données (SGBD), notamment :

- MySQL ;
- MariaDB ;
- PostgreSQL ;
- SQLite ;
- SQL Server ;
- Oracle.

Un même programme PHP peut ainsi changer de SGBD avec peu de modifications.

# 2. Le rôle de PDO

PDO permet principalement de :

- ouvrir une connexion à une base de données ;
- exécuter des requêtes SQL ;
- transmettre des paramètres aux requêtes ;
- récupérer les résultats ;
- gérer les erreurs ;
- utiliser des transactions.

PDO ne connaît rien de votre application.

Il se contente d'exécuter des instructions SQL.

# 3. Créer une connexion

Pour communiquer avec une base MySQL, il faut créer un objet `PDO`.

```
$db = new PDO(
    "mysql:host=localhost;dbname=portfolio;charset=utf8mb4",
    "utilisateur",
    "motdepasse"
);
```

La variable `$db` représente désormais la connexion à la base de données.

Toutes les opérations SQL passeront par cet objet.

# 4. La chaîne DSN

Le premier paramètre est appelé **DSN** (_Data Source Name_).

Exemple :

```
mysql:host=localhost;dbname=portfolio;charset=utf8mb4
```

Il indique :

|Élément|Signification|
|---|---|
|`mysql`|Type de SGBD|
|`host`|Serveur de base de données|
|`dbname`|Nom de la base|
|`charset`|Encodage des caractères|

Selon le SGBD utilisé, le DSN change.

# 5. Configurer PDO

Après la connexion, il est recommandé de configurer certains comportements.

## Gestion des erreurs

```php
$db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
```

En cas d'erreur SQL, PDO génère une exception.

Cette configuration est aujourd'hui considérée comme la bonne pratique.

## Récupération des lignes

```php
$db->setAttribute(PDO::ATTR_DEFAULT_FETCH_MODE, PDO::FETCH_ASSOC);
```

Chaque ligne sera retournée sous forme d'un tableau associatif.

Exemple :

```
[
    "id" => 15,
    "nom" => "Projet Web"
]
```

---

## Désactiver l'émulation des requêtes préparées

```php
$db->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
```

PDO utilisera alors les véritables requêtes préparées de MySQL.

# 6. Exécuter une requête simple

Lorsqu'une requête ne contient aucun paramètre, on peut utiliser `query()`.

Exemple :

```php
$sql = "select id, nom
        from projet";

$resultat = $db->query($sql);
```

La variable `$resultat` contient un objet `PDOStatement`.

Cet objet représente le résultat de la requête.

# 7. Récupérer plusieurs lignes

Pour obtenir toutes les lignes :

```php
$lesLignes = $resultat->fetchAll(PDO::FETCH_ASSOC);
```

Exemple :

```
[
    [
        "id" => 1,
        "nom" => "Portfolio"
    ],
    [
        "id" => 2,
        "nom" => "Site Web"
    ]
]
```

# 8. Récupérer une seule ligne

```php
$ligne = $resultat->fetch(PDO::FETCH_ASSOC);
```

Résultat :

```
[
    "id" => 5,
    "nom" => "Application"
]
```

ou

```
false
```

si aucune ligne n'existe.

# 9. Récupérer une seule valeur

Lorsque seule la première colonne de la première ligne est utile :

```php
$nb = $resultat->fetchColumn();
```

Exemple :

```
select count(*)
from projet;
```

Retour :

```
15
```

# 10. Pourquoi utiliser des requêtes préparées ?

Il ne faut jamais construire une requête SQL par concaténation.

## Exemple 1 : contourner une condition

**Mauvais exemple :**

```php
$id = $_GET["id"];

$sql = "SELECT *
        FROM projet
        WHERE id = " . $id;
```

Si l'utilisateur saisit :

```text
1 OR 1=1
```

la requête exécutée devient :

```sql
SELECT *
FROM projet
WHERE id = 1 OR 1=1
```

La condition `1=1` étant toujours vraie, la requête retourne tous les projets au lieu d'un seul.

## Exemple 2 : exécution d'une instruction supplémentaire

Supposons maintenant le code suivant :

```php
$id = $_GET["id"];

$sql = "SELECT *
        FROM projet
        WHERE id = " . $id;
```

Un utilisateur malveillant pourrait tenter de saisir une valeur comme :

```text
1; DELETE FROM projet
```

La requête construite deviendrait alors :

```sql
SELECT *
FROM projet
WHERE id = 1;

DELETE FROM projet;
```

Si le SGBD ou le pilote autorise l'exécution de plusieurs instructions SQL dans une même requête (ce qui dépend de la configuration utilisée), une telle attaque pourrait provoquer la suppression des données.

De la même manière, un attaquant pourrait tenter d'exécuter une instruction telle que :

```sql
DROP TABLE projet;
```

afin de supprimer complètement une table.

> **Remarque**
>
> Les possibilités exactes d'une injection SQL dépendent du système de gestion de base de données (MySQL, PostgreSQL, SQL Server, etc.), du pilote PDO utilisé et de leur configuration. Certaines configurations empêchent l'exécution de plusieurs instructions dans une même requête, mais il ne faut jamais compter sur cette protection.

## La bonne pratique

La requête doit être écrite avec un paramètre nommé :

```sql
SELECT *
FROM projet
WHERE id = :id
```

Puis la valeur est transmise séparément :

```php
$projet = $select->getRow("SELECT * FROM projet WHERE id = :id",  ["id" => $_GET["id"]]);
```

PDO transmet alors la valeur comme une donnée et non comme une partie de la requête SQL.

Même si l'utilisateur saisit :

```text
1 OR 1=1
```

ou

```text
1; DELETE FROM projet
```

ces caractères sont interprétés comme une simple valeur du paramètre `:id`. Ils ne modifient jamais la structure de la requête SQL.

L'utilisation systématique des requêtes préparées constitue la protection la plus efficace contre les injections SQL.
# 11. Préparer une requête

Une requête contenant des paramètres doit être préparée.

```php
$cmd = $db->prepare($sql);
```

La méthode `prepare()` analyse la requête SQL sans encore l'exécuter.

Elle retourne un objet `PDOStatement`.
# 12. Transmettre les paramètres

Deux méthodes existent.
## Avec execute()

La plus simple :

```php
$cmd->execute(["id" => $id]);
```

## Avec bindValue()

```php
$cmd->bindValue("id", $id, PDO::PARAM_INT);

$cmd->execute();
```

Cette méthode permet notamment de préciser le type du paramètre.

# 13. Exemple complet

```php
$sql = <<<SQL
	select id, nom
	from projet
	where id = :id;
SQL;

$cmd = $db->prepare($sql);

$cmd->execute(["id" => $id]);

$projet = $cmd->fetch(PDO::FETCH_ASSOC);
```

# 14. INSERT

```php
$sql = <<<SQL
	insert into projet(nom)
	values(:nom);
SQL;

$cmd = $db->prepare($sql);

$cmd->execute(["nom" => $nom]);
```

Après l'insertion :

```
$id = $db->lastInsertId();
```

retourne l'identifiant créé lorsque la clé primaire est auto-incrémentée.

# 15. UPDATE

```php
$sql = <<<SQL
	update projet
	set nom = :nom
	where id = :id;
SQL;

$cmd = $db->prepare($sql);

$cmd->execute(["id" => $id, "nom" => $nom]);
```

Le nombre de lignes modifiées est obtenu avec :

```php
$nb = $cmd->rowCount();
```

# 16. DELETE

```php
$sql = <<<SQL
	delete
	from projet
	where id = :id;
SQL;

$cmd = $db->prepare($sql);

$cmd->execute(["id" => $id]);
```

# 17. Les transactions

Une transaction permet de regrouper plusieurs opérations SQL afin qu'elles soient exécutées comme une seule unité.

Début :

```php
$db->beginTransaction();
```

Validation :

```php
$db->commit();
```

Annulation :

```php
$db->rollBack();
```

Exemple :

```php
$db->beginTransaction();

try {
    // plusieurs INSERT ou UPDATE
    $db->commit();
} catch (Exception $e) {
    $db->rollBack();
    throw $e;
}
```

# 18. Fermer une requête

Après avoir terminé la lecture des résultats :

```php
$cmd->closeCursor();
```

Cette méthode libère les ressources utilisées par la requête.

# 19. Les principales classes de PDO

| Classe         | Rôle                                  |
| -------------- | ------------------------------------- |
| `PDO`          | Connexion à la base de données        |
| `PDOStatement` | Requête SQL préparée ou exécutée      |
| `PDOException` | Exception générée en cas d'erreur SQL |

# 20. Les méthodes essentielles à connaître

## Classe `PDO`

| Méthode              | Rôle                                |
| -------------------- | ----------------------------------- |
| `query()`            | Exécuter une requête sans paramètre |
| `prepare()`          | Préparer une requête paramétrée     |
| `beginTransaction()` | Début d'une transaction             |
| `commit()`           | Valider une transaction             |
| `rollBack()`         | Annuler une transaction             |
| `lastInsertId()`     | Récupérer l'identifiant créé        |
| `setAttribute()`     | Configurer PDO                      |
## Classe `PDOStatement`

| Méthode         | Rôle                       |
| --------------- | -------------------------- |
| `execute()`     | Exécuter la requête        |
| `bindValue()`   | Associer un paramètre      |
| `fetch()`       | Lire une ligne             |
| `fetchAll()`    | Lire toutes les lignes     |
| `fetchColumn()` | Lire une seule valeur      |
| `rowCount()`    | Nombre de lignes affectées |
| `closeCursor()` | Libérer les ressources     |

# Résumé

PDO est la couche standard de PHP permettant d'accéder à une base de données. Il fournit une interface commune pour différents SGBD et s'appuie sur deux objets principaux :

- `PDO`, qui représente la connexion à la base de données ;
- `PDOStatement`, qui représente une requête SQL préparée ou exécutée.

Les bonnes pratiques consistent à utiliser des **requêtes préparées**, à **transmettre les valeurs sous forme de paramètres** plutôt que par concaténation, à **gérer les erreurs avec des exceptions** et à **encapsuler plusieurs opérations dans une transaction** lorsque leur exécution doit être atomique.