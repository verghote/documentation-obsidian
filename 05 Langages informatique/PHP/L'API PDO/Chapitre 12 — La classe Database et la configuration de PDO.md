## Objectifs

À la fin de ce chapitre, vous serez capable de :

- comprendre l'intérêt d'une classe de connexion centralisée ;
- éviter la création répétée d'objets PDO ;
- mettre en place un accès unique à la base de données ;
- configurer correctement PDO ;
- externaliser les paramètres de connexion ;
- créer une architecture plus maintenable.

---

# 12.1 Introduction

Dans une application PHP utilisant une base de données, de nombreux scripts ont besoin d'un objet PDO.

Une mauvaise approche consiste à recréer une connexion dans chaque fichier :

```
$db = new PDO(...);
```

Cette solution provoque plusieurs problèmes :

- duplication du code ;
- difficulté de modification des paramètres ;
- risque de créer trop de connexions ;
- configuration non centralisée.

La solution consiste à créer une classe dédiée :

```
Application PHP

        |
        |
        v

Classe Database

        |
        |
        v

Objet PDO unique

        |
        |
        v

SGBDR
```

---

# 12.2 Le principe du singleton

La classe `Database` utilise généralement le principe du **singleton**.

L'objectif est :

> créer une seule instance PDO pendant l'exécution de l'application.

Au lieu de :

```
$db1 = new PDO(...);

$db2 = new PDO(...);

$db3 = new PDO(...);
```

on obtient :

```
$db1 = Database::getInstance();

$db2 = Database::getInstance();

$db3 = Database::getInstance();
```

Les trois variables utilisent le même objet PDO.

---

# 12.3 Structure générale de la classe Database

Exemple :

```
class Database
{

    private static ?PDO $instance = null;



    public static function getInstance(): PDO
    {

        if (self::$instance === null) {

            self::$instance = new PDO(
                ...
            );

        }


        return self::$instance;

    }

}
```

---

# 12.4 Pourquoi un attribut statique privé ?

L'attribut :

```
private static ?PDO $instance;
```

permet de conserver l'objet PDO créé.

Il est :

## Privé

Pour empêcher une modification directe :

```
Database::$instance = null;
```

---

## Statique

Car il appartient à la classe et non à une instance.

On peut donc appeler :

```
Database::getInstance();
```

sans créer d'objet :

```
$db = new Database();
```

---

# 12.5 Création de l'objet PDO

Exemple de connexion MySQL :

```
self::$instance = new PDO(
    "mysql:host=localhost;dbname=formation;charset=utf8",
    "root",
    ""
);
```

La chaîne de connexion contient :

|Élément|Rôle|
|---|---|
|mysql|pilote PDO utilisé|
|host|serveur|
|dbname|base de données|
|charset|encodage|

---

# 12.6 Centraliser la configuration

Les paramètres de connexion ne doivent pas être dispersés dans le code.

Il est préférable d'utiliser un fichier de configuration.

Exemple :

```
config/
 |
 └── database.php
```

Contenu :

```
<?php

return [

    "host" => "localhost",

    "database" => "formation",

    "user" => "root",

    "password" => "",

    "port" => 3306,

    "charset" => "utf8"

];
```

---

# 12.7 Lecture de la configuration

Dans `Database.php` :

```
$config = require(
    RACINE . "/config/database.php"
);
```

Puis :

```
$dsn =
"mysql:host={$config['host']};
dbname={$config['database']};
port={$config['port']};
charset={$config['charset']}";
```

---

# 12.8 Configuration complète de PDO

Lors de la création de PDO, il est possible de transmettre des options.

Exemple :

```
$options = [

    PDO::ATTR_ERRMODE =>
        PDO::ERRMODE_EXCEPTION,


    PDO::ATTR_DEFAULT_FETCH_MODE =>
        PDO::FETCH_ASSOC,


    PDO::ATTR_EMULATE_PREPARES =>
        false

];
```

Puis :

```
self::$instance = new PDO(
    $dsn,
    $config["user"],
    $config["password"],
    $options
);
```

---

# 12.9 `PDO::ATTR_ERRMODE`

Cette option définit la gestion des erreurs PDO.

Trois modes existent.

---

## `PDO::ERRMODE_SILENT`

PDO ne déclenche aucune exception.

Exemple :

```
PDO::ERRMODE_SILENT
```

Il faut ensuite vérifier manuellement :

```
errorInfo()
```

Ce mode est rarement utilisé dans les applications modernes.

---

## `PDO::ERRMODE_WARNING`

PDO génère un avertissement PHP.

Exemple :

```
PDO::ERRMODE_WARNING
```

