## 1. Objet

Cette documentation définit le standard à respecter pour tous les champs de recherche de l'application.

L'objectif est de garantir une présentation et un comportement identiques sur toutes les interfaces, quelle que soit la donnée recherchée ou la technologie utilisée pour proposer les résultats.

Deux types de recherche sont actuellement prévus :

- **recherche par saisie simple** ;
- **recherche avec autocomplétion**.

Ces deux types utilisent la même présentation visuelle et les mêmes règles générales.

# 2. Règles de base 

+ Toute zone de recherche affiche une icône de recherche.

Exemple :

```html
<span class="icone-recherche" aria-hidden="true">⌕</span>
```

+ Chaque champ de recherche dispose d'un `<label>` associé à son champ. Le label est **toujours masqué visuellement**.

Il reste présent pour :

- l'accessibilité ;
- les lecteurs d'écran ;
- l'identification explicite du champ ;
- la maintenance du code.

Exemple :

```html
<label for="licence">
    Numéro de licence
</label>
```

Le CSS `recherche.css` masque automatiquement ce label.

+ Le placeholder doit indiquer clairement ce que l'utilisateur doit saisir.

Exemples :

```html
placeholder="Saisir le n° licence"
```

ou :

```html
placeholder="Saisir les premières lettres du nom"
```

+ Tous les champs de recherche utilisent la même présentation

Les recherches doivent utiliser :

```html
.zone-recherche
.champ-recherche
.input-recherche
.icone-recherche
```

+ La feuille /css/recherche.css contient les styles communs. La hauteur du champ, la couleur de bordure, le rayon des angles, la position de l'icône, le style du focus, le style du placeholder sont centralisés dans `recherche.css`.

# 3. Architecture HTML standard

Une recherche classique doit respecter cette structure :

```html
<div class="zone-recherche">

    <label for="licence">
        Numéro de licence
    </label>

    <div class="champ-recherche">

        <div class="input-recherche">

            <input type="text"
                   id="licence"
                   placeholder="Saisir le n° licence">

            <span class="icone-recherche"
                  aria-hidden="true">⌕</span>

        </div>

    </div>

</div>
```

La hiérarchie est importante :

```text
.zone-recherche
│
├── label
│
└── .champ-recherche
    │
    ├── .input-recherche
    │   │
    │   ├── input
    │   └── .icone-recherche
    │
    └── .messageErreur éventuel
```

`.input-recherche` représente **uniquement la partie visuelle du champ**.
# 5. Deux types de recherche

## 5.1 Recherche par saisie simple

La recherche simple est utilisée lorsque l'utilisateur saisit directement une valeur.

Exemples :

- numéro de licence ;
- numéro de dossier ;
- code ;
- référence ;
- identifiant.

Exemple :

```html
<div class="zone-recherche">

    <label for="licence">
        Numéro de licence
    </label>

    <div class="champ-recherche">

        <div class="input-recherche">

            <input type="text"
                   id="licence"
                   placeholder="Saisir le n° licence">

            <span class="icone-recherche"
                  aria-hidden="true">⌕</span>

        </div>

    </div>

</div>
```

# 6. Recherche avec autocomplétion

La recherche avec autocomplétion conserve exactement la même présentation générale.

La seule différence est que la bibliothèque d'autocomplétion ajoute son propre conteneur autour du champ.

La classe :

```text
recherche-autocomplete
```

permet à `recherche.css` de traiter cette particularité.

Structure conceptuelle :

```text
.zone-recherche.recherche-autocomplete
│
├── label
│
└── .champ-recherche
    │
    └── .input-recherche
        │
        ├── .autoComplete_wrapper
        │   └── input
        │
        └── .icone-recherche
```

La recherche avec autocomplétion doit donc conserver :

- la même largeur ;
- la même hauteur ;
- le même style ;
- le même placeholder ;
- la même icône ;
- le même comportement de focus ;
- le même système de message d'erreur.

Seule la gestion des suggestions diffère.

L'autocomplétion ne doit pas être considérée comme une autre présentation de recherche.

Elle constitue uniquement une variante fonctionnelle.

La règle est donc :

> **Recherche simple et recherche avec autocomplétion ont la même apparence.**

La classe :

```html
recherche-autocomplete
```

sert uniquement à gérer les particularités techniques de la bibliothèque d'autocomplétion.

# 7. Messages d'erreur

Les messages d'erreur sont gérés par la fonction commune :

```javascript
afficherSousLeChamp(inputOuId, message)
```

Exemple :

```javascript
afficherSousLeChamp('licence', 'Le numéro de licence n’existe pas.');
```

Pour supprimer le message :

```javascript
effacerSousLeChamp('licence');
```

Le message d'erreur est placé par la fonction  **sous le champ**.

La structure résultante est :

```html
<div class="champ-recherche">

    <div class="input-recherche">

        <input ...>

        <span class="icone-recherche"
              aria-hidden="true">⌕</span>

    </div>

    <div class="messageErreur">
        Le numéro de licence n'existe pas.
    </div>

</div>
```

Lorsque le champ est invalide, la fonction d'erreur ajoute :

```html
aria-invalid="true"
```

et :

```html
aria-describedby="licence-erreur"
```

Le CSS peut alors modifier l'apparence de la bordure :

```css
input[aria-invalid="true"] {
    border-color: #dc3545;
}
```

