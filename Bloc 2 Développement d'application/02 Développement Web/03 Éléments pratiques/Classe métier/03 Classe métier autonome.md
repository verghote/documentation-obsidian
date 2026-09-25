## Présentation

Toutes les classes métier ne correspondent pas obligatoirement à une table unique.

Certaines classes métier sont dites **autonomes** : elles ne dérivent pas d'une classe technique comme `Table` et ne bénéficient donc pas d'un CRUD automatique.

Elles sont utilisées lorsque les traitements associés à une fonctionnalité nécessitent :

- plusieurs requêtes SQL spécifiques ;
- plusieurs tables ;
- des opérations métier particulières ;
- des traitements qui ne peuvent pas être généralisés.

Ces classes contiennent directement :

- les requêtes SQL nécessaires ;
- les règles métier spécifiques ;
- les méthodes de consultation ;
- les méthodes de mise à jour.

# Différence avec une classe métier héritant de `Table`

## Classe métier dérivée de `Table`

La classe décrit principalement :
- une table ;
- ses colonnes ;
- ses règles de validation.

Les opérations CRUD sont héritées : add(), modify(), delete()

La classe métier ajoute uniquement les particularités.

## Classe métier autonome

La classe ne représente pas forcément une table.

Elle regroupe des opérations correspondant à une fonctionnalité.

Elle écrit directement ses traitements :

```
Projet::getAll();

Projet::ajouter();

Projet::modifierNom();

Projet::supprimer();
```

# Organisation générale d'une classe métier autonome

Une classe autonome est généralement organisée en deux parties :

```
class Projet
{

    // ==============================
    // Méthodes de consultation
    // ==============================


    // ==============================
    // Méthodes de mise à jour
    // ==============================

}
```

# Les méthodes de consultation

Les méthodes de consultation sont généralement déclarées `static`.

Elles ne nécessitent pas la création d'un objet.

Exemple :

```
Projet::getAll();
```

La méthode réalise directement la requête SQL nécessaire.

# La méthode `getAll()`

Cette méthode retourne généralement une collection d'éléments.

Exemple :

```php
public static function getAll(): array
{
    $sql = <<<SQL
        SELECT id, nom
        FROM projet
        ORDER BY nom;
SQL;
    $select = new Select();
    return $select->getRows($sql);
}
```

Elle peut être utilisée pour :

- afficher une liste ;
- remplir une interface ;
- transmettre des données JSON.

Exemple :

```php
$projets = Projet::getAll();
```

Résultat :

```
[
    [
        "id" => 1,
        "nom" => "Site Web"
    ],
    [
        "id" => 2,
        "nom" => "Application mobile"
    ]
]
```

---

# La méthode `getById()`

Cette méthode retourne un élément identifié par sa clé primaire.

Exemple :

```
public static function getById(int $id): mixed
```

Utilisation :

```
$projet = Projet::getById(5);
```

Elle est souvent utilisée :

- avant une modification ;
- avant une suppression ;
- pour afficher un détail.

---

# La méthode `existe()`

Une classe autonome peut proposer des méthodes utilitaires permettant de vérifier une condition.

Exemple :

```php
public static function existe(int $idProjet): bool
{
    $sql = <<<SQL
        SELECT 1
        FROM projet
        WHERE id = :id
SQL;
    $select = new Select();
    return $select->getRow($sql, ['id' => $idProjet]) !== null;
}
```

Cette méthode permet d'éviter de dupliquer des tests :

```php
if (Projet::existe($id)) {

}
```

# Les méthodes de mise à jour

Les opérations de modification sont écrites directement dans la classe.

Elles utilisent généralement :

- `Database::getInstance()` ;
- `prepare()` ;
- `execute()`.

# Ajout d'un enregistrement

Exemple :

```php
public static function ajouter(string $nom): int
{
    $db = Database::getInstance();
    $sql = <<<SQL
        INSERT INTO projet(nom)
        VALUES(:nom)
SQL;
    $cmd = $db->prepare($sql);
    $cmd->execute(["nom" => $nom]);
    return (int)$db->lastInsertId();
}
```

La méthode retourne souvent l'identifiant créé.

Pourquoi ?

Parce que l'ajout est souvent suivi d'autres opérations.

Exemple :

```php
$idProjet = Projet::ajouter($nom);
CompetenceProjet::ajouterUneCompetence($idProjet, $idCompetence);
```

