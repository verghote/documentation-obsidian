## Objectifs

À la fin de ce chapitre, vous serez capable de :

- comprendre le rôle de la classe `PDOStatement` ;
- exécuter une requête préparée ;
- associer des paramètres à une requête SQL ;
- récupérer les résultats sous différentes formes ;
- choisir la méthode de récupération la plus adaptée ;
- gérer correctement le cycle de vie d'un objet `PDOStatement`.

---

# 4.1 Qu'est-ce qu'un `PDOStatement` ?

Lorsqu'une requête SQL est préparée (`prepare()`) ou exécutée (`query()`), PDO retourne un objet de type **`PDOStatement`**.

Cet objet représente une **requête SQL prête à être exécutée ou déjà exécutée**, ainsi que le jeu de résultats associé.

Diagramme non valide ou non pris en charge.

On peut considérer `PDOStatement` comme **l'objet qui encapsule une requête SQL**.

---

# 4.2 Cycle de vie d'un `PDOStatement`

Quel que soit le type de requête, le fonctionnement est toujours le même.

Certaines étapes peuvent être absentes :

- une requête `query()` ne nécessite pas de préparation ;
- une requête `UPDATE` ne retourne généralement aucun résultat.

---

# 4.3 Les principales méthodes

|Méthode|Retour|Description|
|---|---|---|
|`execute()`|bool|Exécute la requête|
|`bindValue()`|bool|Associe une valeur|
|`bindParam()`|bool|Associe une variable|
|`fetch()`|array, object ou false|Retourne une ligne|
|`fetchAll()`|array|Retourne toutes les lignes|
|`fetchColumn()`|mixed|Retourne une seule valeur|
|`fetchObject()`|object|Retourne un objet|
|`rowCount()`|int|Nombre de lignes affectées|
|`closeCursor()`|bool|Libère les ressources|

Ces méthodes couvrent la quasi-totalité des besoins courants.

---

# 4.4 `execute()`

`execute()` lance l'exécution de la requête préparée.

Exemple :

```
$sql = <<<SQL
SELECT *
FROM utilisateur
WHERE id = :id
SQL;

$stmt = $db->prepare($sql);

$stmt->bindValue("id", 15);

$stmt->execute();
```

À partir de ce moment, les résultats peuvent être récupérés.

---

## Passage des paramètres directement

Il est souvent plus simple de transmettre les paramètres dans `execute()`.

```
$stmt->execute([
    "id" => 15
]);
```

Cette écriture est aujourd'hui la plus utilisée.

---

# 4.5 `bindValue()`

`bindValue()` associe une **valeur** à un paramètre.

```
$stmt->bindValue(
    "nom",
    $nom
);
```

La valeur est copiée immédiatement.

Même si `$nom` change ensuite, la requête conservera la valeur initiale.

```
$nom = "Martin";

$stmt->bindValue("nom", $nom);

$nom = "Dupont";

$stmt->execute();
```

La requête utilisera :

```
Martin
```

---

# 4.6 `bindParam()`

`bindParam()` fonctionne différemment.

Il associe une **variable**.

```
$stmt->bindParam(
    "nom",
    $nom
);
```

La valeur est lue uniquement au moment du `execute()`.

```
$nom = "Martin";

$stmt->bindParam("nom", $nom);

$nom = "Dupont";

$stmt->execute();
```

La requête utilisera :

```
Dupont
```

---

# 4.7 `bindValue()` ou `bindParam()` ?

Dans la majorité des applications modernes :

**`bindValue()` est préférable.**

Pourquoi ?

- plus simple ;
- plus lisible ;
- moins de comportements implicites.

`bindParam()` est surtout utile lorsqu'une même requête est exécutée plusieurs fois avec une variable qui change.

Exemple :

```
foreach ($liste as $nom) {

    $stmt->execute([
        "nom" => $nom
    ]);

}
```

Dans ce cas, `execute(array)` est encore plus simple que `bindParam()`.

---

# 4.8 Les types de paramètres

Par défaut, PDO déduit le type.

Il est néanmoins possible de le préciser.

```
$stmt->bindValue(
    "id",
    $id,
    PDO::PARAM_INT
);
```

Les principales constantes sont :

|Constante|Type|
|---|---|
|`PDO::PARAM_INT`|entier|
|`PDO::PARAM_STR`|chaîne|
|`PDO::PARAM_BOOL`|booléen|
|`PDO::PARAM_NULL`|NULL|
|`PDO::PARAM_LOB`|données binaires|

---

# 4.9 `fetch()`

`fetch()` récupère **une seule ligne**.

```
$stmt->execute();

$ligne = $stmt->fetch();
```

Si aucune ligne n'existe :

```
false
```

est retourné.

---

## Cas d'utilisation

- recherche par identifiant ;
- authentification ;
- consultation d'une fiche.

Exemple :

```
$sql = <<<SQL
SELECT *
FROM utilisateur
WHERE id = :id
SQL;

$stmt = $db->prepare($sql);

$stmt->execute([
    "id" => 8
]);

$utilisateur = $stmt->fetch();
```

---

# 4.10 `fetchAll()`

Retourne toutes les lignes.

```
$utilisateurs = $stmt->fetchAll();
```

Exemple :

```
SELECT *
FROM utilisateur
```

