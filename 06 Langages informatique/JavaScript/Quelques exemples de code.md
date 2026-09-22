
Convertir une date du format aaaa-mm-jj au format jj/mm/aaaa 

```javascript
const date = new Date(element.date); 
const options = { weekday: "long", year: "numeric", month: "long", day: "numeric" }; 
const dateFr = `${date.toLocaleString('fr-FR', options)}`;
```


Filtrer les données à afficher

Utilisation de la méthode filter 

```javascript
lesAnnonces
    .filter(element => moisChoisi === -1 || new Date(element.date).getMonth() === moisChoisi)
    .forEach(element => lesCartes.appendChild(creerCarte(element)));
```

Mauvaise solution car elle **crée un nouveau tableau temporaire** en mémoire, ce qui peut être moins performant que de simplement parcourir le tableau original avec une boucle `for...of` ou `for` classique, surtout si le tableau est volumineux.

Utilisation de la boucle for of

```javascript
`const moisChoisi = parseInt(filtreMois.value, 10) - 1; 
// Les mois sont indexés de 0 à 11 
for (const element of lesAnnonces) { 
	if (moisChoisi !== -1 && new Date(element.date).getMonth() !== moisChoisi) { 
		continue; // On passe à l'itération suivante si le mois ne correspond pas 
	} 
	lesCartes.appendChild(creerCarte(element)); 
}`
```

Utilisation de la méthode `Array.prototype.reduce` (moins lisible, mais sans duplication)

```javascript
lesAnnonces.reduce((_, element) => {
    if (moisChoisi === -1 || new Date(element.date).getMonth() === moisChoisi) {
        lesCartes.appendChild(creerCarte(element));
    }
    return _; // On retourne l'accumulateur (inutilisé ici)
}, null);
```

**Cette solution est moins lisible et moins idiomatique** que la boucle `for...of` pour ce cas d'usage.

---

## **Pourquoi `for...of` est la meilleure solution ici ?**

|Critère|`.filter()` + `.forEach()`|`for...of`|`.reduce()`|
|---|---|---|---|
|**Performance**|❌ (Duplique le tableau)|✅|✅|
|**Lisibilité**|✅|✅|❌|
|**Simplicité**|✅|✅|❌|
|**Mémoire**|❌ (Alloue un nouveau tableau)|✅|✅|

**→ `for...of` est le meilleur compromis** : **performant, lisible et simple**.