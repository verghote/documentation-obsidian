# Le format JSON

## 1. Présentation

**JSON** (*JavaScript Object Notation*) est un format texte léger permettant de représenter des **données structurées**. Il est principalement utilisé pour échanger des données entre une application cliente (navigateur, application mobile...) et un serveur.

JSON est aujourd'hui le format d'échange de données le plus utilisé sur le Web. Il présente plusieurs avantages :

- simple à lire et à écrire pour un humain ;
- facile à analyser par les langages de programmation ;
- peu volumineux ;
- indépendant du langage utilisé.

---

## 2. Structure d'un document JSON

Un document JSON peut représenter :

- un objet ;
- un tableau ;
- un tableau contenant plusieurs objets ;
- des objets imbriqués.

### Exemple : tableau d'objets

```json
[
    {
        "id": "alvn",
        "nom": "ALVES",
        "prenom": "Nicolas"
    },
    {
        "id": "zakj",
        "nom": "ZAK",
        "prenom": "Julien"
    }
]
```

En JavaScript :

```javascript
lesEtudiants[0].nom
```

retourne :

```text
ALVES
```

---

### Exemple : tableau associatif

```json
{
    "alvn": {
        "nom": "ALVES",
        "prenom": "Nicolas"
    },
    "zakj": {
        "nom": "ZAK",
        "prenom": "Julien"
    }
}
```

En JavaScript :

```javascript
lesEtudiants["alvn"].nom
```

retourne :

```text
ALVES
```

---

## 3. Syntaxe

Un document JSON est constitué de **paires clé / valeur**.

Une clé est toujours une chaîne de caractères.

Une valeur peut être :

- une chaîne de caractères ;
- un nombre ;
- un booléen (`true` ou `false`) ;
- un objet JSON ;
- un tableau ;
- `null`.

Les différentes paires sont séparées par des virgules.

Exemple :

```json
{
    "nom": "Martin",
    "prenom": "Paul",
    "age": 22,
    "majeur": true
}
```

---

## 4. Les éléments de syntaxe

| Élément | Signification |
|----------|---------------|
| `{ }` | Définit un objet JSON. |
| `[ ]` | Définit un tableau. |
| `:` | Sépare une clé de sa valeur. |
| `,` | Sépare deux éléments d'un objet ou d'un tableau. |

### Objet JSON

```json
{
    "nom": "Martin",
    "prenom": "Paul"
}
```

Chaque propriété possède une clé et une valeur.

---

### Tableau JSON

```json
[
    "Rouge",
    "Vert",
    "Bleu"
]
```

---

### Tableau d'objets

```json
[
    {
        "nom": "Martin",
        "prenom": "Paul"
    },
    {
        "nom": "Durand",
        "prenom": "Julie"
    }
]
```

---

## 5. Règles importantes

- Les clés doivent être entourées de **guillemets doubles**.
- Les chaînes de caractères doivent être entourées de **guillemets doubles**.
- Le dernier élément d'un objet ou d'un tableau **ne doit pas être suivi d'une virgule**.
- Les commentaires ne sont pas autorisés dans un fichier JSON.

✔ Correct

```json
{
    "nom": "Martin",
    "age": 20
}
```

❌ Incorrect

```json
{
    "nom": "Martin",
    "age": 20,
}
```

---

## 6. Les différents types de données

| Type | Exemple |
|-------|----------|
| Chaîne | `"Paul"` |
| Nombre | `25` |
| Décimal | `12.5` |
| Booléen | `true` |
| Objet | `{ "nom":"Martin" }` |
| Tableau | `[1,2,3]` |
| Valeur nulle | `null` |

---

# 7. Utilisation en JavaScript

JSON est directement pris en charge par JavaScript.

## Déclaration d'un objet

```javascript
let unEtudiant = {
    id: 1,
    nom: "Legrand",
    prenom: "Amélie"
};
```

Accès aux propriétés :

```javascript
unEtudiant.id
```

ou

```javascript
unEtudiant["id"]
```

Les deux instructions retournent :

```text
1
```

---

## La classe JSON

JavaScript fournit l'objet **JSON** permettant de convertir des objets JavaScript en texte JSON et inversement.

### JSON.parse()

Convertit une chaîne JSON en objet JavaScript.

```javascript
let chaineJson = '{"code":1,"nom":"Legrand","prenom":"Pierre"}';

let personne = JSON.parse(chaineJson);

console.log(personne.nom);
```

---

### JSON.stringify()

Convertit un objet JavaScript en chaîne JSON.

```javascript
let texteJson = JSON.stringify(unEtudiant);
```

Le résultat est :

```json
{"id":1,"nom":"Legrand","prenom":"Amélie"}
```

---

# 8. Utilisation en PHP

PHP fournit deux fonctions principales.

## json_decode()

Convertit une chaîne JSON en objet PHP.

```php
$chaineJson = '{"code":1,"nom":"Legrand","prenom":"Pierre"}';

$personne = json_decode($chaineJson);

echo $personne->code;
echo $personne->nom;
echo $personne->prenom;
```

---

### Obtenir un tableau associatif

En passant le deuxième paramètre à `true` :

```php
$personne = json_decode($chaineJson, true);

echo $personne['code'];
echo $personne['nom'];
echo $personne['prenom'];
```

---

## json_encode()

Convertit un tableau ou un objet PHP en chaîne JSON.

```php
$chaineJson = json_encode($lesPersonnes);
```

Exemple :

```php
$personne = [
    "nom" => "Martin",
    "prenom" => "Paul"
];

echo json_encode($personne);
```

Résultat :

```json
{"nom":"Martin","prenom":"Paul"}
```

---

# 9. Utilisation en C#

En C#, la bibliothèque la plus utilisée est **Newtonsoft.Json**.

```csharp
using Newtonsoft.Json;

string jsonString = "{\"nom\":\"Jean\",\"age\":30}";

Personne personne = JsonConvert.DeserializeObject<Personne>(jsonString);

Console.WriteLine(personne.Nom);
```

Cette bibliothèque permet :

- de désérialiser une chaîne JSON en objet ;
- de sérialiser un objet en JSON ;
- de parcourir et modifier des documents JSON.

---

# 10. Validation d'un fichier JSON

Avant d'utiliser un fichier JSON, il est conseillé de vérifier sa syntaxe.

Plusieurs validateurs en ligne permettent de détecter rapidement les erreurs.

- https://jsonformatter.curiousconcept.com/
- https://jsonlint.com/
- https://json.parser.online.fr/

Ces outils permettent notamment de :

- vérifier la syntaxe ;
- mettre en forme le document (indentation) ;
- localiser précisément une erreur de syntaxe.

---

# 11. Résumé

JSON est aujourd'hui le format de référence pour l'échange de données entre applications.

Il est :

- léger ;
- lisible ;
- indépendant du langage ;
- facile à manipuler en JavaScript, PHP, C#, Java, Python et dans la plupart des langages modernes.

Ses principales caractéristiques sont :

- les objets sont délimités par `{ }` ;
- les tableaux sont délimités par `[ ]` ;
- les données sont représentées par des paires **clé / valeur** ;
- les clés et les chaînes de caractères sont toujours entourées de guillemets doubles.