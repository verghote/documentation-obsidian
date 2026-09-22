# Configuration JSHint du projet

Ce document décrit les directives utilisées dans le fichier `.jshintrc` du projet.

L'objectif est de contrôler la qualité et la robustesse du code JavaScript tout en conservant certaines règles historiques nécessaires au fonctionnement actuel du projet.

## Configuration complète

```json
{
  "globals": {
    "bootstrap": false
  },
  "esversion": 12,
  "bitwise": true,
  "camelcase": true,
  "newcap": false,
  "curly": true,
  "eqeqeq": true,
  "immed": true,
  "indent": 2,
  "latedef": false,
  "newcap": true,
  "noarg": true,
  "noempty": true,
  "nonew": true,
  "undef": true,
  "unused": true,
  "strict": true,
  "globalstrict": true,
  "trailing": true,
  "debug": false,
  "eqnull": true,
  "evil": false,
  "expr": true,
  "funcscope": true,
  "iterator": true,
  "lastsemic": false,
  "loopfunc": false,
  "multistr": false,
  "proto": false,
  "shadow": false,
  "sub": false,
  "supernew": false,
  "validthis": false,
  "browser": true,
  "devel": true,
  "nomen": true,
  "passfail": false,
  "white": true,
  "-W031": true,
  "-W106": true,
  "-W083": true
}
```


# 1. Variables globales

## `globals`

```json
"globals": {
  "bootstrap": false
}
```

Permet de déclarer à JSHint des variables globales qui ne sont pas définies directement dans le fichier JavaScript.

Cette option est particulièrement utile avec `undef: true`.

### `bootstrap`

```json
"bootstrap": false
```

Indique que `bootstrap` est une variable globale fournie par la bibliothèque Bootstrap.

La valeur `false` indique que le code peut utiliser cette variable mais ne doit pas la modifier.

Par exemple :

```javascript
const modal = new bootstrap.Modal(element);
```

Sans cette déclaration, JSHint pourrait signaler :

```text
'bootstrap' is not defined
```

La documentation JSHint précise que les variables déclarées dans `globals` avec `false` sont considérées comme **en lecture seule**.


# 2. Version JavaScript

## `esversion`

```json
"esversion": 12
```

Indique la version ECMAScript utilisée par le projet.

La valeur `12` correspond à **ECMAScript 2021**.

Cette directive permet notamment à JSHint de reconnaître correctement les fonctionnalités JavaScript modernes.

Elle est importante dans un projet utilisant notamment :

```javascript
const
let
import
export
async / await
```

La documentation officielle recommande `esversion` plutôt que les anciennes options `es5` ou `esnext`.

---

# 3. Règles de programmation

## `bitwise`

```json
"bitwise": true
```

Interdit les opérateurs binaires tels que :

```javascript
&
|
^
~
<<
>>
>>>
```

L'objectif est notamment de détecter les erreurs accidentelles comme :

```javascript
if (a & b)
```

alors que le développeur voulait probablement écrire :

```javascript
if (a && b)
```

JSHint considère que les opérations binaires sont relativement rares dans les applications JavaScript classiques.

---

## `camelcase`

```json
"camelcase": true
```

Demande que les identifiants respectent principalement la convention `camelCase`.

Exemple :

```javascript
nomCoureur
dateNaissance
idCategorie
```

plutôt que :

```javascript
nom_coureur
date_naissance
```

Cette option est cependant **dépréciée dans JSHint** car elle concerne davantage le style de programmation que la détection d'erreurs.

---

## `newcap`

```json
"newcap": true
```

Demande que les fonctions utilisées comme constructeurs avec `new` commencent par une majuscule.

Exemple :

```javascript
const coureur = new Coureur();
```

plutôt que :

```javascript
const coureur = new coureur();
```

Cette règle permet de distinguer visuellement les constructeurs des fonctions classiques.

> **Attention : ta configuration contient deux fois `newcap` :**
> 
> ```json
> "newcap": false,
> ...
> "newcap": true
> ```
> 
> Il est préférable de supprimer la première occurrence et de conserver uniquement :
> 
> ```json
> "newcap": true
> ```
> 
> Dans un objet JSON, une même propriété ne devrait apparaître qu'une seule fois.

JSHint considère également cette option comme dépréciée.

---

## `curly`

```json
"curly": true
```

Impose l'utilisation des accolades pour les blocs conditionnels et les boucles.

JSHint accepte normalement :

```javascript
if (condition)
    faireQuelqueChose();
```

Avec `curly: true`, il faut écrire :

```javascript
if (condition) {
    faireQuelqueChose();
}
```

Cela permet notamment d'éviter des erreurs lors de l'ajout ultérieur d'instructions.

---

## `eqeqeq`

```json
"eqeqeq": true
```

Interdit :

```javascript
==
!=
```

