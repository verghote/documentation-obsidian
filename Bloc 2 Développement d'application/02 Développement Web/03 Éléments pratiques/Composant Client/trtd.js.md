## 1. Présentation

Le composant `trtd.js` est une petite bibliothèque de fonctions JavaScript destinée à simplifier la création dynamique des lignes (`<tr>`) et des cellules (`<td>`) d'un tableau HTML.

Il fournit trois fonctions principales :

- `creerTd()` — crée une cellule `<td>` contenant du texte ou du HTML ;
- `creerTdWithImg()` — crée une cellule `<td>` contenant une image ;
- `creerTr()` — crée une ligne `<tr>` à partir d'un ensemble de cellules.

L'objectif n'est pas de remplacer les fonctionnalités natives du DOM, mais de **factoriser le code répétitif** nécessaire à la construction de tableaux dynamiques.

# 2. Emplacement du composant

Le fichier est situé dans :

```text
public/
└── composant/
    └── fonction/
        └── trtd.js
```

Le composant utilise les modules JavaScript ES6 et doit donc être importé avec `import`.

Exemple :

```javascript
import {
    creerTd,
    creerTdWithImg,
    creerTr
} from '/composant/fonction/trtd.js';
```

Selon l'organisation du projet, le chemin peut naturellement être adapté au fichier JavaScript qui effectue l'import.

# 3. Principe général

La construction classique d'un tableau dynamique consiste à créer successivement :

```text
<table>
    └── <tbody>
        └── <tr>
            ├── <td>
            ├── <td>
            ├── <td>
            └── <td>
```

Avec `trtd.js`, cette construction devient :

```javascript
const lesTds = [
    creerTd('Dupont'),
    creerTd('Jean'),
    creerTd(25, {centrer: true})
];

lesLignes.appendChild(creerTr(lesTds));
```

Le code décrit directement l'intention :

> créer trois cellules, puis créer une ligne avec ces trois cellules.

# 4. Pourquoi utiliser `trtd.js` ?

## 4.1 Réduire le code répétitif

Sans bibliothèque, chaque développeur doit répéter les opérations DOM :

```javascript
const td = document.createElement('td');
td.innerText = categorie.nom;

const td2 = document.createElement('td');
td2.innerText = categorie.id;

const td3 = document.createElement('td');
td3.innerText = categorie.age;
td3.style.textAlign = 'center';

const tr = document.creteeElement('tr');
tr.appendChild(td);
tr.appendChild(td2);
tr.appendChild(td3);

lesLignes.appendChild(tr)

```



Avec `trtd.js` :

```javascript
const lesTds = [
    creerTd(categorie.nom),
    creerTd(categorie.id),
    creerTd(categorie.age, { centrer: true })
];

lesLignes.appendChild(creerTr(lesTds))

```

La logique métier est beaucoup plus visible.

## 4.2 Centraliser les conventions

Le composant permet de centraliser certaines règles de construction des cellules.

Par exemple, toutes les lignes créées avec `creerTr()` utilisent :

```javascript
tr.style.verticalAlign = 'middle';
```

Le développeur n'a donc pas besoin de répéter cette instruction dans chaque traitement.

De la même manière, le centrage d'une cellule s'effectue toujours avec :

```javascript
{ centrer: true }
```

plutôt qu'avec :

```javascript
td.style.textAlign = 'center';
```

## 4.3 Faciliter la lecture du code

Comparez :

```javascript
td.style.textAlign = 'center';
td.classList.add('masquer');
```

avec :

```javascript
creerTd(categorie.annee, {
    centrer: true,
    masquer: true
});
```

La seconde écriture permet de comprendre immédiatement les caractéristiques de la cellule.

# 5. Fonction `creerTd()`

## Syntaxe

```javascript
creerTd(contenu, options)
```

### Paramètres

