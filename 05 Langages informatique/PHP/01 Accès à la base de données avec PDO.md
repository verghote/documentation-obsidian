
# 1. Architecture trois tiers

Pour interagir avec une base de données, PHP dispose d'une extension appelée **PDO (PHP Data Objects)**.

Par rapport aux autres solutions existantes, PDO possède de nombreux avantages :

- connexion à la plupart des systèmes de gestion de bases de données relationnelles (SGBDR) ;
- utilisation d'une interface commune quel que soit le SGBDR ;
- utilisation de requêtes préparées ;
- protection contre les injections SQL lorsque les paramètres sont correctement utilisés.

PDO est donc devenu un élément incontournable de l'accès aux bases de données avec PHP.

L'API PDO fournit une interface d'abstraction de l'accès aux données. Les mêmes méthodes peuvent ainsi être utilisées pour exécuter des requêtes ou récupérer des données, quel que soit le SGBDR utilisé.

L'API PDO comprend notamment :

- `PDO` : représente la connexion à la base de données ;
- `PDOStatement` : représente une requête préparée ou un résultat de requête.

L'architecture générale peut être représentée ainsi :

```text
Client
   │
   │ HTTP
   ▼
Serveur Web / PHP
   │
   │ PDO
   ▼
SGBDR
```

# 2. Connexion à la base de données

PDO permet de se connecter à différents systèmes de gestion de bases de données avec une syntaxe similaire.

Seuls certains paramètres de connexion changent selon le SGBDR utilisé.

## Exemple de connexion à MySQL

```php
$dbHost = 'localhost';
$dbUser = 'root';
$dbPassword = '';
$dbBase = 'formation';
$dbPort = '3306';

try {
    $chaine = "mysql:host=$dbHost;dbname=$dbBase;port=$dbPort";

    $db = new PDO(
        $chaine,
        $dbUser,
        $dbPassword
    );
} catch (PDOException $e) {
    echo "Accès à la base de données impossible, vérifiez les paramètres de connexion.";
    exit();
}
```

D'autres paramètres peuvent être transmis lors de l'instanciation de l'objet `PDO` ou après sa création.
## Exemples de chaînes de connexion

### SQLite

```php
$db = new PDO('sqlite:/chemin/vers/fichier_de_donnees.db');
```
### MySQL

```php
$db = new PDO(
    'mysql:host=localhost;dbname=basededonnees',
    'root',
    ''
);
```
### PostgreSQL

```php
$db = new PDO(
    'pgsql:host=localhost port=5432 dbname=igd user=root password='
);
```
### Microsoft Access

```php
$db = new PDO('odbc:access_inscription');
```
### SQL Server

```php
$db = new PDO(
    'odbc:sqlServer_inscription',
    'sa',
    'BTSSIO'
);
```

Le pilote correspondant au SGBDR doit être installé et activé dans PHP.

## Utilisation de la classe `Database`

Afin d'éviter de répéter le code de connexion dans chaque script, le projet utilise une classe technique `Database`.

Cette classe fournit une méthode statique `getInstance()` permettant d'obtenir l'objet `PDO`.

La connexion devient alors très simple :

```php
$db = Database::getInstance();
```

Tout code nécessitant un accès à la base de données utilise cette méthode.

Exemple :

```php
$db = Database::getInstance();
$sql = "SELECT id, libelle, prix FROM Cours";
$cmd = $db->query($sql);
$lesLignes = $cmd->fetchAll(PDO::FETCH_ASSOC);
```

Le chargement automatique des classes permet d'utiliser directement `Database` sans avoir à inclure manuellement son fichier.

# 3. Méthodes applicables sur un objet PDO

## Constructeur

|Constructeur|Rôle|
|---|---|
|`PDO(string $dsn, string $username, string $password)`|Initialise une connexion à la base de données.|
## Principales méthodes

|Méthode|Retour|Rôle|
|---|---|---|
|`exec(string $sql)`|`int`|Exécute une requête SQL sans paramètre et retourne le nombre de lignes affectées.|
|`query(string $sql)`|`PDOStatement`|Exécute une requête de consultation sans paramètre.|
|`prepare(string $sql)`|`PDOStatement`|Prépare une requête SQL contenant des paramètres.|
|`lastInsertId()`|`string`|Retourne l'identifiant généré par la dernière requête `INSERT`.|
### `exec()`

`exec()` est principalement utilisé pour les requêtes de modification sans paramètre :