Le développeur n'a normalement pas à gérer ces attributs manuellement.

# 8. Dimensions et couleurs

Les dimensions et couleurs communes sont définies dans les variables CSS globales.

Exemple :

```css
:root {

    --entete-bg-start: #0f4c81;
    --entete-bg-end: #1769aa;

    --entete-background:
        linear-gradient(
            135deg,
            var(--entete-bg-start),
            var(--entete-bg-end)
        );

    --champ-background: white;
    --champ-border: #ced4da;
    --champ-focus: var(--entete-bg-end);
    --champ-radius: 8px;
    --champ-height: 42px;
}
```

Le CSS de recherche doit utiliser ces variables plutôt que de redéfinir les valeurs.

Exemple :

```css
border: 1px solid var(--champ-border);
border-radius: var(--champ-radius);
background: var(--champ-background);
height: var(--champ-height);
```

Cela permet de modifier l'apparence globale des recherches depuis un seul endroit.

# 9. Organisation des feuilles CSS

Les responsabilités sont séparées.

## Feuille globale

La feuille CSS générale contient :

- les variables globales ;
- les règles générales de la page ;
- les règles communes aux formulaires ;
- les couleurs et dimensions communes.

Elle définit notamment :

```css
:root {
    --champ-background: white;
    --champ-border: #ced4da;
    --champ-focus: var(--entete-bg-end);
    --champ-radius: 8px;
    --champ-height: 42px;
}
```

## `recherche.css`

Cette feuille contient **tous les styles communs aux champs de recherche** :

```text
.zone-recherche
.champ-recherche
.input-recherche
.icone-recherche
.messageErreur
.recherche-autocomplete
```

## Feuille spécifique à une fonctionnalité

Par exemple :

```text
/coureur/coureur.css
```

doit contenir les styles propres à la fiche coureur :

```text
.fiche-coureur
.fiche-header
.fiche-corps
.bloc-info
.champ
...
```

Elle ne doit pas redéfinir le fonctionnement général des champs de recherche.

# 10. Inclusion des feuilles CSS

Une page utilisant une fiche coureur avec une recherche doit charger :

```html
<link rel="stylesheet" href="/css/recherche.css">
<link rel="stylesheet" href="/coureur/coureur.css">
```

La séparation permet de réutiliser `recherche.css` dans toutes les interfaces.

Chaque fonctionnalité peut ainsi utiliser le même composant de recherche.

# 11. Exemple complet — Recherche par numéro

```html
<div class="zone-recherche">

    <label for="licence">
        Numéro de licence
    </label>

    <div class="champ-recherche">

        <div class="input-recherche">

            <input type="text"
                   id="licence"
                   placeholder="Saisir le n° licence"
                   pattern="[0-9]{6,7}"
                   maxlength="7"
                   minlength="6">

            <span class="icone-recherche"
                  aria-hidden="true">⌕</span>

        </div>

    </div>

</div>
```

# 12. Exemple complet — Recherche par nom avec autocomplétion

```html
<div class="zone-recherche recherche-autocomplete">

    <label for="nomR">
        Nom du licencié
    </label>

    <div class="champ-recherche">

        <div class="input-recherche">

            <!-- AutoComplete.js intervient ici -->

            <input id="nomR"
                   type="text"
                   placeholder="Saisir les premières lettres du nom"
                   autocomplete="off">

            <span class="icone-recherche"
                  aria-hidden="true">⌕</span>

        </div>

    </div>

</div>
```

La bibliothèque d'autocomplétion peut ensuite créer son wrapper sans remettre en cause l'organisation générale.

# 13. Checklist développeur

Avant de valider une nouvelle recherche, vérifier :

- [ ]  Un `<label>` est présent et associé au champ avec `for` / `id`.
- [ ]  Le label est visuellement masqué par `recherche.css`.
- [ ]  Une icône de recherche est toujours présente.
- [ ]  L'icône utilise `aria-hidden="true"`.
- [ ]  Le champ est placé dans `.input-recherche`.
- [ ]  L'icône est placée dans `.input-recherche`.
- [ ]  Le message d'erreur n'est jamais placé dans `.input-recherche`.
- [ ]  Le message d'erreur est géré par `afficherSousLeChamp()`.
- [ ]  Le champ utilise les variables CSS globales.
- [ ]  Les styles communs sont dans `recherche.css`.
- [ ]  Une recherche avec autocomplétion utilise la classe `.recherche-autocomplete`.
- [ ]  Une recherche avec autocomplétion conserve exactement la même apparence qu'une recherche simple.
- [ ]  Aucun style local ne vient modifier inutilement le composant standard.

# 14. Principe général à retenir

Le développeur doit considérer le champ de recherche comme un **composant standard de l'application**.

Il ne doit pas recréer son apparence pour chaque nouvelle interface.

La seule chose qui change d'une recherche à l'autre est :

- l'identifiant du champ ;
- son placeholder ;
- ses contraintes de saisie ;
- la donnée recherchée ;
- éventuellement l'utilisation de l'autocomplétion.

L'apparence reste standardisée.

> **Une recherche = un label accessible masqué + un champ avec icône + éventuellement une autocomplétion + un message d'erreur sous le champ.**

Cette règle constitue le standard de référence pour toutes les futures interfaces de recherche.