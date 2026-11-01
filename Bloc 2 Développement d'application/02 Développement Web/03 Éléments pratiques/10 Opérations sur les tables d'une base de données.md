#  Connexion à la base de données Mysql

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

Les paramètres de connexion sont stockés dans le fichier /config/database.php

```php
return [  
    'host' => 'localhost',  
    'database' => 'vds',  
    'user' => 'vds',  
    'password' => 'rf.7I8iDjbE`g]pDCV]J',  
    'port' => 3306  
];

```
#  Centralisation des requêtes SQL dans les classes métiers

Les requêtes SQL doivent être centralisées dans les classes correspondant aux objets métier concernés.

Cette organisation permet de :

- séparer la logique métier de l'interface ;
- centraliser l'accès aux données ;
- éviter la duplication des requêtes ;
- faciliter la maintenance ;
- rendre le code plus lisible.

# Utilisation de la classe `Select` pour les requêtes de consultation

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
# Exemples de requêtes de mise à jour

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

Remarque:  L'utilisation d'une classe Métier dérivant de la classe Table vous évitent d'avoir à réaliser des requêtes de mise à jour.

#  Mise en place d'une transaction

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


#  Appel d'une procédure ou d'une fonction stockée


```php
$db = Database::getInstance();
$cmd = $db->query("CALL getLesComptes()");
$lesLignes = $cmd->fetchAll(PDO::FETCH_ASSOC);
$cmd->closeCursor();
```

Si la procédure stockée possèdes des paramètres

```php
$cmd = $db->prepare("CALL ajouterCompte(:login,:password,:email,:nom,:prenom)");
$cmd->execute(['email' => $email,'nom' => $nom, 'prenom' => $prenom, 'login' => $login,'password' => $password]);
```

Appel d'une fonction stockée


```php
$db = Database::getInstance();
$cmd = $db->prepare("SELECT activerCompte(:id)");
$cmd->execute(['id' => $id]);
$resultat = $cmd->fetchColumn();
```

Lorsqu'un paramètre doit être transmis avec un type PDO explicite :

```php
$cmd->bindValue('id', $id, PDO::PARAM_INT);
$cmd->bindValue('nom',$nom, PDO::PARAM_STR);
```

Pour une chaîne, la longueur peut également être indiquée avec `bindParam()` lorsque cela est nécessaire.
   

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