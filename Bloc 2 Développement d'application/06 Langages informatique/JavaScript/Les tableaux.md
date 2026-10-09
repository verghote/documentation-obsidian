# 1. Présentation

Un tableau est une structure de données permettant de stocker plusieurs valeurs dans une même variable.

Chaque élément du tableau est associé à un indice numérique correspondant à sa position dans le tableau.

Le premier élément possède toujours l'indice `0`.

Exemple :

```javascript
const lesJours = [
    "Lundi",
    "Mardi",
    "Mercredi",
    "Jeudi",
    "Vendredi",
    "Samedi",
    "Dimanche"
];
````

Représentation du tableau :

|Indice|Valeur|
|---|---|
|0|Lundi|
|1|Mardi|
|2|Mercredi|
|3|Jeudi|
|4|Vendredi|
|5|Samedi|
|6|Dimanche|

Ainsi :

```
lesJours[0]
```

retourne :

```
"Lundi"
```

## 1.1 Les tableaux en JavaScript

En JavaScript, les tableaux sont des **objets particuliers** appartenant à la classe native `Array`.

Contrairement à certains langages où la taille d'un tableau doit être définie à la création, un tableau JavaScript est **dynamique** :

- sa taille évolue automatiquement ;
- des éléments peuvent être ajoutés ou supprimés à tout moment ;
- il peut contenir des types de données différents.

Exemple :

```
const valeurs = [];

valeurs.push(10);
valeurs.push("Bonjour");
valeurs.push(true);
```

Le tableau contient maintenant :

```
[   10, "Bonjour", true]
```

## 1.2 Les différents types de tableaux

Un tableau JavaScript peut contenir :

- des valeurs simples ;
- des objets ;
- d'autres tableaux.
### Tableau de valeurs simples

Les éléments sont directement des valeurs :

```
const lesNotes = [
    15,
    18,
    12,
    20
];
```

Les valeurs peuvent être :

```
const valeurs = [
    10,          // nombre entier
    12.5,        // nombre réel
    "Bonjour",   // chaîne
    true,        // booléen
    null
];
```

### Tableau d'objets

Dans une application web, les tableaux contiennent très souvent des objets représentant des enregistrements provenant d'une base de données.

Exemple :

```javascript
const lesCategories = [
    {
        id: "EA",
        nom: "Éveil Athlétique",
        ageMin: 6,
        ageMax: 7
    },
    {
        id: "BE",
        nom: "Benjamin",
        ageMin: 11,
        ageMax: 13
    }
];
```

Chaque élément du tableau correspond alors à un enregistrement.

Dans cet exemple :

```javascript
lesCategories[1]
```

retourne :

```javascript
{
    id: "BE",
    nom: "Benjamin",
    ageMin: 11,
    ageMax: 13
}
```

Une propriété est ensuite accessible avec l'opérateur `.` :

```javascript
lesCategories[1].nom
```

retourne :

```
"Benjamin"
```

## 1.3 La propriété length

Tous les tableaux possèdent une propriété `length`.

Elle retourne le nombre d'éléments présents dans le tableau.

Exemple :

```javascript
const lesJours = [
    "Lundi",
    "Mardi",
    "Mercredi"
];

console.log(lesJours.length);
```

Résultat :

```
3
```

Cette propriété est souvent utilisée pour parcourir un tableau avec une boucle classique :

```javascript
for(let i = 0; i < lesJours.length; i++) {

    console.log(lesJours[i]);

}
```

# 2. Les tableaux d'éléments simples

Un tableau d'éléments simples contient directement les valeurs manipulées.

Exemple :

```javascript
const lesFruits = [
    "orange",
    "fraise",
    "raisin",
    "kiwi"
];
```

Chaque case contient directement une chaîne de caractères.

# 2.1 La déclaration

La déclaration d'un tableau peut se faire de plusieurs façons.

## Tableau vide

```javascript
let lesValeurs = [];
```

Le tableau existe mais ne contient aucun élément.

## Tableau initialisé

```javascript
let lesJours = [
    "Lundi",
    "Mardi",
    "Mercredi",
    "Jeudi",
    "Vendredi",
    "Samedi",
    "Dimanche"
];
```

## Déclaration avec le constructeur Array

Il est également possible d'utiliser :

```
let lesJours = new Array(
    "Lundi",
    "Mardi",
    "Mercredi"
);
```

Cependant cette notation est rarement utilisée.

La notation avec `[]` est recommandée car elle est plus simple et évite certaines ambiguïtés.

Exemple :

```javascript
new Array(5)
```

ne crée pas un tableau contenant la valeur `5`.

Elle crée un tableau vide de longueur 5 :

```javascript
[
    <5 cases vides>
]
```

# 2.2 Récupérer et modifier des valeurs

L'accès à un élément utilise l'opérateur d'indexation :

```javascript
tableau[indice]
```

Exemple :

```javascript
const lesJours = [
    "Lundi",
    "Mardi",
    "Mercredi"
];

console.log(lesJours[1]);
```

Résultat :

```
"Mardi"
```

## Modifier une valeur

La modification utilise également l'indice :

```javascript
lesJours[1] = "Tuesday";
```

Le tableau devient :

```javascript
[
    "Lundi",
    "Tuesday",
    "Mercredi"
]
```

## Ajouter directement une valeur

Il est possible d'affecter une valeur à un indice qui n'existe pas encore :

```javascript
const nombres = [1,2,3];

nombres[5] = 6;
```

Le résultat est :

```javascript
[
    1,
    2,
    3,
    <2 cases vides>,
    6
]
```

La taille du tableau devient :

```
nombres.length
```

soit :

```
6
```

Cette technique est rarement utilisée.  
Pour ajouter des éléments, on privilégie les méthodes `push`, `unshift` ou `splice`.

# 2.3 La consultation d'un tableau

La consultation d'un tableau consiste à parcourir ses éléments afin de lire ou traiter les valeurs contenues.

JavaScript propose plusieurs structures permettant d'effectuer un parcours.

## La boucle `for`

La boucle classique `for` permet de contrôler précisément l'indice utilisé.

Elle est adaptée lorsque l'on a besoin :

- de connaître la position de l'élément ;
- de modifier directement une case du tableau ;
- de parcourir une partie seulement du tableau.

Exemple :

```javascript
const lesJours = [
    "Lundi",
    "Mardi",
    "Mercredi",
    "Jeudi"
];

for (let i = 0; i < lesJours.length; i++) {
    console.log(lesJours[i]);
}
```

Résultat :

```
Lundi
Mardi
Mercredi
Jeudi
```

## La boucle `for...of`

La boucle `for...of` permet de parcourir directement les valeurs du tableau.

Elle est généralement préférable lorsque l'indice n'est pas nécessaire.

Exemple :

```javascript
for (const jour of lesJours) {

    console.log(jour);

}
```

Résultat :

```
Lundi
Mardi
Mercredi
Jeudi
```

Avantages :

- syntaxe plus simple ;
- évite la gestion manuelle des indices ;
- limite les erreurs liées aux bornes du tableau.

## La boucle `for...in`

La boucle `for...in` parcourt les propriétés d'un objet.

Elle peut fonctionner avec un tableau car les indices sont considérés comme des propriétés.

Exemple :

```javascript
for (const i in lesJours) {
    console.log(lesJours[i]);
}
```

Résultat :

```
Lundi
Mardi
Mercredi
Jeudi
```

Cependant cette boucle est déconseillée pour parcourir un tableau.

Pourquoi ?

Un tableau JavaScript étant un objet, il peut posséder d'autres propriétés que ses indices.

Exemple :

```javascript
lesJours.description = "Liste des jours";
```

La boucle `for...in` parcourra également cette propriété.

Pour un tableau, il faut privilégier :

- `for`
- `for...of`
- `forEach`

# 2.4 La recherche dans un tableau

La recherche consiste à déterminer si une valeur existe dans un tableau ou à récupérer sa position.

# La méthode `includes()`

La méthode `includes()` indique si un tableau contient une valeur.

Elle retourne :

- `true` si la valeur existe ;
- `false` sinon.

Syntaxe :

```javascript
tableau.includes(valeur)
```

Exemple :

```javascript
const lesJours = [
    "Lundi",
    "Mardi",
    "Mercredi"
];

if (lesJours.includes("Mardi")) {
    console.log("Le jour existe");
}
```

Résultat :

```
Le jour existe
```

## Comparaison sensible à la casse

La comparaison est stricte.

Exemple :

```javascript
lesJours.includes("mardi")
```

retourne :

```
false
```

car :

```
"Mardi" !== "mardi"
```

Lorsque les données proviennent d'une saisie utilisateur, il est souvent nécessaire de normaliser les valeurs.

Exemple :

```javascript
const recherche = "mardi";

const existe = lesJours.some(
    jour => jour.toLowerCase() === recherche.toLowerCase()
);
```

# La méthode `indexOf()`

La méthode `indexOf()` retourne la position du premier élément trouvé.

Elle retourne :

- l'indice de l'élément ;
- `-1` si l'élément n'existe pas.

Syntaxe :

```javascript
tableau.indexOf(valeur)
```

Exemple :

```javascript
const position = lesJours.indexOf("Mardi");

