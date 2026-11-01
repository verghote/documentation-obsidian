## Introduction

Les **procédures stockées** (*Stored Procedures*) sont des programmes enregistrés directement dans le serveur MySQL.

Elles permettent de regrouper plusieurs instructions SQL sous un même nom afin de pouvoir les réutiliser facilement depuis une application ou directement depuis MySQL.

Les procédures stockées constituent une véritable couche d'abstraction entre la base de données et les applications clientes.

Le développeur n'a plus besoin de connaître :

- la structure des tables ;
- les relations entre les tables ;
- la syntaxe détaillée des requêtes SQL.

Il lui suffit d'appeler une procédure.

---

# 1. Pourquoi utiliser des procédures stockées ?

Une procédure stockée permet de :

- centraliser les traitements SQL ;
- éviter la duplication du code ;
- améliorer la sécurité ;
- simplifier le développement des applications ;
- encapsuler la logique métier dans la base de données.

Par exemple, au lieu d'écrire cette requête dans toutes les applications :

```sql
SELECT nom, prenom
FROM client
WHERE id = 12;
```

on pourra simplement appeler :

```sql
CALL getClient(12);
```

---

# 2. Création d'une procédure

La création d'une procédure s'effectue avec l'instruction `CREATE PROCEDURE`.

## Syntaxe générale

```sql
CREATE PROCEDURE nomProcedure(paramètres)

BEGIN

    instructions SQL

END;
```

Si la procédure ne contient qu'une seule instruction, les mots-clés `BEGIN` et `END` sont facultatifs.

Exemple :

```sql
CREATE PROCEDURE getClients()

SELECT *
FROM client;
```

---

# 3. Le délimiteur (DELIMITER)

Par défaut, MySQL considère que le caractère `;` termine une instruction SQL.

Or, une procédure contient souvent plusieurs instructions terminées par un point-virgule.

Il faut donc temporairement changer le délimiteur.

## Exemple

```sql
DELIMITER $$

CREATE PROCEDURE getClients()

BEGIN

    SELECT *
    FROM client;

END $$

DELIMITER ;
```

> **Remarque :** certains IDE, comme **JetBrains DataGrip**, gèrent automatiquement les délimiteurs et il n'est pas nécessaire de les modifier.

---

# 4. Les paramètres

Une procédure peut recevoir des paramètres.

Chaque paramètre possède :

- un sens ;
- un nom ;
- un type.

## Syntaxe

```sql
sens nom type
```

Exemple :

```sql
IN idClient INT
```

---

## Les différents sens

| Sens | Description |
|------|-------------|
| IN | Paramètre d'entrée (par défaut) |
| OUT | Paramètre de sortie |
| INOUT | Paramètre d'entrée et de sortie |

---

### Paramètre IN

Le paramètre est transmis à la procédure.

```sql
CREATE PROCEDURE getNom(

    IN idClient INT

)
```

---

### Paramètre OUT

La procédure renvoie une valeur.

```sql
OUT nomClient VARCHAR(30)
```

---

### Paramètre INOUT

Le paramètre est utilisé en entrée puis modifié avant d'être renvoyé.

```sql
INOUT compteur INT
```

---

# 5. SQL SECURITY

Une procédure peut être exécutée avec les droits :

- de son créateur ;
- de l'utilisateur qui l'appelle.

La syntaxe est :

```sql
SQL SECURITY
{
    DEFINER
    | INVOKER
}
```

---

## SQL SECURITY DEFINER

C'est le comportement par défaut.

La procédure est exécutée avec les droits du créateur.

```sql
CREATE PROCEDURE ...

SQL SECURITY DEFINER
```

Cela permet à un utilisateur d'exécuter une procédure même s'il ne possède pas les droits sur les tables concernées.

---

## SQL SECURITY INVOKER

```sql
CREATE PROCEDURE ...

SQL SECURITY INVOKER
```

La procédure est exécutée avec les droits de l'utilisateur courant.

Si celui-ci ne possède pas les privilèges nécessaires, l'exécution échouera.

---

# 6. Exécuter une procédure

Une procédure est exécutée grâce à l'instruction `CALL`.

## Syntaxe

```sql
CALL nomProcedure(paramètres);
```

Exemple :

```sql
CALL getClients();
```

---

# 7. Supprimer une procédure

```sql
DROP PROCEDURE nomProcedure;
```

Pour éviter une erreur si elle n'existe pas :

```sql
DROP PROCEDURE IF EXISTS nomProcedure;
```

---

# 8. Lister les procédures

Afficher toutes les procédures d'une base :

```sql
SHOW PROCEDURE STATUS
WHERE Db = 'nomBase';
```

---

# 9. Afficher le code d'une procédure

```sql
SHOW CREATE PROCEDURE nomProcedure;
```

Cette commande affiche :

- le code SQL ;
- le créateur ;
- les options de sécurité ;
- le jeu de caractères utilisé.

---

# 10. Premier exemple

Créons une procédure qui retourne la liste des clients.

```sql
CREATE PROCEDURE getLesClients()

SELECT

    nom,

    CONCAT(
        rue,
        ' ',
        codePostal,
        ' ',
        ville
    ) AS adresse

FROM client;
```

Exécution :

```sql
CALL getLesClients();
```

Résultat :

| nom | adresse |
|------|----------|
| Martin | 5 rue Victor Hugo 75001 Paris |
| Durand | 10 avenue de Lyon 69000 Lyon |

---

# 11. Procédure avec une variable locale

Une procédure peut utiliser des variables locales.

```sql
CREATE PROCEDURE getNom(

    IN idClient INT

)

BEGIN

    DECLARE nomClient VARCHAR(30);

    SELECT nom

    INTO nomClient

    FROM client

    WHERE id = idClient;

    SELECT nomClient;

END;
```

---

# 12. Vérifier l'existence d'une ligne

Il est recommandé de vérifier qu'un enregistrement existe avant de le lire.

```sql
IF EXISTS(

    SELECT 1

    FROM client

    WHERE id = idClient

)

THEN

    ...

END IF;
```