```php
$db->exec("UPDATE connexion SET etat = NOT etat");
```
### `query()`

`query()` permet d'exécuter directement une requête de consultation sans paramètre :

```php
$cmd = $db->query("SELECT id, nom, prenom FROM connexion");
```
### `prepare()`

`prepare()` permet de préparer une requête contenant des paramètres :

```php
$sql = <<<SQL
    SELECT id, nom, prenom
    FROM connexion
    WHERE id = :id;
SQL;

$cmd = $db->prepare($sql);
$cmd->execute(['id' => $id]);
```

Les paramètres peuvent également être associés avec `bindValue()` ou `bindParam()`.

# 4. Méthodes applicables sur un objet PDOStatement

|Méthode|Retour|Rôle|
|---|---|---|
|`bindValue()`|`bool`|Associe une valeur à un paramètre.|
|`bindParam()`|`bool`|Lie un paramètre à une variable.|
|`execute()`|`bool`|Exécute une requête préparée.|
|`fetch()`|`array\|false`|Récupère la ligne suivante.|
|`fetchAll()`|`array`|Récupère toutes les lignes.|
|`fetchColumn()`|`mixed`|Récupère une colonne d'une ligne.|
|`fetchObject()`|`object\|false`|Récupère une ligne sous forme d'objet.|
|`closeCursor()`|`bool`|Ferme le curseur.|

## `bindValue()`

```php
$cmd->bindValue('id', $id);
```

La valeur est copiée au moment de l'appel.

## `bindParam()`

```php
$cmd->bindParam('id', $id);
```

Le paramètre est lié à la variable. Sa valeur est évaluée lors de l'exécution de la requête.

Dans la plupart des cas, `bindValue()` est plus simple à utiliser.

## `execute()`

```php
$cmd->execute();
```

Ou directement avec les paramètres :

```php
$cmd->execute([
    'id' => $id
]);
```

## `fetch()`

Utiliser `fetch()` lorsque l'on souhaite récupérer une seule ligne ou parcourir les résultats ligne par ligne.

```php
$ligne = $cmd->fetch(PDO::FETCH_ASSOC);
```

## `fetchAll()`

Utiliser `fetchAll()` lorsque toutes les lignes doivent être récupérées dans un tableau :

```php
$lesLignes = $cmd->fetchAll(PDO::FETCH_ASSOC);
```

## `fetchColumn()`

Cette méthode est particulièrement adaptée lorsqu'une requête retourne une seule valeur :

```php
$id = $cmd->fetchColumn();
```
## Formats de récupération

`PDO::FETCH_ASSOC` peut être remplacé par d'autres modes de récupération.

|Mode|Résultat|Exemple|
|---|---|---|
|`PDO::FETCH_ASSOC`|Tableau associatif|`$ligne['nom']`|
|`PDO::FETCH_OBJ`|Objet|`$ligne->nom`|
|`PDO::FETCH_NUM`|Tableau numérique|`$ligne[1]`|

## Avantages des requêtes préparées

Les requêtes préparées présentent plusieurs avantages :

- les paramètres ne doivent pas être entourés de guillemets dans la requête SQL ;
- les apostrophes et caractères spéciaux contenus dans les valeurs sont correctement gérés ;
- les paramètres sont séparés de la requête SQL ;
- elles permettent d'éviter les injections SQL lorsqu'elles sont correctement utilisées ;
- les requêtes répétées peuvent bénéficier d'un meilleur fonctionnement côté SGBDR.

Une attaque classique telle que :

```text
' OR 1=1 #
```

ne permet pas de modifier la structure de la requête lorsqu'elle est transmise comme paramètre d'une requête préparée.
# 5. Centralisation des requêtes SQL dans les classes métiers

Les requêtes SQL doivent être centralisées dans les classes correspondant aux objets métier concernés.

Cette organisation permet de :

- séparer la logique métier de l'interface ;
- centraliser l'accès aux données ;
- éviter la duplication des requêtes ;
- faciliter la maintenance ;
- rendre le code plus lisible.

Une classe métier peut par exemple centraliser les opérations concernant les étudiants.

