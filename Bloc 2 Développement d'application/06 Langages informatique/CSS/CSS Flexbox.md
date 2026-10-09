## 1. Qu'est-ce que Flexbox ?

**Flexbox** est un système de mise en page CSS permettant d'organiser des éléments sur une ligne ou une colonne et de contrôler leur taille, leur alignement et leur répartition dans l'espace disponible.

Flexbox fonctionne avec deux types d'éléments :

- le **conteneur flex** : l'élément auquel on applique `display: flex`
- les **éléments flex** : ses enfants directs

Exemple :

```css
.conteneur {
    display: flex;
}
```

```html
<div class="conteneur">
    <div>Élément 1</div>
    <div>Élément 2</div>
    <div>Élément 3</div>
</div>
```

Les trois `div` deviennent alors des **éléments flex**.

# 2. Le conteneur Flexbox

Le conteneur est l'élément qui possède :

```css
display: flex;
```

Exemple :

```css
.fiche-header {
    display: flex;
}
```

Dans ton interface, `.fiche-header` est donc le **conteneur flex**.

Ses enfants directs :

```html
<label>N° Licence</label>
<input id="licence">
```

sont des **éléments flex**.

# 3. L'axe principal et l'axe secondaire

Flexbox travaille avec deux axes.

Par défaut :

```css
flex-direction: row;
```

L'axe principal est horizontal.

```text
Axe principal →
────────────────────────────>

[ élément ] [ élément ] [ élément ]
```

L'axe secondaire est vertical :

```text
        ↑
        │
        │ axe secondaire
        │
[ élément ][ élément ][ élément ]
```

Si on utilise :

```css
flex-direction: column;
```

les axes sont inversés.

```text
[ élément ]
     ↓
[ élément ]
     ↓
[ élément ]
```

# 4. `flex-direction`

Détermine la direction des éléments.

## Horizontal

```css
flex-direction: row;
```

Résultat :

```text
[ A ] [ B ] [ C ]
```

C'est la valeur par défaut.

## Horizontal inversé

```css
flex-direction: row-reverse;
```

```text
[ C ] [ B ] [ A ]
```

## Vertical

```css
flex-direction: column;
```

```text
[ A ]
[ B ]
[ C ]
```

## Vertical inversé

```css
flex-direction: column-reverse;
```

```text
[ C ]
[ B ]
[ A ]
```

# 5. `justify-content`

`justify-content` contrôle la position des éléments **sur l'axe principal**.

Avec :

```css
display: flex;
flex-direction: row;
```

l'axe principal est horizontal.

## `flex-start`

```css
justify-content: flex-start;
```

```text
[ A ][ B ][ C ]----------------
```

Les éléments sont placés au début.

## `center`

```css
justify-content: center;
```

```text
--------[ A ][ B ][ C ]--------
```

## `flex-end`

```css
justify-content: flex-end;
```

```text
----------------[ A ][ B ][ C ]
```

## `space-between`

```css
justify-content: space-between;
```

```text
[ A ]--------[ B ]--------[ C ]
```

L'espace disponible est réparti **entre** les éléments.

## `space-around`

```css
justify-content: space-around;
```

Les éléments disposent d'un espace autour d'eux.

## `space-evenly`

```css
justify-content: space-evenly;
```

Les espaces sont uniformes.

# 6. `align-items`

`align-items` contrôle l'alignement des éléments **sur l'axe secondaire**.

Dans une ligne horizontale :

```css
flex-direction: row;
```

il contrôle donc l'alignement vertical.

Exemple :

```css
.fiche-header {
    display: flex;
    align-items: center;
}
```

Cela permet notamment de centrer verticalement le label et le champ.

Valeurs courantes :

```css
align-items: flex-start;
align-items: center;
align-items: flex-end;
align-items: stretch;
```

Exemple :

```text
        [ champ ]
        ↑
      center
        ↓
        [ label ]
```

# 7. `gap`

`gap` définit l'espace entre les éléments.

```css
.fiche-header {
    display: flex;
    gap: 1rem;
}
```

Avec :

```text
[ LABEL ]  1rem  [ INPUT ]
```

C'est généralement préférable à l'utilisation de `margin` pour créer l'espace entre les éléments flex.

On peut également utiliser :

```css
column-gap: 1rem;
row-gap: 1rem;
```

ou :

```css
gap: 1rem 2rem;
```

où la première valeur correspond à l'espace vertical et la deuxième à l'espace horizontal selon le contexte.

# 8. `flex-wrap`

Par défaut, les éléments flex restent sur une seule ligne :

```css
flex-wrap: nowrap;
```

