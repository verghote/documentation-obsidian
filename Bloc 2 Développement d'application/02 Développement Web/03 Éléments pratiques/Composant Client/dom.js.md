
Cette bibliothèque fournit des fonctions utilitaires pour générer des composants HTML 

 Référence rapide des fonctions

| Fonction                | Rôle                                                                       |
| ----------------------- | -------------------------------------------------------------------------- |
| getTd                   | Crée une cellule de tableau `<td>`                                         |
| getTdWithImg            | Crée une cellule `<td>` contenant une image                                |
| getTr                   | Crée une ligne de tableau `<tr>` à partir d’un ensemble de cellules `<td>` |
| creerBoutonAction       | Crée un bouton d'action générique                                          |
| creerBoutonModification | Crée un bouton Modifier                                                    |
| creerBoutonSuppression  | Crée un bouton Supprimer                                                   |
| creerBoutonRemplacer    | Crée un bouton Remplacer                                                   |
| creerCheckbox           | Génère une case à cocher standard                                          |
| creerSelect             | Génère une liste déroulante                                                |
| creerInputTexte         | Génère un champ texte                                                      |

#  Fonctions disponibles

## `getTd(contenu, options)`

###  Crée une cellule de tableau `<td>` contenant du texte ou du HTML, avec options de style.

### Options

- `centrer` : centre le contenu horizontalement
- `masquer` : ajoute la classe CSS `masquer`
- `isHTML` : interprète le contenu comme du HTML

###  Exemple

```javascript
const cell = getTd("Bonjour", { centrer: true });
```

```javascript
const cellHTML = getTd("<b>Important</b>", { isHTML: true });
```

```javascript
const cellHidden = getTd("Secret", { masquer: true });
```

---

## `getTdWithImg(src, alt, options)`

###  Crée une cellule `<td>` contenant une image stylisée (souvent ronde ou avatar).

### ⚙️ Options

- `size` : taille de l’image (px)
- `radius` : arrondi de l’image (ex: `50%`)
- `masquer` : ajoute la classe CSS `masquer`

###  Exemple

```javascript
const avatar = getTdWithImg("/img/user.png", "Utilisateur");
```

```javascript
const avatarLarge = getTdWithImg("/img/user.png", "Profil", {    size: 80,    radius: "10%"});
```

---

##  `getTr(lesTds)`

###  Crée une ligne de tableau `<tr>` à partir d’un ensemble de cellules `<td>`.

### 📌 Exemple

```javascript
const row = getTr([getTd("Alice"),getTd("Admin"), getTdWithImg("/img/alice.png", "Alice")]);
```

---
##  Exemple complet d’utilisation

Sans utiliser les fonctions

```javascript
function afficher() { 
    lesLignes.innerHTML = ''; 
    for (const competence of lesCompetences) { 
        const tr = lesLignes.insertRow(); 
        tr.style.verticalAlign = 'middle'; 
        // Colonne : code 
        tr.insertCell().innerText = competence.code; 
        // Colonne Libellé 
        tr.insertCell().innerText = competence.libelle; 
     } 
}
```

Avec l'utilisation des fonctions

```
function afficher() {  
   lesLignes.innerHTML = '';  
   for (const competence of lesCompetences) {  
      const tr = getTr([getTd(competence.code), getTd(competence.libelle)]);  
      lesLignes.appendChild(tr);  
    }  
}
```

---

eerBoutonAction

Crée un bouton d'action stylisé.

 Exemple

```javascript
const bouton = creerBoutonAction({
    icone: '✎',
    couleur: 'orange',
    titre: 'Modifier',
    action: () => modifier()
});
```

---
##  creerBoutonModification

```javascript
const bouton = creerBoutonModification(() => {modifier();});
```

Icône : ✎

##  creerBoutonSuppression

```javascript
const bouton = creerBoutonSuppression(() => {supprimer();});
```

Icône : ✘

---
##  creerBoutonRemplacer

```javascript
const bouton = creerBoutonRemplacer(() => {remplacerDocument();});
```

Icône : ♻️

---
##  creerCheckbox

```javascript
const checkbox = creerCheckbox();
```

Retourne :

```html
<input type="checkbox">
```

avec le style standard de l'application.

---
##  creerSelect

```javascript
const select = creerSelect([
    'form-select',
    'mb-3'
]);
```

Retourne :

```html
<select></select>
```

---

##  creerInputTexte

```javascript
const input = creerInputTexte({
    placeholder: 'Nom du club',
    classes: ['form-control']
});
```

Retourne :

```html
<input type="text">
```

---
