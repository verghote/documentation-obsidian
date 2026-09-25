## 1. Présentation

### Objectif

Le composant `openclose.js` permet de gérer simplement l'ouverture et la fermeture de cartes (`.card`) dans une interface web.

Il prend en charge automatiquement :

- l'affichage ou le masquage du contenu d'une carte ;
- l'affichage d'une icône indiquant l'état de la carte ;
- le clic sur l'en-tête pour ouvrir ou fermer la carte ;
- la mémorisation de l'état de chaque carte dans le `localStorage` du navigateur ;
- la restauration de l'état des cartes lors d'un nouveau chargement de la page ;
- l'arrondissement de l'en-tête lorsque la carte est fermée ;
- la gestion de cartes présentes directement dans le HTML ;
- la gestion de cartes créées dynamiquement en JavaScript.

Le composant est **auto-suffisant** : les styles nécessaires aux icônes sont injectés automatiquement dans la page lors du chargement du module.

# 2. Principe général

Une carte manipulée par le composant possède généralement cette structure :

```html
<div class="card" id="maCarte">
    <div class="card-header">
        Titre de la carte
    </div>

    <div class="card-body">
        Contenu de la carte
    </div>
</div>
```

Le composant considère :

- `.card` comme la carte ;
- `.card-header` comme son en-tête ;
- le premier élément enfant qui n'est pas l'en-tête comme le contenu à afficher/masquer.

L'utilisateur clique sur l'en-tête pour modifier l'état de la carte.

Lorsque la carte est ouverte :

```text
┌──────────────────────────────┐
│ Titre de la carte        ▲   │
├──────────────────────────────┤
│                              │
│ Contenu                      │
│                              │
└──────────────────────────────┘
```

Lorsque la carte est fermée :

```text
┌──────────────────────────────┐
│ Titre de la carte        ▼   │
└──────────────────────────────┘
```

L'état de la carte est mémorisé dans le navigateur.


# 3. Installation

Le chargement du composant `openclose.js` s'effectue dans le fichier JavaScript de la page

```javascript
import {    initialiserToutesLesCartes} from '/composant/fonction/openclose.js';

initialiserToutesLesCartes();
```

# 4. Utilisation sur une page statique

## 4.1. Cas simple

Lorsque les cartes sont directement présentes dans le HTML de la page, il suffit d'appeler :

```javascript
initialiserToutesLesCartes();
```

### Exemple complet

```html
<div class="card" id="presentation">
    <div class="card-header">
        Présentation
    </div>

    <div class="card-body">
        Voici le contenu de la présentation.
    </div>
</div>

<div class="card" id="informations">
    <div class="card-header">
        Informations
    </div>

    <div class="card-body">
        Voici les informations complémentaires.
    </div>
</div>

<script type="module">
    import {
        initialiserToutesLesCartes
    } from '/js/components/openclose.js';

    initialiserToutesLesCartes();
</script>
```

Les deux cartes deviennent alors automatiquement ouvrables et fermables.

# 5. Identifier les cartes

Il est recommandé de donner un identifiant (`id`) aux cartes.

Par exemple :

```html
<div class="card" id="presentation">
```

```html
<div class="card" id="informations">
```

```html
<div class="card" id="configuration">
```

Cet identifiant est utilisé comme clé pour mémoriser l'état de la carte dans le `localStorage`.

Par exemple, le navigateur peut mémoriser une information équivalente à :

```json
{
    "presentation": true,
    "informations": false,
    "configuration": true
}
```

Dans cet exemple :

- `presentation` est ouverte ;
- `informations` est fermée ;
- `configuration` est ouverte.
    
### Pourquoi utiliser un `id` ?

L'utilisation d'un identifiant stable permet de conserver l'état d'une carte même après le rechargement de la page.

Il est donc préférable d'utiliser :

```html
<div class="card" id="client">
```

plutôt que de compter uniquement sur la position de la carte dans le document.

# 6. Choisir l'état initial d'une carte

L'état initial dépend de deux choses :

1. un état éventuellement déjà mémorisé dans le `localStorage` ;
2. l'état d'affichage défini dans le HTML/CSS si aucun état n'est mémorisé.

## 6.1. Carte ouverte par défaut

Pour créer une carte ouverte par défaut :

```html
<div class="card" id="presentation">
    <div class="card-header">
        Présentation
    </div>

    <div class="card-body">
        Contenu visible par défaut.
    </div>
</div>
```

Le contenu étant visible, le composant considérera la carte comme ouverte lors de sa première initialisation.

## 6.2. Carte fermée par défaut

Pour créer une carte fermée par défaut :

```html
<div class="card" id="presentation">
    <div class="card-header">
        Présentation
    </div>

    <div class="card-body" style="display: none;">
        Contenu masqué par défaut.
    </div>
</div>
```