# Modification d'un enregistrement

Exemple :

```php
public static function modifierNom(int $id,string $nom): bool
{
    $db = Database::getInstance();
    $sql = <<<SQL
        UPDATE projet
        SET nom = :nom
        WHERE id = :id;
SQL;
    $cmd = $db->prepare($sql);
    $cmd->execute(["id" => $id, "nom" => $nom]);
    return $cmd->rowCount() > 0;
}
```

La méthode peut retourner un booléen indiquant si une modification a réellement eu lieu.

# Suppression d'un enregistrement

Exemple :

```php
public static function supprimer(int $id): bool
{
    $db = Database::getInstance();
    $sql = <<<SQL
        DELETE FROM projet
        WHERE id = :id;
SQL;
    $cmd = $db->prepare($sql);
    $cmd->execute(["id" => $id]);
    return $cmd->rowCount() > 0;
}
```

Le test est effectué directement grâce au nombre de lignes supprimées.
Il est préférable **de ne pas faire un `SELECT` préalable pour vérifier l'existence de l'id**. La suppression elle-même donne déjà l'information nécessaire grâce au nombre de lignes affectées.

Vérifier l'existence avec un `SELECT` préalable présente plusieurs inconvénients :

- une requête supplémentaire inutile ;
- un risque de décalage entre le test et la suppression (un autre utilisateur pourrait modifier les données entre les deux requêtes) ;
- un code plus long.

La bonne approche est donc de laisser le `DELETE` faire son travail et d'utiliser `rowCount()`.

# Classes métier représentant une table d'association

Certaines tables ne représentent pas un objet métier mais une relation.

Exemple :

```
Projet
   |
   |
competenceProjet
   |
   |
Compétence
```

La table :

```
competenceProjet
----------------
idProjet
idCompetence
```

est une table d'association.

Elle possède donc une classe autonome :

```
CompetenceProjet
```

# Exemple : classe `CompetenceProjet`

Cette classe contient des opérations propres à la relation :

```php
CompetenceProjet::getLesCompetences($idProjet);
```

Retourne les compétences associées.


```php
CompetenceProjet::ajouterUneCompetence($idProjet,$idCompetence);
```

Ajoute une association.

```php
CompetenceProjet::supprimerUneCompetence($idProjet, $idCompetence);
```

Supprime une association.

# Utilisation de `Select`

Les classes autonomes utilisent généralement la classe technique `Select` pour les consultations.

Exemple :

```php
$select = new Select();
return $select->getRows($sql, ["idProjet" => $idProjet]);
```

Avantages :

- moins de code répétitif ;
- utilisation uniforme des requêtes de consultation ;
- séparation entre consultation et mise à jour.

# Utilisation de `Database`

Pour les modifications, la classe utilise directement PDO :

```php
$db = Database::getInstance();
```

Puis :

```pgp
$cmd = $db->prepare($sql);

$cmd->execute();
```

La classe autonome conserve donc la maîtrise complète du traitement SQL.

# Quand créer une classe métier autonome ?

Créer une classe autonome lorsque :

✅ la classe représente une fonctionnalité et non une simple table ;

✅ plusieurs tables sont manipulées ensemble ;

✅ les opérations ne correspondent pas à un CRUD classique ;

✅ les traitements nécessitent des méthodes spécifiques.

---

# Exemples de choix d'architecture

|Besoin|Type de classe|
|---|---|
|Gestion d'une catégorie|Classe héritant de `Table`|
|Gestion d'un utilisateur simple|Classe héritant de `Table`|
|Gestion d'un projet avec ses compétences|Classes autonomes|
|Gestion d'une association plusieurs-à-plusieurs|Classe autonome|
|Calcul statistique|Classe autonome|
|Authentification|Classe autonome|

# Résumé

Les classes métier autonomes complètent les classes dérivées de `Table`.

Elles ne cherchent pas à automatiser le CRUD mais à **encapsuler des traitements métier spécifiques**.

Elles :

- utilisent directement PDO ;
- contiennent leurs propres requêtes SQL ;
- exposent des méthodes orientées métier ;
- peuvent manipuler plusieurs tables ;
- permettent de conserver une architecture claire.

La règle générale est :

> Une classe dérivée de `Table` décrit une table.  
> Une classe métier autonome décrit une fonctionnalité métier.