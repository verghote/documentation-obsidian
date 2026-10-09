## Présentation

Un **Set** est une collection de valeurs **uniques**. Contrairement à un tableau (`Array`), un même élément ne peut être présent qu'une seule fois dans un `Set`.

Il est particulièrement adapté pour représenter un ensemble d'identifiants, de codes ou de clés sans risque de doublon.

```javascript
const competences = new Set();
```

# Création

Créer un ensemble vide :

```javascript
const ensemble = new Set();
```

Créer un ensemble à partir d'un tableau :

```javascript
const ensemble = new Set([2, 5, 8]);
```

Les doublons sont automatiquement supprimés.

```javascript
const ensemble = new Set([1,1,2,3,3]);

console.log(ensemble);
```

Résultat :

```
Set(3) {1, 2, 3}
```

# Ajouter un élément

```javascript
ensemble.add(15);
```

Si la valeur existe déjà, elle n'est pas ajoutée.

```javascript
ensemble.add(15);
ensemble.add(15);

console.log(ensemble.size);
```

Résultat :

```
1
```

# Supprimer un élément

```javascript
ensemble.delete(15);
```

La méthode retourne :

- `true` si l'élément existait ;
- `false` sinon.

Exemple :

```javascript
ensemble.delete(8);
```

# Tester la présence d'un élément

```javascript
ensemble.has(5)
```

Retourne :

- `true`
- `false`

Exemple :

```javascript
if (ensemble.has(12)) {
    console.log("présent");
}
```

# Nombre d'éléments

```javascript
ensemble.size
```

Exemple :

```javascript
console.log(ensemble.size);
```

# Vider complètement un Set

```javascript
ensemble.clear();
```

---

# Parcourir un Set

Avec `for...of` :

```javascript
for (const valeur of ensemble) {
    console.log(valeur);
}
```

ou

```javascript
ensemble.forEach(valeur => {
    console.log(valeur);
});
```

# Conversion en tableau

Très utile lorsqu'une fonction attend un tableau.

```javascript
const tableau = [...ensemble];
```

ou

```javascript
const tableau = Array.from(ensemble);
```

Exemple :

```javascript
const competences = new Set();

competences.add(2);
competences.add(5);
competences.add(8);

const resultat = [...competences];
```

Résultat :

```javascript
[2,5,8]
```

# Conversion d'un tableau en Set

```javascript
const tableau = [2,2,3,5,5];

const ensemble = new Set(tableau);
```

Les doublons disparaissent automatiquement.

# Exemple : gestion des compétences d'un projet

```javascript
const lesCompetencesDuProjet = new Set();
```

Ajout d'une compétence :

```javascript
lesCompetencesDuProjet.add(idCompetence);
```

Suppression :

```javascript
lesCompetencesDuProjet.delete(idCompetence);
```

Envoi au serveur :

```javascript
appelAjax({
    url: "ajax/enregistrer.php",
    data: {
        nom: nom.value,
        lesCompetences: [...lesCompetencesDuProjet]
    }
});
```

Le `Set` est converti en tableau grâce à l'opérateur de décomposition (`...`).

# Comparaison Set / Array

| Opération | Array | Set |
|-----------|------|-----|
| Ajouter | `push()` | `add()` |
| Supprimer | `splice()` | `delete()` |
| Rechercher | `includes()` | `has()` |
| Taille | `length` | `size` |
| Autorise les doublons | Oui | Non |
| Conversion vers tableau | — | `[...set]` |

# Quand utiliser un Set ?

Le `Set` est recommandé lorsque :

- une valeur ne doit apparaître qu'une seule fois ;
- on manipule des identifiants ;
- on souhaite éviter les doublons automatiquement ;
- on effectue beaucoup d'ajouts et de suppressions.

Exemples :

- liste des compétences sélectionnées ;
- rôles attribués à un utilisateur ;
- identifiants d'éléments cochés ;
- tags d'un document ;
- adresses IP déjà traitées.

# Quand préférer un tableau ?

Le tableau (`Array`) reste préférable lorsque :

- l'ordre des éléments est important ;
- on souhaite accéder à un élément par son indice ;
- des doublons sont autorisés ;
- on utilise des méthodes comme `map()`, `filter()`, `reduce()` ou `sort()`.

# Conclusion

Le `Set` est la structure de données idéale pour représenter un ensemble de valeurs uniques. Dans une interface utilisateur, il simplifie considérablement la gestion d'éléments sélectionnés (cases à cocher, identifiants, rôles, compétences…) en supprimant automatiquement les doublons et en offrant des opérations d'ajout, de suppression et de recherche simples et efficaces.