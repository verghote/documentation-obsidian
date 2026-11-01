## 1. Présentation

La classe `List<T>` implémente un tableau dynamique d’éléments de type ou de classe `T`. Elle est définie dans l'espace de noms `System.Collections.Generic`.

- L’opérateur d'indexation `[]` donne accès à un élément de la collection : `lesElements[i]`.
    
- La numérotation (l'indexation) des éléments commence à **0**.
    

🔗 [Documentation officielle Microsoft](https://docs.microsoft.com/fr-fr/dotnet/api/system.collections.generic.list-1?view=netframework-4.8)

## 2. Les membres de la classe

### Constructeurs

|**Constructeur**|**Rôle**|
|---|---|
|`List<T>()`|Instancie une nouvelle collection vide d'éléments de type ou de classe `T`.|
|`List<T>(IEnumerable<T>)`|Initialise une nouvelle instance de la classe `List<T>` qui contient des éléments copiés à partir de la collection spécifiée et possède une capacité suffisante pour accepter le nombre d'éléments copiés.|

### Propriétés

|**Propriété**|**Type**|**Rôle**|
|---|---|---|
|`Count`|`int`|Nombre d’éléments actuellement présents dans la collection.|

### Principales Méthodes

|**Méthode**|**Type de retour**|**Rôle**|
|---|---|---|
|`Add(T x)`|`void`|Ajoute l’objet `x` à la fin de la collection.|
|`Clear()`|`void`|Vide la collection de tous ses éléments.|
|`Contains(T x)`|`bool`|Renvoie `true` si la collection contient l’objet `x`.|
|`IndexOf(T x)`|`int`|Renvoie l’index de l’objet `x` dans la collection, ou `-1` si l’objet n’est pas trouvé.|
|`IndexOf(T x, int p)`|`int`|Idem, mais la recherche commence à partir de la position `p`.|
|`Insert(int p, T x)`|`void`|Insère l’objet `x` en $p$-ième position.|
|`Remove(T x)`|`bool`|Retire l’objet `x` de la collection. Retourne `false` si l'élément est inexistant.|
|`RemoveAt(int p)`|`void`|Retire l’objet situé en $p$-ième position.|
|`Sort()`|`void`|Trie les objets de la collection selon le critère défini dans l'implémentation de `IComparable<T>`.|
|`Sort(IComparer<T> critere)`|`void`|Trie la collection selon le comparateur spécifié.|
|`Reverse()`|`void`|Inverse l'ordre de tri de la collection.|
|`Exists(Predicate<T>)`|`bool`|Retourne `true` s'il existe au moins un élément vérifiant la condition exprimée dans le prédicat.|
|`Find(Predicate<T>)`|`T`|Retourne le premier élément vérifiant la condition, ou `null` si non trouvé.|
|`FindIndex(Predicate<T>)`|`int`|Retourne l'index du premier élément vérifiant la condition, ou `-1` si non trouvé.|
|`FindAll(Predicate<T>)`|`List<T>`|Retourne tous les objets de la collection vérifiant la condition dans une nouvelle collection de même type.|
|`RemoveAll(Predicate<T>)`|`int`|Supprime les objets vérifiant la condition et retourne le nombre d'éléments supprimés.|

### Focus sur les prédicats

Un **prédicat** pointe vers une méthode qui renvoie simplement `true` ou `false`. Il existe trois manières de passer un prédicat en paramètre d'une méthode :

1. **Utilisation d'une expression lambda (notation fléchée) :**
    
    
    ```C#
    lesEtudiants.RemoveAll(etudiant => etudiant.Option == "SISR");
    ```
    
2. **Utilisation d'une méthode (fonction) anonyme :**
    
       
    ```C#
    lesEtudiants.RemoveAll(delegate (Etudiant e) { return e.Option == "SISR"; });
    ```
    
3. **Utilisation d'un pointeur de fonction (méthode classique) :**
    
    ```C#
    lesEtudiants.RemoveAll(etudiantSisr);
    
    // Définition de la méthode
    bool static private etudiantSisr(Etudiant e) { 
        return e.Option == "SISR"; 
    }
    ```

## 3. Rappel sur la comparaison de deux objets

Si l'on compare deux objets par `o1 == o2`, ce sont **par défaut les références (adresses mémoire)** qui sont comparées, et non le contenu des objets.

Pour que deux objets puissent être comparés sur leur contenu, il est nécessaire de :

- Redéfinir la méthode `Equals` de la classe `Object` pour permettre de comparer `o1` et `o2` via `o1.Equals(o2)`.
    
- Redéfinir les opérateurs `==` et `!=`.
    
- Redéfinir la fonction `GetHashCode` de la classe `Object`.
    

Ces redéfinitions sont indispensables si l'on veut modifier le comportement des méthodes `Contains`, `IndexOf` et `Remove` pour qu'elles utilisent le contenu des objets plutôt que leur adresse mémoire.

### Exemple de surcharge dans la classe `Etudiant` :


```C#
public override bool Equals(object o) {
    Etudiant a = (Etudiant)o;
    int res = nom.CompareTo(a.nom);
    if (res == 0)
         res = prenom.CompareTo(a.prenom);
    return res == 0;
}

public static bool operator ==(Etudiant a1, Etudiant a2) {
    if (a1 is null)
        return a2 is null;
    else
        return a2 is null ? false : a1.Equals(a2);
}

public static bool operator !=(Etudiant a1, Etudiant a2) { 
    return !(a1 == a2); 
}

public override int GetHashCode() { 
    return this.ToString().GetHashCode(); 
}
```

_Note : Le recours aux méthodes `Find`, `FindIndex`, `Exists` ou `RemoveAll` avec des expressions lambda peut parfois s'affranchir de ces surcharges._

## 4. Exemples d'utilisation

_Les exemples suivants se basent sur des objets de la classe `Etudiant` possédant 3 propriétés : `Nom`, `Prenom` et `Option`._

### 4.1. Déclaration et instanciation

```C#
// Liste vide
List<Etudiant> lesEtudiants = new List<Etudiant>();

// Avec initialisation des valeurs à la volée
List<Etudiant> lesEtudiantsAvecValeurs = new List<Etudiant>() {
    new Etudiant("BOULONGNE", "Alexandre", "SISR"),  
    new Etudiant("CREUZOT", "Jules", "SLAM")
};
```

### 4.2. Ajout dans la collection

```C#
// En passant par une variable intermédiaire
Etudiant x = new Etudiant("COLLINA", "MATHÉO", "SLAM");
lesEtudiants.Add(x);

// Directement (sans variable)
lesEtudiants.Add(new Etudiant("CUISINIER", "Hugo", "SISR"));
```

### 4.3. Parcours complet

```C#
// Parcours avec foreach (recommandé)
foreach (Etudiant x in lesEtudiants) {
    Console.WriteLine(x.ToString());
}

// Le parcours avec une boucle for classique est également possible
for (int i = 0; i < lesEtudiants.Count; i++) {
    Console.WriteLine(lesEtudiants[i].ToString());
}
```

### 4.4. Récupération de la position (index) d'un objet

```C#
// 'nom' et 'prenom' sont des variables fournies par l'utilisateur
int index = lesEtudiants.FindIndex(x => x.Nom == nom && x.Prenom == prenom);   
// index contient -1 si aucun étudiant ne correspond
```

> **Rappel (Sensibilité à la casse) :** En C#, la comparaison sur les chaînes de caractères est sensible à la casse.
> 
> - `ToUpper()` ou `ToLower()` permettent de contourner le problème mais ne sont pas recommandés.
>     
> - Il est préférable d'utiliser la méthode statique `string.Compare(x, y, true)` (le `true` ignorant la casse).
>     
> - Pour ignorer aussi les accents : `string.Compare(x, y, CultureInfo.CurrentCulture, CompareOptions.IgnoreNonSpace | CompareOptions.IgnoreCase);`
>     

### 4.5. Récupération d'un objet spécifique

```C#
Etudiant x = lesEtudiants.Find(etudiant => 
    string.Compare(etudiant.Nom, nom, true) == 0 && 
    string.Compare(etudiant.Prenom, prenom, true) == 0
);   
// x contient null si aucun étudiant ne correspond
```

### 4.6. Récupérer une sous-collection

```C#
// Filtrage via FindAll (LINQ-like intégré)
List<Etudiant> lesSLAM = lesEtudiants.FindAll(x => x.Option == "SLAM");

// Équivalent par un parcours manuel de la collection
List<Etudiant> lesSLAMParcours = new List<Etudiant>();
foreach (Etudiant x in lesEtudiants) {
    if (x.Option == "SLAM") {
        lesSLAMParcours.Add(x);
    }
}
```

## 5. Le tri des éléments d'une collection

Le tri ne pose pas de problème pour des types simples (entiers, chaînes), mais nécessite un critère de comparaison pour des objets complexes.

### 5.1. Intégration du critère dans la classe (`IComparable<T>`)

La classe doit implémenter l'interface `System.IComparable<NomClasse>` et sa méthode `CompareTo` :

```C#
public class Etudiant : IComparable<Etudiant> {
    public string Nom { get; set; }
    public string Prenom { get; set; }

    public int CompareTo(Etudiant e) {
        int res = Nom.CompareTo(e.Nom);
        if (res == 0)
            res = Prenom.CompareTo(e.Prenom);
        return res;
    }    
}
```

_Utilisation :_

C#

```
lesEtudiants.Sort(); // Utilise automatiquement la logique de CompareTo
```

### 5.2. Définition du critère en dehors de la classe (via expressions lambda)

```C#
// Tri sur un critère unique (chaîne)
lesEtudiants.Sort((x, y) => string.Compare(x.Option, y.Option, true)); 

// Tri sur un critère unique (entier)
lesEtudiants.Sort((x, y) => x.Id - y.Id); 

// Tri sur un critère multiple (Nom puis Prénom)
lesEtudiants.Sort((x, y) => {
    int res = string.Compare(x.Nom, y.Nom);
    if (res != 0)
        return res;
    return string.Compare(x.Prenom, y.Prenom);
}); 
```

## 6. Définir un itérateur dans une classe conteneur

Si une classe personnalisée (ex: `Section`) contient une collection d'objets `List<T>`, il est souvent souhaitable de permettre son parcours direct via un `foreach(T element in C)`.

- **Le problème :** Fournir un accesseur direct `get` sur la liste interne brise l'encapsulation (le code extérieur pourrait modifier la liste sans passer par les méthodes de la classe).
    
- **La solution :** Implémenter l'interface `IEnumerable` de l'espace de noms `System.Collections` et utiliser le mot-clé `yield return`.
    
```C#
using System.Collections;
using System.Collections.Generic;

public class Section : IEnumerable {
    private string code;
    private List<Etudiant> lesEtudiants;

    public IEnumerator GetEnumerator() {
        foreach (Etudiant x in lesEtudiants) {
            yield return x;
        }
    }
}
```

_Utilisation par un programme tiers :_

```C#
Section sio = new Section();
// ... remplissage de la section ...
foreach(Etudiant e in sio) {
    Console.WriteLine(e);
}
```

_Astuce : Il est possible de fixer un ordre de lecture particulier en triant la collection à l'intérieur de l'itérateur :_

```C#
public IEnumerator GetEnumerator() {
    lesEtudiants.Sort((x, y) => string.Compare(x.Option, y.Option));
    foreach (Etudiant x in lesEtudiants) {
        yield return x;
    }
}
```

## 7. Les principales méthodes d'extension LINQ

**LINQ** (Language Integrated Query) propose un ensemble d'extensions applicables sur tous les objets implémentant `IEnumerable<T>` (comme les `List<T>`). Contrairement à la méthode `Sort()` qui modifie la liste en place, les méthodes LINQ créent de nouvelles séquences sans altérer la collection d'origine.

### Les tris (`OrderBy` / `ThenBy`)

```C#
// Tri simple ascendant
foreach(Etudiant e in lesEtudiants.OrderBy(x => x.Nom)) {
    Console.WriteLine(e);
}

// Tri multi-critères (Nom puis Prénom)
foreach(Etudiant e in lesEtudiants.OrderBy(x => x.Nom).ThenBy(x => x.Prenom)) {
    // ...
}

// Tris décroissants : OrderByDescending et ThenByDescending

// Convertir le résultat LINQ en List<Etudiant>
List<Etudiant> listeTriee = lesEtudiants.OrderBy(x => x.Nom).ToList();
```

_(Attention : `OrderBy`/`ThenBy` sont généralement un peu plus lents que `Sort()` sur de petites/moyennes collections car ils allouent de nouvelles structures.)_

### Le filtrage (`Where`)

```C#
foreach (Etudiant e in lesEtudiants.Where(x => x.Promotion == 2024)) {
    Console.WriteLine(e);
}
```

### Les fonctions d'agrégation (`Min`, `Max`, `Average`, `Sum`, `Count`)

```C#
// La plus ancienne promotion
int premierePromotion = lesEtudiants.Min(x => x.Promotion);

// Le nombre d'étudiants dans une promotion spécifique
int nb = lesEtudiants.Count(x => x.Promotion == 2024);
```

### Le regroupement (`GroupBy` et `Select`)

LINQ permet d'effectuer des opérations proches du SQL, comme des regroupements :

```C#
var etudiantsParPromotion = lesEtudiants
    .GroupBy(e => e.Promotion)
    .Select(g => new { 
        Promotion = g.Key, 
        NombreEtudiants = g.Count() 
    })
    .OrderBy(g => g.Promotion)
    .ToList();
```

> **Le mot-clé `var` :** Indique au compilateur de déduire automatiquement le type d'une variable à l'initialisation (typage implicite fort). Le type est figé à la compilation et ne peut plus changer par la suite.