Lors de la première initialisation, le composant détecte que le contenu est masqué et initialise la carte comme fermée.

# 7. Attention à la mémorisation de l'état

Il est important de distinguer **l'état initial défini dans le HTML** et **l'état déjà mémorisé**.

Le fonctionnement est le suivant :

```text
                Initialisation
                      │
                      ▼
          Existe-t-il un état mémorisé ?
                 /             \
               Oui              Non
                │                │
                ▼                ▼
       Utiliser l'état       Examiner le
        du localStorage      HTML/CSS
```

Ainsi, si l'utilisateur a déjà utilisé la page, l'état mémorisé est prioritaire.

### Exemple

La carte est initialement définie comme ouverte :

```html
<div class="card-body">
```

L'utilisateur la ferme.

Le composant mémorise :

```text
presentation = false
```

Si l'utilisateur recharge la page, la carte restera fermée, même si son HTML ne contient pas `display: none`.

# 8. Modifier l'état initial après utilisation

Si l'état d'une carte a déjà été mémorisé, modifier simplement le HTML ne suffit donc pas toujours à changer son état lors des prochains chargements.

Pour remettre toutes les cartes à leur comportement initial, il faut utiliser :

```javascript
import {reinitialiserEtatsCartes} from '/composant/fonction/openclose.js';

reinitialiserEtatsCartes();
```

Cette fonction supprime les états mémorisés.

Au prochain appel de :

```javascript
initialiserToutesLesCartes();
```

les cartes repartiront de leur état défini dans le HTML/CSS.

# 9. Initialiser uniquement certaines cartes

Il n'est pas obligatoire d'initialiser toutes les cartes de la page.

Il est possible de sélectionner les cartes à gérer à partir de leurs identifiants :

```javascript
import {initialiserCartesParIds} from '/composant/fonction/openclose.js';

initialiserCartesParIds([
    'presentation',
    'informations',
    'configuration'
]);
```

Dans cet exemple, seules ces trois cartes sont initialisées.

Cette approche est particulièrement intéressante lorsque la page contient d'autres éléments utilisant également la classe `.card` mais qui ne doivent pas être contrôlés par le composant.

# 10. Utilisation sur une page dynamique

Le composant peut également être utilisé lorsque les cartes sont créées après le chargement initial de la page.

Par exemple :

```javascript
const carte = document.createElement('div');

carte.className = 'card';
carte.id = 'maCarteDynamique';

carte.innerHTML = `
    <div class="card-header">
        Carte dynamique
    </div>

    <div class="card-body">
        Contenu créé dynamiquement.
    </div>
`;

document.querySelector('#conteneur').appendChild(carte);
```

À ce stade, la carte existe dans le DOM mais elle n'est pas encore gérée par `openclose.js`.

Il faut l'initialiser :

```javascript
import {initialiserCarte} from '/composant/fonction/openclose.js';

initialiserCarte(carte);
```

La carte devient alors immédiatement interactive.

# 11. Exemple complet avec création dynamique

HTML :

```html
<div id="conteneur"></div>
```

JavaScript :

```javascript
import { initialiserCarte} from '/composant/fonction/openclose.js';

function creerCarte(id, titre, contenu) {
    const carte = document.createElement('div');

    carte.className = 'card';
    carte.id = id;

    carte.innerHTML = `
        <div class="card-header">
            ${titre}
        </div>

        <div class="card-body">
            ${contenu}
        </div>
    `;

    document
        .getElementById('conteneur')
        .appendChild(carte);

    initialiserCarte(carte);

    return carte;
}

creerCarte(
    'client',
    'Client',
    'Informations concernant le client.'
);

creerCarte(
    'commande',
    'Commande',
    'Informations concernant la commande.'
);
```

Chaque carte créée est immédiatement prise en charge par le composant.

# 12. Créer dynamiquement une carte fermée

Pour créer une carte dynamique fermée dès sa création, le contenu doit être masqué avant l'appel à `initialiserCarte()` :

```javascript
function creerCarteFermee(id, titre, contenu) {
    const carte = document.createElement('div');

    carte.className = 'card';
    carte.id = id;

    carte.innerHTML = `
        <div class="card-header">
            ${titre}
        </div>

        <div class="card-body" style="display: none;">
            ${contenu}
        </div>
    `;

    document
        .getElementById('conteneur')
        .appendChild(carte);

    initialiserCarte(carte);

    return carte;
}
```

La carte sera donc initialement fermée, sauf si un état a déjà été mémorisé pour son identifiant.

# 13. Créer dynamiquement une carte ouverte

Pour une carte ouverte par défaut :

