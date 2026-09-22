## Objectifs

À la fin de ce chapitre, vous serez capable de :

- réaliser des requêtes de consultation avec PDO ;
- choisir entre `query()` et `prepare()` ;
- récupérer une ou plusieurs lignes ;
- utiliser des paramètres dans une requête `SELECT` ;
- organiser les consultations dans des classes métiers ;
- préparer les données avant leur envoi au client.

---

# 7.1 Introduction

Les requêtes de consultation permettent de récupérer des informations stockées dans une base de données.

En SQL, elles utilisent principalement l'instruction :

```
SELECT
```

Exemple :

```
SELECT nom, prenom
FROM utilisateur;
```

Avec PDO, deux méthodes principales permettent d'exécuter une consultation :

- `query()` pour une requête sans paramètre ;
- `prepare()` puis `execute()` pour une requête paramétrée.

---

# 7.2 Utiliser `query()` pour une requête simple

La méthode `query()` est adaptée lorsqu'aucune donnée dynamique n'est intégrée dans la requête.

Exemple :

```
$db = Database::getInstance();

$sql = <<<SQL
SELECT id,
       nom,
       prenom
FROM utilisateur
ORDER BY nom
SQL;

$stmt = $db->query($sql);

$utilisateurs = $stmt->fetchAll(PDO::FETCH_ASSOC);
```

Le résultat obtenu :

```
[
    [
        "id" => 1,
        "nom" => "Martin",
        "prenom" => "Paul"
    ],
    [
        "id" => 2,
        "nom" => "Dupont",
        "prenom" => "Julie"
    ]
]
```

---

# 7.3 Pourquoi ne pas utiliser `query()` avec des variables ?

Une erreur fréquente consiste à écrire :

```
$sql = "
SELECT *
FROM utilisateur
WHERE nom = '$nom'
";
```

Cette écriture est dangereuse.

La valeur provenant de l'application est directement injectée dans le code SQL.

Elle peut provoquer :

- une injection SQL ;
- des erreurs avec certains caractères (`'`) ;
- des comportements inattendus.

La bonne solution est d'utiliser une requête préparée.

---

# 7.4 Requête préparée avec un paramètre

Exemple : récupérer les utilisateurs d'une classe.

Requête SQL :

```
SELECT nom,
       prenom
FROM utilisateur
WHERE idClasse = :idClasse
```

Code PHP :

```
$db = Database::getInstance();

$sql = <<<SQL
SELECT nom,
       prenom
FROM utilisateur
WHERE idClasse = :idClasse
SQL;

$stmt = $db->prepare($sql);

$stmt->execute([
    "idClasse" => $idClasse
]);

return $stmt->fetchAll(PDO::FETCH_ASSOC);
```

---

# 7.5 Consultation d'une seule ligne

Lorsqu'une requête retourne une seule ligne, on utilise généralement `fetch()`.

Exemple :

Recherche d'un utilisateur par son identifiant.

```
$sql = <<<SQL
SELECT id,
       nom,
       prenom,
       email
FROM utilisateur
WHERE id = :id
SQL;

$stmt = $db->prepare($sql);

$stmt->execute([
    "id" => $id
]);

$utilisateur = $stmt->fetch(PDO::FETCH_ASSOC);
```

Résultat :

```
[
    "id" => 15,
    "nom" => "Martin",
    "prenom" => "Paul",
    "email" => "paul@test.fr"
]
```

---

# 7.6 Tester si un enregistrement existe

Il est fréquent de devoir vérifier l'existence d'une ligne.

Exemple :

```
$stmt->execute([
    "id" => $id
]);

$ligne = $stmt->fetch();

if ($ligne !== false) {

    // l'utilisateur existe

}
```

Il ne faut pas utiliser :

```
if ($stmt->rowCount() > 0)
```

car `rowCount()` n'est pas fiable pour les requêtes `SELECT` selon les systèmes de gestion de base de données.

---

# 7.7 Récupérer une seule valeur

Certaines requêtes retournent une seule information.

Exemple :

```
SELECT COUNT(*)
FROM utilisateur;
```

Code PHP :

```
$sql = "
SELECT COUNT(*)
FROM utilisateur
";

$stmt = $db->query($sql);

$total = $stmt->fetchColumn();
```

Résultat :

```
250
```

Autres exemples :

```
SELECT MAX(prix)
FROM produit;
```

```
SELECT email
FROM utilisateur
WHERE id = :id;
```

Dans tous ces cas :

```
fetchColumn()
```

est la méthode adaptée.

---

# 7.8 Recherche avec plusieurs critères

Exemple :

Recherche d'un compte avec le nom et le prénom.

```
$sql = <<<SQL
SELECT login,
       password
FROM compte
WHERE nom = :nom
AND prenom = :prenom
SQL;


$stmt = $db->prepare($sql);

$stmt->execute([
    "nom" => $nom,
    "prenom" => $prenom
]);

$compte = $stmt->fetch(PDO::FETCH_ASSOC);
```

---