L'instruction `EXISTS` est très rapide car MySQL s'arrête dès qu'une ligne est trouvée.

---

# 13. Exemple complet

```sql
CREATE PROCEDURE getNom(

    IN idClient INT

)

BEGIN

    DECLARE nomClient VARCHAR(30);

    IF EXISTS(

        SELECT 1

        FROM client

        WHERE id = idClient

    )

    THEN

        SELECT nom

        INTO nomClient

        FROM client

        WHERE id = idClient;

    ELSE

        SET nomClient = 'Inexistant';

    END IF;

    SELECT nomClient;

END;
```

Exécution :

```sql
CALL getNom(1);
```

Résultat :

```
Martin
```

ou

```
Inexistant
```

---

# 14. Bonnes pratiques

Il est conseillé de :

- utiliser des noms explicites ;
- documenter les paramètres ;
- vérifier les données avant toute modification ;
- centraliser la logique métier ;
- limiter les droits grâce à `SQL SECURITY`;
- privilégier les procédures pour les traitements complexes.

---

# Tableau récapitulatif

| Instruction           | Description                                |
| --------------------- | ------------------------------------------ |
| CREATE PROCEDURE      | Crée une procédure                         |
| CALL                  | Exécute une procédure                      |
| DROP PROCEDURE        | Supprime une procédure                     |
| SHOW PROCEDURE STATUS | Liste les procédures                       |
| SHOW CREATE PROCEDURE | Affiche le code SQL                        |
| DECLARE               | Déclare une variable locale                |
| IN                    | Paramètre d'entrée                         |
| OUT                   | Paramètre de sortie                        |
| INOUT                 | Paramètre entrée/sortie                    |
| SQL SECURITY DEFINER  | Exécution avec les droits du créateur      |
| SQL SECURITY INVOKER  | Exécution avec les droits de l'utilisateur |
# Les procédures stockées MySQL - Partie 2
## Paramètres, exemples et utilisation

---

# 15. Les paramètres de sortie (OUT)

Contrairement à une requête SQL qui retourne directement un résultat, une procédure peut transmettre une ou plusieurs valeurs grâce aux paramètres **OUT**.

Un paramètre `OUT` est une variable qui sera renseignée par la procédure.

## Syntaxe

```sql
OUT nomParametre type
```

Exemple :

```sql
OUT nomClient VARCHAR(30)
```

---

# 16. Exemple : retourner le nom d'un client

```sql
DELIMITER $$

CREATE PROCEDURE getNom(

    IN idClient INT,
    OUT nomClient VARCHAR(30)

)

BEGIN

    IF EXISTS(

        SELECT 1
        FROM client
        WHERE id = idClient

    ) THEN

        SELECT nom

        INTO nomClient

        FROM client

        WHERE id = idClient;

    ELSE

        SET nomClient = 'Inexistant';

    END IF;

END $$

DELIMITER ;
```

---

## Exécution

On utilise une variable utilisateur pour récupérer le résultat.

```sql
CALL getNom(1, @nom);
```

Puis :

```sql
SELECT @nom;
```

Résultat :

```
Martin
```

---

# 17. Les paramètres INOUT

Un paramètre `INOUT` est à la fois :

- une donnée fournie à la procédure ;
- une donnée modifiée puis renvoyée.

## Exemple

```sql
DELIMITER $$

CREATE PROCEDURE incrementer(

    INOUT compteur INT

)

BEGIN

    SET compteur = compteur + 1;

END $$

DELIMITER ;
```

Exécution :

```sql
SET @c = 5;

CALL incrementer(@c);

SELECT @c;
```

Résultat :

```
6
```

---

# 18. Procédure réalisant une mise à jour

Une procédure est souvent utilisée pour encapsuler une modification de la base.

Exemple : mise à jour du solde d'un compte bancaire.

```sql
DELIMITER $$

CREATE PROCEDURE majSolde(

    IN idCompte INT,
    IN sens CHAR(1),
    IN montant DECIMAL(8,2)

)

BEGIN

    IF sens = 'c' THEN

        UPDATE compte

        SET solde = solde + montant

        WHERE id = idCompte;

    ELSE

        UPDATE compte

        SET solde = solde - montant

        WHERE id = idCompte;

    END IF;

END $$

DELIMITER ;
```

---

## Utilisation

```sql
CALL majSolde(

    1,
    'c',
    2000

);
```

Le compte numéro 1 est crédité de 2 000 €.

---

Pour débiter le compte :

```sql
CALL majSolde(

    1,
    'd',
    500

);
```

Le solde est diminué de 500 €.

---

# 19. Utiliser plusieurs paramètres

Une procédure peut recevoir autant de paramètres que nécessaire.

Exemple :

```sql
CREATE PROCEDURE ajouterClient(

    IN nom VARCHAR(40),
    IN prenom VARCHAR(40),
    IN ville VARCHAR(40)

)
```

L'appel devient :

```sql
CALL ajouterClient(

    'Martin',

    'Jean',

    'Paris'

);
```

---

# 20. Valeurs retournées par une procédure

Une procédure peut retourner :

- un jeu de résultats ;
- une valeur via un paramètre `OUT` ;
- plusieurs paramètres `OUT` ;
- plusieurs jeux de résultats.

Exemple :

```sql
SELECT *

FROM client;
```

est parfaitement valide dans une procédure.

---

# 21. Plusieurs résultats

Une procédure peut contenir plusieurs requêtes `SELECT`.

```sql
CREATE PROCEDURE informations()

BEGIN

    SELECT *

    FROM client;

    SELECT *

    FROM commande;

END;
```

Le client recevra deux jeux de résultats.

---

# 22. Utilisation des variables locales

Les variables locales sont déclarées avec `DECLARE`.

```sql
DECLARE nbClients INT;
```

Elles peuvent recevoir le résultat d'une requête.

```sql
SELECT COUNT(*)

INTO nbClients

FROM client;
```

Puis être utilisées :

```sql
IF nbClients = 0 THEN

    SELECT 'Aucun client';

END IF;
```

---

# 23. Procédure avec calcul

