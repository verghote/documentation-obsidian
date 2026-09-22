## Objectifs

À la fin de ce chapitre, vous serez capable de :

- vérifier que l'extension PDO est installée ;
- comprendre le rôle des pilotes (drivers) PDO ;
- connaître la structure d'une chaîne de connexion (**DSN**) ;
- établir une connexion à différents SGBDR ;
- configurer correctement un objet `PDO` ;
- centraliser la création de la connexion dans une classe dédiée.

# 2.1 L'architecture de PDO

PDO est constitué de deux éléments distincts :

- **l'extension PDO**, commune à tous les SGBDR ;
- **un pilote (driver)** propre à chaque moteur de base de données.

L'API reste identique ; seul le pilote change selon le SGBDR utilisé.

# 2.2 Vérifier que PDO est installé

La majorité des distributions PHP installent PDO par défaut.

Il est néanmoins conseillé de vérifier que l'extension est disponible.

```
<?php

phpinfo();
```

Rechercher ensuite une section intitulée :

```
PDO
```

Elle indique notamment :

```
PDO support => enabled
PDO drivers => mysql, pgsql, sqlite
```

Les pilotes disponibles apparaissent également avec :

```
print_r(PDO::getAvailableDrivers());
```

Exemple de résultat :

```
Array
(
    [0] => mysql
    [1] => sqlite
    [2] => pgsql
)
```

---

# 2.3 Les principaux pilotes

|Pilote|SGBDR|
|---|---|
|mysql|MySQL / MariaDB|
|pgsql|PostgreSQL|
|sqlite|SQLite|
|sqlsrv|Microsoft SQL Server|
|oci|Oracle|
|ibm|IBM DB2|
|firebird|Firebird|

Chaque pilote possède son propre format de connexion.

---

# 2.4 Le principe du DSN

Pour ouvrir une connexion, PDO utilise une **Data Source Name (DSN)**.

Le DSN décrit la base de données à laquelle se connecter.

Sa structure générale est :

```
pilote:paramètre1=valeur1;paramètre2=valeur2;...
```

Exemple :

```
mysql:host=localhost;dbname=formation;charset=utf8mb4
```

On retrouve généralement :

- le pilote ;
- le serveur ;
- la base de données ;
- le port ;
- le jeu de caractères.

---

# 2.5 Connexion à MySQL

Connexion minimale :

```
$dsn = "mysql:host=localhost;dbname=formation;charset=utf8mb4";

$pdo = new PDO(
    $dsn,
    "root",
    ""
);
```

Connexion avec un port personnalisé :

```
$dsn = "mysql:host=localhost;port=3307;dbname=formation;charset=utf8mb4";
```

---

# 2.6 Connexion à PostgreSQL

```
$dsn = "pgsql:host=localhost;
        port=5432;
        dbname=formation";

$pdo = new PDO(
    $dsn,
    "postgres",
    "secret"
);
```

---

# 2.7 Connexion à SQLite

SQLite ne fonctionne pas avec un serveur.

La connexion s'effectue directement sur un fichier.

```
$pdo = new PDO(
    "sqlite:data/formation.db"
);
```

Ou :

```
$pdo = new PDO(
    "sqlite:C:/Bases/formation.db"
);
```

---

# 2.8 Connexion à SQL Server

```
$dsn = "sqlsrv:Server=localhost;
        Database=formation";

$pdo = new PDO(
    $dsn,
    "sa",
    "password"
);
```

---

# 2.9 Connexion à Oracle

```
$dsn = "oci:dbname=localhost/XE";

$pdo = new PDO(
    $dsn,
    "system",
    "password"
);
```

---

# 2.10 Le constructeur PDO

La création d'un objet PDO repose sur le constructeur suivant :

```
new PDO(
    string $dsn,
    string $username,
    string $password,
    array $options = []
);
```

Les paramètres sont :

|Paramètre|Description|
|---|---|
|`$dsn`|Chaîne de connexion|
|`$username`|Utilisateur de la base|
|`$password`|Mot de passe|
|`$options`|Configuration de PDO|

---

# 2.11 Les options de configuration

Le quatrième argument du constructeur permet de configurer le comportement de PDO.

Exemple :

```
$options = [

    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,

    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,

    PDO::ATTR_EMULATE_PREPARES => false

];

$pdo = new PDO(
    $dsn,
    $user,
    $password,
    $options
);
```

Les principales options sont détaillées ci-dessous.

---

# 2.12 ATTR_DEFAULT_FETCH_MODE

Détermine le mode de récupération des résultats.

Exemple :

```
PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
```

Il devient alors inutile d'écrire :

```
$stmt->fetch(PDO::FETCH_ASSOC);
```

On peut simplement écrire :

```
$stmt->fetch();
```

Cette configuration améliore la lisibilité du code.

---

# 2.13 ATTR_EMULATE_PREPARES

Cette option indique si PDO doit utiliser :

- les requêtes préparées du SGBDR ;
- ou une simulation réalisée par PDO.

```
PDO::ATTR_EMULATE_PREPARES => false
```

Cette valeur est fortement recommandée.

Les avantages sont :

- meilleure sécurité ;
- meilleure compatibilité avec les types SQL ;
- plan d'exécution optimisé ;
- comportement identique au moteur SQL.