```php
class Etudiant
{
    public static function getAll(): array
    {
        $sql = <<<SQL
            SELECT
                nom,
                prenom,
                DATE_FORMAT(dateNaissance, '%d/%m/%Y') AS dateNaissanceFr,
                sexe,
                libelleCourt,
                photo
            FROM etudiant
            JOIN options ON etudiant.idOption = options.id
            ORDER BY nom, prenom
        SQL;

        $db = Database::getInstance();
        $cmd = $db->query($sql);
        return $cmd->fetchAll(PDO::FETCH_ASSOC);
    }
}
```

PHP ferme automatiquement les curseurs lorsque le script se termine.

Cependant, lorsqu'un script effectue plusieurs requêtes, il est recommandé de libérer explicitement les ressources :

```php
$cmd->closeCursor();
```
# 6. Exemples de requêtes de consultation

## Récupérer plusieurs enregistrements sans paramètre

```php
$sql = <<<SQL
    SELECT id, nom, prenom
    FROM connexion;
SQL;

$db = Database::getInstance();
$cmd = $db->query($sql);
$lesLignes = $cmd->fetchAll(PDO::FETCH_ASSOC);
```
## Récupérer un enregistrement sans paramètre

```php
$sql = <<<SQL
    SELECT id, nom, prenom
    FROM connexion
    WHERE id = 36
SQL;

$db = Database::getInstance();
$cmd = $db->query($sql);
$ligne = $cmd->fetch(PDO::FETCH_ASSOC);
```

## Récupérer plusieurs enregistrements avec une requête paramétrée

```php
$sql = <<<SQL
    SELECT nom, prenom
    FROM etudiant
    WHERE idClasse = :idClasse;
SQL;

$db = Database::getInstance();
$cmd = $db->prepare($sql);
$cmd->execute(['idClasse' => $idClasse]);
$lesLignes = $cmd->fetchAll(PDO::FETCH_ASSOC);
```

## Effectuer un traitement complémentaire côté serveur

Il peut être nécessaire de compléter les données avant de les transmettre au client.

Par exemple, si une table contient le nom d'un fichier image, il peut être intéressant d'ajouter une propriété indiquant si le fichier existe réellement.

```php
$sql = <<<SQL
    SELECT id, nom, prenom, photo
    FROM etudiant;
SQL;

$db = Database::getInstance();
$cmd = $db->query($sql);
$lesLignes = $cmd->fetchAll(PDO::FETCH_ASSOC);

$repertoire = DOSSIER_CLUB;

foreach ($lesLignes as &$ligne) {
    $ligne['present'] = isset($ligne['photo']) && file_exists($repertoire . '/' . $ligne['photo']);
}

unset($ligne);
return $lesLignes;
```

Remarque : il est recommandé d'éviter cette solution et de mettre en place  un classe Service.

## Récupérer un enregistrement avec plusieurs paramètres

```php
$sql = <<<SQL
    SELECT password, login
    FROM connexion
    WHERE nom = :nom
      AND prenom = :prenom
SQL;

$db = Database::getInstance();
$cmd = $db->prepare($sql);
$cmd->execute(['nom' => $nom, 'prenom' => $prenom]);
$ligne = $cmd->fetch(PDO::FETCH_ASSOC);
```
## Récupérer une seule valeur

```php
$sql = <<<SQL
    SELECT password
    FROM connexion
    WHERE login = :login
SQL;

$db = Database::getInstance();
$cmd = $db->prepare($sql);
$cmd->execute(['login' => $login]);

return $cmd->fetchColumn();
```
## Tester l'existence d'un enregistrement

```php
$sql = <<<SQL
    SELECT EXISTS (SELECT 1 FROM connexion  WHERE login = :login)
SQL;

$db = Database::getInstance();
$cmd = $db->prepare($sql);
$cmd->execute(['login' => $login]);

return (bool)$cmd->fetchColumn();
```

# 7. Utilisation de la classe `Select` pour les requêtes de consultation

Afin de réduire la quantité de code nécessaire aux requêtes de consultation, le projet dispose d'une classe technique `Select`.

Elle centralise l'exécution des requêtes de lecture.

Elle fournit notamment les méthodes suivantes :

| Méthode      | Retour         | Rôle                                                 |
| ------------ | -------------- | ---------------------------------------------------- |
| `getRow()`   | `array\|false` | Retourne une ligne sous forme de tableau associatif. |
| `getRows()`  | `array`        | Retourne toutes les lignes sous forme de tableau.    |
| `getValue()` | `mixed`        | Retourne une valeur unique.                          |

Les méthodes acceptent :

1. la requête SQL ;
2. éventuellement un tableau associatif contenant les paramètres.