console.log(position);
```

Résultat :

```
1
```

Si la valeur n'existe pas :

```
lesJours.indexOf("Dimanche")
```

retourne :

```
-1
```

## Choix entre `includes()` et `indexOf()`

|Besoin|Méthode|
|---|---|
|Savoir si une valeur existe|`includes()`|
|Connaître la position|`indexOf()`|

Exemple :

Vérifier l'existence :

```javascript
if (lesJours.includes("Lundi")) {
    ...
}
```

Supprimer un élément :

```javascript
const index = lesJours.indexOf("Lundi");
if(index !== -1) {
    lesJours.splice(index,1);
}
```

# 2.5 La méthode `filter()`

La méthode `filter()` permet de créer un nouveau tableau contenant uniquement les éléments respectant une condition.

Syntaxe :

```javascript
const nouveauTableau = tableau.filter(
    element => condition
);
```

Le tableau d'origine n'est jamais modifié.

## Exemple simple

Tableau des jours :

```javascript
const lesJours = [
    "Lundi",
    "Mardi",
    "Mercredi",
    "Jeudi",
    "Vendredi",
    "Samedi",
    "Dimanche"
];
```

Recherche des jours contenant 5 lettres :

```javascript
const jours5Lettres = lesJours.filter(
    jour => jour.length === 5
);
```

Résultat :

```
[
    "Lundi",
    "Mardi",
    "Jeudi"
]
```

## Obtenir le nombre d'éléments filtrés

Il est possible d'utiliser la propriété `length` :

```javascript
const nombre = lesJours.filter(
    jour => jour.length === 5
).length;
```

## Attention aux performances

La méthode `filter()` est très lisible mais elle crée un nouveau tableau.

Exemple :

```javascript
const resultat = lesValeurs.filter(x => x > 10);
```

JavaScript doit :

1. parcourir tout le tableau ;
2. créer un nouveau tableau ;
3. copier les valeurs correspondantes.

Pour des traitements simples, une boucle peut être plus performante.

Exemple :

```javascript
let nombre = 0;
for (const valeur of lesValeurs) {
    if(valeur > 10) {
        nombre++;
    }
}
```

Ici aucun tableau intermédiaire n'est créé.

# 2.6 L'ajout d'éléments

Les tableaux JavaScript étant dynamiques, il est possible d'ajouter des éléments à tout moment.

# La méthode `push()`

Ajoute un ou plusieurs éléments à la fin du tableau.

Syntaxe :

```javascript
tableau.push(valeur);
```

Exemple :

```javascript
const nombres = [1,2,3];

nombres.push(4);
```

Résultat :

```
[
    1,
    2,
    3,
    4
]
```

La méthode retourne la nouvelle longueur du tableau.

```javascript
const taille = nombres.push(5);

console.log(taille);
```

Résultat :

```
5
```

# La méthode `unshift()`

Ajoute un ou plusieurs éléments au début du tableau.

Exemple :

```javascript
nombres.unshift(0);
```

Résultat :

```
[
    0,
    1,
    2,
    3,
    4
]
```

# La méthode `splice()`

La méthode `splice()` permet d'ajouter des éléments à une position donnée.

Syntaxe :

```javascript
tableau.splice(position, 0, valeur);
```

Paramètres :

|Paramètre|Rôle|
|---|---|
|position|indice d'insertion|
|0|nombre d'éléments supprimés|
|valeur|élément ajouté|

Exemple :

```javascript
const nombres = [1,2,3,4];

nombres.splice(2,0,10);
```

Résultat :

```
[
    1,
    2,
    10,
    3,
    4
]
```

# 2.7 La suppression d'éléments

# La méthode `pop()`

Supprime le dernier élément du tableau.

Exemple :

```javascript
const valeur = nombres.pop();
```

Avant :

```
[1,2,3,4]
```

Après :

```
[1,2,3]
```

La méthode retourne l'élément supprimé.

# La méthode `shift()`

Supprime le premier élément.

Exemple :

```javascript
nombres.shift();
```

Avant :

```
[1,2,3,4]
```

Après :

```
[2,3,4]
```

# La méthode `splice()` pour supprimer

Syntaxe :

```javascript
tableau.splice(position, nombreElements);
```

Exemple :

Supprimer l'élément d'indice 2 :

```
nombres.splice(2,1);
```

Supprimer deux éléments à partir de l'indice 2 :

```javascript
nombres.splice(2,2);
```
## Suppression après recherche

Le cas le plus fréquent consiste à rechercher la position puis supprimer.

Exemple :

```javascript
const index = lesJours.indexOf("Mardi");
if(index !== -1) {
    lesJours.splice(index,1);
}
```

# 2.8 Vider un tableau

La solution la plus simple consiste à réaffecter un tableau vide.

```javascript
lesJours = [];
```

Le tableau précédent est remplacé par un nouveau tableau vide.

# 2.9 Synthèse des opérations de base

|Besoin|Méthode|
|---|---|
|Ajouter à la fin|`push()`|
|Ajouter au début|`unshift()`|
|Ajouter à une position|`splice()`|
|Supprimer le dernier|`pop()`|
|Supprimer le premier|`shift()`|
|Supprimer à une position|`splice()`|
|Tester une existence|`includes()`|
|Trouver une position|`indexOf()`|
|Filtrer|`filter()`|

# 3. Les méthodes d'extension de `Array`

Les tableaux JavaScript possèdent un ensemble de méthodes intégrées permettant de réaliser des traitements avancés sans utiliser systématiquement des boucles classiques.

Ces méthodes sont particulièrement adaptées aux applications métier car elles permettent de :

- rechercher un enregistrement ;
- vérifier une règle métier ;
- filtrer des données ;
- transformer des données ;
- calculer des informations synthétiques.

Elles utilisent généralement une **fonction de rappel** (_callback function_) ou une **fonction fléchée** (_arrow function_).

Exemple :

```
x => x.age >= 18
```

signifie :

> Pour chaque élément `x` du tableau, tester si sa propriété `age` est supérieure ou égale à 18.

Dans les exemples suivants, on utilisera le tableau de catégories suivant :

```javascript
const lesCategories = [
    {
        id: "EA",
        nom: "Éveil Athlétique",
        ageMin: 4,
        ageMax: 6
    },
    {
        id: "PO",
        nom: "Poussin",
        ageMin: 7,
        ageMax: 9
    },
    {
        id: "BE",
        nom: "Benjamin",
        ageMin: 10,
        ageMax: 11
    },
    {
        id: "MI",
        nom: "Minime",
        ageMin: 12,
        ageMax: 13
    }
];
```

# 3.1. La méthode `forEach()`

## Rôle

La méthode `forEach()` permet d'exécuter une action pour chaque élément d'un tableau.

Elle remplace généralement une boucle `for...of`.

Syntaxe :

```javascript
tableau.forEach(element => action);
```

## Exemple : affichage des catégories

Avec une boucle classique :

```javascript
for (const categorie of lesCategories) {
    console.log(categorie.nom);
}
```

Avec `forEach()` :

```javascript
lesCategories.forEach(categorie => {
    console.log(categorie.nom);
});
```

Résultat :

```
Éveil Athlétique
Poussin
Benjamin
Minime
```

## Exemple métier : construire une liste déroulante

```javascript
const liste = document.getElementById("idCategorie");