---

# 2.14 ATTR_ERRMODE

Cette option détermine la manière dont PDO signale les erreurs.

Même si la gestion des erreurs est traitée ailleurs dans votre application, il est important de connaître cette option.

Les trois valeurs possibles sont :

|Constante|Description|
|---|---|
|`PDO::ERRMODE_SILENT`|Aucun message automatique|
|`PDO::ERRMODE_WARNING`|Génère un avertissement PHP|
|`PDO::ERRMODE_EXCEPTION`|Génère une exception|

Dans les chapitres suivants, nous considérerons que cette configuration est déjà prise en charge par l'infrastructure de l'application.

---

# 2.15 Autres options utiles

## ATTR_CASE

Détermine la casse des noms de colonnes.

```
PDO::ATTR_CASE
```

Valeurs possibles :

- `CASE_NATURAL`
- `CASE_LOWER`
- `CASE_UPPER`

---

## ATTR_TIMEOUT

Détermine le délai maximal d'attente.

```
PDO::ATTR_TIMEOUT => 5
```

Ici, PDO attend cinq secondes avant d'abandonner la connexion.

---

## ATTR_PERSISTENT

Active les connexions persistantes.

```
PDO::ATTR_PERSISTENT => true
```

Cette option peut améliorer les performances sur certaines applications fortement sollicitées, mais elle demande une bonne maîtrise de la gestion des connexions.

---

## ATTR_STRINGIFY_FETCHES

Transforme automatiquement les nombres en chaînes.

Cette option est rarement utilisée.

---

# 2.16 Le jeu de caractères

Le jeu de caractères doit être défini dans le DSN.

```
charset=utf8mb4
```

Pourquoi `utf8mb4` ?

Il permet notamment de gérer :

- les caractères accentués ;
- toutes les langues ;
- les emojis ;
- les caractères Unicode.

Aujourd'hui, `utf8mb4` est le choix recommandé pour MySQL et MariaDB.

---

# 2.17 Centraliser la connexion

Créer une connexion dans chaque script est une mauvaise pratique.

Au lieu de cela, il est préférable de centraliser la création de l'objet `PDO` dans une classe dédiée.

Tous les accès à la base utiliseront cette classe.

---

# 2.18 Exemple d'une classe Database

Une implémentation simple repose sur le patron **Singleton** afin qu'une seule connexion soit créée pendant l'exécution du script.

```
class Database
{
    private static ?PDO $instance = null;

    public static function getInstance(): PDO
    {
        if (self::$instance === null) {

            $config = require __DIR__ . '/../config/database.php';

            $dsn = sprintf(
                'mysql:host=%s;port=%d;dbname=%s;charset=%s',
                $config['host'],
                $config['port'],
                $config['database'],
                $config['charset']
            );

            self::$instance = new PDO(
                $dsn,
                $config['user'],
                $config['password'],
                [
                    PDO::ATTR_ERRMODE => $config['errmode'],
                    PDO::ATTR_DEFAULT_FETCH_MODE => $config['fetchmode'],
                    PDO::ATTR_EMULATE_PREPARES => false
                ]
            );
        }

        return self::$instance;
    }
}
```

L'accès à la base devient alors très simple :

```
$db = Database::getInstance();
```

Tous les composants de l'application utilisent la même connexion.

---

# 2.19 Le fichier de configuration

Les paramètres de connexion sont généralement stockés dans un fichier dédié.

Exemple :

```
return [

    'host' => 'localhost',

    'database' => 'formation',

    'user' => 'root',

    'password' => '',

    'port' => 3306,

    'charset' => 'utf8mb4',

    'errmode' => PDO::ERRMODE_EXCEPTION,

    'fetchmode' => PDO::FETCH_ASSOC

];
```

Cette approche présente plusieurs avantages :

- séparation entre le code et la configuration ;
- facilité de déploiement ;
- maintenance simplifiée ;
- possibilité d'avoir plusieurs environnements (développement, test, production).

---

# 2.20 Bonnes pratiques

- Utiliser **une seule connexion PDO** par requête HTTP.
- Centraliser la création de la connexion dans une classe `Database`.
- Définir le jeu de caractères dès la connexion.
- Désactiver l'émulation des requêtes préparées (`ATTR_EMULATE_PREPARES => false`).
- Définir un mode de récupération par défaut (`FETCH_ASSOC` est le plus courant).
- Éviter de créer directement des objets `PDO` dans les contrôleurs ou les vues.

---

# À retenir

- PDO repose sur une **extension générique** associée à un **pilote spécifique** pour chaque SGBDR.
- La connexion est définie par une **chaîne DSN** qui identifie le serveur, la base de données et les paramètres de connexion.
- Le constructeur `PDO` accepte un tableau d'options permettant de configurer son comportement.
- Une configuration cohérente (`FETCH_ASSOC`, `ATTR_EMULATE_PREPARES => false`, jeu de caractères `utf8mb4`) constitue une base solide pour la majorité des applications.
- La création de l'objet `PDO` doit être **centralisée** dans une classe `Database`, généralement implémentée sous la forme d'un **Singleton**, afin de garantir une connexion unique et facilement réutilisable dans toute l'application.