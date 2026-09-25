## Objectifs

À la fin de ce chapitre, vous serez capable de :

- comprendre comment PDO récupère les résultats d'une requête SQL ;
- utiliser les différentes méthodes de récupération proposées par `PDOStatement` ;
- choisir le mode `FETCH` le plus adapté à chaque situation ;
- comparer `fetch()` et `fetchAll()` selon le volume de données ;
- optimiser la consommation mémoire lors de la lecture de grands jeux de résultats.

---

# 6.1 Le jeu de résultats

Lorsqu'une requête `SELECT` est exécutée, le SGBDR retourne un **jeu de résultats** (_Result Set_).

Ce jeu de résultats est représenté en PHP par un objet `PDOStatement`.

L'objet `PDOStatement` ne contient pas directement les données sous forme de tableaux PHP. Il représente un **curseur** positionné sur le résultat de la requête.

---

# 6.2 Le curseur

On peut imaginer le curseur comme un pointeur placé avant la première ligne du résultat.

Exemple :

```
id   nom

1    Martin

2    Dupont

3    Durand
```

Avant le premier appel à `fetch()` :

```
↓

id   nom

1    Martin

2    Dupont

3    Durand
```

Après un premier `fetch()` :

```
id   nom

1    Martin   ← ligne lue

↓

2    Dupont

3    Durand
```

Après un second :

```
id   nom

1    Martin

2    Dupont ← ligne lue

↓

3    Durand
```

Lorsque toutes les lignes ont été lues :

```
false
```

est retourné.

---

# 6.3 Les différentes méthodes de récupération

PDO propose plusieurs méthodes.

|Méthode|Retour|
|---|---|
|`fetch()`|une ligne|
|`fetchAll()`|toutes les lignes|
|`fetchColumn()`|une seule valeur|
|`fetchObject()`|un objet|
|`fetchAll(PDO::FETCH_CLASS)`|collection d'objets|
|`fetchAll(PDO::FETCH_COLUMN)`|une colonne|
|`fetchAll(PDO::FETCH_KEY_PAIR)`|tableau clé → valeur|

Le choix dépend du résultat attendu.

---

# 6.4 `fetch()`

`fetch()` lit **une seule ligne**.

```php
$cmd = $db->query("SELECT id, nom FROM utilisateur");

$ligne = $cmd->fetch();
```

Résultat :

```
[
    "id"=>1,
    "nom"=>"Martin"
]
```

Le curseur avance automatiquement.

Le deuxième appel :

```php
$ligne = $cmd->fetch();
```

retourne :

```
[
    "id"=>2,
    "nom"=>"Dupont"
]
```

---

# 6.5 Parcourir un résultat avec `fetch()`

La lecture ligne par ligne est très fréquente.

```php
while ($ligne = $cmd->fetch()) {
    echo $ligne["nom"];
}
```

Le `while` s'arrête automatiquement lorsque `fetch()` retourne `false`.

Cette approche est très économique en mémoire.

# 6.6 `fetchAll()`

Contrairement à `fetch()`,

`fetchAll()` charge immédiatement **tout le résultat**.

```php
$lesUtilisateurs = $cmd->fetchAll();
```

Résultat :

```
[
    [
        "id"=>1,
        "nom"=>"Martin"
    ],

    [
        "id"=>2,
        "nom"=>"Dupont"
    ],

    [
        "id"=>3,
        "nom"=>"Durand"
    ]
]
```

---

# 6.7 Différence entre `fetch()` et `fetchAll()`

Supposons une table contenant :

```
500 000 utilisateurs
```

Avec `fetchAll()` :

```
Mémoire PHP

█████████████████████████████████████
500 000 lignes
```

Avec `fetch()` :

```
Mémoire PHP

█
1 ligne
```

Le gain mémoire est énorme.

# 6.8 Quand utiliser `fetch()` ?

Utiliser `fetch()` lorsque :

- une seule ligne est attendue ;
- les données sont nombreuses ;
- les lignes sont traitées une par une.

Exemple :

- export CSV
- génération PDF
- traitement statistique
- migration de données

---

# 6.9 Quand utiliser `fetchAll()` ?

Utiliser `fetchAll()` lorsque :

- quelques dizaines ou centaines de lignes sont attendues ;
- les données seront envoyées en JSON ;
- plusieurs parcours du tableau sont nécessaires.

Exemple :

```
echo json_encode(
    $cmd->fetchAll()
);
```

---