lesCategories.forEach(categorie => {
    liste.add(new Option(categorie.nom, categorie.id));
});
```

Chaque catégorie reçue du serveur est automatiquement ajoutée dans une liste HTML.

## Remarque

`forEach()` :

- ne retourne aucune valeur ;
- ne crée pas de nouveau tableau ;
- sert uniquement à réaliser une action.

Exemple incorrect :

```javascript
const noms = lesCategories.forEach(
    c => c.nom
);
```

`noms` vaut :

```
undefined
```

# 3.2. La méthode `some()`

## Rôle

La méthode `some()` permet de savoir si **au moins un élément** du tableau vérifie une condition.

Elle retourne :

- `true` si un élément correspond ;
- `false` sinon.

Syntaxe :

```javascript
tableau.some(element => condition)
```

## Exemple : vérifier l'existence d'un identifiant

Lors de l'ajout d'une catégorie, il faut vérifier que l'identifiant n'existe pas déjà.

```javascript
function idExiste() {
    return lesCategories.some(categorie => categorie.id === id.value);
}
```

Utilisation :

```javascript
if (idExiste()) {
    afficherSousLeChamp(id, "Cet identifiant existe déjà");
    return;
}
```

## Pourquoi utiliser `some()` ?

Une boucle classique devrait parcourir tout le tableau :

```javascript
let trouve = false;
for (const categorie of lesCategories) {
    if (categorie.id === id.value) {
        trouve = true;
    }
}
```

Avec `some()` :

```javascript
const trouve = lesCategories.some(c => c.id === id.value);
```

Le code est :

- plus court ;
- plus lisible ;
- plus proche de l'intention métier.

## Optimisation

`some()` arrête immédiatement son parcours dès qu'un élément est trouvé.

Pour une recherche d'existence, il est donc plus adapté qu'un `filter()` :

```javascript
// inutilement coûteux
lesCategories.filter(c => c.id === "BE").length > 0;
```

car `filter()` continue toujours jusqu'à la fin du tableau.

# 3.3. La méthode `every()`

## Rôle

La méthode `every()` permet de vérifier que **tous les éléments** respectent une condition.

Elle retourne :

- `true` si tous les éléments correspondent ;
- `false` dès qu'un élément ne correspond pas.

## Exemple métier : vérifier que toutes les catégories ont un intervalle valide

Règle :

```
ageMin < ageMax
```

Code :

```javascript
const categoriesValides = lesCategories.every(categorie => categorie.ageMin < categorie.ageMax);
```

Résultat :

```
true
```

## Exemple : contrôle avant validation globale

```javascript
if (!lesCategories.every(c => c.id)) {
    afficherToast("Une catégorie possède un identifiant vide");
}
```

## Différence entre `some()` et `every()`

| Méthode   | Question métier                                            |
| --------- | ---------------------------------------------------------- |
| `some()`  | Existe-t-il au moins un élément qui respecte cette règle ? |
| `every()` | Tous les éléments respectent-ils cette règle ?             |

Exemples :

```
// Existe-t-il une catégorie Benjamin ?
lesCategories.some(c => c.id === "BE");
```

```
// Toutes les catégories ont-elles un âge maximum ?
lesCategories.every(c => c.ageMax);
```

# 3.4. La méthode `find()`

## Rôle

La méthode `find()` recherche le **premier élément** correspondant à une condition.

Elle retourne :

- l'objet trouvé ;
- `undefined` si aucun élément ne correspond.

Syntaxe :

```javascript
tableau.find(element => condition)
```

## Exemple : rechercher une catégorie

```javascript
const categorie = lesCategories.find(c => c.id === "BE");
```

Résultat :

```
{
    id:"BE",
    nom:"Benjamin",
    ageMin:10,
    ageMax:11
}
```

## Utilisation dans une règle métier

Calculer la catégorie d'un coureur :

```javascript
function rechercherCategorie(age) {
    return lesCategories.find(c => age >= c.ageMin && age <= c.ageMax);
}
```

Utilisation :

```javascript
const categorie = rechercherCategorie(11);
if (categorie) {
    console.log(categorie.nom);
}
```

Résultat :

```
Benjamin
```

# 3.5. La méthode `findIndex()`

## Rôle

`findIndex()` fonctionne comme `find()` mais retourne la position de l'élément.

Elle retourne :

- l'indice trouvé ;
- `-1` si aucun élément n'existe.

## Exemple : supprimer une catégorie localement

Recherche :

```javascript
const index = lesCategories.findIndex(
    c => c.id === "BE"
);
```

Résultat :

```
2
```

Suppression :

```javascript
if (index !== -1) {
    lesCategories.splice(index,1);
}
```

## Différence entre `find()` et `findIndex()`

| Méthode       | Retour         |
| ------------- | -------------- |
| `find()`      | l'objet trouvé |
| `findIndex()` | sa position    |

Exemple :

```javascript
const categorie = lesCategories.find(c => c.id === "BE");
```

Retour :

```
{
 id:"BE",
 nom:"Benjamin"
}
```

```javascript
const index = lesCategories.findIndex(c => c.id === "BE");
```

Retour :

```
2
```

# 3.6. La méthode `filter()`

## Rôle

`filter()` retourne un nouveau tableau contenant uniquement les éléments qui vérifient une condition.

Le tableau original n'est jamais modifié.

## Exemple métier : récupérer les catégories jeunes

```javascript
const categoriesJeunes =
    lesCategories.filter(c => c.ageMax < 12);
```

Résultat :

```
[
    {
        id:"EA"
    },
    {
        id:"PO"
    }
]
```

## Exemple : recherche multicritère

```javascript
const resultat =
    lesCategories.filter(c => c.ageMin >= 10 && c.ageMax <= 13);
```

# 3.7. La méthode `map()`

## Rôle

`map()` transforme chaque élément du tableau et retourne un nouveau tableau.

## Exemple : obtenir uniquement les identifiants

```javascript
const lesIds = lesCategories.map(c => c.id);
```

Résultat :

```
[
    "EA",
    "PO",
    "BE",
    "MI"
]
```

## Exemple métier : alimentation d'une liste HTML

```javascript
const options = lesCategories.map(c => ({valeur: c.id, texte: c.nom }));
```

Résultat :

```
[
 {
   valeur:"EA",
   texte:"Éveil Athlétique"
 }
]
```

# 3.8. La méthode `reduce()`

## Rôle

`reduce()` permet de transformer un tableau en une valeur unique.

Utilisations fréquentes :

- somme ;
- moyenne ;
- comptage ;
- regroupement.

Syntaxe :

```javascript
tableau.reduce((accumulateur, element) => traitement, valeurInitiale);
```

## Exemple : nombre total de catégories

```javascript
const nombreCategories = lesCategories.reduce((total, c) => total + 1, 0);
```

Résultat :

```
4
```

## Exemple métier : âge moyen minimum

```javascript
const moyenneAgeMin = lesCategories.reduce((somme,c) => somme + c.ageMin, 0) / lesCategories.length;
```

# 3.9. Choisir la bonne méthode

| Besoin métier                    | Méthode recommandée |
| -------------------------------- | ------------------- |
| Afficher tous les éléments       | `forEach()`         |
| Vérifier qu'une donnée existe    | `some()`            |
| Vérifier une règle générale      | `every()`           |
| Récupérer un objet précis        | `find()`            |
| Récupérer sa position            | `findIndex()`       |
| Obtenir une liste filtrée        | `filter()`          |
| Transformer les données          | `map()`             |
| Calculer une information globale | `reduce()`          |

## Exemple complet : contrôle d'ajout d'une catégorie

Lors de l'ajout d'une catégorie :

```javascript
function verifierCategorie() {
    if (lesCategories.some(c => c.id === id.value)) {
        return "Cet identifiant existe déjà";
    }

    if (lesCategories.some(c => c.nom === nom.value )) {
        return "Ce nom existe déjà";
    }

    if (ageMin.value >= ageMax.value) {
        return "L'âge minimum doit être inférieur à l'âge maximum";
    }

    if (lesCategories.some(c => ageMin.value <= c.ageMax && ageMax.value >= c.ageMin)) {
        return "Cet intervalle chevauche une catégorie existante";
    }
    return null;

}
```

Cette approche permet de reproduire côté client une partie des règles métier déjà présentes côté serveur, tout en conservant le contrôle serveur comme garantie finale.

# 4. Le tri des tableaux en JavaScript

Le tri des données est une opération fréquente dans les applications web :

- tri d'une liste de coureurs par nom ;
- tri des catégories par âge ;
- tri des annonces par date ;
- classement de résultats sportifs.

JavaScript propose plusieurs méthodes pour trier les tableaux.

# 4.1. La méthode `sort()`

## Rôle

La méthode `sort()` permet de trier un tableau.

Attention :

> `sort()` modifie directement le tableau d'origine.

Syntaxe :

```javascript
tableau.sort();
```

## Tri d'un tableau de chaînes

Exemple :

```javascript
const lesNoms = [
    "Martin",
    "Dupont",
    "Bernard",
    "Robert"
];

lesNoms.sort();
```

Résultat :

```
[
    "Bernard",
    "Dupont",
    "Martin",
    "Robert"
]
```

Le tri est effectué selon l'ordre Unicode.

# 4.2. Tri numérique

Attention :

Par défaut, `sort()` trie les valeurs comme des chaînes de caractères.

Exemple :

```javascript
const lesNotes = [8,15, 2, 20];

lesNotes.sort();
```

Résultat inattendu :

```
[
    15,
    2,
    20,
    8
]
```

Pourquoi ?

Car JavaScript compare :

```
"15"
"2"
"20"
"8"
```

## Solution : fournir une fonction de comparaison

Tri croissant :

```javascript
lesNotes.sort((a,b) => a-b);
```

Résultat :

```
[
    2,
    8,
    15,
    20
]
```

---

Tri décroissant :

```javascript
lesNotes.sort((a,b) => b-a);
```

Résultat :

```
[
    20,
    15,
    8,
    2
]
```

# 4.3. Tri d'un tableau d'objets

Dans les applications métier, les tableaux contiennent souvent des objets.

Exemple :

```javascript
const lesCoureurs = [
    {
        nom:"Martin",
        prenom:"Guy",
        age:45
    },
    {
        nom:"Dupont",
        prenom:"Julie",
        age:25
    },
    {
        nom:"Bernard",
        prenom:"Paul",
        age:35
    }
];
```

## Tri sur une propriété simple

Tri par nom :


```javascript
lesCoureurs.sort((a,b) => a.nom.localeCompare(b.nom));
```

Résultat :

```
Bernard
Dupont
Martin
```

## Pourquoi utiliser `localeCompare()` ?

La comparaison directe :

```javascript
a.nom > b.nom
```

ne tient pas correctement compte :

- des accents ;
- des règles linguistiques ;
- de la casse.

`localeCompare()` est conçu pour comparer des chaînes.

Exemple :

```javascript
"Émile".localeCompare("Emile")
```

gère mieux la comparaison humaine.

# 4.4. Tri sur plusieurs critères

Dans une application réelle, un tri utilise souvent plusieurs critères.

Exemple :

Trier les coureurs :

1. par nom ;
2. puis par prénom.

```javascript
lesCoureurs.sort((a,b) => a.nom.localeCompare(b.nom) || a.prenom.localeCompare(b.prenom));
```

## Explication

L'opérateur `||` permet d'enchaîner les comparaisons.

Si les noms sont différents :

```javascript
a.nom.localeCompare(b.nom)
```

retourne une valeur différente de `0`.

Le prénom n'est alors pas testé.

Si les noms sont identiques :

```javascript
0 || comparaisonPrenom
```

la deuxième comparaison est exécutée.

# 4.5. Tri des catégories par âge

Exemple métier :

```javascript
lesCategories.sort((a,b) =>a.ageMin - b.ageMin);
```

Résultat :

```
EA 4-6
PO 7-9
BE 10-11
MI 12-13
```

# 4.6. Inverser un tableau avec `reverse()`

La méthode `reverse()` inverse l'ordre des éléments.

Exemple :

```javascript
const valeurs = [1,2,3,4];

