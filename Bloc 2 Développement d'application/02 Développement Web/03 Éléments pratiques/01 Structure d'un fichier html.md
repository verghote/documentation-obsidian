## 1. Objectif

Toutes les nouvelles interfaces de l'application doivent respecter une structure HTML commune.

L'objectif est de garantir :

- une présentation homogène de toutes les interfaces ;
- une gestion centralisée du style dans les feuilles CSS communes ;
- une séparation claire entre la structure générale et les fonctionnalités spécifiques ;
- un comportement responsive cohérent ;
- une maintenance simplifiée ;
- l'ajout de nouvelles interfaces sans recréer leur propre système de carte, d'entête ou de filtres.

La structure générale repose sur le composant :

```html
<div class="page-card">
```

Le développeur doit donc **réutiliser cette structure** plutôt que créer une nouvelle classe de carte ou une nouvelle organisation d'entête.

# 2. Structure générale obligatoire

La structure de référence est :

```html
<div class="page-card">

    <header class="page-header">

        <div class="page-header-top">

            <h1 class="page-title">
                Titre de la page
            </h1>

            <div class="page-header-actions">
                <!-- Actions éventuelles -->
            </div>

        </div>

        <div class="page-header-filters">
            <!-- Filtres ou recherche éventuels -->
        </div>

    </header>

    <div class="page-body">
        <!-- Contenu principal -->
    </div>

</div>
```

Les éléments suivants sont obligatoires dans une page standard :

- `.page-card`
- `.page-header`
- `.page-header-top`
- `.page-title`
- `.page-body`

Les éléments suivants sont facultatifs :

- `.page-header-actions`
- `.page-header-filters`

> **Important : il n'existe pas de `page-footer` dans la structure de la carte.**
> 
> La carte est intégrée dans la page générale de l'application, laquelle possède déjà sa propre entête et son propre pied de page.

# 3. La carte principale : `.page-card`

La classe `.page-card` représente l'interface fonctionnelle affichée dans la page.

```html
<div class="page-card">
    ...
</div>
```

Elle fournit notamment :

- le fond blanc ;
- la bordure ;
- les coins arrondis ;
- la largeur générale ;
- la gestion du débordement.

# 4. L'entête : `.page-header`

L'entête contient les éléments permettant d'identifier et, éventuellement, de contrôler l'interface.

```html
<header class="page-header">
    ...
</header>
```

L'entête est organisée en deux zones :

```text
.page-header
│
├── .page-header-top
│
└── .page-header-filters
```

La première contient le **titre et les actions**.

La seconde contient les **filtres et les recherches**.

Cette séparation doit être conservée sur toutes les interfaces.

# 5. Première ligne : titre et actions

La première ligne utilise :

```html
<div class="page-header-top">
```

Elle contient obligatoirement le titre et peut éventuellement contenir des actions.

```text
.page-header-top
│
├── .page-title
│
└── .page-header-actions   ← facultatif
```

# 6. Le titre : `.page-title`

Le titre de l'interface est défini avec un élément `<h1>` :

```html
<h1 class="page-title">
    Liste des coureurs
</h1>
```

Le `<h1>` doit être conservé même lorsque le titre n'est pas affiché visuellement.

Dans ce cas, utiliser :

```html
<h1 class="page-title masquer">
    Liste des coureurs
</h1>
```

La classe `masquer` permet de masquer le titre tout en conservant l'information dans le HTML.

# 7. Schéma A — Titre seul

Lorsqu'une interface possède uniquement un titre :

```html
<div class="page-card">

    <header class="page-header">

        <div class="page-header-top">

            <h1 class="page-title">
                Liste des clubs
            </h1>

        </div>

    </header>

    <div class="page-body">
        ...
    </div>

</div>
```

C'est le **schéma minimal**.

# 8. Les actions : `.page-header-actions`

Une action correspond à une opération explicitement déclenchée par l'utilisateur.

Exemples :

- Ajouter ;
- Modifier ;
- Supprimer ;
- Exporter ;
- Télécharger ;
- Imprimer ;
- Actualiser.

Les actions sont placées dans :

```html
<div class="page-header-actions">
```

Exemple :

```html
<div class="page-header-top">

    <h1 class="page-title">
        Liste des catégories
    </h1>

    <div class="page-header-actions">

        <button
                type="button"
                id="btnPdf"
                class="btn btn-primary">
            Télécharger
        </button>

    </div>

</div>
```

# 9. Schéma B — Titre + actions

```html
<div class="page-card">

    <header class="page-header">

        <div class="page-header-top">

            <h1 class="page-title">
                Liste des catégories
            </h1>

            <div class="page-header-actions">

                <button
                        type="button"
                        class="btn btn-primary">
                    Télécharger
                </button>

            </div>

        </div>

    </header>

    <div class="page-body">
        ...
    </div>

</div>
```

