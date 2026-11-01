## Objectifs

À la fin de ce chapitre, vous serez capable de :

- comprendre le rôle de PDO dans une application PHP ;
- connaître les avantages de PDO par rapport aux autres interfaces d'accès aux bases de données ;
- comprendre la place de PDO dans une architecture trois tiers ;
- identifier les principales classes composant l'API PDO ;
- comprendre le cycle de vie d'une requête SQL.

---

# 1.1 Qu'est-ce que PDO ?

**PDO** (**PHP Data Objects**) est l'API officielle de PHP permettant de communiquer avec un **Système de Gestion de Base de Données Relationnelle (SGBDR)**.

Elle fournit une interface unique pour manipuler différents moteurs de bases de données sans modifier la majorité du code de l'application.

Parmi les SGBDR supportés :

- MySQL / MariaDB
- PostgreSQL
- SQLite
- Oracle
- Microsoft SQL Server
- IBM DB2
- Firebird
- Informix
- Sybase

L'objectif de PDO est de séparer le code PHP des spécificités du moteur de base de données.

Ainsi, une application développée avec PDO pourra être migrée plus facilement d'un SGBDR vers un autre.

---

# 1.2 Pourquoi utiliser PDO ?

Avant l'apparition de PDO, PHP proposait plusieurs extensions spécifiques :

- `mysql_*` (supprimée depuis PHP 7)
- `mysqli`
- `pgsql`
- `sqlite`
- etc.

Chaque extension possédait sa propre syntaxe.

Exemple avec MySQL :

```
$result = mysqli_query($connexion, $sql);
```

Exemple avec PostgreSQL :

```
$result = pg_query($connexion, $sql);
```

Changer de moteur impliquait donc de réécrire une partie importante du code.

PDO résout ce problème grâce à une interface commune.

```
$stmt = $pdo->query($sql);
```

Le code reste identique quel que soit le SGBDR.

---

# 1.3 Les avantages de PDO

PDO présente de nombreux avantages.

## Une interface unique

Le même code permet d'interroger différents moteurs de bases de données.

Seule la chaîne de connexion (DSN) change.

---

## Les requêtes préparées

PDO permet d'utiliser des requêtes préparées.

Les paramètres sont transmis séparément de la requête SQL.

Cela apporte plusieurs bénéfices :

- suppression des injections SQL ;
- meilleure gestion des caractères spéciaux ;
- optimisation des requêtes répétitives.

Exemple :

```
$sql = "SELECT * FROM utilisateur
        WHERE email = :email";

$stmt = $pdo->prepare($sql);

$stmt->execute([
    "email" => $email
]);
```

---

## Une API orientée objet

PDO est entièrement orienté objet.

Les principales opérations sont réalisées à l'aide de méthodes.

```
$pdo->prepare(...)
$pdo->query(...)
$stmt->execute()
$stmt->fetch()
```

Le code est plus lisible et plus facilement maintenable.

---

## Les transactions

PDO permet de regrouper plusieurs requêtes dans une transaction.

Une transaction garantit que :

- toutes les requêtes sont exécutées ;
- ou aucune ne l'est.

Cette fonctionnalité est indispensable pour garantir la cohérence des données.

---

## Une meilleure portabilité

La majorité du code SQL reste indépendante du moteur utilisé.

Lors d'un changement de SGBDR, il suffit généralement de modifier :

- le DSN ;
- quelques particularités SQL.

---

## Une récupération flexible des données

PDO permet de récupérer les résultats sous différentes formes :

- tableau associatif ;
- tableau numérique ;
- objet ;
- instance d'une classe ;
- valeur unique.

Cette souplesse facilite l'intégration avec le reste de l'application.

---

# 1.4 Les limites de PDO

PDO simplifie l'accès aux bases de données mais ne masque pas totalement les différences entre les SGBDR.

Par exemple :

- certaines fonctions SQL sont propres à MySQL ;
- les procédures stockées ne sont pas utilisées de la même manière selon les moteurs ;
- la syntaxe des séquences ou de l'auto-incrément varie.