Si l'espace devient insuffisant, on peut autoriser le retour à la ligne :

```css
flex-wrap: wrap;
```

Exemple :

```text
[ A ][ B ][ C ][ D ]
[ E ][ F ]
```

Au lieu de forcer tous les éléments sur la même ligne.

# 9. Les trois propriétés fondamentales des éléments flex

Un élément flex peut être contrôlé avec trois propriétés :

```css
flex-grow
flex-shrink
flex-basis
```

Elles sont généralement regroupées avec la propriété raccourcie :

```css
flex
```

La syntaxe est :

```css
flex: grow shrink basis;
```

Donc :

```css
flex: 0 0 250px;
```

signifie :

```css
flex-grow: 0;
flex-shrink: 0;
flex-basis: 250px;
```

# 10. `flex-grow`

Contrôle la capacité d'un élément à **grandir pour utiliser l'espace disponible**.

Exemple :

```css
.element {
    flex-grow: 1;
}
```

Un élément avec `flex-grow: 1` peut récupérer de l'espace libre.

Exemple :

```text
Conteneur
┌─────────────────────────────────────────┐
│ [ A ] [           B           ]         │
└─────────────────────────────────────────┘
```

Si B possède :

```css
flex-grow: 1;
```

il peut prendre l'espace restant.

Avec :

```css
flex-grow: 0;
```

il ne grandit pas.

# 11. `flex-shrink`

Contrôle la capacité d'un élément à **rétrécir lorsque l'espace manque**.

La valeur par défaut est généralement :

```css
flex-shrink: 1;
```

Cela signifie que l'élément peut rétrécir.

Avec :

```css
flex-shrink: 0;
```

on interdit ce rétrécissement.

Exemple :

```css
input {
    flex-shrink: 0;
}
```

Le navigateur ne réduira pas automatiquement la taille de l'élément pour essayer de faire tenir les autres éléments.

# 12. `flex-basis`

`flex-basis` définit la **taille de base** d'un élément avant la répartition de l'espace disponible.

Exemple :

```css
input {
    flex-basis: 250px;
}
```

La taille de base est alors de `250px`.

`flex-basis` dépend de l'axe principal.

Avec :

```css
flex-direction: row;
```

il correspond essentiellement à la largeur de départ.

Avec :

```css
flex-direction: column;
```

il correspond essentiellement à la hauteur de départ.

# 13. La propriété raccourcie `flex`

Au lieu d'écrire :

```css
flex-grow: 0;
flex-shrink: 0;
flex-basis: 250px;
```

on peut écrire :

```css
flex: 0 0 250px;
```

L'ordre est toujours :

```text
flex: grow shrink basis;
       │     │      │
       │     │      └── taille de base
       │     └───────── capacité à rétrécir
       └─────────────── capacité à grandir
```


# 14. Comprendre `flex: 0 0 250px`

C'est particulièrement important dans ton interface.

```css
#licence {
    flex: 0 0 250px;
}
```

signifie :

```css
flex-grow: 0;
flex-shrink: 0;
flex-basis: 250px;
```

Donc :

> Le champ commence à 250 px, ne grandit pas et ne rétrécit pas.

On peut le visualiser ainsi :

```text
Conteneur de 800 px

┌────────────────────────────────────────────────────────────┐
│                                                            │
│ N° Licence   [       champ de 250 px       ]              │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

L'espace restant n'est pas ajouté au champ.

# 15. `flex: 1`

Une notation très fréquente est :

```css
flex: 1;
```

Elle rend l'élément très flexible.

Elle revient généralement à :

```css
flex-grow: 1;
flex-shrink: 1;
flex-basis: 0%;
```

Exemple :

```css
.colonne {
    flex: 1;
}
```

Avec deux éléments :

```text
┌─────────────────────────────────────────┐
│       A        │        B               │
│                │                        │
│      50 %      │       50 %             │
└─────────────────────────────────────────┘
```

Si les deux éléments ont :

```css
flex: 1;
```

ils se partagent l'espace disponible.

# 16. `flex: auto`

```css
flex: auto;
```

correspond à :

```css
flex: 1 1 auto;
```

L'élément peut :

- grandir ;
- rétrécir ;
- utiliser sa largeur/hauteur définie comme base.

---

# 17. `flex: none`

```css
flex: none;
```

correspond à :

```css
flex: 0 0 auto;
```

L'élément ne grandit pas et ne rétrécit pas.

Sa taille dépend alors notamment de ses propriétés `width` et `height`.

# 18. `width` et `flex-basis`

Dans un conteneur Flexbox, il faut distinguer :

```css
width: 250px;
```

et :

```css
flex-basis: 250px;
```

`width` définit une largeur.

`flex-basis` définit la taille de base utilisée par Flexbox sur l'axe principal.

Par exemple :

```css
input {
    width: 250px;
    flex: 0 0 250px;
}
```

est très explicite :

> Le champ doit avoir une base de 250 px et ne doit ni grandir ni rétrécir.



# 19. Application à ton interface

Ton HTML est :

```html
<div class="fiche-header">

    <label for="licence">
        N° Licence
    </label>

    <input
        type="text"
        id="licence"
        placeholder="Saisir le n° licence"
    >

