## Présentation

La classe `Select` centralise toutes les opérations de consultation de la base de données.

Elle est exclusivement dédiée aux requêtes de lecture SQL (`SELECT`).

Elle complète la classe `Table`, destinée aux opérations de modification des données :

| Classe | Rôle |
|--------|------|
| `Select` | Consultation (`SELECT`) |
| `Table` | Modification (`INSERT`, `UPDATE`, `DELETE`) |

L'objectif principal de `Select` est d'éviter la duplication du code PDO dans les différentes classes d'accès aux données.

Elle fournit une interface simple permettant de récupérer :

- plusieurs lignes ;
- une seule ligne ;
- une seule valeur.

# Rôle dans l'architecture

La classe `Select` constitue le point d'entrée unique pour les consultations SQL.

```text
Application
      │
      ▼
   Select
      │
      ▼
 Base de données
```

Cette organisation permet :

- de regrouper le code de consultation SQL ;
- d'éviter la répétition du code PDO ;
- d'avoir une utilisation homogène des requêtes de lecture ;
- d'utiliser automatiquement les requêtes préparées lorsque des paramètres sont nécessaires.

# Construction

La classe s'utilise simplement :

```php
$select = new Select();
```

La connexion à la base de données est obtenue automatiquement.

L'utilisateur de la classe n'a pas besoin de gérer directement l'objet PDO.

# Choisir la bonne méthode

La classe propose trois méthodes publiques.

```text
              Select
                 │
      ┌──────────┼──────────┐
      ▼          ▼          ▼
   getRows()   getRow()   getValue()
```

Le choix dépend du résultat attendu :

| Besoin | Méthode |
|--------|---------|
| Plusieurs lignes | `getRows()` |
| Une seule ligne | `getRow()` |
| Une seule valeur | `getValue()` |

# Méthode `getRows()`

## Rôle

Retourne plusieurs lignes issues d'une requête SQL.

### Signature

```php
getRows(string $sql, array $lesParametres = []): array
```

### Retour

La méthode retourne un tableau contenant les lignes trouvées.

Chaque ligne est un tableau associatif :

```php
[
    "id" => 15,
    "nom" => "Dupont",
    "ville" => "Lille"
]
```

Si aucune ligne n'est trouvée [] est retourné.
## Exemple

```php
$clients = $select->getRows("SELECT id, nom FROM client ORDER BY nom");

foreach ($clients as $client) {
    echo $client["nom"];
}
```

# Méthode `getRow()`

## Rôle

Retourne une seule ligne correspondant à une recherche.

Cette méthode est adaptée lorsque l'on attend au maximum un enregistrement :

- recherche par identifiant ;
- recherche par clé unique ;
- consultation d'un détail.

### Signature

```php
getRow(string $sql, array $lesParametres = []): ?array
```

### Retour

La méthode retourne :

- un tableau associatif si une ligne est trouvée ;
- `null` si aucune ligne ne correspond.

## Exemple

```php
$membre = $select->getRow("SELECT ... FROM membre WHERE id = :id", ["id" => 12]);

if ($membre !== null) {
    echo $membre["nom"];
}
```

# Méthode `getValue()`

## Rôle

Retourne une seule valeur issue d'une requête.

Cette méthode est particulièrement adaptée aux requêtes :

- `COUNT()`
- `SUM()`
- `AVG()`
- `MIN()`
- `MAX()`

mais également lorsqu'une requête retourne une seule colonne.

### Signature

```php
getValue(string $sql, array $lesParametres = []): mixed
```

## Exemple

```php
$total = $select->getValue("SELECT COUNT(*) FROM membre");

```

Autre exemple :

```php
$nom = $select->getValue("SELECT nom FROM membre  WHERE id = :id", ["id" => 25 ]);
```

## Particularité du retour

`getValue()` conserve le comportement de PDO :

| Situation          | Retour               |
| ------------------ | -------------------- |
| Une ligne trouvée  | Valeur de la colonne |
| Colonne SQL `NULL` | `null`               |
| Aucune ligne       | `false`              |

Ce comportement permet de ne pas confondre une vraie valeur retournée par la base avec une absence de résultat.

# Utilisation des paramètres SQL

Les paramètres doivent être transmis sous forme de tableau associatif.

Exemple :

```php
$clients = $select->getRows(
    "SELECT ...
     FROM client
     WHERE ville = :ville
     AND actif = :actif",
    [
        "ville" => "Lille",
        "actif" => 1
    ]
);
```

Cette écriture présente plusieurs avantages :

- meilleure lisibilité ;
- aucune concaténation de valeurs dans le SQL ;
- protection contre les injections SQL.

# Gestion des ressources

Après chaque lecture, la classe libère automatiquement les ressources utilisées par la requête.

L'utilisateur n'a donc pas besoin de gérer :

- les objets `PDOStatement` ;
- la fermeture des curseurs ;
- la libération des ressources.

# Cycle général d'utilisation

```text
Création de Select
        ▼
Choix de la méthode
        ▼
Transmission du SQL
        ▼
Paramètres éventuels
        ▼
Résultat retourné
```

# Exemples complets

## Liste d'enregistrements

```php
$select = new Select();

$articles = $select->getRows("SELECT ... FROM article ORDER BY date_creation DESC");
```

## Consultation d'un enregistrement

```php
$article = $select->getRow("SELECT ... FROM article WHERE id = :id", ["id" => $id ]);
```

## Calcul d'une valeur

```php
$nombre = $select->getValue("SELECT COUNT(*) FROM article");
```

# Avantages de cette architecture

- Point d'entrée unique pour toutes les consultations SQL.
- Code PDO entièrement factorisé.
- API simple avec seulement trois méthodes publiques.
- Utilisation automatique des requêtes préparées avec paramètres.
- Protection contre les injections SQL.
- Gestion automatique des ressources PDO.
- Séparation claire entre lecture (`Select`) et modification (`Table`).

## Voir aussi

- [[01 La classe Database]]
- [[Table -Utilisation]]
- [[Architecture d'un projet Web]]
- [[PDO]]
- [[Requêtes préparées]]