Exemple de principe :

```php
public function getRows(string $sql, array $lesParametres = []): array
{
    // ...
}
```
## Plusieurs enregistrements sans paramètre

```php
$sql = <<<SQL
    SELECT id, nom, prenom
    FROM connexion;
SQL;

$select = new Select();
return $select->getRows($sql);
```
## Un enregistrement sans paramètre

```php
$sql = <<<SQL
    SELECT id, nom, prenom
    FROM connexion
    WHERE id = 36
SQL;

$select = new Select();
return $select->getRow($sql);
```
## Plusieurs enregistrements avec paramètres

```php
$sql = <<<SQL
    SELECT nom, prenom
    FROM etudiant
    WHERE idClasse = :idClasse
SQL;

$select = new Select();
return $select->getRows($sql, ['idClasse' => $idClasse]);
```
## Un enregistrement avec paramètres

```php
$sql = <<<SQL
    SELECT password, login
    FROM connexion
    WHERE nom = :nom
      AND prenom = :prenom
SQL;

$select = new Select();
return $select->getRow($sql, ['nom' => $nom, 'prenom' => $prenom]);
```
## Une valeur avec paramètres

```php
$sql = <<<SQL
    SELECT password
    FROM connexion
    WHERE login = :login
SQL;

$select = new Select();
return $select->getValue($sql, ['login' => $login]);
```

## Tester l'existence d'un enregistrement

```php
$sql = <<<SQL
    SELECT EXISTS (SELECT 1 FROM connexion  WHERE login = :login)
SQL;

$select = new Select(); 
return (bool)$select->getValue($sql, ['login' => $login]);
```
# 8. Exemples de requêtes de mise à jour

## Création d'un enregistrement

```php
$sql = <<<SQL
    INSERT INTO connexion (nom, prenom, password,email)
    VALUES (:nom, :prenom, :password,:email)
SQL;

$db = Database::getInstance();
$cmd = $db->prepare($sql);
$cmd->execute([
    'nom' => $nom,
    'prenom' => $prenom,
    'password' => $motPasse,
    'email' => $email
]);
```

Si l'identifiant est généré automatiquement :

```php
$id = $db->lastInsertId();
```

## Modification d'un enregistrement

```php
$sql = <<<SQL
    UPDATE connexion
    SET email = :email
    WHERE nom = :nom;
SQL;

$db = Database::getInstance();
$cmd = $db->prepare($sql);
$cmd->execute(['nom' => $nom, 'email' => $email]);
```

## Suppression d'un enregistrement

```php
$sql = <<<SQL
    DELETE FROM connexion
    WHERE id = :id;
SQL;

$db = Database::getInstance();
$cmd = $db->prepare($sql);
$cmd->execute(['id' => $id]);
```

## Requête de mise à jour sans paramètre

```php
$sql = <<SQL
    UPDATE connexion
    SET etat = NOT etat;
SQL;

$db = Database::getInstance();

$db->exec($sql);
```

# 9. Gestion des erreurs

De nombreux éléments du SGBDR peuvent provoquer une erreur lors d'une requête de modification :

- contrainte de clé primaire ;
- contrainte de clé étrangère ;
- contrainte d'unicité ;
- contrainte `NOT NULL` ;
- contrainte `CHECK` ;
- déclencheurs (`TRIGGER`).

Lorsque PDO est configuré avec :

```php
PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
```

une erreur provoque une `PDOException`.

Il est alors possible de traiter l'erreur avec `try/catch`.

```php
try {
    $cmd->execute();
} catch (Exception $e) {
    Erreur::envoyerReponse($e->getMessage());
} finally {
    $cmd->closeCursor();
}
```

Le message généré par le SGBDR est cependant souvent technique et peu adapté à l'utilisateur.
Le recours à un gestionnaire d'erreurs centralisé est conseillé, il évite d'encapsuler toutes les requêtes dans une instruction try/catch

# 10. Ajout ou modification avec des valeurs `NULL`

Une colonne SQL peut accepter la valeur `NULL`.

Il faut distinguer :

```php
null
```

de :

```php
''
```

Une chaîne vide est une valeur de type chaîne alors que `NULL` représente l'absence de valeur.

Par exemple, un formulaire peut transmettre une chaîne vide alors que la colonne SQL doit recevoir `NULL`.

Dans ce cas, le paramètre doit être transmis à PDO avec le type `PDO::PARAM_NULL`.

