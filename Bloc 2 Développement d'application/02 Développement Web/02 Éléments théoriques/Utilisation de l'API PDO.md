## 1. Présentation rapide de PDO

PDO (*PHP Data Objects*) est une extension PHP permettant de communiquer avec une base de données.

Elle fournit une interface commune permettant de travailler avec différents systèmes de gestion de bases de données :

- MySQL ;
- MariaDB ;
- PostgreSQL ;
- SQLite ;
- etc.

Dans notre application, PDO est utilisé pour communiquer avec une base MySQL.

PDO permet notamment :

- d'ouvrir une connexion à la base ;
- d'exécuter des requêtes SQL ;
- d'envoyer des paramètres aux requêtes ;
- de récupérer les résultats ;
- de gérer les erreurs SQL.

Exemple simple :

```php
$db = new PDO(
    "mysql:host=localhost;dbname=portfolio",
    "utilisateur",
    "motdepasse"
);

$resultat = $db->query(
    "select * from projet"
);
````

Cependant, dans une application professionnelle, on ne crée pas une connexion PDO  à chaque requête.

La connexion doit être centralisée.

C'est le rôle de la classe `Database`.

# 2. Création de l'objet PDO dans une application

```php
$db = Database::getInstance();
```

La variable `$db` contient alors un objet PDO connecté à la base.

Toutes les classes métier utilisent cette même connexion.

# 3. La classe Database

La classe `Database` réalise plusieurs opérations :
- lecture de la configuration contenu dans le fichier config/database.php;
- création de la connexion PDO ;
- configuration du comportement PDO ;
- conservation d'une connexion unique.

La méthode :

```php
public static function getInstance(): PDO
```

crée la connexion uniquement lors du premier appel.

Les appels suivants retournent la même connexion.

On utilise donc le principe du **singleton**.

## Création de la connexion PDO

La connexion est créée avec :

```php
$chaine = "mysql:host=$host;dbname=$database;port=$port;charset=utf8mb4";

$db = new PDO($chaine, $user, $password);
```

La chaîne contient :

| Élément | Rôle                       |
| ------- | -------------------------- |
| mysql   | type de SGBD               |
| host    | serveur de base de données |
| dbname  | nom de la base             |
| port    | port MySQL                 |
| charset | encodage utilisé           |

## Configuration PDO

Après création, PDO est configuré.
### Gestion des erreurs

```php
$db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
```

En cas d'erreur SQL, PDO génère une exception.
### Mode de récupération des données

```php
$db->setAttribute(PDO::ATTR_DEFAULT_FETCH_MODE, PDO::FETCH_ASSOC);
```

Les résultats seront retournés sous forme de tableaux associatifs.

Exemple :

```
[
    "id" => 12,
    "nom" => "Mon projet"
]
```

### Désactivation des requêtes préparées simulées

```php
$db->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
```

PDO utilise alors les vraies requêtes préparées du serveur MySQL.

# 4. Le fichier de configuration

Les paramètres de connexion ne doivent jamais être écrits directement dans le code.

Ils sont placés dans : config/database.php

```php
return [
    "host" => "localhost",
    "database" => "portfolio",
    "user" => "root",
    "password" => "",
    "port" => 3306
];
```

La classe `Database` charge ce fichier :

```php
$lesParametres = Config::chargerPhp('database');
```

Ainsi :

- le code PHP reste identique entre développement et production ;
- les mots de passe ne sont pas dans les classes métier.
# 5. Organisation des accès SQL

Dans notre architecture :

```
Contrôleur
      ▼
Service métier
      ▼
Classe métier
      ▼
Classe Table / Select / Database
      ▼
Base MySQL
```

Les classes métier contiennent les requêtes SQL.

Exemple :

```
ClasseMetier\Projet
```

contient les opérations concernant les projets.

# 6. La classe Select

La classe `Select` simplifie les requêtes de consultation.

Elle utilise PDO mais évite de répéter du code.

Elle propose trois méthodes principales.

## getRows() : array

Pour récupérer plusieurs lignes.

Exemple :

```php
$select = new Select();

