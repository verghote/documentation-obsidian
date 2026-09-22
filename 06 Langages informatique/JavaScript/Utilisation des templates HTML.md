## Principe général

Certaines pages doivent afficher une répétition d'éléments ayant tous la même structure :

- cartes ;
- lignes de tableaux personnalisées ;
- blocs d'informations ;
- fiches ;
- composants visuels.

Afin d'éviter de construire ces éléments dynamiquement avec des concaténations HTML dans le JavaScript, l'application utilise le mécanisme natif HTML `<template>`.

Le principe est le suivant :

1. Le HTML définit une structure modèle invisible.
2. Le JavaScript clone cette structure.
3. Les données sont injectées dans le clone.
4. Le composant obtenu est ajouté à la page.

Cette approche permet de conserver la structure HTML dans le fichier `.html` et de garder le JavaScript consacré uniquement au comportement.

# Exemple : affichage de cartes de clubs

## Structure dans `index.html`

Le fichier HTML contient un modèle :

```html
<template id="modeleClub">

    <div class="card carte-club shadow-sm">

        <div class="card-header text-center enteteClub">
            Nom du club
        </div>

        <div class="card-body corpsClub">

            <img class="logoClub"
                 src=""
                 alt=""
                 style="
                    max-height:100%;
                    max-width:100%;
                    object-fit:contain;
                    display:block;
                    margin:auto">

        </div>

        <div class="card-footer text-muted text-center nbLicencies">
            0 licenciés
        </div>

    </div>

</template>
```

## Rôle du template

La balise :

```
<template>
```

contient une structure HTML qui :

- n'est pas affichée directement ;
- n'est pas présente dans le DOM visible ;
- peut être clonée autant de fois que nécessaire.

Le template représente donc un **moule graphique**.

Dans cet exemple, il décrit l'apparence d'une carte représentant un club.

# Récupération du template en JavaScript

Le JavaScript récupère :

- la zone où seront placées les cartes ;
- le modèle servant à créer chaque carte.

```javascript
const lesCartes = document.getElementById('lesCartes');

const modeleClub = document.getElementById('modeleClub');
```

Le template reste inchangé pendant toute la durée de vie de la page.

# Création d'un élément à partir du modèle

La création d'une carte se fait dans une fonction dédiée.

```javascript
function creerCarte(club) {

    const fragment =
        modeleClub.content.cloneNode(true);

    ...
}
```

## Clonage du modèle

La méthode :

```
cloneNode(true)
```

effectue une copie complète du contenu du template.

Le paramètre `true` signifie :

> Copier également tous les éléments enfants.

Le résultat est un `DocumentFragment`, c'est-à-dire un morceau de DOM prêt à être complété puis inséré.

# Alimentation des données

Après clonage, les éléments internes sont récupérés :

```javascript
const entete =
    fragment.querySelector('.enteteClub');

const image =
    fragment.querySelector('.logoClub');

const pied =
    fragment.querySelector('.nbLicencies');
```

Puis les valeurs sont remplacées :

```javascript
entete.textContent = club.nom;

pied.textContent =
    `${club.nb} licenciés`;
```

Le JavaScript ne crée aucune balise HTML.

Il utilise uniquement la structure définie dans le template.

# Gestion des éléments optionnels

Le modèle peut contenir des éléments qui ne sont pas toujours utilisés.

Exemple :

```javascript
if (club.present) {

    image.src =
        `/data/club/${club.fichier}`;

    image.alt =
        `${club.nom} logo`;

} else {

    image.remove();

}
```

Si le club possède un logo :

- l'image est renseignée.

Sinon :

- l'élément `<img>` est supprimé.

Le template reste donc suffisamment générique pour plusieurs situations.

# Ajout dans la page

Une fois le fragment construit :

```javascript
return fragment;
```

Il est ajouté au conteneur principal :

```javascript
for (const club of lesClubs) {

    lesCartes.appendChild(
        creerCarte(club)
    );

}
```

Le navigateur reçoit alors une série de cartes construites à partir du même modèle.

# Pourquoi utiliser `<template>` ?

Cette technique présente plusieurs avantages.

## Séparation claire HTML / JavaScript

Sans template, on serait tenté d'écrire :

```javascript
element.innerHTML =
    "<div class='card'>...</div>";
```

Cette méthode mélange :

- la structure HTML ;
- les données ;
- le comportement.

Avec `<template>` :

- le HTML décrit l'apparence ;
- le JavaScript manipule les données.

## Suppression des duplications

Sans template :

```
<div class="card">
    ...
</div>

<div class="card">
    ...
</div>

<div class="card">
    ...
</div>
```

Le même code HTML est répété plusieurs fois.

Avec un template :

```
<template id="modeleClub">
    ...
</template>
```

Une seule définition suffit.

## Modification graphique simplifiée

Si l'apparence d'une carte change :

Avant :

- modifier le HTML généré dans le JavaScript ;
- rechercher toutes les concaténations concernées.

Avec un template :

- modifier uniquement `index.html`.

Le JavaScript reste inchangé.

# Convention d'utilisation

Dans l'application :

- les templates sont déclarés dans le fichier `index.html` de la page ;
- chaque template possède un identifiant explicite :

```html
<template id="modeleClub">
```

- le JavaScript récupère le modèle correspondant :

```javascript
const modeleClub =
    document.getElementById('modeleClub');
```

- une fonction est créée pour transformer une donnée métier en élément graphique :

```javascript
function creerCarte(donnee)
{
    ...
}
```

# Organisation recommandée

Une page utilisant des templates conserve la même structure générale :

```
clubs/
│
├── index.php
├── index.html
├── index.js
│
└── ajax/
    └── ...
```

Le fichier `index.html` contient :

- la structure générale de la page ;
- les zones d'insertion ;
- les templates.

Le fichier `index.js` contient :

- la récupération des données ;
- le clonage des templates ;
- l'alimentation des éléments ;
- l'insertion dans le DOM.

# Principe architectural

L'utilisation des templates HTML complète naturellement l'architecture générale :

|Élément|Responsabilité|
|---|---|
|`index.php`|Préparer les données métier|
|`interface.php`|Construire la page complète|
|`index.html`|Décrire l'interface et les modèles|
|`<template>`|Définir les composants répétitifs|
|`index.js`|Transformer les données en composants visibles|
|`ajax/*.php`|Fournir les données nécessaires|

Le template HTML devient ainsi un **composant visuel réutilisable**, tout en conservant la philosophie générale du projet : chaque fichier a une responsabilité claire et le code métier reste séparé de la présentation.