## Exemple

Modification du nom et du prénom, tous deux facultatifs :

```php
$db = Database::getInstance();

$email = $_SESSION['user']['email'];

$nom = $_POST['nom'] ?? null;
$prenom = $_POST['prenom'] ?? null;

$sql = "
    UPDATE compte
    SET nom = :nom,
        prenom = :prenom
    WHERE email = :email
";

$cmd = $db->prepare($sql);

$cmd->bindValue(':email', $email);

$cmd->bindValue(
    ':nom',
    $nom,
    $nom === null ? PDO::PARAM_NULL : PDO::PARAM_STR
);

$cmd->bindValue(
    ':prenom',
    $prenom,
    $prenom === null ? PDO::PARAM_NULL : PDO::PARAM_STR
);

try {
    $cmd->execute();
} catch (Exception $e) {
    Erreur::envoyerReponse($e->getMessage());
}
```

Remarque:  L'utilisation d'une classe Métier dérivant de la classe Table vous évitent d'avoir à gérer ce problème.

# 11. Mise en place d'une transaction

Une transaction permet de regrouper plusieurs opérations SQL.

L'objectif est que :
- toutes les opérations soient validées si elles réussissent ;
- aucune modification ne soit conservée si une erreur intervient.
    
Le principe est :

```text
Début de transaction
        │
        ▼
Exécution des requêtes
        │
        ├── Erreur ──► rollback
        │
        └── Succès ──► commit
```

Les principales méthodes PDO sont :

```php
$db->beginTransaction();

$db->rollBack();

$db->commit();
```

## Exemple

Ajout d'un projet et de ses compétences :

```php
public static function ajouterProjet(string $nom, array $lesCompetences): void {

    $db = Database::getInstance();

    $db->beginTransaction();

    try {
        // Ajout du projet
        $sql = "INSERT INTO projet(nom) VALUES (:nom)";
        $cmd = $db->prepare($sql);
        $cmd->execute(['nom' => $nom]);
        $idProjet = $db->lastInsertId();

        // Ajout des compétences
        $sql = "INSERT INTO competenceprojet(idProjet,idCompetence) VALUES (:idProjet, :idCompetence)";
        $cmd = $db->prepare($sql);

        foreach ($lesCompetences as $ligne) {
            $cmd->execute(['idProjet' => $idProjet, 'idCompetence' => $ligne['idCompetence']]);
        }

        $db->commit();
        
    } catch (Exception $e) {
        $db->rollBack();
        Erreur::envoyerReponse($e->getMessage());
    }
}
```

Cette organisation est préférable à la multiplication de blocs `try/catch` autour de chaque requête.

# 12. Appel d'une procédure ou d'une fonction stockée

Une procédure ou une fonction stockée contient un ensemble d'instructions SQL enregistrées directement dans le SGBDR.

Elles sont exécutées sur demande par le SGBDR.
## Avantages

- logique pouvant être exécutée directement sur le serveur de base de données ;
- centralisation de certaines opérations ;
- réduction du code SQL côté application ;
- possibilité de masquer une partie de la structure interne de la base.
## Inconvénients

- dépendance plus forte au SGBDR ;
- logique métier répartie entre PHP et la base de données ;
- charge supplémentaire sur le serveur de base de données ;
- maintenance parfois plus complexe.
## Procédure retournant un jeu d'enregistrements

### Création de la procédure

```sql
CREATE PROCEDURE getLesComptes()
BEGIN
    SELECT email, nom, prenom, login, password
    FROM compte
    ORDER BY nom, prenom;
END
```
### Appel depuis PHP

```php
$db = Database::getInstance();

$cmd = $db->query("CALL getLesComptes()");

$lesLignes = $cmd->fetchAll(PDO::FETCH_ASSOC);

$cmd->closeCursor();
```

## Procédure assurant la création d'un enregistrement

### Création de la procédure

```sql
CREATE PROCEDURE ajouterCompte(
    login VARCHAR(20),
    password VARCHAR(20),
    email VARCHAR(100),
    nom VARCHAR(25),
    prenom VARCHAR(25)
)
BEGIN
    INSERT INTO compte (email, nom, prenom, login, password)
    VALUES (email, nom, prenom, login, password);
END
```
### Appel depuis PHP