$projets = $select->getRows($sql);
```

Résultat :

```
[
    [
        "id" => 1,
        "nom" => "Projet A"
    ],
    [
        "id" => 2,
        "nom" => "Projet B"
    ]
]
```

## getRow() : ?array

Pour récupérer une seule ligne.

Exemple :

```php
$projet = $select->getRow($sql, ["id" => $id]);
```

Résultat :

```
[
    "id" => 3,
    "nom" => "Application web"
]
```

ou :

```
null
```

si aucune ligne n'existe.

## getValue() : mixed

Pour récupérer une seule valeur.

Exemple :

```php
$existe = $select->getValue($sql, ['db' => $dbname, 'table' => $table]);
if ((int)$existe === 0) {  
    throw new Exception("La table `$table` est absente de la base `$dbname`.");  
}
```

# 7. Consultation sans paramètre

Exemple :  Récupérer tous les projets.

```php
public static function getAll(): array
{
    $sql = <<<SQL
        select id, nom
        from projet
        order by nom;
SQL;

    $select = new Select();

    return $select->getRows($sql);
}
```

La requête ne contient aucun paramètre.

Elle retourne plusieurs lignes.

On utilise donc :  getRows()

# 8. Consultation avec paramètre

Exemple :  Récupérer un projet par son identifiant.

SQL :

```
select id, nom
from projet
where id = :id
```

Le paramètre :id  sera remplacé par une valeur.

Méthode :

```php
public static function getById(int $id): mixed
{
    $sql = <<<SQL
        select id, nom
        from projet
        where id = :id;
SQL;

    $select = new Select();

    return $select->getRow($sql, ["id" => $id]);
}
```

# 9. Pourquoi utiliser des paramètres ?

Il ne faut jamais écrire :

```
$sql = "select * from projet where id = ".$id;
```

Cette méthode est dangereuse.

Elle permet des injections SQL.

Il faut utiliser :

```
where id = :id
```

puis :

```
[
 "id" => $id
]
```

PDO transmet alors correctement la valeur.


# 10. Vérifier l'existence d'un élément

Souvent on veut simplement savoir si une ligne existe.

Exemple :

```php
public static function existe(int $idProjet): bool
{
    $sql = <<<SQL
        select 1
        from projet
        where id = :id
SQL;

    $select = new Select();

    return $select->getRow($sql, ["id" => $idProjet]) !== null;
}
```

La requête retourne  1 si la ligne existe.

# 11. Les modifications avec PDO

Les opérations :
- INSERT
- UPDATE
- DELETE

utilisent directement PDO.

On récupère la connexion :

```
$db = Database::getInstance();
```

Puis :

```
$cmd = $db->prepare($sql);
```

# 12. INSERT

Exemple :

```php
public static function ajouter(string $nom): int
{
    $db = Database::getInstance();

    $sql = <<<SQL
        insert into projet(nom)
        values(:nom)
SQL;

    $cmd = $db->prepare($sql);

    $cmd->execute(["nom" => $nom]);

    return (int)$db->lastInsertId();
}
```

Après un INSERT  lastInsertId() permet de récupérer l'identifiant créé.

# 13. UPDATE

Exemple :

```php
public static function modifierNom(int $id, string $nom): bool
{
    $db = Database::getInstance();

    $sql = <<<SQL
        update projet
        set nom = :nom
        where id = :id;
SQL;

    $cmd = $db->prepare($sql);

    $cmd->execute(["id" => $id, "nom" => $nom]);

    return $cmd->rowCount() > 0;
}
```

`rowCount()` indique combien de lignes ont été modifiées.

# 14. DELETE

Exemple :

```php
public static function supprimer(int $id): bool
{
    $db = Database::getInstance();

    $sql = <<<SQL
        delete from projet
        where id = :id;
SQL;

    $cmd = $db->prepare($sql);

    $cmd->execute(["id" => $id]);

    return $cmd->rowCount() > 0;
}
```

Si aucune ligne n'est supprimée rowCount() == 0
# 15. Exemple avec une table d'association

La table :

```
competenceprojet
```

associe :

```
projet
  |
  |
competence
```

Ajouter une compétence :

```sql
insert into competenceprojet(idProjet,idCompetence)
values (:idProjet,:idCompetence);
```

Code PHP :

```php
$db = Database::getInstance();

$cmd = $db->prepare($sql);

$cmd->execute(["idProjet" => $idProjet, "idCompetence" => $idCompetence]);
```

# Conclusion

Dans cette architecture :

- PDO réalise la communication avec MySQL ;
- `Database` crée une connexion unique ;
- `Select` simplifie les lectures ;
- les classes métier contiennent les requêtes SQL ;
- les paramètres préparés protègent contre les injections SQL.

La règle principale est :

> Une classe métier connaît le SQL de son domaine, mais elle ne crée jamais directement la connexion PDO.