## Table des matières

1. Présentation [[Mémento CSS#1. Présentation]]
2. Déclaration des styles [[Mémento CSS#2. Déclaration des styles]]
3. Définition d'un style [[Mémento CSS#3. Définition d'un style]]
4. Les sélecteurs [[Mémento CSS#4. Les sélecteurs]]
5. Exemples de règles CSS [[Mémento CSS#5. Exemples de règles CSS]]
6. Les principales propriétés [[Mémento CSS#6. Les principales propriétés]]    
7. Mettre en forme un texte [[Mémento CSS#7. Mettre en forme un texte]]
8. Les unités de mesure (px, em, rem, %) [[Mémento CSS#8. Les unités de mesure (px, em, rem, %)]]
9. La notion de bloc (Box Model & Display) [[Mémento CSS#8. Les unités de mesure (px, em, rem, %)]]
10. Positionnement des éléments [[Mémento CSS#10. Positionnement des éléments]]
11. Pseudo-classes [[Mémento CSS#11. Pseudo-classes]]
12. Sélecteurs avancés [[Mémento CSS#12. Sélecteurs avancés]]
13. Impression (`@media print`) [[Mémento CSS#13. Impression (`@media print`)]]
14. Validation d'une feuille de style [[Mémento CSS#14. Validation d'une feuille de style]]
15. Media Queries & Design Responsive [[Mémento CSS#15. Media Queries & Design Responsive]]
16. La règle `!important` [[Mémento CSS#16. La règle `!important`]]
17. Variables CSS (Propriétés personnalisées) [[Mémento CSS#17. Variables CSS (Propriétés personnalisées)]]
18. Fonctions CSS (`calc`, `min`, `max`, `clamp`) [[Mémento CSS#18. Les fonctions CSS]]
## 1. Présentation

Les feuilles de style (CSS) offrent un moyen centralisé de définir des catégories de styles visuels applicables aux éléments d'une page HTML (couleurs, bordures, arrière-plans, dimensions, disposition relative et interactivité simple).

- **Normalisation W3C :** CSS1 (1996), CSS2 (1998), CSS3 (depuis les années 2000 : Flexbox, Grid, animations, variables, etc.).
    
- **Support :** CSS3 est universellement pris en charge par tous les navigateurs modernes.
    

## 2. Déclaration des styles

### A. Fichier externe (recommandé)

HTML

```html
<!-- Fichier local -->
<link rel="stylesheet" type="text/css" href="css/style.css">

<!-- Fichier distant via CDN -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.2/dist/css/bootstrap.min.css">
```

> **Avantages d'un CDN :** Réduit la charge du serveur local, améliore la vitesse via la mise en cache globale et la proximité géographique.
> 
> **Inconvénient :** Dépendance envers un serveur externe.

### B. Inclus dans l'en-tête HTML

HTML

```html
<style type="text/css">
  body { background-color: #f4f4f4; }
</style>
```

### C. Style en ligne (Inline)

HTML

```html
<p style="font-size: 16pt;">Texte stylisé directement</p>
```

### D. Importation via CSS

CSS

```css
@import url("https://cdn.jsdelivr.net/npm/bootstrap-icons@1.5.0/font/bootstrap-icons.css");
```

## 3. Définition d'un style

Une règle CSS se compose d'un **sélecteur** et d'un **bloc de déclaration** :

CSS

```css
selecteur {
    propriete: valeur;
    propriete: valeur;
}
```

## 4. Les sélecteurs

|**Sélecteur**|**S'applique sur**|**Exemple**|
|---|---|---|
|`nomBalise`|Une balise HTML spécifique|`h1 { color: red; }`|
|`.nomClasse`|Les éléments ayant cette classe|`.photo { float: left; margin: 2em; }`|
|`#nomId`|L'élément unique ayant cet ID|`#pied { border: 1px solid blue; }`|
|`*`|Tous les éléments (universel)|`* { margin: 0; }`|

### Combinaisons & Relations

- `h1.photo` : Balise `<h1>` portant la classe `photo`.
    
- `E[attr]` : Élément possédant l'attribut `attr` (`a[title]`).
    
- `E[attr="val"]` : Valeur exacte (`a[href="[https://site.com](https://site.com)"]`).
    
- `E[attr^="val"]` : Commençant par `val` (`a[href^="#"]`).
    
- `E[attr$="val"]` : Se terminant par `val` (`a[href$=".pdf"]`).
    
- `E[attr*="val"]` : Contenant la sous-chaîne `val`.
    
- `A B` : Élément `B` descendant de `A`.
    
- `A > B` : Élément `B` enfant direct de `A`.
    
- `A + B` : Élément `B` frère adjacent direct de `A`.
    
- `A ~ B` : Tous les éléments `B` frères situés après `A`.
    
## 5. Exemples de règles CSS

CSS

```css
body {
    background-color: #FFFFCC; /* Couleur de fond */
    font-family: Arial, Helvetica, sans-serif; /* Police */
    font-size: 1em;            /* Taille relative */
    line-height: 24px;         /* Hauteur de ligne */
    color: #333333;            /* Couleur du texte */
}

.retrait20 {
    padding-left: 20px;        /* Marge interne gauche */
    padding-right: 10px;       /* Marge interne droite */
}

img {
    vertical-align: middle;    /* Alignement vertical */
    border: none;              /* Pas de bordure */
}

ul { 
    margin: 0 0 16px 0;        /* Marges externes : Haut Droite Bas Gauche (H D B G) */
    padding: 0 0 0 48px;       /* Marges internes */
    list-style-position: outside;
}

ul li { 
    list-style-image: url('/img/puce.png'); /* Puce personnalisée */
    padding: 0;
    margin: 0 48px 8px 0;
}
```

## 6. Les principales propriétés

### Arrière-plan (Background)

- `background-color`: Couleur de fond (`#cfcfcf`).
    
- `background-image`: Image de fond (`url('/header.jpg')`).
    
- `background-repeat`: `no-repeat` | `repeat` | `repeat-x` | `repeat-y`.
    
- `background-position`: Position (`top left`, `center center`, `1% 50%`).
    
- `background-size`: `cover` | `contain` | `100%`.
    
- **Raccourci :** `background: white url('/img/bg.png') top/cover no-repeat;`

### Bordures & Boîte

- `border`: `1px solid #0000FF;` (Spécifique : `border-top`, `border-right`, etc.)
    
- `border-radius`: Coin arrondi (`8px`, `2em`).
    
- `border-collapse`: `collapse;` (Pour les tableaux HTML).
    
- `box-shadow`: Ombre portée (`8px 8px 0px #aaa`).
    
- `display`: `block` | `inline` | `inline-block` | `flex` | `grid` | `none`.
    
- `opacity`: Transparence de 0 à 1 (`0.25`).
    
- `visibility`: `hidden` (masqué mais conserve sa place) | `visible`.
    
- `overflow`: `visible` | `auto` | `scroll` | `hidden`.
    
- `z-index`: Ordre d'empilement superposé (`1`, `10`, `999`).
    
### Marges & Dimensions

- `margin`: Marges extérieures (`margin-top`, `margin-right`, etc.). `margin: auto` permet de centrer un bloc avec largeur définie.
    
- `padding`: Marges intérieures (`padding-left`, etc.).
    
- `width` / `height`: Largeur / Hauteur.
    
- `min-width` / `max-width`, `min-height` / `max-height`: Contraintes de dimensions.
    

### Typographie & Texte

- `font-family`: Liste de polices fallback (`Verdana, sans-serif`).
    
- `font-weight`: Épaisseur (`bold`, `normal`, `600`).
    
- `font-size`: Taille du texte.
    
- `font-style`: `italic` | `normal` | `oblique`.
    
- `font-variant`: `small-caps` (petites capitales).
    
- `line-height`: Hauteur d'interligne.
    
- `color`: Couleur du texte.
    
- `text-align`: `left` | `right` | `center` | `justify`.
    
- `text-decoration`: `underline` | `overline` | `line-through` | `none`.
    
- `text-transform`: `uppercase` | `lowercase` | `capitalize`.
    
- `letter-spacing`: Espacement entre caractères.
    
- `text-shadow`: Ombre sur le texte (`2px 2px 8px #FF0000`).
    
- `vertical-align`: Alignement vertical en ligne (`top`, `middle`, `bottom`).
    

## 7. Mettre en forme un texte

- Les polices dépendent du système de l'utilisateur. Privilégiez les polices système courantes ou les webfonts (ex: Google Fonts).
    
- Utilisez de préférence des unités relatives (`em`, `rem`, `%`) pour une meilleure accessibilité. Par défaut, la taille de police du navigateur est de **16px**.
    

## 8. Les unités de mesure (px, em, rem, %)

### 8.1. Le Pixel (`px`)

Unités absolue fixe.

- **Avantage :** Précis et facile à utiliser.
    
- **Inconvénient :** Inflexible pour le responsive design ; nécessite de recalculer chaque taille manuellement via Media Queries.
    

### 8.2. Les unités relatives au parent (`em` et `%`)

Relatives à la taille de police de l'élément parent (`1em = 100%`).

- **Formule :** $\text{Taille en em} = \frac{\text{Taille désirée en px}}{\text{Taille du parent en px}}$
    
- **Problème d'imbrication :** L'effet cascade multiplie la taille :
    
    CSS
    
    ```
    body { font-size: 14px; }
    li { font-size: 1.2em; } /* li = 16.8px */
    li li { font-size: 1.2em; } /* li imbriqué = 16.8 * 1.2 = 20.16px ! */
    ```
    

### 8.3. L'unité relative à la racine (`rem`)

`rem` = **Root EM**. Se base uniquement sur la taille de police définie sur l'élément racine (`html` ou `body`).

- Évite l'effet d'accumulation d'héritage de l'unité `em`.
    
- Prévisible et recommandé pour le design moderne et accessible.
    

## 9. La notion de bloc (Box Model & Display)

Dans le modèle CSS, tout élément HTML est contenu dans une boîte rectangulaire composée de : **Content**, **Padding**, **Border** et **Margin**.

```
+-----------------------------------+
|               MARGIN              |
|  +-----------------------------+  |
|  |           BORDER            |  |
|  |  +-----------------------+  |  |
|  |  |        PADDING        |  |  |
|  |  |  +-----------------+  |  |  |
|  |  |  |     CONTENT     |  |  |  |
|  |  |  +-----------------+  |  |  |
|  |  +-----------------------+  |  |
|  +-----------------------------+  |
+-----------------------------------+
```

### 9.1. Les balises de type Bloc (`block`)

- Crée automatiquement un retour à la ligne avant et après.
    
- Prend toute la largeur disponible par défaut.
    
- **Exemples :** `<div>`, `<h1>`-`<h6>`, `<p>`, `<ul>`, `<ol>`, `<li>`, `<form>`, `<table>`, `<article>`, `<section>`.
    

### 9.2. Les balises de type En ligne (`inline`)

- Ne crée pas de retour à la ligne (se place à côté du texte/élément adjacent).
    
- La taille dépend uniquement de son contenu (ignore `width` et `height`).
    
- N'applique pas les marges verticales (`margin-top` / `margin-bottom`).
    
- **Exemples :** `<span>`, `<a>`, `<img>`, `<strong>`, `<em>`, `<label>`.
    

### 9.3. Les balises de type `inline-block`

- Se positionne sur la même ligne (comportement inline).
    
- Accepte les dimensions (`width`, `height`) et toutes les marges (comportement block).
    

## 10. Positionnement des éléments

### 10.1. Position Absolue (`absolute`) et Fixe (`fixed`)

- `position: absolute`: Sort l'élément du flux. Se positionne par rapport au premier ancêtre positionné (`relative`, `absolute` ou `fixed`).
    
- `position: fixed`: Positionné par rapport à la fenêtre du navigateur (reste fixe lors du défilement/scroll).
    

### 10.2. Position Relative (`relative`)

Conserve la place d'origine dans le flux et applique un décalage visuel par rapport à sa position initiale (`top`, `bottom`, `left`, `right`).

CSS

```
.jaune {
    position: relative;
    bottom: 5px;
    left: 3em;
    background-color: #ffff00;
}
```

### 10.3. Positionnement par Flottants (`float`)

Permet de placer des éléments côte à côte (ex: image entourée de texte).

- `float: left | right;`
    
- `clear: left | right | both;` : Interrompt le flottement.
    

CSS

```
nav { float: left; width: 150px; min-height: 400px; }
section { min-height: 400px; }
h2 { clear: both; } /* Se place en dessous des éléments flottants */
```

### 10.4. Positionnement en `inline-block`

CSS

```
nav { display: inline-block; width: 150px; height: 400px; vertical-align: top; }
section { display: inline-block; width: 400px; height: 400px; vertical-align: top; }
```

### 10.5. Layouts Modernes : Flexbox & Grid

- `display: flex`: Modèle unidimensionnel (alignement en ligne ou colonne flexible).
    
- `display: grid`: Modèle bidimensionnel (lignes et colonnes).
    

## 11. Pseudo-classes

Permettent de cibler un élément selon son état ou son contexte :

- `:hover` : Au survol du curseur.
    
- `:active` : Lors du clic/maintien.
    
- `:focus` : Quand l'élément reçoit le focus clavier ou clic (champs de texte, liens).
    
- `:checked` : Case à cocher (`checkbox`) ou bouton radio sélectionné.
    
- `:disabled` / `:enabled` : Élément désactivé ou actif.
    
- `:valid` / `:invalid` : Validation de formulaire en temps réel.
    
- `:not(sélecteur)` : Négation (ex: `div:not(#main)`).
    
- `:first-child` / `:last-child` : Premier / dernier enfant du parent.
    
- `:nth-child(n)` : $n^{\text{ème}}$ enfant (ex: `tr:nth-child(even)` ou `tr:nth-child(2n)` pour les lignes paires).
    

### Exemple d'états pour les liens (`<a>`)

CSS

```
a:link { color: red; }      /* Lien non visité */
a:visited { color: blue; }  /* Lien visité */
a:hover { color: green; }   /* Lien survolé */
a:active { color: lime; }    /* Lien au moment du clic */
```

## 12. Sélecteurs avancés

|**Sélecteur**|**Description**|
|---|---|
|`E:first-letter`|Première lettre du texte de l'élément.|
|`E:before` / `E:after`|Insère du contenu généré avant ou après l'élément (via la propriété `content`).|
|`[attribute]`|Présence de l'attribut.|
|`[attribute="valeur"]`|Égalité exacte de valeur.|

## 13. Impression (`@media print`)

Spécifiez une feuille de style uniquement dédiée à l'impression pour masquer les éléments inutiles (menus, pieds de page) :

HTML

```
<link rel="stylesheet" media="print" href="css/print.css">
```

Ou directement dans le fichier CSS :

CSS

```
@media print {
    nav, footer, .no-print {
        display: none !important;
    }
}
```

## 14. Validation d'une feuille de style

Pour vérifier la conformité W3C de votre code CSS :

- Validator officiel W3C : [http://jigsaw.w3.org/css-validator/](http://jigsaw.w3.org/css-validator/)
    

## 15. Media Queries & Design Responsive

Permet d'adapter le style selon le type de média ou la taille de l'écran.

### Syntaxe générale

CSS

```
/* Écrans de taille <= 576px (Smartphones) */
@media (max-width: 576px) {
    body { font-size: 14px; }
}

/* Écrans >= 576px (Tablettes) */
@media (min-width: 576px) { ... }

/* Plage spécifique */
@media (576px <= width <= 768px) { ... }
@media (min-width: 576px) and (max-width: 768px) { ... }

/* Combinaison avec types de média et orientation */
@media screen and (max-width: 800px) {
    .masquer-mobile { display: none; }
}
@media (min-height: 680px), screen and (orientation: portrait) { ... }
```

## 16. La règle `!important`

Force la priorité d'une règle CSS par-dessus toutes les autres définitions, quelle que soit la spécificité du sélecteur.

CSS

```
body {
    background-color: #E1E1E1 !important; /* Prioritaire */
    background-color: #C0C0C0;            /* Ignoré */
}
```

> ⚠️ **Attention :** À utiliser de manière exceptionnelle (ex: surcharge de classes d'un framework externe), car elle rend le débogage complexe.

## 17. Variables CSS (Propriétés personnalisées)

Les variables CSS permettent de stocker des valeurs réutilisables dans tout le document. Déclarées avec le préfixe `--`, elles s'utilisent avec la fonction `var()`.

CSS

```
/* Déclaration globale dans la racine :root */
:root {
    --primary-color: #ffffff;
    --text-color: #282828;
    --text-error: #ff0000;
    --border-color: #68a0dd;
}

body {
    color: var(--text-color);
    background-color: var(--primary-color);
}

.error {
    color: var(--text-error);
    border: 1px solid var(--border-color);
}
```

## 18. Les fonctions CSS

### `calc()`

Calcule dynamiquement des expressions mathématiques mélangeant différentes unités.

CSS

```
body {
    font-size: calc(14px + (18 - 14) * ((100vw - 300px) / (1600 - 300)));
}
.container {
    width: calc(100% - 40px);
}
```

### `min()`

Sélectionne la plus petite valeur parmi les arguments fournis.

CSS

```
.responsive-font {
    font-size: min(5vw, 30px); /* Ne dépassera jamais 30px */
}
```

### `max()`

Sélectionne la plus grande valeur parmi les arguments fournis.

CSS

```
.flexible-width {
    width: max(300px, 50%); /* Fera au minimum 300px */
}
```

### `clamp()`

Maintient une valeur entre une limite minimale et maximale (`clamp(MIN, VALEUR_CIBLE, MAX)`).

CSS

```
.dynamic-padding {
    padding: clamp(10px, 5vw, 50px);
}

:root {
    --taille-police: clamp(12px, 0.5vw + 12px, 20px);
}

html {
    font-size: var(--taille-police);
}
```