```php
$cmd = $db->prepare("CALL ajouterCompte(:login,:password,:email,:nom,:prenom)");

$cmd->execute(['email' => $email,'nom' => $nom, 'prenom' => $prenom, 'login' => $login,'password' => $password]);
```

## Fonction retournant une valeur

Exemple de fonction permettant d'activer un compte :

```sql
CREATE FUNCTION activerCompte(
    code VARCHAR(64)
)
RETURNS VARCHAR(150)
DETERMINISTIC
BEGIN
    SET @id = 0;

    SELECT id, actif
    INTO @id, @actif
    FROM compte
    WHERE SHA1(email) = code;

    IF @id = 0 THEN
        SET @reponse = 'Paramètre invalide.';
    ELSEIF @actif = 1 THEN
        SET @reponse = 'Ce compte a déjà été activé.';
    ELSE
        UPDATE compte
        SET actif = 1
        WHERE id = @id;
        SET @reponse =
            'Votre compte est maintenant actif, vous pouvez vous connecter';
    END IF;
    RETURN @reponse;
END
```

### Appel depuis PHP

```php
$db = Database::getInstance();

$cmd = $db->prepare("SELECT activerCompte(:id)");

$cmd->execute(['id' => $id]);

$resultat = $cmd->fetchColumn();
```

## Paramètres typés

Lorsqu'un paramètre doit être transmis avec un type PDO explicite :

```php
$cmd->bindValue('id', $id, PDO::PARAM_INT);
$cmd->bindValue('nom',$nom, PDO::PARAM_STR);
```

Pour une chaîne, la longueur peut également être indiquée avec `bindParam()` lorsque cela est nécessaire.

## Paramètre `OUT`

Une procédure peut théoriquement posséder un paramètre de sortie :

```sql
CREATE PROCEDURE ajouterCompte(
    login VARCHAR(20),
    password VARCHAR(20),
    email VARCHAR(100),
    nom VARCHAR(25),
    prenom VARCHAR(25),
    OUT id INT
)
BEGIN
    INSERT INTO compte (email, nom, prenom, login, password )
    VALUES (email, nom, prenom, login, password);
    SET id = (SELECT @@IDENTITY);
END
```

Cependant, l'utilisation directe d'un paramètre `OUT` avec PDO et MySQL peut poser des problèmes.
Une solution de contournement consiste à faire retourner l'identifiant par la procédure elle-même.
### Procédure

```sql
CREATE PROCEDURE ajouterCompte(
    login VARCHAR(20),
    password VARCHAR(20),
    email VARCHAR(100),
    nom VARCHAR(25),
    prenom VARCHAR(25)
)
BEGIN
    INSERT INTO compte (email, nom, prenom, login, password)
    VALUES (email, nom, prenom, login, password);

    SELECT LAST_INSERT_ID();
END
```
### PHP

```php
$cmd = $db->prepare("CALL ajouterCompte(:login, :password, :email, :nom,:prenom)");

$cmd->execute(['email' => $email,'nom' => $nom,'prenom' => $prenom,'login' => $login,'password' => $password]);

$id = $cmd->fetchColumn();
```

# 13. Classe `Database` et configuration de PDO

La classe `Database` permet d'éviter de créer plusieurs connexions PDO dans l'application.

Elle fournit une méthode :

```php
Database::getInstance()
```

Cette méthode doit :

1. vérifier si une instance PDO existe déjà ;
2. retourner cette instance si elle existe ;
3. créer l'instance sinon ;
4. appliquer la configuration PDO ;
5. conserver l'instance créée.

Cette organisation permet d'utiliser une seule connexion PDO dans le contexte de l'application.

## Configuration de PDO

La création d'un objet PDO nécessite notamment :

- le serveur ;
- la base de données ;
- l'utilisateur ;
- le mot de passe ;
- le port ;
- le jeu de caractères.

D'autres paramètres sont particulièrement importants.

### `PDO::ATTR_ERRMODE`

Définit le comportement de PDO en cas d'erreur.

Les principales valeurs sont :

| Valeur                   | Comportement                                      |
| ------------------------ | ------------------------------------------------- |
| `PDO::ERRMODE_SILENT`    | Les erreurs doivent être contrôlées manuellement. |
| `PDO::ERRMODE_WARNING`   | PDO génère un avertissement PHP.                  |
| `PDO::ERRMODE_EXCEPTION` | PDO lance une `PDOException`.                     |

La valeur recommandée est :

```php
PDO::ERRMODE_EXCEPTION
```