```sql
DELIMITER $$

CREATE PROCEDURE calculTVA(

    IN prixHT DECIMAL(8,2)

)

BEGIN

    DECLARE prixTTC DECIMAL(8,2);

    SET prixTTC = prixHT * 1.20;

    SELECT

        prixHT,

        prixTTC;

END $$

DELIMITER ;
```

Exécution :

```sql
CALL calculTVA(100);
```

Résultat :

| prixHT | prixTTC |
|---------|----------|
|100.00|120.00|

---

# 24. Appeler une procédure depuis une autre procédure

Une procédure peut appeler une autre procédure.

```sql
CALL majSolde(

    1,

    'c',

    500

);
```

Cela permet de découper les traitements en plusieurs modules.

---

# 25. Procédures et transactions

Les procédures sont souvent utilisées avec les transactions.

```sql
START TRANSACTION;

CALL majSolde(

    1,

    'd',

    500

);

CALL majSolde(

    2,

    'c',

    500

);

COMMIT;
```

En cas d'erreur :

```sql
ROLLBACK;
```

---

# 26. Afficher des informations

Une procédure peut simplement afficher un message.

```sql
SELECT 'Traitement terminé';
```

ou

```sql
SELECT

    NOW(),

    CURRENT_USER();
```

---

# 27. Les procédures comme couche d'abstraction

Une bonne pratique consiste à interdire l'accès direct aux tables.

L'application appelle uniquement des procédures.

```
Application

        │

        ▼

Procédures stockées

        │

        ▼

Tables
```

Cette architecture présente plusieurs avantages :

- meilleure sécurité ;
- indépendance vis-à-vis de la structure des tables ;
- maintenance simplifiée.

---

# 28. Bonnes pratiques

Il est conseillé de :

- utiliser des noms explicites (`ajouterClient`, `majSolde`, etc.) ;
- vérifier les paramètres avant leur utilisation ;
- limiter le nombre de paramètres ;
- privilégier les paramètres `OUT` plutôt que les `SELECT` lorsque l'on retourne une seule valeur ;
- documenter chaque procédure ;
- utiliser des transactions pour les mises à jour multiples.

---

# Tableau récapitulatif

| Élément           | Description                |
| ----------------- | -------------------------- |
| IN                | Paramètre d'entrée         |
| OUT               | Paramètre de sortie        |
| INOUT             | Paramètre entrée/sortie    |
| CALL              | Appel d'une procédure      |
| SELECT ... INTO   | Affectation d'une variable |
| DECLARE           | Variable locale            |
| START TRANSACTION | Début d'une transaction    |
| COMMIT            | Validation                 |
| ROLLBACK          | Annulation                 |

# Les procédures stockées MySQL - Partie 3
# Les fonctions stockées

## Introduction

Une **fonction stockée** (*Stored Function*) est un programme enregistré dans le serveur MySQL qui **retourne obligatoirement une valeur**.

Elle est comparable à une fonction d'un langage de programmation comme C#, Java ou PHP.

Contrairement à une procédure stockée, une fonction peut être utilisée directement dans une requête SQL.

Exemples :

```sql
SELECT getNom(1);
```

```sql
SELECT prixHT, calculTVA(prixHT)
FROM produit;
```

---

# 29. Procédure ou fonction ?

Les procédures et les fonctions sont très proches mais n'ont pas le même objectif.

| Procédure | Fonction |
|-----------|----------|
| Peut retourner plusieurs résultats | Retourne une seule valeur |
| Utilise CALL | Utilisée dans un SELECT |
| Peut modifier les données | Ne doit pas modifier l'état de la base |
| Paramètres IN, OUT et INOUT | Paramètres IN uniquement |
| Pas de RETURN obligatoire | RETURN obligatoire |

En règle générale :

- **une procédure réalise une action** ;
- **une fonction calcule une valeur**.

---

# 30. Création d'une fonction

La création d'une fonction s'effectue avec l'instruction `CREATE FUNCTION`.

## Syntaxe

```sql
CREATE FUNCTION nomFonction(

    paramètre type,
    ...

)

RETURNS type

BEGIN

    instructions

    RETURN valeur;

END;
```

La clause `RETURNS` indique le type de la valeur retournée.

---

# 31. Premier exemple

Créons une fonction qui double un nombre.

```sql
DELIMITER $$

CREATE FUNCTION doubleValeur(

    valeur INT

)

RETURNS INT

BEGIN

    RETURN valeur * 2;

END $$

DELIMITER ;
```

Utilisation :

```sql
SELECT doubleValeur(15);
```

Résultat

```
30
```

---

# 32. Fonction retournant le nom d'un client

```sql
DELIMITER $$

CREATE FUNCTION getNom(

    idClient INT

)

RETURNS VARCHAR(30)

BEGIN

    DECLARE nomClient VARCHAR(30)
    DEFAULT 'Inexistant';

    IF EXISTS(

        SELECT 1

        FROM client

        WHERE id = idClient

    )

    THEN

        SELECT nom

        INTO nomClient

        FROM client

        WHERE id = idClient;

    END IF;

    RETURN nomClient;

END $$

DELIMITER ;
```

Utilisation

```sql
SELECT getNom(1);
```

Résultat

```
Martin
```

---

# 33. Utiliser une fonction dans une requête

L'un des principaux intérêts des fonctions est leur intégration dans les requêtes SQL.

```sql
SELECT

    id,

    getNom(id)

FROM client;
```

Ou encore :

```sql
SELECT

    nom,

    calculTVA(prix)

FROM produit;
```

Chaque ligne appelle automatiquement la fonction.

---

# 34. Paramètres d'une fonction

Contrairement aux procédures, une fonction ne possède que des paramètres d'entrée.

Les mots-clés

```
IN

OUT

INOUT
```

ne sont pas autorisés.

Exemple :

```sql
CREATE FUNCTION calculTVA(

    prix DECIMAL(8,2)

)
```

---

# 35. L'instruction RETURN

Toute fonction doit obligatoirement retourner une valeur.

```sql
RETURN prix * 1.20;
```

