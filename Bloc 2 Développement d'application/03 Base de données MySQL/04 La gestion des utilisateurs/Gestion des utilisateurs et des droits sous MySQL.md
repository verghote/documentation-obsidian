## 1. Présentation

Le langage SQL permet de créer des utilisateurs et de leur donner des droits sur les objets gérés par un serveur de base de données (bases de données, tables, vues, etc.).

Il existe deux moyens principaux pour paramétrer les droits d'accès sous MySQL :

- **Utilisation des commandes SQL** : `CREATE USER`, `GRANT` et `REVOKE`.
    
- **Utilisation d'une application cliente** (MySQL WorkBench, DataGrip, PhpMyAdmin, etc.).
    

Le serveur MySQL stocke toutes les informations concernant les utilisateurs, leurs privilèges et autorisations dans la base de données système **`mysql`**. 

Elle contient notamment les tables suivantes :

- **`mysql.user`** : Contient les informations sur les utilisateurs (nom, hachage du mot de passe, privilèges globaux et paramètres d'authentification).
- **`mysql.db`** : Contient les droits accordés aux utilisateurs au niveau des bases de données.
- **`mysql.tables_priv`** : Contient les droits accordés sur les tables.
- **`mysql.columns_priv`** : Contient les droits accordés sur les colonnes des tables.
- **`mysql.procs_priv`** : Contient les droits accordés sur les procédures stockées et fonctions.

## 2. Création et gestion des utilisateurs

Sous MySQL, un compte utilisateur est identifié par un nom d'utilisateur et un nom d'hôte (machine depuis laquelle il peut se connecter), séparés par le symbole `@`.

### 2.1. Création d'un utilisateur

|**Propriété**|**Valeur / Code SQL**|
|---|---|
|**Syntaxe**|`CREATE USER [IF NOT EXISTS] nomUser@nomServeur IDENTIFIED BY 'motDePasse';`|
|**Exemple**|`CREATE USER 'sa'@'localhost' IDENTIFIED BY 'BTSIO';`|
|**Remarque**|Par défaut, un nouvel utilisateur a uniquement le droit `USAGE` (il peut juste se connecter, mais n'a aucun privilège).|
### 2.2. Suppression d'un utilisateur

|**Propriété**|**Valeur / Code SQL**|
|---|---|
|**Syntaxe**|`DROP USER [IF EXISTS] nomUser@nomServeur;`|
|**Exemple**|`DROP USER 'sa'@'localhost';`|
|**Remarque**|La suppression d'un utilisateur entraîne la suppression automatique et définitive de tous ses droits.|

### 2.3. Modification du mot de passe

|**Propriété**|**Valeur / Code SQL**|
|---|---|
|**Syntaxe**|`ALTER USER nomUser@nomServeur IDENTIFIED BY 'nouveauMotDePasse';`|
|**Exemple**|`ALTER USER 'root'@'localhost' IDENTIFIED BY 'BTSSIO';`|
|**Remarque**|Fonctionnalité disponible à partir de MySQL 5.7.6 et supérieur.|

### 2.4. Modification du nom (Renommage)

|**Propriété**|**Valeur / Code SQL**|
|---|---|
|**Syntaxe**|`RENAME USER nomUser@nomServeur TO nouveauNomUser@nomServeur;`|
|**Exemple**|`RENAME USER 'sa'@'localhost' TO 'admin'@'localhost';`|
|**Remarque**|Permet de renommer un compte existant tout en conservant ses privilèges associés.|
### Remarques importantes :

- Le délimiteur d'apostrophe (`'`) doit être utilisé si le nom de l'utilisateur ou de l'hôte contient des caractères spéciaux ou des tirets.
- Le nom du serveur peut être remplacé par `%` pour autoriser la connexion depuis n'importe quel hôte : `CREATE USER 'sa'@'%' IDENTIFIED BY 'BTSIO';`.
- **Attention :** Si l'on omet l'hôte (ex: `CREATE USER 'sa'`), MySQL utilise `%` par défaut.
- Le mot de passe est techniquement facultatif, mais fortement déconseillé.

## 3. Attribution des droits

La syntaxe générale de la commande `GRANT` est la suivante :

SQL

```
GRANT droit [, droit…] 
ON *.* | nomBase.* | nomBase.nomTable 
TO user [, user…] 
[WITH GRANT OPTION];
```

> **Attention au `WITH GRANT OPTION` :** Cette clause permet à l'utilisateur de transmettre à son tour ses privilèges à d'autres utilisateurs. 
> À utiliser avec une extrême prudence, car MySQL ne gère pas la suppression en cascade des droits.

### Les niveaux de privilèges :

1. **Niveau global (`*.*`)** : S'applique à toutes les bases du serveur. Stocké dans `mysql.user`.
2. **Niveau base de données (`nomBase.*`)** : S'applique à toutes les tables d'une base spécifique. Stocké dans `mysql.db`.
3. **Niveau table (`nomBase.nomTable`)** : S'applique à une table précise. Stocké dans `mysql.tables_priv`.
4. **Niveau colonne (`nomBase.nomTable (col1, col2)`)** : S'applique à des colonnes spécifiques. Stocké dans `mysql.columns_priv`.

### 3.1. Liste des principaux droits

#### Droits sur les instructions

- **`ALL [PRIVILEGES]`** : Tous les droits (sauf `WITH GRANT OPTION`).
- **`ALTER`** : Autorise l'utilisation de `ALTER TABLE`.
- **`CREATE`** / **`DROP`** : Autorise la création et la suppression de tables ou bases.
- **`EXECUTE`** : Autorise l'exécution de procédures et fonctions stockées (`GRANT EXECUTE ON PROCEDURE nomPro TO nomUser;`).
- **`REFERENCES`** : Nécessaire pour définir des contraintes de clés étrangères (`FOREIGN KEY`).
- **`INDEX`** : Autorise la création et suppression d'index.
- **`TRIGGER`** : Autorise la création de déclencheurs.
- **`USAGE`** : Synonyme de "aucun droit" (l'utilisateur peut uniquement se connecter et voir la base système `information_schema`).

#### Droits sur les données (CRUD)

- **`SELECT`** : Consultation (possibilité de limiter à certaines colonnes : `GRANT SELECT (id, nom) ON Table TO user;`).
- **`INSERT`** : Ajout d'enregistrements.
- **`UPDATE`** : Modification (possibilité de limiter à certaines colonnes).
- **`DELETE`** : Suppression d'enregistrements.

### 3.2. Exemples d'attribution

```sql
-- Administrateur complet avec droit de transmission
GRANT ALL PRIVILEGES ON *.* TO 'sa'@'localhost' WITH GRANT OPTION;

-- Tous les droits sur une base spécifique
GRANT ALL PRIVILEGES ON atlantic.* TO 'admin'@'localhost';

-- Droits restreints de type CRUD sur une table
GRANT SELECT, INSERT, UPDATE, DELETE ON atlantic.client TO 'gestion'@'localhost';

-- Droits de consultation ciblés sur quelques colonnes
GRANT SELECT (id, nom) ON atlantic.client TO 'gestion'@'localhost';
```

## 4. Particularité du droit `EXECUTE` et sécurité des procédures

Par défaut, une procédure stockée est exécutée avec les privilèges de l'utilisateur qui l'a **créée** (mode `DEFINER`).

Cela permet de mettre en place une interface de sécurité : un utilisateur sans aucun droit sur une table sensible peut exécuter une procédure autorisée qui va manipuler cette table pour lui.

```sql
-- Création d'une procédure avec les droits de son créateur (comportement par défaut)
CREATE PROCEDURE ajoutSalarie(IN nom VARCHAR(20), IN prenom VARCHAR(20)) 
SQL SECURITY DEFINER
BEGIN
    INSERT INTO salarie(nom, prenom) VALUES (nom, prenom);
END;
```

Si l'on souhaite au contraire que la procédure s'exécute avec les droits de l'utilisateur qui l'appelle (nécessitant donc que _cet utilisateur_ ait les droits sur la table), on utilise la clause **`SQL SECURITY INVOKER`** :

```sql
CREATE PROCEDURE ajoutSalarie(...) 
SQL SECURITY INVOKER
BEGIN
    -- ...
END;
```
## 5. La suppression des droits (`REVOKE`)

Pour retirer des privilèges à un utilisateur, on utilise la commande `REVOKE` :

```sql
-- Supprimer tous les privilèges globaux
REVOKE ALL ON *.* FROM 'sa'@'localhost';
REVOKE GRANT OPTION ON *.* FROM 'sa'@'localhost';

-- Supprimer les droits sur une table précise
REVOKE ALL ON atlantic.client FROM 'gestion'@'localhost';

-- Supprimer un droit ciblé sur une colonne
REVOKE SELECT(id, nom) ON atlantic.client FROM 'gestion'@'localhost';
```
## 6. Commandes utiles d'administration

- **`SHOW PRIVILEGES;`** : Affiche la liste de tous les droits gérés par le serveur.
- **`SHOW GRANTS;`** : Affiche les droits de l'utilisateur actuellement connecté.
- **`SHOW GRANTS FOR 'user'@'host';`** : Affiche les droits d'un utilisateur spécifique.
- **`FLUSH PRIVILEGES;`** : Recharge les tables de privilèges en mémoire (nécessaire si les tables de la base `mysql` ont été modifiées directement par des requêtes `INSERT`/`UPDATE`).
- **`SHOW DATABASES [LIKE 'pattern'];`** : Liste les bases de données accessibles.
## 7. La notion de rôle

Un **rôle** permet de regrouper un ensemble de permissions afin de les attribuer globalement à un ou plusieurs utilisateurs.

```sql
-- 1. Création des rôles
CREATE ROLE IF NOT EXISTS 'administrateurs', 'lecteurs', 'redacteurs';

-- 2. Attribution des droits aux rôles
GRANT ALL ON atlantic.* TO 'administrateurs';
GRANT SELECT ON atlantic.* TO 'lecteurs';
GRANT INSERT, UPDATE, DELETE ON atlantic.* TO 'redacteurs';

-- 3. Attribution des rôles aux utilisateurs
GRANT 'lecteurs' TO 'dupont'@'localhost', 'durand'@'localhost';

-- 4. Définition d'un rôle par défaut (pour qu'il soit actif dès la connexion)
ALTER USER 'dupont'@'localhost' DEFAULT ROLE 'lecteurs';
```