|Paramètre|Type|Obligatoire|Défaut|Description|
|---|---|---|---|---|
|`contenu`|`string`|Oui|—|Texte ou HTML à placer dans la cellule|
|`options`|`Object`|Non|`{}`|Options de configuration|
|`options.centrer`|`boolean`|Non|`false`|Centre horizontalement le contenu|
|`options.masquer`|`boolean`|Non|`false`|Ajoute la classe CSS `masquer`|
|`options.isHTML`|`boolean`|Non|`false`|Interprète le contenu comme du HTML|

## Exemple simple

```javascript
const td = creerTd('Dupont');
```

Produit :

```html
<td>Dupont</td>
```

## Centrer le contenu

```javascript
const td = creerTd('25', { centrer: true});
```

Produit notamment :

```html
<td style="text-align: center;">25</td>
```

## Masquer une colonne

```javascript
const td = creerTd(categorie.annee, { masquer: true});
```

La cellule reçoit la classe :

```html
<td class="masquer">2026</td>
```

Le fonctionnement réel de l'affichage dépend alors du CSS associé à la classe `.masquer`.

Par exemple :

```css
.masquer {
    display: none;
}
```

# 6. Attention à `isHTML`

Par défaut, `creerTd()` utilise :

```javascript
td.innerText = contenu;
```

Le contenu est donc considéré comme du texte.

Exemple :

```javascript
creerTd('<strong>Dupont</strong>');
```

affichera littéralement :

```text
<strong>Dupont</strong>
```

Pour insérer volontairement du HTML :

```javascript
creerTd('<strong>Dupont</strong>', {
    isHTML: true
});
```

Le résultat sera :

```html
<td>
    <strong>Dupont</strong>
</td>
```

### ⚠️ Sécurité

`isHTML: true` doit être utilisé uniquement lorsque le contenu HTML est maîtrisé.

Il ne faut pas injecter directement dans `innerHTML` une donnée provenant d'un utilisateur ou d'une source non fiable.

À privilégier :

```javascript
creerTd(nomUtilisateur);
```

plutôt que :

```javascript
creerTd(nomUtilisateur, {
    isHTML: true
});
```

La version texte est plus sûre car `innerText` n'interprète pas le contenu comme du HTML.

# 7. Fonction `creerTdWithImg()`

Cette fonction permet de créer une cellule contenant une image.
## Syntaxe

```javascript
creerTdWithImg(src, alt, options)
```

### Paramètres

|Paramètre|Type|Obligatoire|Défaut|Description|
|---|---|---|---|---|
|`src`|`string`|Oui|—|Chemin ou URL de l'image|
|`alt`|`string`|Non|`''`|Texte alternatif|
|`options`|`Object`|Non|`{}`|Options d'affichage|
|`options.size`|`number`|Non|`40`|Largeur et hauteur en pixels|
|`options.radius`|`string`|Non|`50%`|Rayon des bordures|
|`options.masquer`|`boolean`|Non|`false`|Ajoute la classe `masquer`|

## Exemple

```javascript
const tdPhoto = creerTdWithImg(
    '/images/coureurs/dupont.jpg',
    'Jean Dupont'
);
```

Produit une cellule contenant une image de `40 × 40 px`.

## Modifier la taille

```javascript
const tdPhoto = creerTdWithImg(
    '/images/coureurs/dupont.jpg',
    'Jean Dupont',
    {
        size: 60
    }
);
```

## Image carrée

```javascript
const tdPhoto = creerTdWithImg(
    '/images/coureurs/dupont.jpg',
    'Jean Dupont',
    {
        size: 50,
        radius: '0'
    }
);
```

# 8. Fonction `creerTr()`

Cette fonction construit une ligne `<tr>` à partir d'un tableau de cellules.

## Syntaxe

```javascript
creerTr(lesTds)
```

### Paramètre

|Paramètre|Type|Défaut|Description|
|---|---|---|---|
|`lesTds`|`HTMLTableCellElement[]`|`[]`|Tableau de cellules `<td>`|

## Exemple

```javascript
const lesTds = [
    creerTd('Dupont'),
    creerTd('Jean'),
    creerTd('25', { centrer: true })
];

const tr = creerTr(lesTds);

lesLignes.appendChild(tr);
```

