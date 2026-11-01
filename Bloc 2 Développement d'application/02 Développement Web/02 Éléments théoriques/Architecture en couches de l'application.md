# Pourquoi découper une application en couches ?

Une application web réalise plusieurs tâches très différentes :
- recevoir une requête HTTP ;
- vérifier les données reçues ;
- appliquer les règles métier ;
- communiquer avec la base de données ;
- renvoyer une réponse au navigateur.

Si toutes ces responsabilités sont mélangées dans un même fichier, le code devient rapidement difficile à maintenir.

L'objectif est donc de répartir le travail entre plusieurs couches, chacune ayant une responsabilité bien précise.

# Vue d'ensemble

```
Navigateur
      ▼
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

Chaque couche ne dialogue qu'avec la couche immédiatement inférieure.

# Répartition des responsabilités

| Couche            | Responsabilité                              |
| ----------------- | ------------------------------------------- |
| Contrôleur        | Gérer la requête HTTP                       |
| Service           | Appliquer les règles métier globales        |
| Classe métier     | Gérer une entité métier et ses requêtes SQL |
| Table             | Fournir les opérations CRUD génériques      |
| Database / Select | Communiquer avec PDO                        |
| PDO               | Communiquer avec MySQL                      |
|                   |                                             |
# 1. Le contrôleur

Le contrôleur constitue le point d'entrée d'une requête HTTP.

Son rôle est volontairement limité.

Il doit :
- vérifier la méthode HTTP (`GET`, `POST`...) ;
- récupérer les paramètres de la requête ;
- vérifier leur format général ;
- appeler le service métier ;
- renvoyer la réponse JSON.

Il ne doit jamais contenir de règles métier.

Exemple :

```php
Requete::verifierPost();

$idCategorie = Requete::postString('idCategorie');

ReponseJson::envoyerLesDonnees(ServiceCategorie::getLesCoureurs($idCategorie));
```

Le contrôleur répond simplement à la question :

> Que demande le navigateur ?

# 2. Le service métier

Le service métier orchestre le traitement.

Il connaît les règles fonctionnelles de l'application.

Le **service** répond à la question : _"Comment réaliser cette action demandée par l'utilisateur ?"_

Exemples :
- un utilisateur ne peut supprimer que ses propres projets ;
- plusieurs opérations SQL doivent être exécutées dans une même transaction.

Le service peut faire intervenir plusieurs classes métier.

Exemple :

```
Suppression d'une catégorie
        ▼
vérifier que la catégorie existe
        ▼
vérifier qu'aucun coureur n'y appartient
        ▼
supprimer la catégorie
        ▼