valeurs.reverse();
```

Résultat :

```
[4, 3, 2, 1 ]
```

Attention :

`reverse()` modifie le tableau original.

# 4.7. La méthode `toReversed()`

Les versions modernes de JavaScript proposent :

```javascript
toReversed()
```

Cette méthode :

- ne modifie pas le tableau original ;
- retourne une nouvelle copie inversée.

Exemple :

```javascript
const valeurs = [1,2,3,4];

const inverse = valeurs.toReversed();
```

Résultat :

```
inverse :[4,3,2,1]
```

Le tableau initial reste :

```
[1,2,3, 4]
```

# 4.8. La méthode `toSorted()`

De la même manière, `toSorted()` réalise un tri sans modifier le tableau.

Avec `sort()` :

```javascript
const copie = lesCoureurs.sort(comparaison);
```

Le tableau original est modifié.

Avec `toSorted()` :

```javascript
const classement = lesCoureurs.toSorted((a,b) => a.nom.localeCompare(b.nom));
```

Le tableau original reste inchangé.

# 4.9. Exemple complet : classement sportif

Données :

``` javascript
const resultats = [
    {
        nom:"Martin",
        temps:245
    },

    {
        nom:"Dupont",
        temps:230
    },

    {
        nom:"Bernard",
        temps:250
    }

];
```

Classement du meilleur temps au moins bon :

```javascript
const classement = resultats.toSorted((a,b)=> a.temps-b.temps);
```

Résultat :

```
Dupont 230
Martin 245
Bernard 250
```

# 4.10. Synthèse des méthodes de tri

| Méthode        | Modifie le tableau ? | Utilisation                                  |
| -------------- | -------------------- | -------------------------------------------- |
| `sort()`       | Oui                  | Tri classique                                |
| `toSorted()`   | Non                  | Tri avec conservation des données originales |
| `reverse()`    | Oui                  | Inversion directe                            |
| `toReversed()` | Non                  | Copie inversée                               |

# Bonnes pratiques

Dans une application web avec des données reçues du serveur :

✅ Utiliser `toSorted()` lorsque l'ordre original doit être conservé.

Exemple :

```javascript
const categoriesTriees = lesCategories.toSorted((a,b)=>a.ageMin-b.ageMin);
```

Ainsi :

- les données reçues du serveur restent intactes ;
- plusieurs affichages avec des tris différents sont possibles ;
- les traitements métier restent prévisibles.

# 5. Les tableaux associatifs en JavaScript

## 5.1. Présentation

Dans certains langages (comme PHP), un **tableau associatif** permet d'associer une valeur à une clé.

Exemple en PHP :

```javascript
$lesClasses = [
    "slam1" => 16,
    "sisr1" => 14,
    "dcg2" => 12
];
```

La valeur est récupérée à partir de sa clé :

```javascript
$lesClasses["slam1"];
```

Résultat :

```
16
```

En JavaScript, il n'existe pas réellement de tableau associatif.

La structure équivalente est un **objet JavaScript** dont les propriétés jouent le rôle des clés.

Exemple :

```javascript
const lesClasses = {
    "slam1": 16,
    "sisr1": 14,
    "dcg2": 12
};
```

Ici :

- `slam1`, `sisr1`, `dcg2` sont des propriétés de l'objet ;
- `16`, `14`, `12` sont les valeurs associées.

# 5.2. Différence entre tableau et objet associatif

Un tableau classique utilise des indices numériques :

```javascript
const lesPrenoms = ["Paul", "Julie", "Marc"];
```

Représentation :

| Indice | Valeur |
| ------ | ------ |
| 0      | Paul   |
| 1      | Julie  |
| 2      | Marc   |

Accès :

```javascript
lesPrenoms[1];
```

Résultat :

```
Julie
```

---

Un objet utilisé comme tableau associatif utilise des clés :

```javascript
const lesClasses = {

    "slam1":16,
    "sisr1":14

};
```

Représentation :

|Clé|Valeur|
|---|---|
|slam1|16|
|sisr1|14|

Accès :

```javascript
lesClasses["slam1"];
```

Résultat :

```
16
```

# 5.3. Déclaration

Deux syntaxes sont possibles.

## Syntaxe JSON

C'est la forme la plus utilisée :

```javascript
const lesClasses = {
    "slam1":16,
    "sisr1":14,
    "dcg2":12
};
```

## Syntaxe avec propriétés

Lorsque le nom de la clé respecte les règles JavaScript :

```javascript
const lesClasses = {
    slam1:16,
    sisr1:14,
    dcg2:12
};
```

Les deux écritures sont équivalentes.

# 5.4. Accès aux valeurs

Deux syntaxes existent.

## Avec les crochets

C'est la syntaxe recommandée.

```javascript
lesClasses["slam1"];
```

Résultat :

```
16
```

## Avec l'opérateur point

Possible uniquement si la clé est connue à l'avance :

```javascript
lesClasses.slam1;
```

Résultat :

```
16
```

# 5.5. Utilisation avec une variable

L'accès par crochets est indispensable lorsque la clé est contenue dans une variable.

Exemple :

```javascript
const formation = "slam1";

console.log(lesClasses[formation]);
```

Résultat :

```
16
```

Avec la notation point, cela serait impossible :

```javascript
lesClasses.formation;
```

Cette instruction chercherait une propriété appelée :

```
formation
```

et non :

```
slam1
```

# 5.6. Ajouter ou modifier une valeur

En JavaScript, l'ajout et la modification utilisent la même syntaxe.

## Ajouter une nouvelle clé

```javascript
lesClasses["slam2"] = 15;
```

L'objet devient :

```
{
    slam1:16,
    sisr1:14,
    dcg2:12,
    slam2:15
}
```

## Modifier une valeur existante

```javascript
lesClasses["slam1"] = 18;
```

Résultat :

```
{
    slam1:18,
    sisr1:14,
    dcg2:12
}
```

# 5.7. Parcourir un objet associatif

La boucle adaptée est `for...in`.

Syntaxe :

```javascript
for (const cle in objet) {
    traitement;
}
```

Exemple :

```javascript
for (const classe in lesClasses) {
    console.log(classe, lesClasses[classe] );
}
```

Résultat :

```
slam1 16
sisr1 14
dcg2 12
```

# 5.8. Vérifier l'existence d'une clé

La méthode correcte est l'opérateur `in`.

Exemple :

```javascript
if ("slam1" in lesClasses) {
    console.log("Classe trouvée");
}
```

Résultat :

```
Classe trouvée
```

## Attention aux valeurs particulières

Il ne faut pas tester simplement la valeur :

```javascript
if (lesClasses["dcg2"]) {

}
```

Car certaines valeurs sont considérées comme fausses en JavaScript :

```
false
0
""
null
undefined
```

Exemple :

```javascript
const valeurs = {
    classe1: 0,
    classe2: null,
    classe3: false
};
```

Test :

```javascript
if(valeurs.classe1)
```

retourne :

```
false
```

alors que la clé existe bien.

La bonne méthode :

```javascript
if ("classe1" in valeurs)
```

# 5.9. Supprimer une clé

La suppression s'effectue avec l'opérateur `delete`.

Exemple :

```javascript
delete lesClasses["slam1"];
```

Résultat :

```
{
    sisr1:14,
    dcg2:12
}
```

---

Autre syntaxe :

```javascript
delete lesClasses.slam1;
```

# 5.10. Récupérer les clés et les valeurs

JavaScript fournit plusieurs méthodes utiles.

## `Object.keys()`

Retourne un tableau contenant les clés.

Exemple :

```javascript
const classes =
    Object.keys(lesClasses);
```

Résultat :

```
[
    "slam1",
    "sisr1",
    "dcg2"
]
```

## `Object.values()`

Retourne un tableau contenant les valeurs.

```javascript
const effectifs =
    Object.values(lesClasses);