```javascript
function creerCarteOuverte(id, titre, contenu) {
    const carte = document.createElement('div');

    carte.className = 'card';
    carte.id = id;

    carte.innerHTML = `
        <div class="card-header">
            ${titre}
        </div>

        <div class="card-body">
            ${contenu}
        </div>
    `;

    document
        .getElementById('conteneur')
        .appendChild(carte);

    initialiserCarte(carte);

    return carte;
}
```

Le contenu étant visible au moment de l'initialisation, la carte sera considérée comme ouverte, à condition qu'aucun état ne soit déjà enregistré pour cet identifiant.

# 14. Ouvrir ou fermer toutes les cartes

Le composant fournit également une fonction permettant de modifier l'état de toutes les cartes.

## Fermer toutes les cartes

```javascript
import {basculerToutesLesCartes} from '/composant/fonction/openclose.js';

basculerToutesLesCartes(false);
```
## Ouvrir toutes les cartes

```javascript
basculerToutesLesCartes(true);
```

L'état est également mémorisé.

# 15. Exemple avec des boutons « Tout ouvrir » / « Tout fermer »

```html
<button id="ouvrirTout">
    Tout ouvrir
</button>

<button id="fermerTout">
    Tout fermer
</button>
```

```javascript
import {initialiserToutesLesCartes, basculerToutesLesCartes} from '/composant/fonction/openclose.js';

initialiserToutesLesCartes();

document
    .getElementById('ouvrirTout')
    .addEventListener('click', () => {
        basculerToutesLesCartes(true);
    });

document
    .getElementById('fermerTout')
    .addEventListener('click', () => {
        basculerToutesLesCartes(false);
    });
```

# 16. API du composant

Le composant expose les fonctions suivantes.

| Fonction                       | Utilisation                                    |
| ------------------------------ | ---------------------------------------------- |
| `initialiserCarte()`           | Initialise une carte                           |
| `initialiserToutesLesCartes()` | Initialise toutes les `.card` de la page       |
| `initialiserCartesParIds()`    | Initialise uniquement les cartes indiquées     |
| `obtenirElementsCarte()`       | Récupère les éléments constitutifs d'une carte |
| `basculerAffichage()`          | Ouvre, ferme ou inverse l'état d'une carte     |
| `basculerToutesLesCartes()`    | Ouvre ou ferme toutes les cartes               |
| `reinitialiserEtatsCartes()`   | Supprime les états mémorisés                   |

# 17. `initialiserCarte()`

```javascript
initialiserCarte(carte);
```

Initialise une carte individuelle.

Exemple :

```javascript
const carte = document.getElementById('presentation');

initialiserCarte(carte);
```

La fonction peut également recevoir un index :

```javascript
initialiserCarte(carte, 0);
```

L'index permet notamment d'identifier une carte qui ne possède pas d'`id`.

**Recommandation :** donner systématiquement un `id` aux cartes afin de garantir une mémorisation stable de leur état.

# 18. `initialiserToutesLesCartes()`

```javascript
initialiserToutesLesCartes();
```

Recherche toutes les cartes :

```css
.card
```

présentes dans le document et les initialise.

C'est la méthode la plus simple pour une page statique.

# 19. `initialiserCartesParIds()`

```javascript
initialiserCartesParIds([
    'carte1',
    'carte2'
]);
```

Cette fonction permet de contrôler précisément les cartes à initialiser.

Elle est particulièrement adaptée aux pages complexes contenant de nombreux composants Bootstrap utilisant la classe `.card`.

# 20. `basculerAffichage()`

Cette fonction permet de contrôler directement l'affichage d'une carte.

```javascript
basculerAffichage(
    contenu,
    icone,
    id
);
```

Sans quatrième paramètre, l'état est inversé :

```javascript
basculerAffichage(contenu, icone, id);
```

Pour forcer l'ouverture :

```javascript
basculerAffichage(
    contenu,
    icone,
    id,
    true
);
```

Pour forcer la fermeture :

```javascript
basculerAffichage(
    contenu,
    icone,
    id,
    false
);
```

# 21. `basculerToutesLesCartes()`

Pour ouvrir toutes les cartes :

```javascript
basculerToutesLesCartes(true);
```

Pour fermer toutes les cartes :

```javascript
basculerToutesLesCartes(false);
```

# 22. `reinitialiserEtatsCartes()`

Cette fonction efface la mémorisation des états :

```javascript
reinitialiserEtatsCartes();
```

Elle supprime la clé :

```text
etatCartes
```

du `localStorage`.

Après cette opération, les cartes ne disposent plus d'un état mémorisé.

# 23. Icônes

Le composant ajoute automatiquement une icône dans l'en-tête lorsqu'elle n'est pas déjà présente.

Deux classes sont utilisées :

