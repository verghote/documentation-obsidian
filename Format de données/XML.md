# Le format XML

## 1. Présentation

**XML** (*eXtensible Markup Language*) est un langage de balisage permettant de représenter, stocker et échanger des données structurées.

Contrairement au HTML, dont les balises sont prédéfinies pour afficher des pages Web, XML permet de **créer ses propres balises** afin de décrire la nature des données.

XML est utilisé dans de nombreux domaines :

- échanges de données entre applications ;
- fichiers de configuration ;
- services Web (SOAP) ;
- bureautique (DOCX, XLSX, ODT...) ;
- fichiers de description.

Bien que JSON soit aujourd'hui privilégié pour les échanges entre applications Web, XML reste très présent dans les systèmes d'information et les logiciels professionnels.

---
# 2. Structure d'un document XML

Un document XML est constitué :

- d'un **élément racine** unique ;
- d'éléments (balises) pouvant contenir :
  - du texte ;
  - d'autres éléments ;
  - des attributs.

Exemple :

```xml
<?xml version="1.0" encoding="UTF-8"?>

<etudiant>
    <id>alvn</id>
    <nom>ALVES</nom>
    <prenom>Nicolas</prenom>
</etudiant>
```

---
## Exemple avec plusieurs étudiants

```xml
<?xml version="1.0" encoding="UTF-8"?>

<etudiants>

    <etudiant>
        <id>alvn</id>
        <nom>ALVES</nom>
        <prenom>Nicolas</prenom>
    </etudiant>

    <etudiant>
        <id>zakj</id>
        <nom>ZAK</nom>
        <prenom>Julien</prenom>
    </etudiant>

</etudiants>
```

---

# 3. Syntaxe

Un document XML est constitué de **balises ouvrantes** et **fermantes**.

Chaque élément possède :
- un nom ;
- éventuellement des attributs ;
- un contenu.

Exemple :

```xml
<nom>Martin</nom>
```

Les éléments peuvent être imbriqués.

```xml
<personne>
    <nom>Martin</nom>
    <prenom>Paul</prenom>
</personne>
```

---
# 4. Les éléments de syntaxe

| Élément | Signification |
|----------|---------------|
| `<balise>` | Balise ouvrante. |
| `</balise>` | Balise fermante. |
| `<balise />` | Balise vide. |
| `attribut="valeur"` | Attribut d'un élément. |
| `<?xml ... ?>` | Déclaration XML. |

---
## Élément simple

```xml
<nom>Martin</nom>
```

---
## Élément contenant plusieurs sous-éléments

```xml
<personne>
    <nom>Martin</nom>
    <prenom>Paul</prenom>
</personne>
```

---
ment avec attribut

```xml
<personne id="15">
    <nom>Martin</nom>
    <prenom>Paul</prenom>
</personne>
```

---

## Élément vide

```xml
<photo />
```

---
# 5. Règles importantes

Un document XML doit respecter plusieurs règles :

- il possède un **élément racine unique** ;
- chaque balise ouverte doit être fermée ;
- les balises sont sensibles à la casse ;
- les attributs doivent être entre guillemets ;
- les éléments doivent être correctement imbriqués.

✔ Correct

```xml
<personne>
    <nom>Martin</nom>
</personne>
```

❌ Incorrect

```xml
<personne>
    <nom>Martin
</personne>
```

---

✔ Correct

```xml
<b>
    <i>Texte</i>
</b>
```

❌ Incorrect

```xml
<b>
    <i>Texte</b>
</i>
```

---
# 6. Les attributs

Les attributs permettent d'ajouter des informations à un élément.

Exemple :

```xml
<livre isbn="9782100808822">
    <titre>Algorithmique</titre>
</livre>
```

L'attribut est :

```text
isbn
```

Sa valeur est :

```text
9782100808822
```

---
# 7. Utilisation en JavaScript

JavaScript permet de lire un document XML grâce à la classe **DOMParser**.

```javascript
const texte = `
<personne>
    <nom>Martin</nom>
    <prenom>Paul</prenom>
</personne>
`;

const parser = new DOMParser();

const xml = parser.parseFromString(texte, "text/xml");

console.log(xml.getElementsByTagName("nom")[0].textContent);
```

Résultat :

```text
Martin
```

---
# 8. Utilisation en PHP

PHP fournit plusieurs bibliothèques pour manipuler XML.

La plus simple est **SimpleXML**.

```php
$xml = simplexml_load_file("personnes.xml");

echo $xml->personne[0]->nom;
```

---

Création d'un document XML :

```php
$xml = new SimpleXMLElement('<personne/>');

$xml->addChild('nom', 'Martin');
$xml->addChild('prenom', 'Paul');

echo $xml->asXML();
```

Résultat :

```xml
<?xml version="1.0"?>

<personne>
    <nom>Martin</nom>
    <prenom>Paul</prenom>
</personne>
```

---
# 9. Utilisation en C#

En C#, XML est pris en charge par l'espace de noms **System.Xml**.

```csharp
using System.Xml;

XmlDocument document = new XmlDocument();

document.Load("personnes.xml");

Console.WriteLine(
    document.SelectSingleNode("//nom").InnerText
);
```

Il est également possible d'utiliser **LINQ to XML**, qui simplifie la manipulation des documents XML.

---
# 10. Validation d'un document XML

Un document XML peut être validé :

- en vérifiant sa syntaxe ;
- en le comparant à une définition de structure.

Deux mécanismes existent :

- **DTD** (*Document Type Definition*) ;
- **XSD** (*XML Schema Definition*).

Le schéma XSD est aujourd'hui le plus utilisé car il permet de définir précisément :

- les éléments autorisés ;
- les attributs ;
- les types de données ;
- les valeurs possibles.

---

# 11. Comparaison XML / JSON

| XML | JSON |
|------|------|
| Langage de balisage | Format de données |
| Balises ouvrantes et fermantes | Paires clé / valeur |
| Plus verbeux | Plus compact |
| Très structuré | Plus léger |
| Validation possible par XSD | Validation possible par JSON Schema |
| Utilisé dans de nombreuses applications professionnelles | Très utilisé pour les API Web |

---
# 12. Outils de validation

Plusieurs outils permettent de vérifier la syntaxe d'un document XML.

- https://www.xmlvalidation.com/
- https://codebeautify.org/xmlvalidator
- https://www.freeformatter.com/xml-validator-xsd.html

Ces outils permettent notamment :

- de vérifier la syntaxe ;
- d'afficher les erreurs ;
- de mettre en forme le document ;
- de valider le document par rapport à un schéma XSD.

---
# 13. Résumé

XML est un langage de balisage permettant de représenter des données structurées.

Ses principales caractéristiques sont :

- les données sont décrites par des balises ;
- chaque document possède un élément racine unique ;
- les balises peuvent contenir des éléments imbriqués ;
- les attributs permettent d'ajouter des informations ;
- un document XML peut être validé grâce à un schéma (DTD ou XSD).

XML reste largement utilisé dans les applications professionnelles, les échanges de données entre systèmes et les formats de fichiers bureautiques, même si JSON est aujourd'hui devenu le format privilégié pour les échanges entre applications Web.