# 7.9 Recherche avec une partie de texte

Les paramètres fonctionnent également avec `LIKE`.

Exemple :

Recherche des utilisateurs dont le nom commence par une lettre.

```
$sql = <<<SQL
SELECT nom,
       prenom
FROM utilisateur
WHERE nom LIKE :nom
SQL;


$stmt = $db->prepare($sql);

$stmt->execute([
    "nom" => $nom . "%"
]);
```

Le paramètre contient donc :

```
Mar%
```

qui correspondra à :

```
Martin
Marcel
Martinez
```

---

# 7.10 Trier dynamiquement les résultats

Une difficulté fréquente concerne le tri.

On ne peut pas écrire :

```
ORDER BY :colonne
```

car un paramètre ne peut remplacer qu'une valeur.

La solution consiste à contrôler la valeur reçue avant de construire la requête.

Exemple :

```
$colonnesAutorisees = [
    "nom",
    "prenom",
    "dateNaissance"
];

if (!in_array($tri, $colonnesAutorisees)) {
    $tri = "nom";
}


$sql = "
SELECT *
FROM utilisateur
ORDER BY $tri
";
```

La liste blanche empêche l'injection SQL.

---

# 7.11 Préparer les données avant envoi au client

Le serveur peut enrichir les données avant transmission.

Exemple :

Une table contient une image :

```
photo = "martin.jpg"
```

On souhaite savoir si le fichier existe réellement.

```
$sql = "
SELECT id,
       nom,
       photo
FROM utilisateur
";

$stmt = $db->query($sql);

$utilisateurs = $stmt->fetchAll(PDO::FETCH_ASSOC);


$repertoire = RACINE . "/data/photos";


foreach ($utilisateurs as &$utilisateur) {

    $utilisateur["present"] =
        isset($utilisateur["photo"])
        && file_exists(
            "$repertoire/{$utilisateur["photo"]}"
        );
}
```

Le client reçoit alors :

```
{
    "id":12,
    "nom":"Martin",
    "photo":"martin.jpg",
    "present":true
}
```

---

# 7.12 Centraliser les consultations dans des classes métiers

Une bonne architecture évite de placer les requêtes SQL directement dans les scripts d'affichage.

Exemple :

```
class Utilisateur
{

    public static function getAll(): array
    {

        $sql = <<<SQL
        SELECT id,
               nom,
               prenom,
               email
        FROM utilisateur
        ORDER BY nom
        SQL;


        $db = Database::getInstance();

        $stmt = $db->query($sql);

        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }

}
```

Utilisation :

```
$utilisateurs = Utilisateur::getAll();
```

Avantages :

- séparation entre interface et données ;
- réutilisation du code ;
- maintenance facilitée ;
- meilleure organisation du projet.

---

# 7.13 Exemple complet : recherche d'étudiants

Classe métier :

```
class Etudiant
{

    public static function getParClasse(int $idClasse): array
    {

        $sql = <<<SQL
        SELECT nom,
               prenom,
               dateNaissance
        FROM etudiant
        WHERE idClasse = :idClasse
        ORDER BY nom
        SQL;


        $db = Database::getInstance();

        $stmt = $db->prepare($sql);

        $stmt->execute([
            "idClasse" => $idClasse
        ]);

        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }

}
```

Appel :

```
$liste = Etudiant::getParClasse(3);
```

---

# 7.14 Récupération et format JSON

Dans une application Web moderne, les résultats sont souvent transmis au client JavaScript.

Exemple :

```
$data = Etudiant::getParClasse(3);

echo json_encode($data);
```

Le navigateur reçoit :

```
[
 {
   "nom":"Martin",
   "prenom":"Paul"
 },
 {
   "nom":"Dupont",
   "prenom":"Julie"
 }
]
```

JavaScript peut directement exploiter cet objet.

---

# 7.15 Bonnes pratiques

- Utiliser `query()` uniquement lorsqu'aucun paramètre dynamique n'est nécessaire.
- Utiliser systématiquement `prepare()` pour les données provenant de l'application.
- Utiliser `fetch()` lorsqu'une seule ligne est attendue.
- Utiliser `fetchAll()` pour les listes destinées à l'affichage ou au JSON.
- Utiliser `fetchColumn()` lorsqu'une seule valeur est nécessaire.
- Ne jamais concaténer une donnée utilisateur dans une requête SQL.
- Centraliser les requêtes dans des classes métiers.
- Préparer les données avant leur envoi au client lorsque cela simplifie le traitement JavaScript.

---

# À retenir

- Une consultation PDO repose principalement sur `query()` ou `prepare()`.
- Les requêtes préparées doivent être utilisées dès qu'une valeur dynamique intervient.
- `fetch()` est adapté aux résultats uniques.
- `fetchAll()` est adapté aux listes.
- `fetchColumn()` est utilisé pour récupérer une seule valeur.
- Les classes métiers permettent de séparer la logique d'accès aux données du reste de l'application.
- PDO facilite la création d'applications sécurisées grâce aux requêtes paramétrées.