Sans instruction `RETURN`, la fonction est invalide.

---

# 36. Le mot-clé DETERMINISTIC

Une fonction peut être déclarée **DETERMINISTIC**.

```sql
CREATE FUNCTION calculTVA(...)

RETURNS DECIMAL(8,2)

DETERMINISTIC
```

Cela signifie que :

> Pour les mêmes paramètres, la fonction retournera toujours le même résultat.

Exemple :

```
calculTVA(100)

↓

120
```

La fonction retournera toujours 120.

---

## Pourquoi utiliser DETERMINISTIC ?

Cela permet à MySQL d'optimiser certaines exécutions.

Le serveur peut réutiliser un résultat déjà calculé au lieu de relancer la fonction.

---

## Exemples de fonctions déterministes

```sql
doubleValeur()
```

```sql
calculTVA()
```

```sql
prixTTC()
```

---

## Fonctions non déterministes

Certaines fonctions retournent une valeur différente à chaque appel.

Exemples :

```sql
NOW()
```

```sql
CURRENT_TIMESTAMP()
```

```sql
RAND()
```

Ces fonctions ne sont donc pas déterministes.

---

# 37. Suppression d'une fonction

```sql
DROP FUNCTION getNom;
```

Pour éviter une erreur :

```sql
DROP FUNCTION IF EXISTS getNom;
```

---

# 38. Afficher les fonctions

Afficher toutes les fonctions d'une base :

```sql
SHOW FUNCTION STATUS

WHERE Db = 'nomBase';
```

---

# 39. Voir le code d'une fonction

```sql
SHOW CREATE FUNCTION getNom;
```

---

# 40. Exemple : calcul de l'âge

```sql
DELIMITER $$

CREATE FUNCTION age(

    naissance DATE

)

RETURNS INT

DETERMINISTIC

BEGIN

    RETURN TIMESTAMPDIFF(

        YEAR,

        naissance,

        CURDATE()

    );

END $$

DELIMITER ;
```

Utilisation :

```sql
SELECT

    nom,

    age(dateNaissance)

FROM client;
```

---

# 41. Exemple : catégorie de salaire

```sql
CREATE FUNCTION categorieSalaire(

    salaire DECIMAL(8,2)

)

RETURNS VARCHAR(20)

DETERMINISTIC

BEGIN

    IF salaire > 50000 THEN

        RETURN 'Élevé';

    END IF;

    RETURN 'Standard';

END;
```

Utilisation

```sql
SELECT

    nom,

    categorieSalaire(salaire)

FROM employe;
```

---

# 42. Les limitations des fonctions

Une fonction ne doit pas modifier la base de données.

Les instructions suivantes sont à éviter :

```sql
INSERT
```

```sql
UPDATE
```

```sql
DELETE
```

```sql
ALTER TABLE
```

```sql
DROP TABLE
```

Les fonctions sont destinées au calcul de valeurs.

Les traitements modifiant les données doivent être réalisés dans des procédures stockées.

---

# 43. Bonnes pratiques

Une fonction doit :

- effectuer un calcul ;
- être courte ;
- retourner une seule valeur ;
- être déterministe lorsque c'est possible ;
- ne pas modifier les données.

---

# Tableau récapitulatif

| Élément | Description |
|----------|-------------|
| CREATE FUNCTION | Créer une fonction |
| RETURNS | Type de retour |
| RETURN | Valeur retournée |
| DETERMINISTIC | Résultat toujours identique |
| DROP FUNCTION | Supprimer une fonction |
| SHOW FUNCTION STATUS | Lister les fonctions |
| SHOW CREATE FUNCTION | Afficher le code |
| SELECT fonction() | Exécuter une fonction |

---

# Procédure ou fonction ?

Utiliser une **procédure** lorsque :

- on modifie des données ;
- on retourne plusieurs informations ;
- on réalise un traitement métier.

Utiliser une **fonction** lorsque :

- on calcule une valeur ;
- on souhaite utiliser le résultat dans un SELECT ;
- on retourne une seule information.

# Les procédures stockées MySQL - Partie 4
# Utilisation depuis une application (C# et PHP)

## Introduction

Les procédures et fonctions stockées prennent tout leur intérêt lorsqu'elles sont appelées depuis une application.

Le principe est simple :

```
Application

        │

        ▼

 Procédure / Fonction

        │

        ▼

   Base de données
```

L'application ne manipule plus directement les tables mais appelle des procédures ou des fonctions.

Cette architecture présente plusieurs avantages :

- meilleure sécurité ;
- code SQL centralisé ;
- maintenance simplifiée ;
- évolution plus facile de la base de données.

---

# 44. Appeler une procédure depuis C#

Les exemples suivants utilisent :

- **MySqlConnector**
- **ADO.NET**

Une connexion est d'abord ouverte :

```csharp
using var cnx = new MySqlConnection(chaineConnexion);

cnx.Open();
```

---

# 45. Procédure retournant un jeu de résultats

Supposons la procédure suivante :

```sql
CREATE PROCEDURE getLesClients()

SELECT

    nom,
    prenom

FROM client;
```

L'appel en C# est le suivant :

```csharp
using var cmd = new MySqlCommand()

{
    Connection = cnx,
    CommandText = "getLesClients",
    CommandType = CommandType.StoredProcedure
};

using var reader = cmd.ExecuteReader();

while (reader.Read())
{
    string nom = reader["nom"].ToString();

    string prenom = reader["prenom"].ToString();

    Console.WriteLine($"{prenom} {nom}");
}
```

---

# 46. Lecture des colonnes

Une colonne peut être lue :

par son nom

```csharp
reader["nom"]
```

ou par son indice

```csharp
reader.GetString(0);
```

La lecture par nom est généralement plus lisible.

---

# 47. Procédure avec paramètres

Supposons la procédure suivante.

```sql
CREATE PROCEDURE getNom(

    IN idClient INT

)

BEGIN

    SELECT nom

    FROM client

    WHERE id = idClient;

END;
```

L'appel devient :

