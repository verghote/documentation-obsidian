
# Principe

Certaines classes métier n'ont pour rôle que de **consulter les données**. Contrairement aux classes dérivées de `Table`, elles ne réalisent **aucune opération de création, de modification ou de suppression**.

Elles regroupent les requêtes SQL permettant de récupérer les informations nécessaires à l'application.

Ces classes sont généralement utilisées pour :

- alimenter des listes déroulantes ;
- afficher des tableaux ;
- effectuer des recherches ;
- vérifier l'existence d'un enregistrement ;
- fournir des données de référence.

Les exemples typiques sont les classes `Club` et `Competence`.

# Caractéristiques

Une classe métier de consultation possède généralement les caractéristiques suivantes :

- elle **n'hérite pas** de la classe `Table` ;
- toutes ses méthodes sont **statiques** ;
- elle ne possède aucun attribut d'instance ;
- elle utilise principalement la classe `Select` pour exécuter les requêtes SQL ;
- elle peut utiliser directement `Database` lorsqu'un traitement PDO particulier est nécessaire (par exemple `fetchAll(PDO::FETCH_COLUMN)`).

# Structure générale

```php
class Club
{
    public static function getAll(): array
    {
        ...
    }

    public static function getById(string $id): array|false
    {
        ...
    }
}
```

# Utilisation de la classe `Select`

La majorité des consultations utilisent la classe `Select`, qui encapsule l'exécution des requêtes SQL.

## Retour de plusieurs lignes

```php
$sql = <<<SQL
    select id, nom
    from club
    order by nom
SQL;

$select = new Select();

return $select->getRows($sql);
```

La méthode `getRows()` retourne un tableau contenant toutes les lignes du résultat.

## Retour d'une seule ligne

Lorsqu'une seule ligne est attendue, on utilise `getRow()`.

```php
$sql = <<<SQL
    select id, nom
    from club
    where id = :id
SQL;

$select = new Select();

return $select->getRow($sql, [
    'id' => $id
]);
```

# Méthodes de consultation

Les méthodes sont généralement nommées selon leur objectif.

## `getAll()`

Retourne l'ensemble des enregistrements.

### Exemple

```php
public static function getAll(): array
{
    $sql = <<<SQL
        select id, nom
        from club
        order by nom
SQL;

    $select = new Select();

    return $select->getRows($sql);
}
```

## `getById()`

Retourne un enregistrement identifié par sa clé.

### Exemple

```php
public static function getById(string $id): array|false
{
    $sql = <<<SQL
        select id, nom
        from club
        where id = :id
SQL;

    $select = new Select();

    return $select->getRow($sql, [
        'id' => $id
    ]);
}
```

## Méthodes de recherche

Une méthode peut recevoir un ou plusieurs critères.

### Exemple

```php
public static function getLesCompetences(string $idBloc, string $idDomaine = '*'): array  {  
        $lesParametres['idBloc'] = $idBloc;  
        $sql = <<<SQL  
            select id, concat('C.', idBloc, '.', idDomaine, '.', idCompetence) as code, libelle            from competence            where idBloc = :idBlocSQL;  
        // prise en compte du domaine si l'idDomaine n'a pas la valeur par défaut  
        if ($idDomaine !== '*') {  
            $sql .= " and  idDomaine = :idDomaine";  
            $lesParametres['idDomaine'] = $idDomaine;  
        }  
  
        $select = new Select();  
        return $select->getRows($sql, $lesParametres);  
}
```

La requête SQL est alors construite dynamiquement selon les paramètres reçus.

## Méthodes retournant des listes

Ces méthodes servent principalement à alimenter des listes déroulantes.

### Exemple

```php
public static function getListe(): array  {  
        $sql = <<<SQL  
           select id, nom, (select count(*) from coureur where coureur.idClub = club.id) as nb
           from club
           order by nom;
SQL;  
        $select = new Select();
}
```

Les colonnes retournées sont généralement limitées à l'identifiant et au libellé.

## Méthodes de vérification

Une classe de consultation peut également proposer des méthodes retournant un booléen.

### Exemple

```php
public static function existe(int $id): bool  {  
        $sql = <<<SQL  
        select 1
        from competence        
        where id = :id;
SQL;  
 
        $select = new Select();  
  
        return $select->getRow($sql, ['id' => $id]) !== null;  
}
```

ou

```php
public static function appartientAuBloc(int $idCompetence, int $idBloc): bool
{
    $sql = <<<SQL  
        select 1        
        from competence        
        where id = :idCompetence          
        and idBloc = :idBloc;
SQL;  
    $select = new Select();  
    return $select->getRow($sql, ['idCompetence' => $idCompetence, 'idBloc'       => $idBloc]) !== null;
}
```

Ces méthodes évitent de dupliquer des requêtes de contrôle dans plusieurs services.

# Paramètres SQL

Toutes les requêtes utilisent des paramètres nommés.
### Exemple

```php
$sql = <<<SQL
    select id, libelle
    from competence
    where idBloc = :idBloc
SQL;

return $select->getRows($sql, ['idBloc' => $idBloc]);
```

Cette approche protège contre les injections SQL et facilite la lecture des requêtes.

# Traitements complémentaires

Une méthode de consultation peut enrichir les données récupérées avant de les retourner.

### Exemple

```php
foreach ($lesLignes as &$ligne) {

    $ligne['present'] =
        isset($ligne['fichier'])
        && is_file(self::DIR . $ligne['fichier']);
}
```

Dans la classe `Club`, une colonne supplémentaire (`present`) indique si le logo du club est effectivement présent sur le disque.

---

# Utilisation directe de `Database`

Dans certains cas, la classe `Database` est utilisée directement lorsque les méthodes de `Select` ne sont pas adaptées.

### Exemple

```php
$sql = "select id from club";

$db = Database::getInstance();

$cmd = $db->query($sql);

$lesIds = $cmd->fetchAll(PDO::FETCH_COLUMN);
```

Cette approche est notamment utilisée pour récupérer rapidement une liste de valeurs d'une seule colonne.

---

# Bonnes pratiques

Une classe métier de consultation :

- ne contient aucune opération d'écriture (`INSERT`, `UPDATE`, `DELETE`) ;
- regroupe toutes les requêtes relatives à un même domaine métier ;
- expose uniquement des méthodes statiques ;
- utilise des paramètres SQL nommés ;
- retourne des tableaux simples (`array`) ou des booléens (`bool`) selon le besoin ;
- peut effectuer un léger post-traitement des résultats avant de les retourner.

---

# Différences avec une classe dérivée de `Table`

| Classe de consultation | Classe dérivée de `Table` |
|-------------------------|---------------------------|
| Ne réalise que des lectures | Réalise les opérations CRUD |
| N'hérite pas de `Table` | Hérite de `Table` |
| Méthodes statiques | Méthodes d'instance pour les opérations CRUD |
| Utilise `Select` | Hérite des traitements génériques de `Table` |
| Aucune validation métier d'écriture | Validation générique et règles métier avant écriture |
| Ne définit ni colonnes ni contraintes | Définit les colonnes, contraintes et règles métier |