Plusieurs actions peuvent être placées dans `.page-header-actions` :

```html
<div class="page-header-actions">

    <button
            type="button"
            class="btn btn-primary">
        Ajouter
    </button>

    <button
            type="button"
            class="btn btn-secondary">
        Exporter
    </button>

</div>
```

# 10. Différence entre action et filtre

Cette distinction est essentielle.

## Action

Une action **déclenche une opération**.

Exemples :

```text
Ajouter
Modifier
Supprimer
Exporter
Télécharger
Imprimer
Actualiser
```

Elle appartient à :

```html
.page-header-actions
```

## Filtre

Un filtre **modifie les données affichées**.

Exemples :

```text
Catégorie
Club
Saison
Mois
Sexe
Recherche par nom
Recherche par licence
```

Il appartient à :

```html
.page-header-filters
```

Il ne faut donc pas placer un filtre dans `.page-header-actions`.

# 11. Les filtres : `.page-header-filters`

Lorsqu'une interface possède un ou plusieurs critères de recherche ou de filtrage, ils sont regroupés dans :

```html
<div class="page-header-filters">
    ...
</div>
```

Cette zone constitue toujours une **seconde ligne de l'entête**.

Exemple :

```html
<header class="page-header">

    <div class="page-header-top">

        <h1 class="page-title">
            Liste des coureurs
        </h1>

    </div>

    <div class="page-header-filters">

        ...
        
    </div>

</header>
```

La règle générale est donc :

> **Première ligne : titre et actions.**
> 
> **Deuxième ligne : filtres et recherches.**

# 12. Les zones de recherche : `.zone-recherche`

Les filtres utilisant le composant de recherche standard utilisent :

```html
<div class="zone-recherche">
    ...
</div>
```

Exemple :

```html
<div class="page-header-filters">

    <div class="zone-recherche">

        <label for="idCategorie">
            Catégorie
        </label>

        <select id="idCategorie">
        </select>

    </div>

</div>
```

La CSS générale masque par défaut le `<label>`.

Le label doit néanmoins être présent dans le HTML afin de conserver une structure accessible.

# 13. Afficher volontairement le label d'un filtre

Par défaut, les labels des filtres sont masqués visuellement.

Cela permet notamment d'obtenir une interface compacte :

```text
┌───────────────────────────────────────────┐
│                                           │
│ [ Sélectionner une catégorie          ▼ ] │
│                                           │
└───────────────────────────────────────────┘
```

Si le contexte nécessite l'affichage du label, la classe :

```html
afficher
```

peut être ajoutée au `<label>`.

Exemple :

```html
<div class="zone-recherche">

    <label
            for="idCategorie"
            class="afficher">
        Catégorie
    </label>

    <select id="idCategorie">
    </select>

</div>
```

On obtient alors :

```text
Catégorie

[ Sélectionner une catégorie              ▼ ]
```

> Le label ne doit donc pas être supprimé du HTML simplement parce qu'il n'est pas affiché.

# 14. Plusieurs filtres

Plusieurs zones de recherche peuvent être placées dans `.page-header-filters`.

Exemple :

```html
<div class="page-header-filters">

    <div class="zone-recherche">

        <label for="idCategorie">
            Catégorie
        </label>

        <select id="idCategorie">
        </select>

    </div>

    <div class="zone-recherche">

        <label for="idClub">
            Club
        </label>

        <select id="idClub">
        </select>

    </div>

    <div class="zone-recherche">

        <label for="sexe">
            Sexe
        </label>

        <select id="sexe">
        </select>

    </div>

</div>
```

La hiérarchie est donc :

```text
.page-header
│
├── .page-header-top
│   └── .page-title
│
└── .page-header-filters
    ├── .zone-recherche
    ├── .zone-recherche
    └── .zone-recherche
```

# 15. Recherche textuelle

Une recherche textuelle utilise le composant de recherche standard.

Exemple :

```html
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
                        aria-hidden="true">
                    ⌕
                </span>

            </div>

        </div>

    </div>

</div>
```

Le composant peut également être utilisé avec le système d'autocomplétion prévu par l'application.

# 16. Schéma C — Titre + filtres

```html
<div class="page-card">

    <header class="page-header">

        <div class="page-header-top">

            <h1 class="page-title">
                Courses annoncées
            </h1>

        </div>

        <div class="page-header-filters">

            <div class="zone-recherche">

                <label for="filtreMois">
                    Mois
                </label>

                <select id="filtreMois">
                    <option value="0">
                        Tous les mois
                    </option>
                </select>

            </div>

        </div>

    </header>

    <div class="page-body">
        ...
    </div>

</div>
```