</div>
```

Ton conteneur :

```css
.fiche-header {
    display: flex;
    align-items: center;
    justify-content: flex-start;
    gap: 1rem;
}
```

Le label :

```css
.fiche-header label {
    white-space: nowrap;
    flex-shrink: 0;
}
```

Le champ :

```css
#licence {
    width: 250px;
    flex: 0 0 250px;
    box-sizing: border-box;
}
```

Tu obtiens :

```text
┌───────────────────────────────────────────────────────────┐
│                                                           │
│ N° Licence    [       Saisir le n° licence       ]       │
│               <----------- 250 px ----------->            │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

Le champ ne prend donc pas automatiquement tout l'espace disponible.

---

# 20. Responsive avec Flexbox

Sur un écran étroit, tu peux changer la direction :

```css
@media screen and (max-width: 600px) {

    .fiche-header {
        flex-direction: column;
    }

    #licence {
        width: 100%;
        max-width: 100%;
        flex: 0 0 auto;
    }
}
```

Sur ordinateur :

```text
N° Licence    [ champ ]
```

Sur mobile :

```text
N° Licence

[          champ          ]
```

---

# 21. Les propriétés du conteneur

Les principales propriétés Flexbox appliquées au **parent** sont :

|Propriété|Rôle|
|---|---|
|`display: flex`|Active Flexbox|
|`flex-direction`|Définit la direction|
|`justify-content`|Aligne sur l'axe principal|
|`align-items`|Aligne sur l'axe secondaire|
|`flex-wrap`|Autorise le retour à la ligne|
|`gap`|Définit l'espace entre les éléments|
|`align-content`|Gère plusieurs lignes Flexbox|

Exemple :

```css
.conteneur {
    display: flex;
    flex-direction: row;
    justify-content: flex-start;
    align-items: center;
    gap: 1rem;
    flex-wrap: nowrap;
}
```

---

# 22. Les propriétés des éléments

Les principales propriétés appliquées aux **enfants** sont :

|Propriété|Rôle|
|---|---|
|`flex`|Contrôle global de la flexibilité|
|`flex-grow`|Autorise la croissance|
|`flex-shrink`|Autorise le rétrécissement|
|`flex-basis`|Définit la taille de base|
|`align-self`|Modifie l'alignement d'un élément|
|`order`|Modifie l'ordre d'affichage|

---

# 23. Mémo rapide

### Conteneur

```css
display: flex;
```

Active Flexbox.

### Direction

```css
flex-direction: row;
```

Horizontal.

```css
flex-direction: column;
```

Vertical.

### Alignement horizontal

```css
justify-content: center;
```

### Alignement vertical

```css
align-items: center;
```

### Espacement

```css
gap: 1rem;
```

### Élément flexible

```css
flex: 1;
```

Prend l'espace disponible.

### Élément fixe

```css
flex: 0 0 250px;
```

250 px, sans grandir ni rétrécir.

---

# 24. La règle à retenir

La propriété :

```css
flex
```

se lit toujours :

```text
flex: GROW  SHRINK  BASIS;
       │       │       │
       │       │       └── taille de départ
       │       └────────── peut rétrécir ?
       └────────────────── peut grandir ?
```

Exemple :

```css
flex: 0 0 250px;
```

se lit :

```text
0       → ne grandit pas
0       → ne rétrécit pas
250px   → taille de base de 250px
```

C'est exactement pourquoi cette déclaration est utile pour ton champ `#licence`.

---

## Ressources

- [Documentation MDN — Flexbox](https://developer.mozilla.org/fr/docs/Learn_web_development/Core/CSS_layout/Flexbox)
- [Documentation MDN — propriété `flex`](https://developer.mozilla.org/fr/docs/Web/CSS/Reference/Properties/flex)
- [Documentation MDN — `flex-basis`](https://developer.mozilla.org/fr/docs/Web/CSS/Reference/Properties/flex-basis)
- [Documentation MDN — contrôle des proportions des éléments Flex](https://developer.mozilla.org/fr/docs/Web/CSS/Guides/Flexible_box_layout/Controlling_flex_item_ratios)