La ligne obtenue est équivalente à :

```html
<tr>
    <td>Dupont</td>
    <td>Jean</td>
    <td style="text-align:center">25</td>
</tr>
```

# 9. Exemple complet : affichage des catégories

Voici un exemple représentatif de l'utilisation du composant.

```javascript
// affichage de la liste des catégories
for (const categorie of lesCategories) {

    const lesTds = [
        creerTd(categorie.nom),
        creerTd(categorie.id),
        creerTd(categorie.age, {
            centrer: true
        }),
        creerTd(categorie.annee, {
            centrer: true,
            masquer: true
        })
    ];

    // Ajout de la ligne dans le tbody
    lesLignes.appendChild(creerTr(lesTds));
}
```

Cette écriture présente plusieurs avantages.

La structure de chaque ligne est immédiatement identifiable :

```text
nom
id
âge
année
```

Les options sont également explicites :

```javascript
{ centrer: true }
```

et :

```javascript
{
    centrer: true,
    masquer: true
}
```

Le code métier reste ainsi indépendant des détails techniques de création du DOM.

# 10. Ce qu'il faudrait écrire sans `trtd.js` avec `insertRow()` et `insertCell()`

Sans le composant, une première solution consiste à utiliser les méthodes natives des tableaux HTML :

```javascript
// affichage de la liste des catégories
for (const categorie of lesCategories) {

    const ligne = lesLignes.insertRow();

    const celluleNom = ligne.insertCell();
    celluleNom.innerText = categorie.nom;

    const celluleId = ligne.insertCell();
    celluleId.innerText = categorie.id;

    const celluleAge = ligne.insertCell();
    celluleAge.innerText = categorie.age;
    celluleAge.style.textAlign = 'center';

    const celluleAnnee = ligne.insertCell();
    celluleAnnee.innerText = categorie.annee;
    celluleAnnee.style.textAlign = 'center';
    celluleAnnee.classList.add('masquer');
}
```

Cette solution fonctionne parfaitement.

`insertRow()` et `insertCell()` sont des API natives du DOM et ne présentent aucun problème particulier.

Cependant, le code devient rapidement répétitif lorsque l'application comporte beaucoup de tableaux dynamiques.

Avec `trtd.js`, les mêmes opérations deviennent :

```javascript
for (const categorie of lesCategories) {

    const lesTds = [
        creerTd(categorie.nom),
        creerTd(categorie.id),
        creerTd(categorie.age, { centrer: true }),
        creerTd(categorie.annee, {
            centrer: true,
            masquer: true
        })
    ];

    lesLignes.appendChild(creerTr(lesTds));
}
```

La différence principale est donc la **factorisation de la construction des cellules**.

# 11. Ce qu'il faudrait écrire avec `document.createElement()`

Une autre solution consiste à utiliser directement les API génériques du DOM.

Sans `trtd.js`, il faudrait écrire :

```javascript
// affichage de la liste des catégories
for (const categorie of lesCategories) {

    const tr = document.createElement('tr');

    tr.style.verticalAlign = 'middle';

    const tdNom = document.createElement('td');
    tdNom.innerText = categorie.nom;
    tr.appendChild(tdNom);

    const tdId = document.createElement('td');
    tdId.innerText = categorie.id;
    tr.appendChild(tdId);

    const tdAge = document.createElement('td');
    tdAge.innerText = categorie.age;
    tdAge.style.textAlign = 'center';
    tr.appendChild(tdAge);

    const tdAnnee = document.createElement('td');
    tdAnnee.innerText = categorie.annee;
    tdAnnee.style.textAlign = 'center';
    tdAnnee.classList.add('masquer');
    tr.appendChild(tdAnnee);

    lesLignes.appendChild(tr);
}
```

Cette version est encore plus verbeuse.

Pour seulement quatre cellules, cela représente beaucoup de lignes de code par rapport à la logique réellement nécessaire.