## `PDO::ATTR_DEFAULT_FETCH_MODE`

Définit le mode de récupération par défaut.

Par exemple :

```php
PDO::FETCH_ASSOC
```

permet d'obtenir directement des tableaux associatifs avec `fetch()` et `fetchAll()`.

Cela évite d'avoir à préciser systématiquement :

```php
$cmd->fetch(PDO::FETCH_ASSOC);
```

## `PDO::ATTR_EMULATE_PREPARES`

Ce paramètre contrôle l'utilisation des requêtes préparées émulées.

Dans le projet, il est recommandé de désactiver l'émulation :

```php
PDO::ATTR_EMULATE_PREPARES => false
```

## Exemple de configuration

Les paramètres peuvent être regroupés dans un fichier de configuration :

```php
return [
    'host' => 'vds2025',
    'database' => 'vds2025',
    'user' => 'root',
    'password' => '',
    'port' => 3306,
    'charset' => 'utf8mb4',

    'errmode' => PDO::ERRMODE_EXCEPTION,
    'fetchmode' => PDO::FETCH_ASSOC,

    'emulate_prepares' => false
];
```

La classe `Database` peut ensuite utiliser cette configuration lors de la création de PDO.

## Validation des données par le SGBDR

Il est possible d'ajouter des règles de validation directement dans la base de données afin d'éviter les conversions implicites du SGBDR.

Par exemple, selon la configuration du SGBDR, une donnée invalide peut parfois être convertie automatiquement en une valeur par défaut.

Il peut être préférable d'utiliser :

- des contraintes SQL ;
- des contraintes `CHECK` ;
- des déclencheurs `BEFORE` lorsque des messages métier spécifiques doivent être générés.

# 15. Cas particulier : appel Ajax

Pour une requête Ajax, la réponse doit être retournée au format **JSON**.

L'intérêt de cette organisation est de permettre au module JavaScript `ajax.js` de centraliser le traitement des erreurs.

En cas de succès, la réponse JSON est automatiquement convertie en objet JavaScript.

Il est également possible de fournir une fonction de rappel spécifique en cas d'erreur.

## Exemple côté JavaScript

```javascript
import { appelAjax } from "/composant/fonction/ajax.js";

function chargerLesCompetencesDuProjet(idProjet) {
    msg.innerHTML = "";
    appelAjax({
        url: 'ajax/getlescompetences.php',
        data: {
            idProjet: idProjet
        },
        success: data => {
            lesCompetences = data;
            afficherLesCompetences(lesCompetences);
        }
    });
}
```

## Exemple côté PHP

Fichier :

```text
ajax/getlescompetences.php
```

Le paramètre peut être récupéré et validé avant d'effectuer la requête :

```php
<?php

use ClasseMetier\Projet;
use ClasseMetier\CompetenceProjet;
use ClasseTechnique\Requete;
use ClasseTechnique\ReponseJson;
use ClasseTechnique\UserException;

/** @noinspection PhpIncludeInspection */
require $_SERVER['DOCUMENT_ROOT'] . '/../bootstrap/bootstrap.php';


// Récupération contrôlée du paramètre attendue idPRojet
$idProjet = Requete::postInt('idProjet');

// Si le projet n'existe pas, on envoie une erreur
if (!Projet::existe($idProjet)) {
    throw new UserException("Ce projet n'existe pas.");
}

// récupération des compétences du projet et envoi de la réponse au format json
ReponseJson::envoyerLesDonnees(CompetenceProjet::getLesCompetences($idProjet));
```

## Principe général

L'architecture peut être résumée ainsi :

```text
Navigateur
    │
    │ requête HTTP / Ajax
    ▼
PHP
    │
    ├── Validation des paramètres
    │
    ├── Classe métier
    │
    ├── Select / Database
    │
    ▼
SGBDR
    │
    ▼
Résultat
    │
    ├── Page HTML
    │
    └── Réponse JSON
            │
            ▼
       JavaScript
```

Cette organisation permet de séparer clairement :

- **l'interface utilisateur** ;
- **la gestion des requêtes HTTP** ;
- **la logique métier** ;
- **l'accès aux données** ;
- **la gestion de la base de données** ;
- **la réponse envoyée au navigateur**.

L'utilisation de `Database`, `Select`, des classes métiers et des requêtes préparées permet ainsi de conserver un accès aux données centralisé, sécurisé et maintenable.