# 6.10 `fetchColumn()`

Cette méthode retourne uniquement une colonne.

Exemple :

```
SELECT COUNT(*)
FROM utilisateur
```

```
$total = $cmd->fetchColumn();
```

Retour :

```
125
```

---

Autre exemple :

```
SELECT nom,
       prenom
FROM utilisateur
```

```
echo $cmd->fetchColumn(1);
```

retourne :

```
Pierre
```

---

# 6.11 `fetchObject()`

Cette méthode construit un objet.

```
$utilisateur = $cmd->fetchObject();
```

Au lieu de :

```
$ligne["nom"]
```

on écrit

```
$utilisateur->nom
```

L'objet retourné est une instance de `stdClass`.

---

# 6.12 Les modes FETCH

Les méthodes `fetch()` et `fetchAll()` acceptent un mode de récupération.

```
$cmd->fetch(PDO::FETCH_ASSOC);
```

---

## FETCH_ASSOC

Le plus utilisé.

```
[
    "id"=>1,
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
    0=>1,
    1=>"Martin"
]
```

Accès :

```
$ligne[1];
```

---

## FETCH_BOTH

Combine les deux.

```
[
    0=>1,

    "id"=>1,

    1=>"Martin",

    "nom"=>"Martin"
]
```

---

## FETCH_OBJ

Retourne un objet.

```
$ligne->nom;
```

---

# 6.13 Les modes avancés

PDO propose également des modes moins connus.

|Mode|Description|
|---|---|
|FETCH_CLASS|construit des objets|
|FETCH_COLUMN|retourne une seule colonne|
|FETCH_KEY_PAIR|tableau clé → valeur|
|FETCH_GROUP|groupe les lignes|
|FETCH_UNIQUE|indexation par clé|
|FETCH_FUNC|appelle une fonction|

Ces modes seront étudiés dans un chapitre avancé consacré aux techniques d'hydratation des objets.

---

# 6.14 Itérer directement sur un `PDOStatement`

Un `PDOStatement` est parcourable.

On peut écrire :

```
foreach ($cmd as $ligne) {

    echo $ligne["nom"];

}
```

PDO appelle implicitement `fetch()`.

Cette écriture est élégante et très utilisée.

---

# 6.15 Fermer le curseur

Lorsque toutes les données ont été lues, il est recommandé de fermer le curseur.

```
$cmd->closeCursor();
```

Cela libère immédiatement les ressources utilisées par le SGBDR.

Même si PHP ferme automatiquement les curseurs à la fin du script, un appel explicite est recommandé lorsque plusieurs requêtes sont exécutées successivement.

---

# 6.16 Résumé

|Situation|Méthode recommandée|
|---|---|
|Une ligne|`fetch()`|
|Plusieurs lignes|`fetchAll()`|
|Une valeur|`fetchColumn()`|
|Un objet|`fetchObject()`|
|Lecture progressive|`fetch()`|
|Export volumineux|`fetch()`|
|Réponse JSON|`fetchAll()`|

---

# Bonnes pratiques

- Utiliser `fetch()` lorsqu'une seule ligne est attendue.
- Utiliser `fetchColumn()` pour les fonctions d'agrégation (`COUNT`, `SUM`, `MAX`, etc.).
- Éviter `fetchAll()` sur des jeux de données très volumineux.
- Définir `PDO::FETCH_ASSOC` comme mode de récupération par défaut dans la configuration de `PDO`.
- Appeler `closeCursor()` lorsque plusieurs requêtes sont exécutées sur une même connexion.
- Réserver les modes de récupération avancés (`FETCH_CLASS`, `FETCH_GROUP`, etc.) aux besoins spécifiques.

---

# À retenir

- Un `PDOStatement` représente le résultat d'une requête SQL et s'appuie sur un **curseur** permettant de parcourir les lignes retournées.
- `fetch()` lit les résultats **ligne par ligne**, ce qui limite la consommation mémoire et convient aux traitements volumineux.
- `fetchAll()` charge **l'ensemble des lignes** en mémoire et facilite les traitements lorsque le volume de données est raisonnable.
- `fetchColumn()` est la méthode la plus adaptée lorsqu'une seule valeur est attendue.
- `fetchObject()` retourne directement un objet, tandis que les différents modes `FETCH` permettent d'adapter la représentation des résultats aux besoins de l'application.
- Le choix de la méthode de récupération influence directement les performances et l'utilisation mémoire de l'application.