```csharp
using var cmd = new MySqlCommand()

{
    Connection = cnx,
    CommandText = "getNom",
    CommandType = CommandType.StoredProcedure
};

cmd.Parameters.AddWithValue("idClient", 5);

using var reader = cmd.ExecuteReader();
```

---

# 48. Procédure avec paramètre OUT

Supposons :

```sql
CREATE PROCEDURE getNom(

    IN idClient INT,

    OUT nomClient VARCHAR(30)

)
```

Le paramètre de sortie est déclaré comme tel.

```csharp
cmd.Parameters.Add(

    "nomClient",

    MySqlDbType.VarChar,

    30

);

cmd.Parameters["nomClient"].Direction =
    ParameterDirection.Output;
```

Après l'exécution :

```csharp
cmd.ExecuteNonQuery();

string nom =
    cmd.Parameters["nomClient"].Value.ToString();
```

---

# 49. Exemple complet

Procédure :

```sql
CREATE PROCEDURE ajouterRendezVous(

    IN _idPraticien INT,

    IN _idMotif INT,

    IN _dateHeure DATETIME,

    OUT _idVisite INT

)
```

Appel :

```csharp
using var cmd = new MySqlCommand()

{
    Connection = cnx,
    CommandText = "ajouterRendezVous",
    CommandType = CommandType.StoredProcedure
};

cmd.Parameters.AddWithValue("_idPraticien", idPraticien);

cmd.Parameters.AddWithValue("_idMotif", idMotif);

cmd.Parameters.AddWithValue("_dateHeure", uneDate);

cmd.Parameters.Add("_idVisite", MySqlDbType.Int32);

cmd.Parameters["_idVisite"].Direction =
    ParameterDirection.Output;

cmd.ExecuteNonQuery();

int idVisite =
(int)cmd.Parameters["_idVisite"].Value;
```

---

# 50. Gestion des erreurs

Une procédure peut générer une exception.

En C#, on utilise :

```csharp
try
{
    cmd.ExecuteNonQuery();
}
catch(MySqlException e)
{
    Console.WriteLine(e.Message);
}
```

Toutes les erreurs SQL sont transformées en exceptions .NET.

---

# 51. Appeler une fonction depuis C#

Une fonction s'appelle comme une requête SQL.

Exemple :

```sql
SELECT getNom(5);
```

En C# :

```csharp
using var cmd =

new MySqlCommand(

    "SELECT getNom(@id);",

    cnx

);

cmd.Parameters.AddWithValue("id",5);

string nom =
    cmd.ExecuteScalar().ToString();
```

`ExecuteScalar()` est la méthode idéale lorsqu'une seule valeur est retournée.

---

# 52. Pourquoi ExecuteScalar ?

Trois méthodes principales existent.

| Méthode | Utilisation |
|----------|-------------|
| ExecuteReader() | Lecture de plusieurs lignes |
| ExecuteNonQuery() | INSERT, UPDATE, DELETE |
| ExecuteScalar() | Une seule valeur |

---

# 53. Appeler une procédure en PHP

Les exemples utilisent **PDO**.

Connexion :

```php
$db = new PDO(
    $dsn,
    $user,
    $password
);
```

---

# 54. Procédure retournant un jeu de résultats

Procédure :

```sql
CALL getLesClients();
```

PHP :

```php
$curseur =
$db->query("CALL getLesClients();");

while($ligne =
$curseur->fetch(PDO::FETCH_ASSOC))
{

    echo $ligne["nom"];

}
```

---

# 55. Récupérer toutes les lignes

Au lieu de parcourir le curseur :

```php
$clients =
$curseur->fetchAll(
    PDO::FETCH_ASSOC
);
```

On obtient directement un tableau.

---

# 56. Procédure avec paramètres

```php
$curseur =
$db->prepare(

"CALL ajouterCompte(

    :login,

    :password,

    :email,

    :nom,

    :prenom

)"

);

$curseur->bindParam(
    "login",
    $login
);

$curseur->bindParam(
    "password",
    $password
);

$curseur->bindParam(
    "email",
    $email
);

$curseur->bindParam(
    "nom",
    $nom
);

$curseur->bindParam(
    "prenom",
    $prenom
);

$curseur->execute();
```

---

# 57. Appeler une fonction en PHP

Une fonction s'appelle dans un SELECT.

```php
$curseur =
$db->prepare(

"SELECT getNom(:id)"

);

$curseur->bindParam(
    "id",
    $idClient
);

$curseur->execute();

$nom =
$curseur->fetchColumn();
```

---

# 58. Gestion des erreurs en PHP

Toutes les erreurs SQL peuvent être interceptées.

```php
try
{

    $curseur->execute();

}
catch(Exception $e)
{

    echo $e->getMessage();

}
```

---

# 59. Architecture recommandée

Une bonne architecture est la suivante :

```
Application

        │

        ▼

Classes DAO

        │

        ▼

Procédures stockées

        │

        ▼

Base de données
```

Le code SQL est centralisé.

Les développeurs travaillent principalement avec des méthodes.

Exemple :

```csharp
clientDao.Ajouter(...);

compteDao.MajSolde(...);

visiteDao.Creer(...);
```

Chaque méthode appelle une procédure.

---

# 60. Avantages

L'utilisation des procédures stockées depuis une application permet :

- d'éviter les requêtes SQL dispersées dans le code ;
- d'améliorer la sécurité ;
- de limiter les risques d'injection SQL ;
- de simplifier la maintenance ;
- de réduire le trafic réseau lorsque plusieurs opérations sont regroupées dans une procédure ;
- de réutiliser facilement la logique métier.

---

# Tableau récapitulatif

| C# | Utilisation |
|-----|-------------|
| ExecuteReader() | Lire plusieurs lignes |
| ExecuteScalar() | Lire une seule valeur |
| ExecuteNonQuery() | Mise à jour |
| CommandType.StoredProcedure | Appel d'une procédure |
| Parameters.AddWithValue() | Paramètre d'entrée |