```

Résultat :

```
[
    16,
    14,
    12
]
```

---

## `Object.entries()`

Retourne un tableau contenant les couples clé/valeur.

```javascript
const elements = Object.entries(lesClasses);
```

Résultat :

```
[
    ["slam1",16],
    ["sisr1",14],
    ["dcg2",12]
]
```

---

Cette méthode permet d'utiliser ensuite les méthodes d'extension des tableaux.

Exemple :

```javascript
Object.entries(lesClasses).forEach(
    ([classe,effectif]) => {
        console.log(classe, effectif);
    }
);
```

# 5.11. Exemple métier : association identifiant / libellé

Les objets associatifs sont très utilisés pour stocker des correspondances.

Exemple :

```javascript
const lesSexes = { M:"Masculin", F:"Féminin"};
```

Utilisation :

```javascript
selectSexe.add(new Option(lesSexes.M, "M"));
```

# 5.12. Exemple métier : cache côté client

Supposons que le serveur transmette :

```javascript
const lesClubs = {

    "08001":"Amiens UC",
    "08002":"Abbeville AC",
    "08003":"Beauvais Oise"

};
```

Pour afficher le nom d'un club :

```javascript
const nomClub = lesClubs[idClub];
```

Avantage :

- accès immédiat par identifiant ;
- pas besoin de parcourir un tableau.

# 5.13. Tableau ou objet associatif : quel choix ?

|Besoin|Structure adaptée|
|---|---|
|Liste d'éléments à parcourir|Tableau `Array`|
|Recherche par position|Tableau|
|Recherche par identifiant|Objet associatif|
|Données venant d'une table SQL|Tableau d'objets|
|Correspondance clé → valeur|Objet|

# 5.14. Exemple avec les données serveur

Un serveur PHP peut envoyer :

```javascript
$lesSexes = ["M" => "Masculin", "F" => "Féminin"];

echo json_encode($lesSexes);
```

JavaScript reçoit :

```javascript
const lesSexes = {M:"Masculin", F:"Féminin"};
```

Utilisation :

```javascript
console.log(
    lesSexes["M"]
);
```

Résultat :

```
Masculin
```

# 5.15. À retenir

Un "tableau associatif" JavaScript est en réalité un **objet utilisé comme dictionnaire**.

Points essentiels :

✅ utiliser `[]` pour accéder aux clés dynamiques ;  
✅ utiliser `in` pour tester l'existence d'une clé ;  
✅ utiliser `for...in` pour parcourir les propriétés ;  
✅ utiliser `Object.keys()`, `Object.values()` et `Object.entries()` pour transformer un objet en tableau exploitable ;  
✅ ne pas utiliser les méthodes de tableau (`push`, `pop`, `length`) sur ces objets.

# Partie 6 — Copie des tableaux et gestion des références

## 6.1. Le comportement particulier des tableaux en JavaScript

En JavaScript, un tableau est un **objet**.

Cela signifie qu'une variable contenant un tableau ne contient pas directement les données du tableau, mais une **référence vers l'objet tableau stocké en mémoire**.

Conséquence importante : une simple affectation ne crée pas une copie.

Exemple :

```javascript
const lesNotes = [12, 15, 18];

const copie = lesNotes;

copie.push(20);

console.log(lesNotes);
```

Résultat :

```
[12, 15, 18, 20]
```

Le tableau `lesNotes` a été modifié alors que l'on pensait travailler sur `copie`.

Explication :

```
lesNotes ─────┐
              │
              ▼
        [12,15,18,20]
              ▲
              │
copie ────────┘
```

Les deux variables pointent vers **le même objet tableau**.

On parle de :

- référence partagée ;
- copie par référence ;
- aliasing.

# 6.2. Copier un tableau simple

Pour créer un nouveau tableau indépendant, il faut effectuer une copie.

## 6.2.1. L'opérateur de propagation `...`

La méthode la plus utilisée consiste à utiliser l'opérateur de propagation (_spread operator_).

Syntaxe :

```javascript
const nouveauTableau = [...ancienTableau];
```

Exemple :

```javascript
const lesNotes = [12, 15, 18];

const copie = [...lesNotes];

copie.push(20);

console.log(lesNotes);
console.log(copie);
```

Résultat :

```
lesNotes : [12,15,18]

copie : [12,15,18,20]
```

Les deux tableaux sont maintenant indépendants.

Schéma :

```
lesNotes
    |
    ▼
[12,15,18]


copie
    |
    ▼
[12,15,18,20]
```

## 6.2.2. Utilisation de `slice()`

La méthode `slice()` permet également de créer une copie.

Sans paramètre :

```javascript
const copie = lesNotes.slice();
```

Elle retourne un nouveau tableau contenant tous les éléments.

Exemple :

```javascript
const lesJours = [
    "Lundi",
    "Mardi",
    "Mercredi"
];

const copie = lesJours.slice();

copie.push("Jeudi");
```

Résultat :

```
lesJours :
[
"Lundi",
"Mardi",
"Mercredi"
]


copie :
[
"Lundi",
"Mardi",
"Mercredi",
"Jeudi"
]
```

## 6.2.3. Utilisation de `concat()`

La concaténation avec un tableau vide permet également d'obtenir une copie.

```javascript
const copie = [].concat(lesJours);
```

Exemple :

```javascript
const nombres = [1,2,3];

const copie = [].concat(nombres);

copie.push(4);

console.log(nombres);
```

Résultat :

```
[1,2,3]
```

## 6.2.4. Utilisation de `Array.from()`

La méthode `Array.from()` crée un nouveau tableau à partir d'un objet itérable.

Exemple :

```javascript
const valeurs = [10,20,30];

const copie = Array.from(valeurs);
```

Résultat :

```
valeurs ───► [10,20,30]

copie ─────► [10,20,30]
```

# 6.3. Les copies superficielles (_shallow copy_)

Les méthodes :

- `...`
- `slice()`
- `concat()`
- `Array.from()`

réalisent uniquement une **copie superficielle**.

Cela signifie que les éléments du tableau sont copiés, mais pas les objets contenus dans le tableau.

Exemple avec un tableau d'objets :

```javascript
const lesPersonnes = [
    {
        nom : "Martin",
        age : 18
    },
    {
        nom : "Dupont",
        age : 20
    }
];


const copie = [...lesPersonnes];
```

On obtient :

```
lesPersonnes
      |
      ▼
[
  ┌───────────────┐
  │ nom : Martin  │
  │ age : 18      │
  └───────────────┘
]


copie
      |
      ▼
[
  ┌───────────────┐
  │ nom : Martin  │
  │ age : 18      │
  └───────────────┘
]
```

Les tableaux sont différents, mais les objets internes sont partagés.

---

Modification :

```javascript
copie[0].age = 25;
```

Résultat :

```javascript
console.log(lesPersonnes[0].age);
```

Affiche :

```
25
```

Pourquoi ?

Parce que :

```
copie[0]
     |
     ▼
  objet Personne
     ▲
     |
lesPersonnes[0]
```

Les deux tableaux contiennent une référence vers le même objet.

# 6.4. Copie profonde (_deep copy_)

Lorsque le tableau contient des objets, il peut être nécessaire de créer une copie totalement indépendante.

On parle alors de **copie profonde**.

## 6.4.1. Utilisation de JSON.parse et JSON.stringify

Une solution classique consiste à convertir le tableau en JSON puis à le reconstruire.

```javascript
const copie = JSON.parse(
    JSON.stringify(lesPersonnes)
);
```

Fonctionnement :

```
Objet JavaScript
        |
        | JSON.stringify()
        ▼
Chaîne JSON
        |
        | JSON.parse()
        ▼
Nouvel objet JavaScript
```

Exemple :

```javascript
const personnes = [
    {
        nom:"Martin",
        age:18
    }
];

const copie = JSON.parse(JSON.stringify(personnes));

copie[0].age = 30;
```

Résultat :

```
console.log(personnes[0].age);
```

Affiche :

```
18
```

Le tableau original n'est pas modifié.

# 6.5. Limites de la copie JSON

La copie par JSON fonctionne uniquement avec des données compatibles JSON.

Elle ne conserve pas :

- les fonctions ;
- les objets `Date` ;
- les propriétés `undefined` ;
- certains objets JavaScript spécifiques.

Exemple :

```javascript
const personne = {
    nom : "Martin",
    dateNaissance : new Date()
};


const copie = JSON.parse(
    JSON.stringify(personne)
);
```

La propriété :

```
dateNaissance
```

devient une simple chaîne.

# 6.6. La copie dans un contexte métier

Dans une application web, la copie de tableau est très utilisée.

Exemple : modification d'une catégorie.

Le serveur transmet :

```javascript
const lesCategories = getData("lesCategories");
```

On souhaite modifier localement la liste avant validation.

Il faut éviter :

```javascript
const categoriesModifiees = lesCategories;
```

car toute modification affecterait directement les données originales.

Préférer :

```javascript
const categoriesModifiees = [...lesCategories];
```

Exemple :

```javascript
const categoriesModifiees = [...lesCategories];

categoriesModifiees.push({
    id : "M0",
    nom : "Master",
    ageMin : 35,
    ageMax : 40
});
```

La liste originale reste inchangée.

# 6.7. Choisir la bonne méthode de copie

|Situation|Méthode recommandée|
|---|---|
|Tableau simple de nombres ou chaînes|`[...tableau]`|
|Copie partielle d'un tableau|`slice()`|
|Création depuis un objet iterable|`Array.from()`|
|Tableau contenant des objets simples|`[...tableau]` si les objets ne sont pas modifiés|
|Tableau contenant des objets qui doivent être indépendants|copie profonde|
|Données complexes|bibliothèque spécialisée ou `structuredClone()`|

# 6.8. La méthode moderne `structuredClone()`

Les versions récentes de JavaScript proposent une méthode native permettant une copie profonde.

Syntaxe :

```javascript
const copie = structuredClone(original);
```

Exemple :

```javascript
const personnes = [
    {
        nom:"Martin",
        age:18
    }
];