# 17. Schéma D — Titre + actions + filtres

Il s'agit du schéma le plus complet.

```html
<div class="page-card">

    <header class="page-header">

        <div class="page-header-top">

            <h1 class="page-title">
                Liste des coureurs
            </h1>

            <div class="page-header-actions">

                <button
                        type="button"
                        class="btn btn-primary">
                    Exporter
                </button>

                <button
                        type="button"
                        class="btn btn-secondary">
                    Actualiser
                </button>

            </div>

        </div>

        <div class="page-header-filters">

            <div class="zone-recherche">

                <label for="idCategorie">
                    Catégorie
                </label>

                <select id="idCategorie">
                </select>

            </div>

            <div class="zone-recherche">

                <label for="idClub">
                    Club
                </label>

                <select id="idClub">
                </select>

            </div>

            <div class="zone-recherche">

                <label for="search">
                    Recherche
                </label>

                <div class="champ-recherche">

                    <div class="input-recherche">

                        <input
                                type="text"
                                id="search"
                                placeholder="Rechercher..."
                                autocomplete="off">

                        <span
                                class="icone-recherche"
                                aria-hidden="true">
                            ⌕
                        </span>

                    </div>

                </div>

            </div>

        </div>

    </header>

    <div class="page-body">

        <!-- Contenu -->

    </div>

</div>
```

# 18. Le corps : `.page-body`

Le corps contient exclusivement le contenu principal de la fonctionnalité.

```html
<div class="page-body">

    <!-- Contenu spécifique -->

</div>
```

Le contenu peut être :

- un tableau ;
- une liste ;
- des informations ;
- plusieurs blocs ;
- des cartes secondaires ;
- un formulaire ;
- une zone de résultats ;
- tout autre composant propre à la fonctionnalité.

La structure générale ne doit pas être recréée à l'intérieur de `.page-body`.

# 19. Exemple avec un tableau

```html
<div class="page-card">

    <header class="page-header">

        <div class="page-header-top">

            <h1 class="page-title">
                Liste des clubs
            </h1>

        </div>

    </header>

    <div class="page-body">

        <table id="leTableau">

            <thead>
                <tr>
                    <th>Code</th>
                    <th>Nom</th>
                </tr>
            </thead>

            <tbody id="lesLignes"></tbody>

        </table>

    </div>

</div>
```

La CSS générale assure la présentation du tableau.

Une feuille CSS spécifique n'est nécessaire que si l'interface possède des besoins particuliers.

# 20. Exemple avec recherche et tableau

```html
<div class="page-card">

    <header class="page-header">

        <div class="page-header-top">

            <h1 class="page-title">
                Liste des coureurs
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
                                aria-hidden="true">
                            ⌕
                        </span>

                    </div>

                </div>

            </div>

        </div>

    </header>

    <div class="page-body">

        <table id="leTableau">

            <thead>
                ...
            </thead>

            <tbody id="lesLignes"></tbody>

        </table>

    </div>

</div>
```

# 21. Schéma visuel général

La structure doit être comprise comme suit :

```text
┌──────────────────────────────────────────────────────────────┐
│ PAGE-CARD                                                    │
│                                                              │
│ ┌──────────────────────────────────────────────────────────┐ │
│ │ PAGE-HEADER                                              │ │
│ │                                                          │ │
│ │ ┌──────────────────────────────────────────────────────┐ │ │
│ │ │ PAGE-HEADER-TOP                                      │ │ │
│ │ │                                                      │ │ │
│ │ │ Titre de la page                    Actions éventuelles│ │ │
│ │ │                                                      │ │ │
│ │ └──────────────────────────────────────────────────────┘ │ │
│ │                                                          │ │
│ │ ┌──────────────────────────────────────────────────────┐ │ │
│ │ │ PAGE-HEADER-FILTERS                                  │ │ │
│ │ │                                                      │ │ │
│ │ │ Filtre 1     Filtre 2     Recherche...              │ │ │
│ │ │                                                      │ │ │
│ │ └──────────────────────────────────────────────────────┘ │ │
│ │                                                          │ │
│ └──────────────────────────────────────────────────────────┘ │
│                                                              │
│ ┌──────────────────────────────────────────────────────────┐ │
│ │ PAGE-BODY                                                │ │
│ │                                                          │ │
│ │              Contenu principal                           │ │
│ │                                                          │ │
│ └──────────────────────────────────────────────────────────┘ │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

Il n'existe **aucun pied de page propre à `page-card`**.

Le pied de page éventuel appartient à la structure générale de l'application et non à cette carte fonctionnelle.

# 22. Les quatre schémas de référence

Pour créer une nouvelle interface, le développeur choisit principalement entre quatre configurations.

| Schéma | Titre | Actions | Filtres |
| ------ | ----- | ------- | ------- |
| A      | Oui   | Non     | Non     |
| B      | Oui   | Oui     | Non     |
| C      | Oui   | Non     | Oui     |
| D      | Oui   | Oui     | Oui     |

### Schéma A

```text
Carte
└── Entête
    └── Titre

