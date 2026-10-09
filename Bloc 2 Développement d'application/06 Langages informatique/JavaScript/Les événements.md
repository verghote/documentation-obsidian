# 1. Présentation

Un événement est une action ou un changement d'état détecté par le navigateur et pouvant déclencher l'exécution d'un traitement JavaScript.

Un événement peut être provoqué :
- par l'utilisateur :
  - clic sur un bouton ;
  - saisie dans un champ ;
  - déplacement de la souris ;
  - appui sur une touche ;

- par le navigateur :
  - chargement d'une page ;
  - fin de chargement d'une image ;
  - modification de la taille de la fenêtre.

En JavaScript, on associe une fonction appelée **gestionnaire d'événement** (ou *event listener*) à un élément HTML.

Lorsque l'événement survient, le navigateur appelle automatiquement cette fonction.

Exemple :

```javascript
const bouton = document.getElementById('btn');

bouton.addEventListener('click', function () {
    console.log("Le bouton a été cliqué");
});
```

# 2. L'objet Event

Lorsqu'un événement est déclenché, JavaScript transmet automatiquement un objet contenant des informations sur cet événement.

Cet objet est généralement nommé `e`.

Exemple :

```javascript
bouton.addEventListener('click', function(e) {
    console.log(e.type);
});
```

## Principales propriétés

| Propriété / méthode | Rôle |
|---|---|
| `e.type` | Nom de l'événement déclenché |
| `e.target` | Élément réellement à l'origine de l'événement |
| `e.currentTarget` | Élément possédant le gestionnaire |
| `e.preventDefault()` | Annule l'action automatique du navigateur |
| `e.stopPropagation()` | Stoppe la propagation de l'événement |

## Exemple avec target et currentTarget

```html
<div id="bloc">
    <button id="btn">Valider</button>
</div>
```

```javascript
document.getElementById('bloc').addEventListener('click', function(e){
    console.log(e.target.id);
    console.log(e.currentTarget.id);
});
```

Si on clique sur le bouton :

```
target          → btn
currentTarget   → bloc
```

# 3. Associer un événement

## 3.1 Avec addEventListener (méthode recommandée)

C'est la méthode moderne à privilégier.

Syntaxe :

```javascript
element.addEventListener(evenement,fonction);
```

Exemple :

```javascript
const bouton = document.getElementById('btn');

bouton.addEventListener('click', verifier);

function verifier(e) {
    console.log("Vérification");
}
```

Il est également possible d'utiliser une fonction fléchée :

```javascript
bouton.addEventListener('click', (e) => {
    console.log("clic");
});
```

Lorsque la fonction fléchée contient une seule instruction, il est possible de supprimer les accolades

```javascript
bouton.addEventListener('click', (e) => console.log("clic"));
```
## Avantages de addEventListener

### Plusieurs traitements possibles

Contrairement à `onclick`, plusieurs fonctions peuvent être associées au même événement.

```javascript
bouton.addEventListener('click', fonction1);
bouton.addEventListener('click', fonction2);
```

Les deux fonctions seront exécutées.

### Suppression d'un événement

Pour supprimer un gestionnaire, il faut utiliser la même fonction :

```javascript
function verifier(){
    console.log("test");
}

bouton.addEventListener('click', verifier);

bouton.removeEventListener('click', verifier);
```

Un événement ajouté avec `addEventListener` ne peut être supprimé que si la référence de la fonction utilisée lors de l'ajout est conservée. Une fonction anonyme ou une fonction fléchée écrite directement dans `addEventListener` ne peut pas être supprimée.

Dans votre exemple :

```
bouton.addEventListener('click', (e) => console.log("clic"));
```

la fonction fléchée est créée **anonymement**. Elle n'a pas de référence que l'on peut réutiliser ensuite.

Donc ceci **ne fonctionnera pas** :

```javascript
bouton.removeEventListener(
    'click',
    (e) => console.log("clic")
);
```

Même si le code semble identique, JavaScript considère qu'il s'agit d'une **nouvelle fonction différente**.

## Solution 1 : donner un nom à la fonction (recommandée)

```javascript
const afficherClic = (e) => {
    console.log("clic");
};

bouton.addEventListener('click', afficherClic);
```

Pour supprimer :

```
bouton.removeEventListener('click', afficherClic);
```

Ici JavaScript retrouve exactement la même référence de fonction.

## Solution 2 : utiliser une fonction classique

Même principe :

```javascript
function afficherClic(e) {
    console.log("clic");
}

bouton.addEventListener('click', afficherClic);
```

Suppression :

```
bouton.removeEventListener('click', afficherClic);
```

## Solution 3 : conserver la fonction dans une variable

On peut aussi faire :

```javascript
const gestionnaireClic = (e) => console.log("clic");

bouton.addEventListener('click', gestionnaireClic);
```

Puis :

```
bouton.removeEventListener('click', gestionnaireClic);
```

## Cas où la suppression n'est pas nécessaire

Dans beaucoup de scripts modernes, on ne supprime jamais les événements car :