| PHP (PDO)     | Utilisation            |
| ------------- | ---------------------- |
| prepare()     | Préparer une requête   |
| bindParam()   | Associer un paramètre  |
| execute()     | Exécuter               |
| fetch()       | Lire une ligne         |
| fetchAll()    | Lire toutes les lignes |
| fetchColumn() | Lire une seule valeur  |

# Les procédures stockées MySQL - Partie 5
# Gestion des erreurs

## Introduction

Comme dans n'importe quel langage de programmation, une instruction SQL peut provoquer une erreur.

Par exemple :

- ajout d'une clé primaire déjà existante ;
- violation d'une clé étrangère ;
- suppression d'une table référencée ;
- suppression d'une colonne utilisée par une contrainte ;
- erreur de syntaxe SQL ;
- valeur NULL interdite ;
- dépassement de taille d'un champ.

Une bonne procédure stockée doit être capable :

- de détecter les erreurs prévisibles ;
- de générer ses propres erreurs métier ;
- d'intercepter les erreurs système.

---

# 61. Les erreurs SQL

Quelques erreurs fréquentes :

| Code | Description |
|------:|-------------|
|1062|Clé primaire ou unique déjà existante|
|1048|Valeur NULL interdite|
|1451|Impossible de supprimer une ligne référencée|
|1452|Clé étrangère inexistante|
|1091|Objet inexistant lors d'un DROP|
|1828|Suppression impossible d'une colonne liée à une contrainte|

Exemples :

```
Error Code: 1062
Duplicate entry '1' for key PRIMARY
```

```
Error Code: 1452
Cannot add or update a child row
```

```
Error Code: 1828
Cannot drop column ...
```

---

# 62. Éviter les erreurs

Le meilleur traitement d'erreur est souvent de l'éviter.

Avant une insertion, il est préférable de vérifier qu'une ligne n'existe pas déjà.

Exemple :

```sql
IF EXISTS(

    SELECT 1

    FROM client

    WHERE id = idClient

)

THEN

    ...

END IF;
```

---

# 63. Utiliser EXISTS

`EXISTS` est très rapide.

MySQL s'arrête dès qu'il trouve une ligne.

Exemple :

```sql
IF EXISTS(

    SELECT 1

    FROM visite

    WHERE id = _idVisite

)

THEN

    ...

END IF;
```

---

# 64. Générer une erreur avec SIGNAL

Depuis MySQL 5.5, il est possible de générer une erreur personnalisée.

Syntaxe :

```sql
SIGNAL SQLSTATE '45000'

SET MESSAGE_TEXT = 'Message';
```

Le code SQLSTATE **45000** est réservé aux erreurs définies par le développeur.

---

# 65. Exemple

```sql
IF EXISTS(

    SELECT 1

    FROM visite

    WHERE id = _idVisite

)

THEN

    SIGNAL SQLSTATE '45000'

    SET MESSAGE_TEXT =
        'Cette visite existe déjà';

END IF;
```

La procédure s'interrompt immédiatement.

L'application reçoit alors une exception.

---

# 66. Exemple complet

```sql
CREATE PROCEDURE ajouterVisite(

    IN idVisite INT

)

BEGIN

    IF EXISTS(

        SELECT 1

        FROM visite

        WHERE id = idVisite

    )

    THEN

        SIGNAL SQLSTATE '45000'

        SET MESSAGE_TEXT =
            'Cette visite existe déjà';

    END IF;

    INSERT INTO visite(id)

    VALUES(idVisite);

END;
```

---

# 67. Gestion des erreurs côté application

L'erreur est automatiquement transmise au langage hôte.

En PHP :

```php
try
{

    $curseur->execute();

}
catch(Exception $e)
{

    echo $e->getMessage();

}
```

En C# :

```csharp
try
{
    cmd.ExecuteNonQuery();
}
catch(MySqlException e)
{
    Console.WriteLine(e.Message);
}
```

---

# 68. Les gestionnaires d'erreurs (HANDLER)

Certaines erreurs ne peuvent pas être évitées.

Exemple :

- suppression d'une colonne utilisée dans une clé étrangère ;
- erreur de verrouillage ;
- erreur de syntaxe dans une requête dynamique.

MySQL permet de définir un **gestionnaire d'erreurs**.

---

# 69. Syntaxe

```sql
DECLARE

    EXIT HANDLER

FOR numeroErreur

instruction;
```

ou

```sql
DECLARE

    CONTINUE HANDLER

FOR numeroErreur

instruction;
```

---

# 70. EXIT ou CONTINUE ?

Deux comportements sont possibles.

## EXIT

La procédure s'arrête.

```
Erreur

↓

Gestionnaire

↓

Fin de la procédure
```

---

## CONTINUE

La procédure continue après le traitement.

```
Erreur

↓

Gestionnaire

↓

Suite des instructions
```

---

# 71. Déclaration d'un gestionnaire

Les gestionnaires doivent toujours être déclarés :

1. après les variables locales ;
2. avant les instructions SQL.

Ordre correct :

```sql
DECLARE variable;

DECLARE HANDLER ...;

Instructions...
```

---

# 72. Exemple

Supposons l'erreur :

```
1828

Cannot drop column...
```

On souhaite afficher un message plus compréhensible.

```sql
DECLARE EXIT HANDLER

FOR 1828

SET resultat =
'Le champ est utilisé dans une contrainte.';
```

---

# 73. Exemple complet

```sql
CREATE PROCEDURE suppressionColonne(

    IN nomTable VARCHAR(30),

    IN nomColonne VARCHAR(30)

)

BEGIN

    DECLARE resultat VARCHAR(80);

    DECLARE EXIT HANDLER

    FOR 1828

    SET resultat =
    'La colonne est utilisée dans une contrainte.';
```

Puis :

```sql
SELECT resultat;
```

---

# 74. Vérifier l'existence d'une table

Avant de supprimer une colonne, il est prudent de vérifier que la table existe.

```sql
SELECT 1

FROM information_schema.tables

WHERE table_name = nomTable;
```

---

# 75. Vérifier l'existence d'une colonne

```sql
SELECT 1

FROM information_schema.columns

WHERE table_name = nomTable

AND column_name = nomColonne;
```