const copie = structuredClone(personnes);

copie[0].age = 25;
```

Résultat :

```javascript
personnes[0].age
```

reste égal à :

```
18
```

Cette méthode est aujourd'hui préférable à :

```javascript
JSON.parse(JSON.stringify(objet))
```

lorsqu'elle est disponible.

# 6.9. Synthèse

|Opération|Résultat|
|---|---|
|`tab2 = tab1`|partage la même référence|
|`[...tab1]`|copie superficielle|
|`tab1.slice()`|copie superficielle|
|`Array.from(tab1)`|copie superficielle|
|`JSON.parse(JSON.stringify(tab1))`|copie profonde limitée|
|`structuredClone(tab1)`|copie profonde moderne|

Dans une application métier, la règle générale est :

- utiliser **`...`** pour dupliquer rapidement une liste ;
- utiliser **`findIndex()` + modification directe** lorsque l'on travaille sur une ligne précise ;
- utiliser **`structuredClone()`** lorsqu'une copie indépendante complète d'un ensemble de données est nécessaire avant traitement.

# Partie 7 — Les tableaux associatifs en JavaScript et les objets utilisés comme dictionnaires

## 7.1. Présentation

Dans certains langages comme PHP, un **tableau associatif** est une structure permettant d'associer une valeur à une clé.

Exemple en PHP :

```javascript
$lesCategories = [
    "M0" => "Master 0",
    "M1" => "Master 1",
    "M2" => "Master 2"
];

echo $lesCategories["M1"];
```

Résultat :

```
Master 1
```

La clé permet d'accéder directement à une valeur.

En JavaScript, il n'existe pas réellement de **tableau associatif**.

Les tableaux JavaScript (`Array`) utilisent uniquement des **indices numériques** :

```javascript
const jours = [
    "Lundi",
    "Mardi",
    "Mercredi"
];

console.log(jours[1]);
```

Résultat :

```
Mardi
```

Les indices sont automatiquement générés :

```
0 → Lundi
1 → Mardi
2 → Mercredi
```

Pour obtenir un fonctionnement proche d'un tableau associatif, JavaScript utilise un **objet**.

# 7.2. Utilisation d'un objet comme tableau associatif

Un objet JavaScript est constitué de :

- propriétés (ou clés) ;
- valeurs associées.

Syntaxe :

```javascript
const objet = {
    cle1 : valeur1,
    cle2 : valeur2
};
```

Exemple :

```javascript
const lesClasses = {
    "slam1" : 16,
    "sisr1" : 14,
    "slam2" : 15,
    "sisr2" : 13
};
```

On peut représenter cette structure ainsi :

|Clé|Valeur|
|---|---|
|slam1|16|
|sisr1|14|
|slam2|15|
|sisr2|13|

Accès à une valeur :

```
console.log(lesClasses["slam1"]);
```

Résultat :

```
16
```

# 7.3. Les deux syntaxes d'accès

Il existe deux façons d'accéder aux propriétés d'un objet.

## 7.3.1. La notation avec un point

```javascript
objet.propriete
```

Exemple :

```javascript
console.log(lesClasses.slam1);
```

Résultat :

```
16
```

Cette notation est simple et lisible.

## 7.3.2. La notation avec crochets

```javascript
objet["propriete"]
```

Exemple :

```javascript
console.log(lesClasses["slam1"]);
```

Résultat :

```
16
```

Cette notation est obligatoire lorsque le nom de la clé est contenu dans une variable.

Exemple :

```javascript
const classe = "slam1";

console.log(lesClasses[classe]);
```

Résultat :

```
16
```

# 7.4. Ajouter ou modifier une valeur

En JavaScript, il n'y a pas de distinction entre ajout et modification.

Si la clé existe :

```javascript
lesClasses["slam1"] = 18;
```

La valeur est modifiée.

Résultat :

```
{
    slam1 : 18,
    sisr1 : 14,
    slam2 : 15,
    sisr2 : 13
}
```

---

Si la clé n'existe pas :

```javascript
lesClasses["dcg1"] = 20;
```

La propriété est ajoutée.

Résultat :

```
{
    slam1 : 16,
    sisr1 : 14,
    slam2 : 15,
    sisr2 : 13,
    dcg1 : 20
}
```

# 7.5. Suppression d'une propriété

La suppression s'effectue avec l'opérateur `delete`.

Exemple :

```javascript
delete lesClasses["dcg1"];
```

La propriété disparaît de l'objet.

Autre syntaxe :

```javascript
delete lesClasses.dcg1;
```

Cependant la notation avec crochets est préférable car elle fonctionne également avec une variable :

```javascript
const classe = "dcg1";

delete lesClasses[classe];
```

# 7.6. Parcourir un objet associatif

Un objet n'étant pas un tableau, les méthodes :

```javascript
push()
pop()
filter()
map()
find()
```

ne sont pas disponibles.

La consultation utilise principalement la boucle `for...in`.

Exemple :

```javascript
for (const classe in lesClasses) {
    console.log(classe, lesClasses[classe]);
}
```

Résultat :

```
slam1 16
sisr1 14
slam2 15
sisr2 13
```

# 7.7. Tester l'existence d'une clé

Il faut distinguer :

- la clé inexistante ;
- une clé existante dont la valeur vaut `0`, `false` ou `null`.

Exemple :

```javascript
const valeurs = {
    a : 10,
    b : 0,
    c : false,
    d : null
};
```

Un test classique serait :

```javascript
if (valeurs["b"]) {
    console.log("Existe");
}
```

Mais le résultat est faux car :

```
Boolean(0)
```

vaut :

```
false
```

La clé existe pourtant.

## 7.7.1. Utilisation de l'opérateur `in`

La bonne méthode :

```javascript
if ("b" in valeurs) {
    console.log("La clé existe");
}
```

Résultat :

```
La clé existe
```

L'opérateur `in` vérifie la présence de la propriété indépendamment de sa valeur.

# 7.8. Récupérer les clés et les valeurs

JavaScript fournit plusieurs méthodes utiles.

## 7.8.1. `Object.keys()`

Retourne un tableau contenant les clés.

Exemple :

```javascript
const cles = Object.keys(lesClasses);
```

Résultat :

```
[
    "slam1",
    "sisr1",
    "slam2",
    "sisr2"
]
```

On peut ensuite utiliser les méthodes classiques des tableaux :

```javascript
Object.keys(lesClasses).forEach(cle => { console.log(cle); });
```

## 7.8.2. `Object.values()`

Retourne uniquement les valeurs.

Exemple :

```javascript
const effectifs = Object.values(lesClasses);
```

Résultat :

```
[16, 14, 15, 13 ]
```

Il devient possible d'utiliser les méthodes des tableaux :

```javascript
const total = Object.values(lesClasses)
    .reduce((somme, valeur) => somme + valeur, 0);
```

Résultat :

```
58
```

## 7.8.3. `Object.entries()`

Retourne un tableau contenant des couples :

```
[clé, valeur]
```

Exemple :

```javascript
const lignes = Object.entries(lesClasses);
```

Résultat :

```
[
    ["slam1",16],
    ["sisr1",14],
    ["slam2",15],
    ["sisr2",13]
]
```

Utilisation :

```javascript
for(const [classe, effectif] of Object.entries(lesClasses)) {
    console.log(classe, effectif);
}
```

# 7.9. Exemple métier : accès rapide aux catégories

Dans une application de gestion de coureurs, on peut recevoir :

```javascript
const lesCategories = [
    {
        id:"M0",
        nom:"Master 0"
    },
    {
        id:"M1",
        nom:"Master 1"
    }
];
```

Une recherche avec `find()` nécessite un parcours :

```javascript
const categorie = lesCategories.find(
    c => c.id === "M1"
);
```

Pour de nombreuses recherches, il peut être intéressant de créer un dictionnaire :

```javascript
const categoriesParId = {};
for(const categorie of lesCategories) {
    categoriesParId[categorie.id] = categorie;
}
```

On obtient :

```
{
    M0 : {
        id:"M0",
        nom:"Master 0"
    },

    M1 : {
        id:"M1",
        nom:"Master 1"
    }
}
```

La recherche devient immédiate :

```
const categorie = categoriesParId["M1"];
```

# 7.10. Tableau ou objet associatif : quel choix ?

|Besoin|Structure recommandée|
|---|---|
|Liste d'éléments à parcourir|`Array`|
|Filtrer, rechercher avec `find()`|`Array`|
|Ajouter/supprimer des éléments|`Array`|
|Accéder rapidement par identifiant|Objet dictionnaire|
|Associer une clé unique à une valeur|Objet|
|Utiliser `map`, `filter`, `reduce`|`Array`|

# 7.11. Alternative moderne : `Map`

JavaScript possède également une structure dédiée aux associations clé/valeur :

```javascript
const classes = new Map();