└── Corps
```

### Schéma B

```text
Carte
└── Entête
    └── Titre + Actions

└── Corps
```

### Schéma C

```text
Carte
└── Entête
    ├── Titre
    └── Filtres

└── Corps
```

### Schéma D

```text
Carte
└── Entête
    ├── Titre + Actions
    └── Filtres

└── Corps
```

# 23. Ce que le développeur doit faire

Lorsqu'une nouvelle interface est créée, le développeur doit :

1. utiliser `.page-card` comme conteneur principal ;
2. créer un `.page-header` ;
3. placer le titre dans `.page-header-top` ;
4. placer les actions éventuelles dans `.page-header-actions` ;
5. placer les recherches et filtres dans `.page-header-filters` ;
6. placer le contenu fonctionnel dans `.page-body` ;
7. utiliser les composants CSS existants lorsque ceux-ci répondent au besoin ;
8. créer une CSS spécifique uniquement lorsque la fonctionnalité possède réellement un besoin particulier.

# 24. Ce que le développeur ne doit pas faire

Il ne faut pas :

- créer une nouvelle classe de carte pour une fonctionnalité ;
- créer une nouvelle structure d'entête ;
- placer les filtres à côté du titre dans `.page-header-top` ;
- placer les filtres dans `.page-header-actions` ;
- supprimer le `<label>` parce qu'il est visuellement masqué ;
- recréer le style des champs de recherche dans chaque fonctionnalité ;
- créer un `page-footer` à l'intérieur de la carte ;
- dupliquer dans une CSS spécifique les règles déjà fournies par la CSS générale.

# 25. Principe architectural

L'organisation recherchée est la suivante :

```text
                         APPLICATION
                              │
              ┌───────────────┴───────────────┐
              │                               │
       STRUCTURE COMMUNE                FONCTIONNALITÉ
              │                               │
        page-card                         coureur
        page-header                       club
        page-title                        projet
        page-body                         course
        page-header-actions               recherche
        page-header-filters               etc.
              │                               │
              └───────────────┬───────────────┘
                              │
                         INTERFACE
```

La **structure commune** est définie une seule fois dans les feuilles de style générales.

La **fonctionnalité** apporte uniquement ce qui lui est propre.

# 26. Règle d'or

Lorsqu'une nouvelle interface doit être développée, le développeur doit commencer par déterminer :

> **Quel est le titre de mon interface ?**
> 
> **Ai-je des actions ?**
> 
> **Ai-je des filtres ou une recherche ?**
> 
> **Quel est le contenu à afficher dans le corps ?**

Il choisit ensuite l'un des quatre schémas :

```text
                    NOUVELLE INTERFACE
                           │
                           ▼
                    ┌───────────────┐
                    │     Titre ?   │
                    └───────┬───────┘
                            │
                  ┌─────────┴─────────┐
                  │                   │
               Actions ?           Filtres ?
                  │                   │
                  └─────────┬─────────┘
                            ▼
                     Schéma A / B / C / D
                            │
                            ▼
                       .page-body
                            │
                            ▼
                  Contenu spécifique
```

Le développeur n'a donc pas à concevoir une nouvelle architecture graphique pour chaque page.

Il doit **réutiliser la structure commune et ne développer que la fonctionnalité**.

# 27. Résumé

La structure de toutes les interfaces repose sur le modèle :

```html
<div class="page-card">

    <header class="page-header">

        <div class="page-header-top">

            <h1 class="page-title">
                Titre
            </h1>

            <div class="page-header-actions">
                <!-- Actions éventuelles -->
            </div>

        </div>

        <div class="page-header-filters">
            <!-- Filtres éventuels -->
        </div>

    </header>

    <div class="page-body">
        <!-- Contenu spécifique -->
    </div>

</div>
```

La règle fondamentale est :

```text
                    PAGE-CARD
                       │
              ┌────────┴────────┐
              │                 │
          PAGE-HEADER        PAGE-BODY
              │                 │
        ┌─────┴─────┐           │
        │           │           │
       TOP       FILTERS     CONTENU
        │
   ┌────┴────┐
   │         │
 TITRE    ACTIONS
```

Cette architecture constitue le **contrat de présentation des interfaces** de l'application.

Toute nouvelle page doit s'y conformer afin de préserver la cohérence visuelle, la maintenabilité et l'évolution homogène de l'application.