- le bouton existe pendant toute la durée de vie de la page ;
- le gestionnaire reste nécessaire.

Par exemple :

```
btnEnregistrer.addEventListener('click', enregistrer);
```

n'a généralement pas besoin d'être supprimé.

La suppression devient utile principalement lorsque :

- on crée et détruit dynamiquement des éléments ;
- on active temporairement un comportement ;
- on évite des doublons lors d'une initialisation répétée.

Exemple :

```javascript
function activerModeEdition() {
    bouton.addEventListener('click', modifier);
}

function desactiverModeEdition() {
    bouton.removeEventListener('click', modifier);
}
```

# 4. Anciennes méthodes (à éviter)

## Utilisation d'une propriété onclick

```javascript
bouton.onclick = function(){
    console.log("clic");
};
```

Cette méthode fonctionne mais elle possède une limitation :

```javascript
bouton.onclick = fonction1;
bouton.onclick = fonction2;
```

La deuxième instruction écrase la première.

## Attribut HTML onclick

Exemple :

```html
<button onclick="verifier()">
Valider
</button>
```

Cette méthode est déconseillée car elle mélange :

- le HTML (présentation) ;
- le JavaScript (traitement).

Il est préférable de séparer les responsabilités.

# 5. Les principaux événements

## Souris

| Événement | Utilisation |
|-|-|
| `click` | clic simple |
| `dblclick` | double clic |
| `mousedown` | bouton souris enfoncé |
| `mouseup` | bouton souris relâché |
| `mousemove` | déplacement souris |
| `mouseenter` | entrée dans un élément |
| `mouseleave` | sortie d'un élément |

## Formulaires

| Événement | Utilisation |
|-|-|
| `focus` | champ sélectionné |
| `blur` | champ quitté |
| `input` | modification immédiate d'un champ |
| `change` | valeur modifiée puis validation |
| `submit` | envoi d'un formulaire |

Exemple :

```javascript
nom.addEventListener('input', function(){
    console.log(this.value);
});
```

## Clavier

| Événement | Utilisation |
|-|-|
| `keydown` | touche enfoncée |
| `keyup` | touche relâchée |

Exemple :

```javascript
document.addEventListener('keydown', function(e){
    if(e.key === "Enter"){
        rechercher();
    }
});
```

La propriété importante est :  e.key 
Elle retourne la touche utilisée :

```
"A"
"Enter"
"Escape"
"ArrowLeft"
```
# 6. Empêcher le comportement par défaut

Certains éléments HTML possèdent un comportement automatique.

Exemple :  Un lien recharge une page.

```javascript
lien.addEventListener('click', function(e){
    e.preventDefault();
});
```

Le navigateur n'exécutera pas son action normale.
# 7. Propagation des événements

Les événements se propagent dans l'arbre DOM.

Exemple :

```html
<div id="parent">
    <button id="bouton">
        OK
    </button>
</div>
```

Un clic sur le bouton déclenche :

```
button
 ↓
div
 ↓
body
 ↓
document
```

Cette remontée est appelée :

**bubbling** (bouillonnement).

## Arrêter la propagation

```javascript
bouton.addEventListener('click', function(e){
    e.stopPropagation();
});
```

L'événement ne remontera pas vers les éléments parents.

# 8. Chargement du JavaScript

Le script doit pouvoir accéder aux éléments HTML.

La solution recommandée est :

```html
<script type="module" src="script.js"></script>
```

ou :

```html
<script src="script.js" defer></script>
```

Le navigateur attend alors que le HTML soit analysé avant d'exécuter le JavaScript.

# 9. Déclencher un événement par programmation

Il est possible de provoquer un événement :

```javascript
champ.dispatchEvent(
    new Event('change')
);
```

Le gestionnaire associé à `change` sera exécuté.
Normalement, un événement est déclenché par une action extérieure (clic utilisateur, saisie clavier...). Avec `dispatchEvent()`, JavaScript peut provoquer lui-même cet événement comme si l'utilisateur avait réalisé l'action.
Dans les tests automatisés, on peut ainsi simuler des actions utilisateur.
# 10. Les temporisations

## Exécuter une fois après un délai

```javascript
setTimeout(afficherMessage, 3000);
```

La fonction est exécutée après 3 secondes.

## Exécuter régulièrement

```javascript
const timer = setInterval(actualiser, 1000);
```

La fonction est exécutée toutes les secondes.

Pour arrêter :

```javascript
clearInterval(timer);
```

# 11. Bonnes pratiques

À retenir :

✅ Utiliser `addEventListener()`  
✅ Séparer HTML et JavaScript  
✅ Utiliser `e.target` et `e.currentTarget` avec compréhension  
✅ Utiliser `preventDefault()` uniquement lorsque nécessaire  
✅ Utiliser `input` pour surveiller une saisie en temps réel  
✅ Utiliser `keydown` plutôt que `keypress` (obsolète)  
✅ Préférer `defer` ou `type="module"` pour charger les scripts  