classes.set("slam1",16);
classes.set("sisr1",14);
```

Lecture :

```javascript
console.log(classes.get("slam1"));
```

Résultat :

```
16
```

Contrairement aux objets :

- les clés peuvent être de n'importe quel type ;
- l'ordre est conservé ;
- des méthodes spécifiques existent.

Exemple :

```javascript
classes.has("slam1");
```

Retourne :

```
true
```

Cependant dans une application web utilisant principalement JSON, les objets restent souvent privilégiés car ils sont directement compatibles avec les échanges serveur/client.

# 7.12. Synthèse

En JavaScript :

- un **Array** représente une liste ordonnée d'éléments ;
- un **Object** peut jouer le rôle d'un tableau associatif ;
- un objet n'utilise pas les méthodes d'extension de `Array` ;
- `Object.keys()`, `Object.values()` et `Object.entries()` permettent de transformer un objet en tableau exploitable ;
- `Map` est une structure spécialisée pour les associations clé/valeur.

Dans une application métier :

- utiliser un tableau pour représenter une collection d'enregistrements venant d'une base SQL ;
- utiliser un objet dictionnaire pour accélérer les recherches par identifiant ;
- choisir la structure en fonction de l'opération principale à réaliser.

# Partie 8 — Les tableaux et les échanges de données JSON entre serveur et client

## 8.1. Rôle des tableaux dans une application web

Dans une application web moderne, les tableaux JavaScript sont très souvent utilisés pour représenter des **collections de données provenant du serveur**.

Exemples :

- liste des catégories ;
- liste des clubs ;
- liste des coureurs ;
- liste des utilisateurs ;
- liste des produits.

Ces données sont généralement issues d'une base de données côté serveur (PHP + SQL), puis transmises au navigateur sous forme de **JSON**.

Le cycle général est :

```
Base de données
       |
       ▼
     PHP
       |
       | json_encode()
       ▼
      JSON
       |
       | transmission HTTP
       ▼
 Navigateur
       |
       | JSON.parse()
       ▼
 Tableau JavaScript
```

# 8.2. Conversion d'un tableau PHP en tableau JavaScript

## Exemple côté PHP

Supposons que le serveur possède un tableau de catégories :

```php
$lesCategories = [
    [
        "id" => "M0",
        "nom" => "Master 0",
        "ageMin" => 35,
        "ageMax" => 39
    ],
    [
        "id" => "M1",
        "nom" => "Master 1",
        "ageMin" => 40,
        "ageMax" => 49
    ]
];
```

Le tableau PHP contient :

- un tableau principal ;
- contenant plusieurs tableaux associatifs ;
- représentant des objets métier.

## Conversion en JSON

La fonction :

```php
json_encode()
```

transforme cette structure en JSON.

```php
$json = json_encode($lesCategories);
```

Résultat :

```
[
    {
        "id":"M0",
        "nom":"Master 0",
        "ageMin":35,
        "ageMax":39
    },
    {
        "id":"M1",
        "nom":"Master 1",
        "ageMin":40,
        "ageMax":49
    }
]
```

# 8.3. Réception côté JavaScript

Le navigateur reçoit une chaîne JSON.

Exemple :

```javascript
const json = `
[
 {
    "id":"M0",
    "nom":"Master 0",
    "ageMin":35,
    "ageMax":39
 }
]
`;
```

Cette chaîne doit être convertie en objet JavaScript.

On utilise :

```javascript
JSON.parse()
```

Exemple :

```javascript
const lesCategories = JSON.parse(json);
```

Résultat :

```
[
    {
        id:"M0",
        nom:"Master 0",
        ageMin:35,
        ageMax:39
    }
]
```

La variable `lesCategories` est maintenant un véritable tableau JavaScript.

# 8.4. Exploitation d'un tableau reçu du serveur

Une fois reçu, le tableau peut utiliser toutes les méthodes d'extension de `Array`.

Exemple :

```javascript
const categorie = lesCategories.find(c => c.id === "M1");
```

Résultat :

```
{
    id:"M1",
    nom:"Master 1",
    ageMin:40,
    ageMax:49
}
```

## Exemple : alimentation d'une liste déroulante

Données reçues :

```
const lesClubs = [
    {
        id:"001",
        nom:"Amiens UC"
    },
    {
        id:"002",
        nom:"Beauvais AC"
    }
];
```

Création des options :

```javascript
for(const club of lesClubs){
    idClub.add(new Option(club.nom, club.id));
}
```

Résultat HTML :

```
<option value="001">
    Amiens UC
</option>

<option value="002">
    Beauvais AC
</option>
```

# 8.5. Recherche dans les données reçues

## Exemple : vérifier l'existence d'une catégorie

Avant un ajout, on peut contrôler l'unicité d'un identifiant.

```javascript
function idExiste(idRecherche){
    return lesCategories.some(categorie => categorie.id === idRecherche);
}
```

Utilisation :

```javascript
if(idExiste(id.value)){
    afficherSousLeChamp(id, "Cette catégorie existe déjà");
}
```

La méthode `some()` est particulièrement adaptée car elle s'arrête dès qu'un élément correspond.

# 8.6. Contrôle métier côté client

Le tableau reçu du serveur peut permettre de réaliser des contrôles avant l'envoi.

Exemple : vérifier un chevauchement d'âge.

Données :

```javascript
const nouvelleCategorie = {
    id:"M2",
    ageMin:45,
    ageMax:55
};
```

Contrôle :

```javascript
function chevauchement(categorie){
    return lesCategories.some(c => categorie.ageMin <= c.ageMax && categorie.ageMax >= c.ageMin);
}
```

Explication :

Deux intervalles se chevauchent lorsque :

```
Nouvel intervalle :
      |-----------|

Ancien intervalle :
          |-----------|
```

La condition :

```javascript
ageMin1 <= ageMax2
```

et

```javascript
ageMax1 >= ageMin2
```

est vraie.

# 8.7. Modification locale puis synchronisation serveur

Dans certaines situations, on modifie d'abord le tableau côté client puis on envoie les changements au serveur.

Exemple :

```javascript
const categoriesModifiees = [...lesCategories];

const index = categoriesModifiees.findIndex(c => c.id === "M1");

if(index !== -1){
    categoriesModifiees[index].nom = "Master 1 modifié";
}
```

Le serveur reçoit ensuite :

```javascript
appelAjax({
    url:"/ajax/modifier.php",
    data:{
        tableName:"Categorie",
        primaryKey:"M1",
        columns:{
            nom:"Master 1 modifié"
        }
    }

});
```

# 8.8. Envoi d'un tableau JavaScript vers PHP

Le sens inverse est également fréquent.

Exemple :

```javascript
const lesIds = [
    "M0",
    "M1",
    "M2"
];
```

Pour envoyer ce tableau au serveur :

```javascript
appelAjax({
    url:"/ajax/traitement.php",
    data:{
        categories:
        JSON.stringify(lesIds)
    }
});
```

Le tableau devient :

```
[
    "M0",
    "M1",
    "M2"
]
```

Côté PHP :

```php
$categories =
json_decode($_POST['categories'], true);
```

Résultat :

```
[
    "M0",
    "M1",
    "M2"
]
```

# 8.9. Tableau d'objets envoyé vers PHP

Exemple JavaScript :

```javascript
const coureurs = [
    {
        licence:"123",
        nom:"Martin"
    },
    {
        licence:"456",
        nom:"Dupont"
    }
];
```

Envoi :

```javascript
data:{
    coureurs:
    JSON.stringify(coureurs)
}
```

JSON transmis :

```
[
    {
        "licence":"123",
        "nom":"Martin"
    },
    {
        "licence":"456",
        "nom":"Dupont"
    }
]
```

Réception PHP :

```
$coureurs = json_decode($_POST['coureurs'], true);
```

On obtient :

```
[
    [
        "licence"=>"123",
        "nom"=>"Martin"
    ],
    [
        "licence"=>"456",
        "nom"=>"Dupont"
    ]
]
```

# 8.10. Tableau reçu automatiquement par le framework

Dans un framework applicatif, il est préférable d'éviter de gérer directement :

```php
$_POST
```

Le traitement doit être centralisé.

Exemple :

```php
$columns = Requete::postArray('columns');
```

Le framework assure :

- la récupération ;
- la conversion JSON ;
- le contrôle du format ;
- la gestion des erreurs.

# 8.11. Bonnes pratiques

## Côté serveur

Préférer :

```php
json_encode($donnees);
```

plutôt qu'une construction manuelle :

```php
echo "[...]";
```

Car `json_encode()` gère :

- les caractères spéciaux ;
- les accents ;
- les guillemets ;
- les caractères Unicode.

## Côté client

Préférer :

```javascript
const personne = getData("personne");
```

plutôt que :

```javascript
JSON.parse(document.getElementById("personne").textContent);
```

Le framework centralise ainsi la récupération des données.

## Toujours vérifier les données reçues

Même si les contrôles existent côté client :

```javascript
if(!Array.isArray(lesCategories)){
    throw new Error("Format incorrect");
}
```

Les contrôles métier définitifs restent toujours côté serveur.

# 8.12. Synthèse

Dans une application web :

| Origine    | Structure            |
| ---------- | -------------------- |
| Base SQL   | lignes SQL           |
| PHP        | tableaux associatifs |
| JSON       | texte structuré      |
| JavaScript | tableaux et objets   |

Le tableau JavaScript devient donc le point central permettant :

- d'afficher des données ;
- de rechercher rapidement un élément ;
- d'appliquer des contrôles métier ;
- de préparer des modifications ;
- d'échanger des informations avec le serveur.

La maîtrise des tableaux et de leurs méthodes (`find`, `some`, `filter`, `map`, `reduce`) est donc essentielle pour construire des interfaces web dynamiques et cohérentes.

# 9. Bonnes pratiques et choix des méthodes de tableaux en JavaScript

## 9.1. Choisir la bonne méthode selon le besoin

JavaScript propose de nombreuses méthodes sur les tableaux. Le choix de la méthode dépend principalement de l'objectif recherché.

|Besoin|Méthode conseillée|Résultat|
|---|---|---|
|Vérifier si une valeur existe|`includes()`|`true` ou `false`|
|Trouver un élément|`find()`|l'objet trouvé ou `undefined`|
|Trouver la position d'un élément|`findIndex()`|indice ou `-1`|
|Vérifier qu'au moins un élément respecte une règle|`some()`|`true` ou `false`|
|Vérifier que tous les éléments respectent une règle|`every()`|`true` ou `false`|
|Transformer chaque élément|`map()`|nouveau tableau|
|Extraire certains éléments|`filter()`|nouveau tableau filtré|
|Calculer une valeur globale|`reduce()`|une valeur finale|
|Parcourir pour effectuer une action|`forEach()`|aucun retour|

# 9.2. Ne pas utiliser `filter()` pour une simple recherche

Une erreur fréquente consiste à utiliser `filter()` pour rechercher un seul élément.

Exemple :

```javascript
const categories = lesCategories.filter(c => c.id === "BE");