PDO ne remplace donc pas la connaissance du langage SQL.

---

# 1.5 Les principales classes de PDO

L'API PDO est composée de plusieurs classes.

Dans la majorité des applications, deux classes sont utilisées quotidiennement.

## La classe PDO

Cette classe représente la connexion à la base de données.

Elle permet notamment de :

- ouvrir une connexion ;
- préparer une requête ;
- exécuter une requête ;
- démarrer une transaction ;
- récupérer le dernier identifiant généré.

Schéma simplifié :

```
Application
      │
      ▼
   Objet PDO
      │
      ▼
 Base de données
```

---

## La classe PDOStatement

Une requête SQL préparée ou exécutée est représentée par un objet `PDOStatement`.

Cet objet permet :

- d'associer des paramètres ;
- d'exécuter la requête ;
- de récupérer les résultats ;
- de fermer le curseur.

```
PDO
 │
 ├── prepare()
 │
 ▼
PDOStatement
 │
 ├── execute()
 ├── fetch()
 ├── fetchAll()
 └── closeCursor()
```

---

# 1.6 Fonctionnement général de PDO

Le fonctionnement de PDO suit toujours les mêmes étapes.

Toutes les opérations de consultation et de modification suivent ce cycle.

---

# 1.7 Architecture trois tiers

PDO s'intègre naturellement dans une architecture trois tiers.

Chaque niveau possède un rôle précis.

## Le client

Le client correspond au navigateur.

Il est responsable :

- de l'affichage ;
- des interactions utilisateur ;
- des appels AJAX.

Le client ne dialogue jamais directement avec la base de données.

---

## Le serveur PHP

Le serveur constitue le cœur de l'application.

Il :

- reçoit les requêtes HTTP ;
- contrôle les données ;
- exécute les traitements métiers ;
- interroge la base via PDO ;
- construit la réponse.

---

## Le SGBDR

Le système de gestion de base de données :

- stocke les données ;
- exécute les requêtes SQL ;
- garantit l'intégrité des informations.

---

# 1.8 Cycle complet d'une requête

Lorsqu'un utilisateur demande des informations :

Cette séquence est identique pour la majorité des applications web.

---

# 1.9 Place de PDO dans une architecture applicative

Dans une application correctement structurée, PDO n'est généralement jamais utilisé directement dans les contrôleurs ou les vues.

Une architecture classique est la suivante :

```
Client
   │
Contrôleur
   │
Service métier
   │
Repository / DAO
   │
Database (PDO)
   │
Base de données
```

Cette organisation présente plusieurs avantages :

- séparation des responsabilités ;
- meilleure réutilisation du code ;
- facilité de maintenance ;
- simplification des tests ;
- centralisation des requêtes SQL.

Les chapitres suivants montreront comment mettre en place cette organisation.

---

# 1.10 Les grandes étapes d'un développement avec PDO

Le développement d'une fonctionnalité reposant sur une base de données suit généralement les étapes suivantes :

1. obtenir une connexion (`PDO`) ;
2. préparer la requête SQL ;
3. associer les paramètres éventuels ;
4. exécuter la requête ;
5. récupérer les résultats ;
6. exploiter les données dans l'application ;
7. fermer le curseur si nécessaire.

Cette séquence sera utilisée tout au long de ce document.

---

# À retenir

- **PDO** est l'API standard de PHP pour accéder aux bases de données relationnelles.
- Elle fournit une interface commune à de nombreux SGBDR.
- Les deux classes principales sont **PDO** et **PDOStatement**.
- Les requêtes préparées améliorent la sécurité et la maintenabilité du code.
- PDO s'intègre naturellement dans une architecture trois tiers.
- Une requête suit toujours le même cycle : **connexion → préparation → exécution → récupération des résultats**.
- Une bonne organisation consiste à centraliser l'accès aux données dans une couche dédiée (Repository, DAO ou classes métier) plutôt que de disperser les requêtes SQL dans toute l'application.