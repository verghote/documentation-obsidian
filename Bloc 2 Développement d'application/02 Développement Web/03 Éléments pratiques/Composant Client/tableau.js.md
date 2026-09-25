# Introduction

Le module `tableau.js` propose des utilitaires pour :
- **Mélanger** un tableau d’éléments.
- **Activer le tri** sur un tableau HTML à partir de données dynamiques.
- **Exporter** un tableau au format **CSV** ou **JSON**.

# Installation

Importez le module dans votre projet :

```javascript
import { melanger, activerTri, exporterCSV, exporterJSON } from './tableau.js';
````

#  Fonctions Exportées

### 1. `melanger(lesElements)`

**Description** :  
Mélange aléatoirement un tableau d’éléments en utilisant l’algorithme de Fisher-Yates.

**Paramètres** :

| Paramètre     | Type    | Obligatoire | Description                            |
| ------------- | ------- | ----------- | -------------------------------------- |
| `lesElements` | `Array` | Oui         | Tableau à mélanger (modifié en place). |

**Exemple d’utilisation** :


```javascript
const monTableau = [1, 2, 3, 4, 5];
melanger(monTableau);
console.log(monTableau); // Exemple de résultat : [3, 1, 5, 2, 4]
```

**Remarques** :

- Modifie le tableau **en place** (pas de retour de valeur).
- Utilise `Math.random()` pour la génération aléatoire.

### 2. `activerTri(options)`

**Description** :  
Active le tri interactif sur un tableau HTML, avec gestion des en-têtes cliquables, des flèches de tri, et des types de données (texte, nombre, date).

**Paramètres** (`options`) :

| Paramètre    | Type       | Obligatoire | Description                                                                                            |
| ------------ | ---------- | ----------- | ------------------------------------------------------------------------------------------------------ |
| `idTable`    | `string`   | Oui         | ID de la balise `<table>` à rendre triable.                                                            |
| `getData`    | `Function` | Oui         | Fonction **sans argument** retournant le tableau de données à trier (ex: `() => mesDonnees`).          |
| `afficher`   | `Function` | Oui         | Fonction pour **rafraîchir l’affichage** du tableau après tri (ex: `(data) => afficherTableau(data)`). |
| `triInitial` | `Object`   | Non         | Objet pour afficher une flèche de tri initiale **sans trier** (ex: `{colonne: "nom", sens: "asc"}`).   |

**Structure HTML requise** :

- La balise `<table>` doit avoir un `<thead>` avec une ligne d’en-tête (`<tr>`).
- Chaque cellule d’en-tête (`<th>`) **doit** avoir un attribut `data-champ` pour identifier la colonne.
- Optionnel : `data-type="number"` ou `data-type="date"` pour un tri adapté.

**Exemple d’utilisation** :

javascript

Copier

```html
<table id="monTableau">
  <thead>
    <tr>
      <th data-champ="nom">Nom</th>
      <th data-champ="age" data-type="number">Âge</th>
      <th data-champ="dateNaissance" data-type="date">Date de naissance</th>
    </tr>
  </thead>
  <tbody id="corpsTableau"></tbody>
</table>
```

```JavaScript
const mesDonnees = [
  { nom: "Alice", age: 30, dateNaissance: "15/05/1990" },
  { nom: "Bob", age: 25, dateNaissance: "10/08/1995" },
];

function afficherTableau(data) {
  const tbody = document.getElementById("corpsTableau");
  tbody.innerHTML = data.map(item => `
    <tr>
      <td>${item.nom}</td>
      <td>${item.age}</td>
      <td>${item.dateNaissance}</td>
    </tr>
  `).join("");
}

