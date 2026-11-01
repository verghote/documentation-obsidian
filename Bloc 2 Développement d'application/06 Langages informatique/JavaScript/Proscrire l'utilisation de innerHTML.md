
innerHTML a été introduit par Microsoft pour Internet Explorer 4 en 1997 avant d'être standardisé bien plus tard par le W3C/HTML5 face à son immense popularité auprès des développeurs.

D'un point de vue architectural et de la sécurité, **bannir `innerHTML` est une évidence*.

### Pourquoi `innerHTML` pose problème ?

1. **Le risque de sécurité (XSS) :** `innerHTML` interprète du texte brut comme du code HTML. Si les données proviennent d'un utilisateur malveillant (ou d'une API compromise), cela ouvre la porte à des failles d'injection de code (XSS).
    
2. **La destruction du DOM existant :** `innerHTML` détruit et reconstruit tout l'arbre d'éléments enfants à chaque affectation. Cela supprime au passage tous les écouteurs d'événements (`addEventListener`) qui étaient attachés à ces enfants.
    
3. **Le détournement de la philosophie du DOM :** Au lieu de manipuler des objets et des nœuds du DOM de manière structurée, on manipule de simples chaînes de caractères.
    
### Par quoi le remplacer ? 

Il suffit de manipuler directement les nœuds du DOM via les méthodes natives.


```javascript
// ❌ Version avec innerHTML
const id = document.getElementById('id');
id.innerHTML = `<span class="badge-id">${epreuve.id}</span>`;
```

On utilise la création d'éléments par programmation :

```javascript
// ✅ Version 100% pur DOM
const spanId = document.createElement('span');
spanId.className = 'badge-id';
spanId.textContent = epreuve.id;

const id = document.getElementById('id');
id.replaceChildren(spanId);
```



```javascript
msg.innerHTML = genererMessage(reponseErreur.error, 'error');
==>
msg.replaceChildren(genererMessage(reponseErreur.error, 'error'));
```

```javascript
msg.innerHTML = genererMessage(reponseErreur.error, 'error');
==>
msg.replaceChildren(genererMessage(reponseErreur.error, 'error'));
```

Fichiers concernés
menu.js
ajax.js
affichier.js : genererMessage renvoie un élément DOM plutôt qu’une chaîne HTML. Les options isHTML des cellules conservent leur comportement via DOMParser.

dom.js  crrerTd
trtd.js
formulaire.js
pawword.js
tableau.js