Le script continue son exécution.

---

## `PDO::ERRMODE_EXCEPTION`

PDO déclenche une exception.

Exemple :

```
PDO::ERRMODE_EXCEPTION
```

C'est le mode recommandé.

Il permet d'utiliser :

```
try
{

}
catch(Exception $e)
{

}
```

---

# 12.10 `PDO::ATTR_DEFAULT_FETCH_MODE`

Cette option définit le format par défaut des résultats.

Sans configuration :

```
$stmt->fetch(PDO::FETCH_ASSOC);
```

Avec :

```
PDO::ATTR_DEFAULT_FETCH_MODE =>
PDO::FETCH_ASSOC
```

on peut écrire :

```
$stmt->fetch();
```

Le résultat sera automatiquement :

```
[
    "nom" => "Martin",
    "prenom" => "Paul"
]
```

---

# 12.11 Les différents modes de récupération

## Tableau associatif

```
PDO::FETCH_ASSOC
```

Résultat :

```
$ligne["nom"];
```

---

## Objet

```
PDO::FETCH_OBJ
```

Résultat :

```
$ligne->nom;
```

---

## Tableau numérique

```
PDO::FETCH_NUM
```

Résultat :

```
$ligne[0];
```

---

# 12.12 `PDO::ATTR_EMULATE_PREPARES`

Cette option est importante :

```
PDO::ATTR_EMULATE_PREPARES => false
```

Elle oblige PDO à utiliser les requêtes préparées natives du SGBDR.

Avantages :

- meilleure sécurité ;
- meilleure gestion des paramètres ;
- protection contre les injections SQL ;
- meilleure compatibilité avec certains types SQL.

Dans une application professionnelle, cette option doit rester à :

```
false
```

---

# 12.13 Exemple complet de classe Database

```
class Database
{

    private static ?PDO $instance = null;



    public static function getInstance(): PDO
    {

        if (self::$instance === null) {


            $config =
                require RACINE .
                "/config/database.php";


            $dsn =
            "mysql:host={$config['host']};
            dbname={$config['database']};
            port={$config['port']};
            charset={$config['charset']}";


            self::$instance = new PDO(

                $dsn,

                $config["user"],

                $config["password"],

                [

                    PDO::ATTR_ERRMODE =>
                        PDO::ERRMODE_EXCEPTION,


                    PDO::ATTR_DEFAULT_FETCH_MODE =>
                        PDO::FETCH_ASSOC,


                    PDO::ATTR_EMULATE_PREPARES =>
                        false

                ]

            );

        }


        return self::$instance;

    }

}
```

---

# 12.14 Utilisation dans une classe métier

Exemple :

```
class Etudiant
{

    public static function getAll(): array
    {

        $db = Database::getInstance();


        $sql = "
        SELECT *
        FROM etudiant
        ORDER BY nom
        ";


        $stmt = $db->query($sql);


        return $stmt->fetchAll();

    }

}
```

La classe métier ne connaît :

- ni le serveur ;
- ni l'utilisateur SQL ;
- ni le mot de passe.

Elle utilise uniquement :

```
Database::getInstance()
```

---

# 12.15 Avantages de cette architecture

## Maintenance facilitée

Changer de serveur :

```
localhost
```

vers :

```
serveur-production
```

nécessite uniquement une modification de configuration.

---

## Sécurité améliorée

Les informations sensibles :

- utilisateur ;
- mot de passe ;
- serveur ;

sont regroupées dans un emplacement contrôlé.

---

## Code plus propre

Les classes métiers restent concentrées sur leur rôle :

```
Classe métier
       |
       |
       v
Database
       |
       |
       v
PDO
       |
       |
       v
Base de données
```

---

# 12.16 Bonnes pratiques

- Ne jamais créer plusieurs connexions PDO inutilement.
- Centraliser la connexion dans une classe dédiée.
- Stocker les paramètres dans un fichier de configuration.
- Utiliser `PDO::ERRMODE_EXCEPTION`.
- Désactiver l'émulation des requêtes préparées.
- Définir un mode de récupération adapté.
- Ne jamais écrire les identifiants de connexion directement dans les classes métiers.
- Utiliser la classe `Database` partout dans l'application.

---

# À retenir

- La classe `Database` centralise la création de l'objet PDO.
- Le singleton garantit une connexion unique pendant l'exécution.
- La configuration PDO influence la sécurité et la simplicité du code.
- `ERRMODE_EXCEPTION`, `FETCH_ASSOC` et `EMULATE_PREPARES=false` sont des réglages couramment recommandés.
- Une architecture propre sépare la connexion, l'accès aux données et la logique métier.