Résultat :

```
[
    [...],
    [...],
    [...]
]
```

---

## Quand utiliser `fetchAll()` ?

Lorsque :

- le nombre de lignes reste raisonnable ;
- toutes les données doivent être envoyées au client ;
- on souhaite parcourir plusieurs fois les résultats.

---

## Quand l'éviter ?

Si la requête retourne :

- plusieurs centaines de milliers de lignes ;
- plusieurs millions de lignes.

Dans ce cas, `fetch()` sera beaucoup plus économique.

---

# 4.11 `fetchColumn()`

Retourne une seule colonne.

Exemple :

```
SELECT COUNT(*)
FROM utilisateur
```

```
$stmt->execute();

$total = $stmt->fetchColumn();
```

Très utile pour :

- `COUNT()`
- `SUM()`
- `AVG()`
- `MAX()`
- `MIN()`

---

## Choisir une autre colonne

```
$stmt->fetchColumn(2);
```

retourne la troisième colonne.

---

# 4.12 `fetchObject()`

Retourne la ligne sous forme d'objet.

```
$utilisateur = $stmt->fetchObject();
```

Accès :

```
echo $utilisateur->nom;
```

au lieu de

```
echo $utilisateur["nom"];
```

---

# 4.13 Les différents modes FETCH

PDO peut construire plusieurs représentations.

## FETCH_ASSOC

```
[
    "id"=>12,
    "nom"=>"Martin"
]
```

Accès :

```
$ligne["nom"];
```

---

## FETCH_NUM

```
[
    0=>12,
    1=>"Martin"
]
```

Accès :

```
$ligne[1];
```

---

## FETCH_OBJ

```
stdClass
```

Accès :

```
$ligne->nom;
```

---

## FETCH_BOTH

```
[
    0=>12,
    "id"=>12
]
```

Les deux accès sont possibles.

---

> Les autres modes (`FETCH_CLASS`, `FETCH_GROUP`, `FETCH_KEY_PAIR`, etc.) seront étudiés dans un chapitre consacré aux modes de récupération avancés.

---

# 4.14 `rowCount()`

Retourne le nombre de lignes affectées.

Exemple :

```
$stmt->execute();

echo $stmt->rowCount();
```

Pour :

```
UPDATE utilisateur
SET actif = 0
```

on obtient par exemple :

```
18
```

---

### Attention

Avec un `SELECT`, `rowCount()` **n'est pas fiable sur tous les SGBDR**.

Il ne faut donc pas écrire :

```
if ($stmt->rowCount() > 0)
```

pour tester l'existence d'une ligne.

Préférer :

```
$ligne = $stmt->fetch();

if ($ligne !== false) {

}
```

---

# 4.15 `closeCursor()`

Libère les ressources associées à la requête.

```
$stmt->closeCursor();
```

Cette méthode est recommandée lorsque plusieurs requêtes sont exécutées successivement sur la même connexion.

PHP ferme automatiquement les curseurs en fin de script, mais un appel explicite améliore la gestion des ressources.

---

# 4.16 Exemple complet

```
$db = Database::getInstance();

$sql = <<<SQL
SELECT id,
       nom,
       prenom
FROM utilisateur
WHERE id = :id
SQL;

$stmt = $db->prepare($sql);

$stmt->execute([
    "id" => 12
]);

$utilisateur = $stmt->fetch();

$stmt->closeCursor();

return $utilisateur;
```

---

# 4.17 Tableau récapitulatif

|Méthode|Utilisation|
|---|---|
|`execute()`|Exécute la requête|
|`bindValue()`|Associe une valeur|
|`bindParam()`|Associe une variable|
|`fetch()`|Une ligne|
|`fetchAll()`|Toutes les lignes|
|`fetchColumn()`|Une valeur|
|`fetchObject()`|Une ligne sous forme d'objet|
|`rowCount()`|Nombre de lignes affectées|
|`closeCursor()`|Libère les ressources|

---

# Bonnes pratiques

- Privilégier `execute(array)` lorsque les paramètres sont peu nombreux.
- Utiliser `bindValue()` lorsque le type doit être précisé (`PARAM_INT`, `PARAM_NULL`, etc.).
- Réserver `bindParam()` aux cas où une variable doit être réutilisée entre plusieurs exécutions.
- Utiliser `fetch()` lorsqu'une seule ligne est attendue.
- Utiliser `fetchColumn()` lorsqu'une seule valeur est attendue.
- Éviter `fetchAll()` sur des jeux de données volumineux.
- Appeler `closeCursor()` avant d'exécuter une nouvelle requête importante sur la même connexion.

---

# À retenir

- Un objet **`PDOStatement`** représente une requête SQL préparée ou exécutée.
- La méthode `execute()` déclenche l'exécution de la requête.
- Les paramètres peuvent être transmis avec `execute(array)`, `bindValue()` ou `bindParam()`.
- `fetch()`, `fetchAll()`, `fetchColumn()` et `fetchObject()` permettent de récupérer les résultats sous différentes formes.
- Le choix de la méthode de récupération dépend du nombre de lignes et du format attendu.
- La fermeture explicite du curseur avec `closeCursor()` est une bonne pratique, notamment lorsque plusieurs requêtes sont exécutées successivement.