et impose :

```javascript
===
!==
```

Exemple :

```javascript
if (age === 18) {
}
```

plutôt que :

```javascript
if (age == 18) {
}
```

Cette règle évite les conversions automatiques de types réalisées par `==` et `!=`.

---

# 4. Fonctions immédiatement exécutées

## `immed`

```json
"immed": true
```

Contrôle les fonctions immédiatement exécutées, également appelées **IIFE**.

Exemple :

```javascript
(function () {
    // code
}());
```

JSHint demande ici que l'appel soit explicitement entouré de parenthèses.

Cette option est **dépréciée** dans les versions modernes de JSHint.

---

# 5. Indentation

## `indent`

```json
"indent": 2
```

Indique une indentation de **2 espaces**.

Exemple :

```javascript
function afficher() {
  if (condition) {
    console.log("OK");
  }
}
```

Cette option est aujourd'hui **dépréciée** dans JSHint, la mise en forme étant plutôt considérée comme le rôle d'un formateur de code ou de l'IDE.

---

# 6. Déclaration des variables

## `latedef`

```json
"latedef": false
```

Autorise l'utilisation d'une variable ou d'une fonction avant sa déclaration.

Exemple :

```javascript
afficher();

function afficher() {
    console.log("OK");
}
```

Avec `latedef: true`, JSHint signalerait l'utilisation avant déclaration.

Avec :

```json
"latedef": false
```

ce comportement n'est pas signalé.

---

## `undef`

```json
"undef": true
```

Signale l'utilisation de variables qui n'ont pas été déclarées.

Exemple :

```javascript
const nom = "Dupont";

console.log(nom);
console.log(prenom);
```

JSHint signalera `prenom` comme variable non définie.

C'est une règle particulièrement intéressante car elle permet de détecter les fautes de frappe :

```javascript
dateNaissance
```

contre :

```javascript
dateNaisssance
```

La documentation JSHint considère cette option comme particulièrement utile pour détecter les variables mal orthographiées.

---

## `unused`

```json
"unused": true
```

Signale les variables ou fonctions déclarées mais jamais utilisées.

Exemple :

```javascript
const nom = "Dupont";
const prenom = "Jean";

console.log(nom);
```

JSHint signalera `prenom` comme inutilisé.

Cette règle permet notamment de nettoyer progressivement le code devenu inutile.

---

# 7. Contrôle du mode strict

## `strict`

```json
"strict": true
```

Demande à JSHint de contrôler l'utilisation du mode strict JavaScript.

Avec `true`, JSHint attend une directive :

```javascript
"use strict";
```

au niveau des fonctions.

C'est notamment cette configuration qui peut provoquer l'avertissement :

```text
W097: Use the function form of "use strict".
```

---

## `globalstrict`

```json
"globalstrict": true
```

Autorise l'utilisation du mode strict au niveau global du fichier :

```javascript
"use strict";
```

C'est cette directive qui est importante dans **ta configuration actuelle**, puisque tu souhaites conserver :

```javascript
"use strict";
```

au début de tes fichiers.

JSHint indique toutefois que `globalstrict` est aujourd'hui **dépréciée** et recommande théoriquement :

```json
"strict": "global"
```

à la place.

Dans ton projet, cette modification ne doit cependant pas être faite sans vérifier le comportement de Qodana/JSHint que tu utilises actuellement.

---

# 8. Options liées aux avertissements

## `trailing`

```json
"trailing": true
```

Contrôle les espaces ou caractères blancs placés en fin de ligne.

Cette directive ne doit pas être confondue avec la gestion des virgules finales dans les tableaux ou objets.

---

## `debug`

```json
"debug": false
```

Avec `false`, JSHint continue à signaler les instructions :

```javascript
debugger;
```

Cela permet d'éviter qu'une instruction de débogage oubliée reste dans le code.

---

## `eqnull`

```json
"eqnull": true
```

Autorise explicitement :

```javascript
variable == null
```

Ce test possède la particularité de détecter à la fois :

```javascript
null
```

et :

```javascript
undefined
```

Exemple :

```javascript
if (valeur == null) {
    // valeur est null ou undefined
}
```

Cette règle désactive donc l'avertissement habituellement généré pour `== null`.

---

## `evil`

```json
"evil": false
```

JSHint continue à signaler l'utilisation de :

```javascript
eval()
```

C'est généralement souhaitable car `eval()` peut poser des problèmes de sécurité et rendre le code plus difficile à analyser.

---

## `expr`

```json
"expr": true
```

Autorise certaines expressions utilisées seules.

Par exemple :

```javascript
condition && afficher();
```

JSHint pourrait normalement considérer certaines expressions de ce type comme suspectes.

Cette option réduit donc certains avertissements.

---

# 9. Portée des variables

## `funcscope`