---

# 76. Les requêtes dynamiques

Une difficulté apparaît lorsqu'on souhaite passer le nom d'une table en paramètre.

Ceci est impossible :

```sql
ALTER TABLE nomTable
DROP nomColonne;
```

MySQL interprète `nomTable` comme un texte.

Il faut utiliser une requête préparée.

---

# 77. Préparer une requête

```sql
SET @sql =

CONCAT(

'ALTER TABLE ',

nomTable,

' DROP ',

nomColonne

);
```

---

# 78. Exécuter une requête préparée

```sql
PREPARE requete

FROM @sql;

EXECUTE requete;

DEALLOCATE PREPARE requete;
```

La requête est construite puis exécutée.

---

# 79. Exemple complet

```sql
SET @sql = CONCAT(

'ALTER TABLE ',

nomTable,

' DROP ',

nomColonne

);

PREPARE requete

FROM @sql;

EXECUTE requete;

DEALLOCATE PREPARE requete;
```

---

# 80. Pourquoi utiliser des requêtes préparées ?

Parce que les objets SQL ne peuvent pas être passés directement en paramètre.

Par exemple :

- nom de table ;
- nom de colonne ;
- nom d'index ;
- nom d'une vue.

---

# 81. Limitation importante

Les requêtes dynamiques sont autorisées dans :

- les procédures stockées.

Elles sont interdites dans :

- les fonctions ;
- les déclencheurs (Triggers).

---

# 82. Conseils

Toujours :

- vérifier les paramètres avant toute modification ;
- utiliser SIGNAL pour les erreurs métier ;
- utiliser HANDLER pour les erreurs système ;
- écrire des messages d'erreur compréhensibles ;
- privilégier EXIT lorsque l'erreur rend la poursuite du traitement impossible.

---

# Tableau récapitulatif

| Instruction | Utilisation |
|-------------|-------------|
| SIGNAL | Générer une erreur |
| SQLSTATE '45000' | Erreur personnalisée |
| DECLARE HANDLER | Déclarer un gestionnaire |
| EXIT HANDLER | Arrêter la procédure |
| CONTINUE HANDLER | Continuer l'exécution |
| PREPARE | Préparer une requête dynamique |
| EXECUTE | Exécuter la requête |
| DEALLOCATE PREPARE | Libérer la requête |
| EXISTS | Vérifier l'existence d'une ligne |
| information_schema | Métadonnées de la base |

---

# Bonnes pratiques

✔ Vérifier les données avant les INSERT, UPDATE ou DELETE.

✔ Utiliser `SIGNAL` pour les erreurs fonctionnelles (métier).

✔ Utiliser `DECLARE HANDLER` pour intercepter les erreurs SQL que l'on ne peut pas empêcher.

✔ Renvoyer des messages explicites afin de faciliter le débogage de l'application.

✔ Utiliser les requêtes préparées uniquement lorsque les noms des objets SQL (tables, colonnes, index...) doivent être construits dynamiquement.

# Les procédures stockées MySQL - Partie 6
# Bonnes pratiques et guide de conception

## Introduction

Les procédures stockées, les fonctions, les déclencheurs (*Triggers*) et les événements (*Events*) permettent de déplacer une partie de la logique métier dans la base de données.

Cette approche présente de nombreux avantages mais également quelques inconvénients. Il est donc important de savoir **quand** utiliser chaque mécanisme.

---

# 83. Avantages des procédures stockées

## Sécurité

Les utilisateurs n'ont pas besoin d'accéder directement aux tables.

Ils exécutent uniquement des procédures.

```
Application

        │

        ▼

 Procédures stockées

        │

        ▼

      Tables
```

Les droits peuvent être accordés uniquement sur les procédures.

Cela limite les risques :

- de modification accidentelle ;
- de suppression involontaire ;
- d'injection SQL.

---

## Centralisation du code

Une requête complexe n'est écrite qu'une seule fois.

Exemple :

```
Application Web
Application Mobile
Application Bureau

        │

        ▼

getClient()
```

Si la structure de la base évolue, seule la procédure doit être modifiée.

---

## Performances

Les procédures sont analysées et optimisées par le serveur.

Elles permettent également de limiter les échanges réseau.

Sans procédure :

```
SELECT ...

UPDATE ...

INSERT ...

DELETE ...
```

Plusieurs allers-retours sont nécessaires.

Avec une procédure :

```
CALL traitementComplet();
```

Une seule communication est réalisée.

---

## Maintenance

La logique métier est regroupée dans la base.

Elle est plus facile à maintenir.

Exemple :

```
majSolde()

↓

Modification des règles

↓

Toutes les applications bénéficient immédiatement
de la mise à jour.
```

---

## Interface entre la base et l'application

L'application ne connaît plus les tables.

Elle appelle uniquement des procédures.

Exemple :

```sql
CALL ajouterCommande(...);
```

Au lieu de :

```sql
INSERT INTO commande ...

INSERT INTO ligneCommande ...

UPDATE stock ...

INSERT INTO facture ...
```

Toute cette logique est encapsulée.

---

# 84. Inconvénients

## Complexité

Une partie du code est :

- dans l'application ;
- dans la base.

Le débogage peut être plus difficile.

---

## Portabilité

Les procédures MySQL utilisent une syntaxe spécifique.

Un passage vers :

- SQL Server ;
- Oracle ;
- PostgreSQL

nécessitera généralement des adaptations.

---

## Maintenance

Une mauvaise organisation peut conduire à :

- des centaines de procédures ;
- des dépendances difficiles à comprendre ;
- du code dupliqué.

---

## Performances

Une procédure mal conçue peut être moins performante qu'une requête SQL optimisée.

Par exemple :

```
Boucle

↓

1000 UPDATE

↓

Très lent
```

alors qu'un simple :

```sql
UPDATE ...
WHERE ...
```

serait beaucoup plus rapide.

---

# 85. Procédure ou requête SQL ?

Une simple requête est suffisante lorsque :