```css
.iconeOuvrir
```

et :

```css
.iconeFermer
```

Le composant injecte automatiquement les styles suivants :

```css
.iconeOuvrir::before {
    content: '▼';
}

.iconeFermer::before {
    content: '▲';
}
```

Ainsi :

- `▼` indique que la carte est fermée et peut être ouverte ;
- `▲` indique que la carte est ouverte et peut être fermée.

Aucune feuille CSS supplémentaire n'est nécessaire pour ces icônes.

# 24. En-tête arrondi lorsque la carte est fermée

Lorsque la carte est fermée, le composant ajoute automatiquement :

```css
arrondi-seul
```

sur le `.card-header`.

Le style injecté produit :

```css
.card-header.arrondi-seul {
    border-bottom-left-radius: 0.375rem;
    border-bottom-right-radius: 0.375rem;
}
```

Cela permet d'obtenir un rendu cohérent avec une carte Bootstrap dont le corps est masqué.

Lorsque la carte est rouverte, cette classe est automatiquement supprimée.

# 25. Mémorisation dans le navigateur

Le composant utilise le `localStorage`.

La clé utilisée est :

```text
etatCartes
```

La mémorisation est réalisée automatiquement à chaque changement d'état.

L'utilisateur n'a donc aucune action particulière à effectuer.

Exemple :

1. l'utilisateur ouvre une carte ;
2. l'état est enregistré ;
3. l'utilisateur recharge la page ;
4. la carte retrouve son état précédent.

Cette mémorisation est propre au navigateur utilisé.

Par conséquent, un autre navigateur ou un autre poste ne possède pas nécessairement les mêmes états.

# 26. Recommandations pour une utilisation fiable

### Toujours donner un `id` aux cartes

Préférer :

```html
<div class="card" id="detailsClient">
```

à :

```html
<div class="card">
```

### Utiliser des identifiants stables

Un identifiant comme :

```text
client
```

est préférable à un identifiant généré à partir de la position de la carte.

### Définir clairement l'état initial dans le HTML

Carte ouverte :

```html
<div class="card-body">
```

Carte fermée :

```html
<div class="card-body" style="display: none;">
```

### Ne pas oublier l'initialisation des cartes dynamiques

Après avoir ajouté une carte au DOM :

```javascript
initialiserCarte(carte);
```

est nécessaire.

# 27. Exemple complet recommandé

## HTML

```html
<div class="card" id="presentation">
    <div class="card-header">
        Présentation
    </div>

    <div class="card-body">
        Contenu visible au premier affichage.
    </div>
</div>

<div class="card" id="options">
    <div class="card-header">
        Options
    </div>

    <div class="card-body" style="display: none;">
        Contenu masqué au premier affichage.
    </div>
</div>
```

## JavaScript

```javascript
import {initialiserToutesLesCartes} from '/composant/fonction/openclose.js';

initialiserToutesLesCartes();
```

Dans cet exemple :

- `presentation` est ouverte par défaut ;    
- `options` est fermée par défaut ;    
- chaque carte peut être ouverte ou fermée en cliquant sur son en-tête ;
- l'état est mémorisé automatiquement ;
- après un rechargement, l'état précédemment utilisé par l'utilisateur est restauré.

# 28. Résumé

Le fonctionnement du composant peut être résumé ainsi :

```text
                openclose.js
                     │
                     ▼
              Recherche les cartes
                     │
                     ▼
             Lit le localStorage
                     │
          ┌──────────┴──────────┐
          │                     │
    État mémorisé          Pas d'état
          │                     │
          ▼                     ▼
   Utilise cet état       Utilise l'état
                          défini dans HTML
          │                     │
          └──────────┬──────────┘
                     ▼
             Initialise la carte
                     │
                     ▼
       Clic sur l'en-tête de carte
                     │
                     ▼
            Ouvre / ferme la carte
                     │
                     ▼
          Mémorise le nouvel état
```

Le principe essentiel est donc :

> **Le HTML définit l'état initial d'une carte tant qu'aucun état n'a déjà été mémorisé. Ensuite, le `localStorage` devient prioritaire.**

Pour une **page statique**, l'utilisation recommandée est :

```javascript
initialiserToutesLesCartes();
```

Pour une **carte créée dynamiquement**, l'utilisation recommandée est :

```javascript
initialiserCarte(carte);
```

Pour une carte **ouverte par défaut**, son contenu doit être visible au moment de l'initialisation.

Pour une carte **fermée par défaut**, son contenu doit être masqué au moment de l'initialisation :

```html
style="display: none;"
```

Enfin, pour repartir des états définis dans le HTML et ignorer les préférences précédemment mémorisées :

```javascript
reinitialiserEtatsCartes();
```