if (categories.length > 0) {
    console.log(categories[0]);
}
```

Cette solution fonctionne mais elle n'est pas optimale.

Pourquoi ?

- `filter()` parcourt tout le tableau ;
- il crée un nouveau tableau ;
- il continue la recherche même après avoir trouvé l'élément.

La méthode adaptée est `find()` :

```javascript
const categorie = lesCategories.find(c => c.id === "BE");

if (categorie) {
    console.log(categorie);
}
```

Avantages :

- arrêt dès que l'élément est trouvé ;
- aucun tableau temporaire créé ;
- code plus clair.

# 9.3. Utiliser `findIndex()` pour modifier ou supprimer un élément

Lorsque l'on souhaite modifier ou supprimer un élément d'un tableau, il faut généralement connaître sa position.

Exemple : suppression d'un coureur.

```javascript
const index = lesCoureurs.findIndex(c => c.licence === "123456");

if (index !== -1) {
    lesCoureurs.splice(index, 1);
}
```

`findIndex()` est adapté car :

- il retourne directement la position ;
- il permet ensuite d'utiliser `splice()`.

# 9.4. `some()` pour les contrôles d'unicité

Dans une application métier, les contrôles d'unicité sont très fréquents.

Exemple : vérifier que l'identifiant d'une catégorie n'existe pas.

```javascript
function idExiste() {
    return lesCategories.some(
        c => c.id === id.value
    );
}
```

Utilisation :

```javascript
if (idExiste()) {
    afficherSousLeChamp(id, "Cet identifiant existe déjà.");
}
```

Pourquoi `some()` est préférable ?

- Il correspond exactement au besoin métier :
        > Existe-t-il au moins un élément répondant à cette condition ?    
- Il retourne immédiatement dès qu'un élément est trouvé.

# 9.5. `every()` pour valider une collection

La méthode `every()` permet de vérifier une règle générale.

Exemple :

Toutes les catégories doivent avoir un âge minimum inférieur à 100.

```javascript
const valide = lesCategories.every(c => c.ageMin < 100);
```

Dans un contexte métier :

```javascript
if (!lesCategories.every(c => c.ageMin < c.ageMax)) {
    afficherErreur("Certaines catégories possèdent un intervalle invalide.");
}
```

# 9.6. `map()` pour transformer les données reçues du serveur

Les données reçues en JSON ne correspondent pas toujours directement au format attendu par l'interface.

Exemple :

Données reçues :

```
[
    {
        id: "BE",
        nom: "Benjamin"
    },
    {
        id: "M1",
        nom: "Master 1"
    }
]
```

Créer une liste destinée à une liste déroulante :

```javascript
const options = lesCategories.map(
    c => ({
        value: c.id,
        text: c.nom
    })
);
```

Résultat :

```
[
    {
        value:"BE",
        text:"Benjamin"
    },
    {
        value:"M1",
        text:"Master 1"
    }
]
```

# 9.7. `reduce()` pour les calculs métier

`reduce()` permet de transformer un tableau en une seule valeur.

## Exemple : nombre total de coureurs

```javascript
const total = lesCoureurs.reduce((somme, coureur) => somme + 1, 0);
```

Résultat :

```
nombre de coureurs
```

## Exemple : nombre de coureurs par catégorie

Données :

```
const coureurs = [
    {nom:"Martin", categorie:"BE"},
    {nom:"Durand", categorie:"BE"},
    {nom:"Dupont", categorie:"M1"}
];
```

Calcul :

```javascript
const repartition = coureurs.reduce(
    (resultat, c) => {

        if (!resultat[c.categorie]) {
            resultat[c.categorie] = 0;
        }

        resultat[c.categorie]++;

        return resultat;

    },
    {}
);
```

Résultat :

```
{
    BE:2,
    M1:1
}
```

# 9.8. Méthodes qui créent un nouveau tableau

Certaines méthodes ne modifient jamais le tableau original.

Elles retournent une nouvelle structure.

|Méthode|Modification du tableau original|
|---|---|
|`map()`|Non|
|`filter()`|Non|
|`slice()`|Non|
|`concat()`|Non|
|`toSorted()`|Non|
|`toReversed()`|Non|

Exemple :

```javascript
const tri = lesCategories.toSorted(
    (a,b)=>a.nom.localeCompare(b.nom)
);
```

Le tableau `lesCategories` reste inchangé.

# 9.9. Méthodes qui modifient le tableau

Certaines méthodes agissent directement sur le tableau.

|Méthode|Action|
|---|---|
|`push()`|ajoute à la fin|
|`pop()`|supprime le dernier|
|`shift()`|supprime le premier|
|`unshift()`|ajoute au début|
|`splice()`|ajoute ou supprime|
|`sort()`|trie le tableau|
|`reverse()`|inverse l'ordre|

Exemple :

```javascript
lesCategories.push(
    {
        id:"M2",
        nom:"Master 2"
    }
);
```

Le tableau original est modifié.

# 9.10. Performance des recherches

Pour un petit tableau :

```javascript
lesCategories.some(c => c.id === "BE");
```

est parfaitement adapté.

Cependant, si un tableau contient plusieurs milliers d'éléments et que les recherches sont fréquentes, une structure associative peut être plus efficace.

Exemple :

Tableau :

```
[
 {id:"BE", nom:"Benjamin"},
 {id:"M1", nom:"Master 1"}
]
```

Recherche :

```javascript
find()
```

Complexité :

```
O(n)
```

Création d'un index :

```javascript
const categoriesIndex = {};

for (const categorie of lesCategories) {
    categoriesIndex[categorie.id] = categorie;
}
```

Recherche :

```javascript
const categorie = categoriesIndex["BE"];
```

Complexité :

```
O(1)
```

Cette technique est particulièrement intéressante pour :

- les listes de référence ;
- les autocomplétions ;
- les recherches répétées ;
- les contrôles d'existence.

# 9.11. Recommandations dans une application métier

Dans une application utilisant un framework PHP/JavaScript :

## Pour vérifier une règle métier :

Utiliser :

```
some()
every()
```

Exemples :

```javascript
// unicité
lesCategories.some(c => c.id === id.value)

// cohérence globale
lesCategories.every(c => c.ageMin < c.ageMax)
```

## Pour récupérer une donnée :

Utiliser :

```javascript
find()
```

Exemple :

```javascript
const club = lesClubs.find(
    c => c.id === idClub.value
);
```

## Pour modifier un élément connu :

Utiliser :

```javascript
findIndex()
```

puis :

```javascript
splice()
```

## Pour construire des données d'affichage :

Utiliser :

```javascript
map()
```

## Pour réaliser un calcul :

Utiliser :

```javascript
reduce()
```

## Pour filtrer une liste affichée :

Utiliser :

```javascript
filter()
```

# Conclusion

Les méthodes d'extension de `Array` permettent d'écrire un code JavaScript plus proche du raisonnement métier.

Dans une application web :

- `some()` répond aux questions **"Existe-t-il au moins un élément ?"**
- `every()` répond aux questions **"Tous les éléments respectent-ils cette règle ?"**
- `find()` répond aux questions **"Quel est l'élément recherché ?"**
- `findIndex()` répond aux questions **"À quelle position se trouve-t-il ?"**
- `filter()` répond aux questions **"Quels éléments correspondent ?"**
- `map()` répond aux questions **"Comment transformer mes données ?"**
- `reduce()` répond aux questions **"Comment calculer une synthèse ?"**

Ces méthodes constituent la base du traitement des données reçues du serveur et permettent d'implémenter efficacement les contrôles métier côté client.