# 12. Quel est le véritable intérêt de `trtd.js` ?

Le composant n'apporte pas une nouvelle capacité au navigateur.

Toutes les opérations réalisées par `trtd.js` pourraient être réalisées directement avec le DOM.

Son intérêt est ailleurs :

> **`trtd.js` constitue une couche d'abstraction légère permettant de standardiser la création des lignes et cellules des tableaux dans l'application.**

Cette abstraction apporte notamment :

### Lisibilité

Le développeur voit immédiatement :

```javascript
creerTd(categorie.age, { centrer: true })
```

plutôt qu'une succession de manipulations DOM.

### Cohérence

Toutes les cellules et toutes les lignes sont créées selon les mêmes conventions.

### Maintenance

Si la manière de créer les cellules évolue, la modification peut être centralisée dans `trtd.js`.

Par exemple, si demain le centrage doit être réalisé avec une classe CSS plutôt qu'avec un style inline, une modification de :

```javascript
creerTd()
```

peut suffire.

Le code métier :

```javascript
creerTd(categorie.age, { centrer: true })
```

reste inchangé.

### Réduction du code

La construction répétitive du DOM est déplacée dans le composant.

### Standardisation entre développeurs

Deux développeurs travaillant sur deux tableaux différents utiliseront la même convention :

```javascript
creerTd(valeur)
creerTd(valeur, { centrer: true })
creerTd(valeur, { masquer: true })
creerTr(lesTds)
```

# 13. Quand utiliser `creerTd()` plutôt que du DOM natif ?

Pour les tableaux standards de l'application, l'utilisation de `trtd.js` est recommandée.

Utiliser :

```javascript
creerTd()
creerTdWithImg()
creerTr()
```

permet de conserver une convention homogène.

Les API natives :

```javascript
document.createElement()
```

et :

```javascript
insertRow()
insertCell()
```

restent parfaitement valides et peuvent être utilisées lorsqu'un traitement très particulier nécessite un contrôle DOM spécifique.

Le composant ne doit donc pas être considéré comme une obligation technique, mais comme **la convention recommandée pour les tableaux dynamiques du projet**.

# 14. Résumé des fonctions

|Fonction|Rôle|
|---|---|
|`creerTd()`|Créer une cellule `<td>` textuelle ou HTML|
|`creerTdWithImg()`|Créer une cellule `<td>` contenant une image|
|`creerTr()`|Créer une ligne `<tr>` à partir de cellules|

## Options de `creerTd()`

|Option|Défaut|Fonction|
|---|---|---|
|`centrer`|`false`|Centre horizontalement le contenu|
|`masquer`|`false`|Ajoute la classe `.masquer`|
|`isHTML`|`false`|Interprète le contenu comme du HTML|

## Options de `creerTdWithImg()`

|Option|Défaut|Fonction|
|---|---|---|
|`size`|`40`|Taille de l'image|
|`radius`|`'50%'`|Rayon des bordures|
|`masquer`|`false`|Ajoute la classe `.masquer`|

# 15. Exemple recommandé

Pour un tableau dynamique, la forme recommandée est :

```javascript
import {
    creerTd,
    creerTr
} from '/composant/fonction/trtd.js';

// affichage de la liste des catégories
for (const categorie of lesCategories) {

    const lesTds = [
        creerTd(categorie.nom),
        creerTd(categorie.id),
        creerTd(categorie.age, {
            centrer: true
        }),
        creerTd(categorie.annee, {
            centrer: true,
            masquer: true
        })
    ];

    lesLignes.appendChild(creerTr(lesTds));
}
```

Cette forme doit être privilégiée car elle sépare clairement :

1. **les données** : `categorie.nom`, `categorie.id`, etc. ;
2. **la présentation de la cellule** : `centrer`, `masquer` ;
3. **la construction du DOM** : prise en charge par `trtd.js`.

Ainsi, le code appelant reste centré sur **ce que doit contenir le tableau**, tandis que le composant se charge de **la manière de construire les éléments HTML**.