- une seule instruction SQL est nécessaire ;
- aucun traitement particulier n'est réalisé.

Exemple :

```sql
SELECT *

FROM client;
```

Une procédure devient intéressante lorsque :

- plusieurs requêtes sont exécutées ;
- des contrôles sont nécessaires ;
- plusieurs tables sont modifiées.

---

# 86. Procédure ou fonction ?

Utiliser une procédure lorsque :

- plusieurs résultats sont retournés ;
- une mise à jour est réalisée ;
- plusieurs tables sont modifiées.

Utiliser une fonction lorsque :

- une valeur est calculée ;
- cette valeur doit être utilisée dans un `SELECT`.

Exemple :

```sql
SELECT

nom,

age(dateNaissance)

FROM client;
```

---

# 87. Procédure ou Trigger ?

Un Trigger est exécuté automatiquement.

```
INSERT

↓

Trigger

↓

Traitement
```

Une procédure est exécutée volontairement.

```
CALL ajouterCommande()
```

Les traitements métiers importants sont généralement plus lisibles dans une procédure.

---

# 88. Procédure ou Event ?

Les événements permettent d'exécuter automatiquement une procédure.

Exemple :

```
Tous les jours

↓

CALL archiverCommandes();
```

Les événements sont donc souvent utilisés avec les procédures stockées.

---

# 89. Organisation recommandée

Il est conseillé de regrouper les procédures par domaine fonctionnel.

Exemple :

```
Clients

    ajouterClient()

    modifierClient()

    supprimerClient()

    rechercherClient()
```

```
Produits

    ajouterProduit()

    modifierProduit()

    supprimerProduit()
```

```
Commandes

    ajouterCommande()

    annulerCommande()

    facturerCommande()
```

---

# 90. Convention de nommage

Une convention facilite la maintenance.

Exemple :

| Action | Nom |
|---------|-----|
|Ajouter|ajouterClient|
|Modifier|modifierClient|
|Supprimer|supprimerClient|
|Rechercher|rechercherClient|
|Compter|compterClients|
|Mettre à jour|majSolde|

---

# 91. Utiliser les transactions

Lorsque plusieurs tables sont modifiées, il est recommandé d'utiliser une transaction.

```sql
START TRANSACTION;

...

COMMIT;
```

En cas d'erreur :

```sql
ROLLBACK;
```

---

# 92. Documenter les procédures

Chaque procédure devrait préciser :

- son rôle ;
- les paramètres ;
- les valeurs retournées ;
- les erreurs possibles.

Exemple :

```sql
/*
Procédure :
    ajouterClient

Paramètres :

    nom

    prenom

Retour :

    identifiant créé

Auteur :

    ...
*/
```

---

# 93. Bonnes pratiques

✔ Utiliser des noms explicites.

✔ Limiter la taille des procédures.

✔ Factoriser le code.

✔ Éviter les traitements dupliqués.

✔ Toujours vérifier les paramètres.

✔ Gérer les erreurs avec `SIGNAL`.

✔ Utiliser des transactions lorsque plusieurs tables sont modifiées.

✔ Préférer une fonction lorsqu'un simple calcul est nécessaire.

✔ Commenter les traitements complexes.

---

# 94. À éviter

❌ Écrire une procédure de plusieurs milliers de lignes.

❌ Faire des boucles lorsqu'une requête SQL suffit.

❌ Ignorer les erreurs SQL.

❌ Donner directement accès aux tables aux utilisateurs.

❌ Dupliquer la logique métier dans plusieurs procédures.

---

# 95. Comparatif des objets MySQL

| Objet | Retourne une valeur | Modifie les données | Exécution |
|--------|--------------------|---------------------|-----------|
| Procédure | Oui | Oui | CALL |
| Fonction | Oui (une seule) | Non* | SELECT |
| Trigger | Non | Oui | Automatique |
| Event | Non | Oui | Planifiée |

\* Une fonction ne doit pas modifier l'état de la base de données.

---

# 96. Quand utiliser quoi ?

| Besoin | Solution |
|---------|----------|
|Calculer une valeur|Fonction|
|Ajouter un client|Procédure|
|Mettre à jour plusieurs tables|Procédure|
|Historiser automatiquement une modification|Trigger|
|Nettoyer automatiquement les anciennes données|Event|
|Calculer une TVA|Fonction|
|Créer une facture|Procédure|
|Journaliser une suppression|Trigger|
|Archiver chaque nuit|Event|

---

# 97. Mémo des principales instructions

## Procédures

```sql
CREATE PROCEDURE
```

```sql
CALL
```

```sql
DROP PROCEDURE
```

---

## Fonctions

```sql
CREATE FUNCTION
```

```sql
RETURN
```

```sql
SELECT fonction(...)
```

---

## Variables

```sql
DECLARE
```

```sql
SET
```

```sql
SELECT ... INTO
```

---

## Conditions

```sql
IF
```

```sql
CASE
```

---

## Boucles

```sql
WHILE
```

```sql
LOOP
```

```sql
REPEAT
```

```sql
LEAVE
```

```sql
ITERATE
```

---

## Gestion des erreurs

```sql
SIGNAL
```

```sql
DECLARE HANDLER
```

```sql
EXIT
```

```sql
CONTINUE
```

---

## Transactions

```sql
START TRANSACTION
```

```sql
COMMIT
```

```sql
ROLLBACK
```

---

# Conclusion

Le langage procédural de MySQL permet de développer de véritables programmes directement dans le serveur de bases de données.

Les **procédures stockées** sont destinées à encapsuler les traitements métier et les opérations de mise à jour.

Les **fonctions stockées** permettent de calculer et de retourner une valeur exploitable dans les requêtes SQL.

Les **déclencheurs (Triggers)** automatisent des traitements en réaction aux modifications des données, tandis que les **événements (Events)** planifient l'exécution de tâches à une date ou selon une fréquence donnée.

En combinant ces différents mécanismes avec les variables, les structures conditionnelles, les boucles, les transactions et la gestion des erreurs, il est possible de réaliser des traitements robustes, sécurisés et facilement réutilisables au sein des applications.