```json
"funcscope": true
```

Supprime certains avertissements concernant les variables `var` déclarées dans un bloc puis utilisées en dehors de celui-ci.

Exemple :

```javascript
function test() {

    if (true) {
        var valeur = 10;
    }

    console.log(valeur);
}
```

Avec `var`, la variable appartient à la portée de la fonction et non au bloc `if`.

Cette règle est surtout héritée du fonctionnement historique de `var`. Elle est beaucoup moins importante dans du code moderne utilisant :

```javascript
let
const
```

---

# 10. Options historiques ou spécifiques

## `iterator`

```json
"iterator": true
```

Autorise l'utilisation de :

```javascript
__iterator__
```

Cette propriété est ancienne et n'est pas supportée de manière uniforme par les navigateurs.

Elle est donc essentiellement conservée pour compatibilité avec du code ancien.

---

## `lastsemic`

```json
"lastsemic": false
```

Concerne un cas très particulier de point-virgule manquant à la fin d'une instruction située dans un bloc écrit sur une seule ligne.

Cette option a une utilité très limitée et concerne principalement certains générateurs de code.

---

## `loopfunc`

```json
"loopfunc": false
```

JSHint continue à signaler les fonctions déclarées à l'intérieur des boucles.

Exemple :

```javascript
for (let i = 0; i < 10; i++) {

    bouton.onclick = function () {
        console.log(i);
    };

}
```

Cette construction peut historiquement provoquer des problèmes liés aux variables capturées par les fonctions.

Avec `loopfunc: false`, JSHint conserve donc cette vérification.

---

## `multistr`

```json
"multistr": false
```

N'autorise pas les anciennes chaînes JavaScript écrites sur plusieurs lignes avec `\`.

Exemple historique :

```javascript
const texte = "Première ligne\
Deuxième ligne";
```

Cette technique est ancienne et n'est normalement plus nécessaire avec les chaînes modernes et les template literals.

---

## `proto`

```json
"proto": false
```

JSHint continue à signaler l'utilisation de :

```javascript
__proto__
```

Cette propriété possède un comportement particulier sur les prototypes JavaScript et son utilisation directe est généralement déconseillée.

---

## `shadow`

```json
"shadow": false
```

Signale les situations dans lesquelles une variable masque une variable portant le même nom dans une portée extérieure.

Exemple :

```javascript
const nom = "Dupont";

function afficher() {
    const nom = "Martin";
}
```

La variable interne `nom` masque la variable externe.

Avec `shadow: false`, JSHint signale ce type de situation.

---

## `sub`

```json
"sub": false
```

Signale les accès utilisant la notation crochets lorsqu'une notation avec point serait possible.

Par exemple :

```javascript
objet["nom"]
```

au lieu de :

```javascript
objet.nom
```

Cette option est **dépréciée** par JSHint car elle relève davantage du style de programmation que de la correction du code.

---

## `supernew`

```json
"supernew": false
```

Signale certaines constructions inhabituelles utilisant `new`.

Par exemple :

```javascript
new Object;
```

ou des constructions du type :

```javascript
new function () {
    // ...
};
```

Ces constructions sont valides mais généralement peu lisibles.

---

## `validthis`

```json
"validthis": false
```

Contrôle certains usages de :

```javascript
this
```

dans des fonctions où JSHint pourrait considérer que `this` risque d'être utilisé de manière incorrecte en mode strict.

Avec `false`, JSHint conserve ces avertissements.

---

# 11. Environnement d'exécution

## `browser`

```json
"browser": true
```

Indique que le code est exécuté dans un navigateur.

JSHint connaît alors les variables globales fournies par le navigateur, notamment :

```javascript
window
document
navigator
FileReader
```

Cette option est indispensable pour ton JavaScript côté client.

---

## `devel`

```json
"devel": true
```

Autorise les objets et fonctions généralement utilisés pendant le développement, notamment :

```javascript
console
alert
```

Cela permet notamment d'utiliser :

```javascript
console.log("Test");
```

sans avertissement concernant `console`.

---

# 12. Noms d'identifiants

## `nomen`

```json
"nomen": true
```

Autorise certains identifiants comportant des caractères normalement considérés comme particuliers, notamment le caractère `_`.

Cette option est principalement liée aux conventions de nommage.

---

# 13. Contrôle de l'analyse

## `passfail`

```json
"passfail": false
```

Demande à JSHint de **ne pas s'arrêter à la première erreur**.

JSHint continue donc son analyse et peut retourner plusieurs problèmes pour un même fichier.

C'est particulièrement utile avec Qodana, car cela permet d'obtenir une vision plus complète des problèmes présents dans un fichier.

---

# 14. Désactivation d'avertissements spécifiques

Les directives `-Wxxx` permettent de désactiver des avertissements JSHint identifiés par leur numéro.

