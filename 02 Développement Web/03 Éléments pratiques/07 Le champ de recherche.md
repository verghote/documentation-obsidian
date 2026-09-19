
Le composant de recherche permet d'ajouter un champ de saisie permettant à l'utilisateur de rechercher une information dans une page.

Il peut être utilisé seul ou associé à `autoComplete.js` pour proposer des résultats au fur et à mesure de la saisie.

## 1. Champ de recherche simple

Pour ajouter un champ de recherche, utiliser la structure HTML suivante :

```html
<div class="zone-recherche">
    <label for="search">
        Licence, nom ou club
    </label>

    <div class="champ-recherche">
        <div class="input-recherche">
            <input
                type="text"
                id="search"
                placeholder="Licence, nom ou club..."
                autocomplete="off">

            <span
                class="icone-recherche"
                aria-hidden="true">⌕</span>
        </div>
    </div>
</div>
```

Le résultat est un champ de recherche avec :

- un libellé ;
- un champ de saisie ;
- une icône de recherche ;
- un affichage adapté aux différentes tailles d'écran.

### Les classes à utiliser

La mise en forme est assurée par la feuille de style du composant :

```html
<link rel="stylesheet" href="/css/recherche.css">
```

Les principales classes sont :

|Classe|Rôle|
|---|---|
|`zone-recherche`|Conteneur général du champ de recherche.|
|`champ-recherche`|Conteneur du champ.|
|`input-recherche`|Conteneur du champ de saisie et de l'icône.|
|`icone-recherche`|Icône affichée à droite du champ.|

**Il est recommandé de conserver cette structure et ces classes** afin de bénéficier automatiquement du style défini pour l'application.

## 2. Ajouter le champ dans l'en-tête d'une page

Lorsqu'un champ de recherche doit apparaître dans l'en-tête d'une page, il est généralement placé dans `page-header-filters` :

```html
<header class="page-header">

    <div class="page-header-top">
        <h1 class="page-title">
            Rechercher un coureur
        </h1>
    </div>

    <div class="page-header-filters">

        <div class="zone-recherche">

            <label for="search">
                Licence, nom ou club
            </label>

            <div class="champ-recherche">

                <div class="input-recherche">

                    <input
                        type="text"
                        id="search"
                        placeholder="Licence, nom ou club..."
                        autocomplete="off">

                    <span
                        class="icone-recherche"
                        aria-hidden="true">⌕</span>

                </div>

            </div>

        </div>

    </div>

</header>
```

Le développeur n'a pas besoin d'ajouter de CSS spécifique : le composant est déjà prévu pour s'intégrer dans l'en-tête de page.

## 3. Utiliser une recherche avec autocomplétion

Lorsque la recherche doit proposer des résultats pendant la saisie, il faut utiliser `autoComplete.js`.

La structure HTML est légèrement différente.

```html
<div class="zone-recherche recherche-autocomplete">

    <label for="search">
        Nom du licencié
    </label>

    <div class="champ-recherche">

        <div class="input-recherche">

            <div class="autoComplete_wrapper">

                <input
                    id="search"
                    type="text"
                    placeholder="Saisir les premières lettres du nom"
                    autocomplete="off">

            </div>

            <span
                class="icone-recherche"
                aria-hidden="true">⌕</span>

        </div>

    </div>

</div>
```

La classe supplémentaire :

```html
recherche-autocomplete
```

permet d'appliquer les règles spécifiques à une recherche utilisant `autoComplete.js`.

Le conteneur :

```html
<div class="autoComplete_wrapper">
```

est nécessaire au fonctionnement et au positionnement de la liste des suggestions.

## 4. Différence entre les deux types de recherche

Il faut distinguer deux situations.

### Recherche simple

L'utilisateur saisit une valeur et l'application déclenche ensuite le traitement prévu.

```text
Saisie
   ↓
Champ de recherche
   ↓
Traitement de la recherche
```

Structure :

```html
<div class="zone-recherche">
    ...
</div>
```

### Recherche avec autocomplétion

Le composant propose des résultats pendant la saisie.

```text
Saisie
   ↓
autoComplete.js
   ↓
Recherche
   ↓
Liste de suggestions
   ↓
Sélection
```

Structure :

```html
<div class="zone-recherche recherche-autocomplete">
    ...
    <div class="autoComplete_wrapper">
        <input ...>
    </div>
    ...
</div>
```

## 5. Attributs du champ

Quelques attributs HTML sont importants.

|Attribut|Rôle|
|---|---|
|`id`|Identifie le champ et permet notamment de l'associer à son `label`.|
|`type="text"`|Définit un champ de saisie texte.|
|`placeholder`|Indique à l'utilisateur ce qu'il peut rechercher.|
|`autocomplete="off"`|Désactive les suggestions automatiques du navigateur afin de laisser la gestion des suggestions au composant.|
|`pattern`|Permet éventuellement de limiter les caractères acceptés.|

Par exemple :

```html
<input
    type="text"
    id="search"
    placeholder="Licence, nom ou club..."
    pattern="[A-Za-z0-9 -]+"
    autocomplete="off">
```

Le `pattern` est **optionnel**. Il doit être utilisé uniquement lorsque les caractères autorisés sont connus et doivent être contrôlés.

## 6. Le `label`

Chaque champ doit avoir un `label` associé :

```html
<label for="search">
    Nom du licencié
</label>
```

Le `for` du `label` doit correspondre au `id` du champ :

```text
for="search"
     ↓
id="search"
```

Cela permet notamment aux technologies d'assistance d'identifier correctement le champ.

Le texte du label doit expliquer ce que l'utilisateur peut rechercher :

```html
<label for="search">
    Nom du licencié
</label>
```

ou :

```html
<label for="search">
    Licence, nom ou club
</label>
```

## 7. Exemple recommandé

Pour une recherche avec autocomplétion, le modèle à utiliser est donc :

```html
<link rel="stylesheet" href="/css/recherche.css">

<div class="zone-recherche recherche-autocomplete">

    <label for="search">
        Nom du licencié
    </label>

    <div class="champ-recherche">

        <div class="input-recherche">

            <div class="autoComplete_wrapper">

                <input
                    id="search"
                    type="text"
                    placeholder="Saisir les premières lettres du nom"
                    autocomplete="off">

            </div>

            <span
                class="icone-recherche"
                aria-hidden="true">⌕</span>

        </div>

    </div>

</div>
```

Puis la configuration JavaScript d'`autoComplete.js` peut être ajoutée :

```javascript
const autoCompleteJS = new autoComplete({
    selector: "#search",

    data: {
        src: lesCoureurs,
        keys: ["nomPrenom"]
    },

    events: {
        input: {
            selection: (event) => {

                const selection =
                    event.detail.selection.value;

                search.value =
                    selection.nomPrenom;

                // traitement de la sélection
            }
        }
    }
});
```

## 8. Structure à retenir

Pour un développeur, la structure à retenir est simplement :

```text
zone-recherche
│
├── label
│
└── champ-recherche
    │
    └── input-recherche
        │
        ├── input
        │
        └── icone-recherche
```

Avec autocomplétion :

```text
zone-recherche recherche-autocomplete
│
├── label
│
└── champ-recherche
    │
    └── input-recherche
        │
        ├── autoComplete_wrapper
        │   └── input
        │
        └── icone-recherche
```

**En pratique :** le développeur doit principalement reprendre la structure HTML fournie, utiliser `/css/recherche.css`, puis ajouter la logique JavaScript nécessaire à la recherche. La feuille de style prend en charge l'apparence, le positionnement et le comportement responsive du champ.