activerTri({
  idTable: "monTableau",
  getData: () => mesDonnees,
  afficher: afficherTableau,
  triInitial: { colonne: "nom", sens: "asc" },
});
```

**Comportement** :

- Cliquer sur un en-tête trie la colonne correspondante.
- La flèche `▲`/`▼` indique le sens du tri.
- Le tri est **stable** (l’ordre initial est préservé pour les valeurs égales).
- Les types supportés : `text` (défaut), `number`, `date` (format `jj/mm/aaaa`).

### 3. `exporterCSV(data, nomFichier)`

**Description** :  
Exporte un tableau de données au format **CSV** (séparateur `;`), avec encodage UTF-8 et protection contre les injections Excel.

**Paramètres** :

|Paramètre|Type|Obligatoire|Description|
|---|---|---|---|
|`data`|`Array`|Oui|Tableau d’objets à exporter (ex: `[{nom: "Alice", age: 30}, ...]`).|
|`nomFichier`|`string`|Oui|Nom du fichier (sans extension).|

**Exemple d’utilisation** :

javascript

Copier

```
const data = [
  { nom: "Alice", age: 30 },
  { nom: "Bob", age: 25 },
];
exporterCSV(data, "utilisateurs");
```

→ Télécharge `utilisateurs.csv`.

**Spécificités** :

- Ajoute un **BOM** pour Excel.
- Échappe les guillemets (`"` → `""`).
- Protège contre les formules Excel (ex: `=COMMAND|' /C calc'!A0` → `'=COMMAND...`).
- Remplace les caractères interdits dans le nom de fichier par `_`.

### 4. `exporterJSON(data, nomFichier)`

**Description** :  
Exporte un tableau de données au format **JSON**, avec indentation pour une meilleure lisibilité.

**Paramètres** :

|Paramètre|Type|Obligatoire|Description|
|---|---|---|---|
|`data`|`Array`|Oui|Tableau d’objets à exporter.|
|`nomFichier`|`string`|Oui|Nom du fichier (sans extension).|

**Exemple d’utilisation** :

javascript

Copier

```
const data = [
  { nom: "Alice", age: 30 },
  { nom: "Bob", age: 25 },
];
exporterJSON(data, "utilisateurs");
```

→ Télécharge `utilisateurs.json`.

**Spécificités** :

- Utilise `JSON.stringify(data, null, 2)` pour un JSON lisible.
- Nettoie le nom de fichier (comme pour `exporterCSV`).

##  Bonnes Pratiques

1. **Tri** :
    
    - Toujours fournir `getData` et `afficher` pour un tri réactif.
    - Utilisez `data-type` pour les colonnes numériques ou de dates.
    - Évitez de mélanger les types dans une même colonne.
2. **Export** :
    
    - Vérifiez que `data` n’est pas vide avant d’exporter.
    - Pour le CSV, assurez-vous que les clés des objets sont cohérentes.
3. **Performances** :
    
    - Pour de grands tableaux, limitez les appels à `getData` (cachez les données si possible).
4. **Accessibilité** :
    
    - Ajoutez des attributs `aria-sort` aux en-têtes pour améliorer l’accessibilité.

---

##  Limitations Connues

- Le tri des dates suppose le format `jj/mm/aaaa`.
- `melanger` utilise `Math.random()`, non cryptographiquement sécurisé.
- Les exports ne gèrent pas les données binaires ou les objets imbriqués complexes.

---

## 📚 Exemple Complet

javascript

Copier

```javascript
import { activerTri, exporterCSV, exporterJSON } from './tableau.js';

// Données
const utilisateurs = [
  { id: 1, nom: "Alice", age: 30, dateInscription: "15/05/2020" },
  { id: 2, nom: "Bob", age: 25, dateInscription: "10/08/2021" },
  { id: 3, nom: "Charlie", age: 35, dateInscription: "01/01/2019" },
];

// Affichage
function afficherUtilisateurs(data) {
  const tbody = document.querySelector("#tableUtilisateurs tbody");
  tbody.innerHTML = data.map(u => `
    <tr>
      <td>${u.id}</td>
      <td>${u.nom}</td>
      <td>${u.age}</td>
      <td>${u.dateInscription}</td>
    </tr>
  `).join("");
}

// Initialisation
activerTri({
  idTable: "tableUtilisateurs",
  getData: () => utilisateurs,
  afficher: afficherUtilisateurs,
});

// Boutons d'export
document.getElementById("btnExportCSV").addEventListener("click", () => {
  exporterCSV(utilisateurs, "utilisateurs_export");
});
document.getElementById("btnExportJSON").addEventListener("click", () => {
  exporterJSON(utilisateurs, "utilisateurs_export");
});
```