écrire une trace dans l'historique
```

Toutes ces opérations constituent une seule règle métier.

Le contrôleur ne devrait jamais connaître cette logique.

La couche Service n'est nécessaire que si il faut :
+ orchestrer plusieurs objets
+ réaliser des contrôles nécessitant plusieurs accès ou plusieurs objets (contrôle utilisant une autre table par exemple)
+ réaliser une transaction ou u enchainement d'opérations sur plusieurs tables
# 3. Les classes métier

Chaque classe métier représente une table principale de la base.

Exemples :

```
Categorie
Projet
Coureur
Club
```

Une classe métier connaît uniquement son propre domaine.
La **classe métier** répond à la question : _"Quelles sont les règles de cette table ?"_

Elle contient :
- les requêtes SQL de consultation  concernant cette table ;
- les contrôles métier propres à cette table par exemple "Une catégorie ne peut pas être supprimée si des coureurs lui sont associés."
- la description des colonnes ;
- les contraintes d'unicité.
- les points d'extension (hooks) permettant d'intégrer des traitements spécifiques avant ou après l'action de mise à jour demandé

Elle ne doit pas gérer plusieurs tables simultanément.

Le terme point d'extension (**hook**) désigne un **point d'accroche** prévu dans un programme pour permettre d'ajouter un traitement sans modifier le fonctionnement général.

L'idée est la suivante : Le classe Table exécute son traitement normal, puis il "accroche" éventuellement du code supplémentaire à un endroit précis.
Imaginons une méthode `add()` simplifiée :

```php
public function add(array $data): bool {
    if (!$this->checkColumns($data)) {
        return false;
    }
    // Insertion SQL
    $this->insert();
    return true;
}
```

Pour permettre aux classes filles d'effectuer un traitement juste avant l'insertion il faut ajouter 

```php
public function add(array $data): bool{
    if (!$this->checkColumns($data)) {
        return false;
    }
    if (!$this->beforeInsert()) {
        return false;
    }
    $this->insert();
    return true;
}
```

`beforeInsert()` est un **hook**. Par défaut, il ne fait rien :

```php
protected function beforeInsert(): bool
{
    return true;
}
```

Mais une classe fille peut le redéfinir :

```php
protected function beforeInsert(): bool {
    if ($this->getValue('age') < 18) {
        $this->addError('age', "Âge insuffisant.");
        return false;
    }
    return true;
}
```

La méthode `add()` ne change jamais.  Elle appelle simplement le hook.

Les hooks de type before permettent d'ajouter des contrôles ou de préparer les données avant l'opération.
Les  hooks de type after permettent d'ajouter des traitements **internes** à l'entité après que l'opération SQL s'est correctement terminée.
Par exemple :
- vider un cache ;
- mettre à jour un compteur ;
- enregistrer une trace ;
- recalculer une information dépendante.

En revanche, dès que le traitement devient une **fonctionnalité métier complète** (envoi d'e-mail, facturation, paiement, synchronisation avec une autre application, etc.), il faut passer par un service.
# 4. La classe Table

Toutes les classes métier héritent de `Table`.

Cette classe fournit les traitements communs :

- validation des colonnes ;
- contrôle des colonnes autorisées ;
- contrôle des colonnes obligatoires ;
- contrôle des contraintes d'unicité ;
- ajout ;
- modification ;
- suppression.


Ainsi une classe métier ne décrit que ce qui lui est spécifique.

Exemple :

```
Categorie

↓

defineTable()

defineColumns()

beforeDelete()

beforeUpdate()

afterInsert()

afterUpdate()

afterDelete()

```

Toute la mécanique SQL est déjà fournie par `Table`.

# 5. Les classes techniques

Les classes techniques ne connaissent rien du métier.

Elles rendent simplement des services.

Exemples :

```
Database
Select
Requete
Jeton
ReponseJson
Session
Config
```

Leur rôle est de fournir des outils réutilisables.

# 6. La base de données

La base de données stocke les informations.

Elle ne connaît ni les contrôleurs ni les services.

Elle répond uniquement aux requêtes SQL.

# Exemple complet

Supposons que l'utilisateur souhaite supprimer une catégorie.

Le déroulement est le suivant :

```
Navigateur
      │
      ▼
controleurSupprimerCategorie.php
      │
      ▼
ServiceCategorie::supprimer()
      │
      ▼
Categorie::delete()
      │
      ▼
Table::delete()
      │
      ▼
Database
      │
      ▼
MySQL
```

Chaque couche réalise uniquement son travail.


# Où placer un traitement ?

Cette question revient très souvent.

## Vérifier qu'un paramètre POST existe

→ Contrôleur

```
Requete::postString()
```

## Vérifier qu'un identifiant est bien formé

→ Contrôleur

```
M10
SEN
A
```

Le contrôleur vérifie uniquement le format.

## Vérifier que la catégorie existe

→ Classe métier

```
Categorie::exists()
```

La classe métier connaît sa table.

---

## Vérifier qu'une catégorie est vide avant suppression

→ Classe métier

```
beforeDelete()
```

Il s'agit d'une règle propre aux catégories.

---

## Supprimer une catégorie puis enregistrer un historique

→ Service métier

Deux opérations différentes doivent être réalisées ensemble.

---

## Construire une requête SQL

→ Classe métier

## Ouvrir une connexion MySQL

→ Database

Jamais ailleurs.

# Ce qu'il faut retenir

Chaque couche possède une responsabilité unique.

```
Contrôleur
    ↓
reçoit la requête

Service
    ↓
coordonne le métier

Classe métier
    ↓
connaît une table

Table
    ↓
fournit le CRUD générique

Database / Select
    ↓
utilisent PDO

PDO
    ↓
communique avec MySQL
```

Plus chaque couche reste spécialisée, plus l'application est simple à comprendre, à maintenir et à faire évoluer.

Une même opération peut traverser plusieurs couches, mais chaque couche ne réalise qu'un seul type de travail.