## `-W031`

```json
"-W031": true
```

Désactive l'avertissement JSHint **W031**.

Cette directive permet de ne pas faire remonter ce contrôle particulier dans Qodana.

---

## `-W106`

```json
"-W106": true
```

Désactive l'avertissement JSHint **W106**.

Cette désactivation est notamment utile lorsque les conventions de nommage imposées par le projet ne correspondent pas exactement aux conventions attendues par JSHint.

---

## `-W083`

```json
"-W083": true
```

Désactive l'avertissement **W083**, qui concerne les fonctions déclarées dans des boucles et utilisant des variables provenant d'une portée extérieure.

C'est particulièrement pertinent avec du JavaScript moderne utilisant :

```javascript
for (const element of elements) {
    const action = () => {
        console.log(element);
    };
}
```

La gestion des portées de `let` et `const` rend cette situation différente des anciens problèmes liés à `var`.

---

# 15. Résumé des directives

|Directive|Valeur|Rôle principal|
|---|--:|---|
|`globals.bootstrap`|`false`|Déclare `bootstrap` comme variable globale en lecture seule|
|`esversion`|`12`|Utilisation d'ECMAScript 2021|
|`bitwise`|`true`|Interdit les opérateurs binaires|
|`camelcase`|`true`|Contrôle les conventions `camelCase`|
|`newcap`|`true`|Contrôle le nommage des constructeurs|
|`curly`|`true`|Impose les accolades|
|`eqeqeq`|`true`|Impose `===` et `!==`|
|`immed`|`true`|Contrôle les IIFE|
|`indent`|`2`|Indentation de 2 espaces|
|`latedef`|`false`|Autorise l'utilisation avant déclaration|
|`noarg`|`true`|Interdit `arguments.caller` et `arguments.callee`|
|`noempty`|`true`|Signale les blocs vides|
|`nonew`|`true`|Signale les `new` dont le résultat n'est pas utilisé|
|`undef`|`true`|Signale les variables non déclarées|
|`unused`|`true`|Signale les variables inutilisées|
|`strict`|`true`|Contrôle l'utilisation du mode strict|
|`globalstrict`|`true`|Autorise/contrôle le `use strict` global|
|`trailing`|`true`|Contrôle les espaces en fin de ligne|
|`debug`|`false`|Signale les instructions `debugger`|
|`eqnull`|`true`|Autorise `== null`|
|`evil`|`false`|Signale `eval()`|
|`expr`|`true`|Autorise certaines expressions seules|
|`funcscope`|`true`|Autorise certaines utilisations de variables `var` hors bloc|
|`iterator`|`true`|Autorise `__iterator__`|
|`lastsemic`|`false`|Contrôle un cas particulier de point-virgule final|
|`loopfunc`|`false`|Signale les fonctions dans les boucles|
|`multistr`|`false`|Interdit les anciennes chaînes multilignes|
|`proto`|`false`|Signale `__proto__`|
|`shadow`|`false`|Signale les variables qui masquent une variable extérieure|
|`sub`|`false`|Contrôle l'utilisation de `obj["propriete"]`|
|`supernew`|`false`|Signale certaines constructions inhabituelles avec `new`|
|`validthis`|`false`|Contrôle certains usages de `this`|
|`browser`|`true`|Environnement navigateur|
|`devel`|`true`|Autorise `console`, `alert`, etc.|
|`nomen`|`true`|Autorise certains noms avec `_`|
|`passfail`|`false`|Analyse le fichier jusqu'au bout|
|`-W031`|`true`|Désactive W031|
|`-W106`|`true`|Désactive W106|
|`-W083`|`true`|Désactive W083|

---

## Point d'attention : `newcap`

Dans la configuration fournie, `newcap` apparaît deux fois :

```json
"newcap": false,
...
"newcap": true,
```

Il faut corriger cela. La configuration doit contenir **une seule occurrence**.

Si tu souhaites conserver le comportement actuel, garde :

```json
"newcap": true
```

et supprime :

```json
"newcap": false
```

---

## Point d'attention : options dépréciées

Plusieurs options présentes dans cette configuration sont aujourd'hui considérées comme **dépréciées par JSHint**, notamment :

```text
camelcase
immed
indent
newcap
noempty
sub
```

JSHint indique que ces règles de style ont vocation à sortir progressivement de son périmètre, qui se concentre davantage sur la **correction du code**.

Cela ne signifie pas qu'il faut immédiatement les supprimer de ton projet : **tant qu'elles produisent le comportement que tu souhaites et que Qodana les prend correctement en charge, elles peuvent rester dans ta configuration actuelle**.

La documentation officielle de référence pour les options JSHint est disponible ici : [Documentation officielle des options JSHint](https://jshint.com/docs/options/?utm_source=chatgpt.com)