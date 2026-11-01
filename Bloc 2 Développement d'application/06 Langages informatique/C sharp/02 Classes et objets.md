# Introduction : Fondamentaux de la Programmation Orientée Objet (POO)

La POO a pour but de concevoir des composants logiciels réutilisables. Au lieu d'écrire des séquences d'instructions monolithiques, l'application est pensée comme un assemblage d'objets autonomes qui interagissent entre eux. Les piliers fondamentaux reposent sur :

1. **L'Encapsulation :** C'est le principe de regrouper les données (attributs) et les traitements (méthodes) au sein d'une même structure (la classe), tout en masquant l'état interne de l'objet pour le protéger des modifications corrompues ou non autorisées de l'extérieur.
    
2. **État et Comportement :** Un objet possède un **état** (ses attributs à un instant donné) et un **comportement** (ses méthodes qui permettent de faire évoluer cet état ou d'interagir avec d'autres objets).

# 1. Conventions de Nommage (Spécificité BTS SIO)

> ⚠️ **Particularité importante en BTS SIO :** Bien que les standards officiels du langage C# préconisent le `PascalCase` pour les méthodes (ex: `CalculerTotal()`), **les sujets et attendus en BTS SIO utilisent traditionnellement le `camelCase` pour les méthodes** (ex: `calculerTotal()`). Soyez vigilants et adaptez-vous aux consignes de vos enseignants ou des sujets d'examen !

|**Élément**|**Convention**|**Exemple**|
|---|---|---|
|**Classes, Interfaces, Enums**|`PascalCase` (Première lettre en majuscule)|`Article`, `IComparable`|
|**Propriétés C# (moderne)**|`PascalCase`|`Prix`, `Designation`|
|**Méthodes et Accesseurs**|`camelCase` _(Spécificité BTS SIO)_|`getPrix()`, `calculerMontant()`|
|**Attributs privés (Champs)**|`camelCase` ou `_camelCase`|`reference`, `_reference`|
|**Paramètres et variables locales**|`camelCase`|`nouveauPrix`, `taux`|
## 2. Structure Générale d'une Classe

Voici la structure de référence d'une classe complète en C# regroupant l'ensemble des bonnes pratiques :

```C#
namespace Metier
{
    p4ublic class Article : IComparable<Article>
    {
        // 1. Attributs statiques (portée classe)
        private static double tauxTVA = 20.0;

        // 2. Attributs d'instance (privés et sécurisés)
        private readonly int reference;
        private string designation;
        private decimal prix;
        private List<string> historiqueModifications = new();

        // 3. Constructeurs
        public Article() { } // Constructeur par défaut

        public Article(int reference, string designation, decimal prix)
        {
            this.reference = reference;
            this.designation = designation;
            this.prix = prix;
        }

        // 4. Accesseurs traditionnels
        public decimal getPrix() => prix;

        // 5. Propriétés (Encapsulation moderne)
        public decimal Prix
        {
            get => prix;
            set => prix = value >= 0 ? value : throw new ArgumentException("Le prix ne peut pas être négatif.");
        }

        // 6. Méthodes métier
        public void appliquerRemise(decimal pourcentage)
        {
            if (pourcentage > 0 && pourcentage <= 100)
                prix -= prix * (pourcentage / 100m);
        }

        // 7. Égalité et Opérateurs
        public override bool Equals(object? obj) => obj is Article article && reference == article.reference;
        public override int GetHashCode() => HashCode.Combine(reference);
        public static bool operator ==(Article? a1, Article? a2) => ReferenceEquals(a1, a2) || (a1 is not null && a1.Equals(a2));
        public static bool operator !=(Article? a1, Article? a2) => !(a1 == a2);

        // 8. Tri naturel (IComparable)
        public int CompareTo(Article? other) => other is null ? 1 : reference.CompareTo(other.reference);
    }
}
```
# 3. Les Attributs (Champs / Fields)

Les attributs stockent l'état brut de l'objet. Par essence, pour garantir l'encapsulation, ils doivent **toujours** être déclarés en `private`.

- **Le modificateur `readonly` :** Utilisé pour les données immuables après la création de l'objet (comme la référence unique d'un article). Une fois le constructeur exécuté, la valeur ne peut plus jamais être modifiée.
    
- **Les attributs statiques (`static`) :** Ils appartiennent à la classe elle-même et non à une instance particulière. Ils sont partagés par l'ensemble des objets créés à partir de cette classe (ex: un taux de TVA commun à tous les articles).
    
- **Initialisation directe :** Initialiser un attribut à sa déclaration (ex: `private List<string> historique = new();`) garantit qu'il disposera d'une référence valide en mémoire dès la construction de l'objet, évitant ainsi les redoutables exceptions de type `NullReferenceException` si le développeur oublie d'initialiser la liste dans un constructeur.
# 4. Les Constructeurs

Le constructeur est une méthode spéciale qui porte le même nom que la classe, ne possède **aucun type de retour** (pas même `void`), et s'exécute automatiquement lors de l'instanciation (`new`). Son rôle unique est d'initialiser l'état de l'objet.
### A. Constructeur principal et chaînage (`this`)

Il est possible de surcharger les constructeurs pour proposer différentes signatures selon les besoins d'initialisation :

```C#
public Article(int reference, string designation, decimal prix, int quantite)
{
    this.reference = reference;
    this.designation = designation;
    this.prix = prix;
    this.quantite = quantite;
}

// Surcharge déléguant l'initialisation au constructeur principal via le mot-clé this
public Article(int reference, string designation, decimal prix) 
    : this(reference, designation, prix, 0) { }
```

### B. Le constructeur par copie

Il permet de créer un nouvel objet en dupliquant strictement les valeurs d'un objet existant. Cela évite le simple partage de référence mémoire (où deux variables pointent vers le même emplacement dans le _Heap_) en garantissant un clone totalement indépendant :

```C#
public Article(Article autreArticle)
{
    if (autreArticle == null) {
        throw new ArgumentNullException(nameof(autreArticle));
    }    
    this.reference = autreArticle.reference;
    this.designation = autreArticle.designation;
    this.prix = autreArticle.prix;
}
```

## 5. Les Accesseurs Traditionnels (`get...` et `set...`)

Avant l'apparition des propriétés, l'encapsulation passait par des méthodes explicites :

- **Les Getters (`get...`) :** Permettent de lire la valeur d'un attribut privé en lecture seule.
    
- **Les Setters (`set...`) :** Permettent de modifier l'attribut tout en appliquant un contrôle de validité.

Pour valider une modification sans bloquer brutalement le programme, un setter traditionnel retourne traditionnellement un booléen (`true` en cas de succès, `false` si la valeur est incorrecte) :

```C#
public class Article
{
    private decimal prix;

  // Getter traditionnel
    public decimal getPrix()
    {
        return prix;
    }


    // Setter traditionnel avec retour booléen
    public bool setPrix(decimal nouveauPrix)
    {
        if (nouveauPrix >= 0)
        {
            prix = nouveauPrix;
            return true; // Succès
        }
        return false; // Échec (valeur invalide)
    }
}
```

> **Limite majeure :** Le code appelant est obligé de tester systématiquement le retour de la méthode (`if (!monArticle.setPrix(-10)) { ... }`), ce qui alourdit l'écriture et est souvent omis par les développeurs.

```C#
public class Article
{
    private decimal prix;

    public decimal getPrix() => prix;

    // Setter traditionnel avec contrôle de validité
    public void setPrix(decimal nouveauPrix)
    {
        if (nouveauPrix >= 0)
        {
            prix = nouveauPrix;
        }
        else
        {
            throw new ArgumentException("Le prix ne peut pas être négatif !");
        }
    }
}
```
# 6. Les Méthodes Métier

Elles définissent le comportement dynamique de l'objet et encapsulent la logique applicative (calculs, modifications de l'état interne, règles de gestion) :

``` C#
public class Article
{
    private decimal prix;
    private int quantiteStock;

    public void appliquerRemise(decimal pourcentage)
    {
        if (pourcentage > 0 && pourcentage <= 100)
        {
            prix -= prix * (pourcentage / 100m);
        }
    }
}
```

# 7. Égalité d'Objets et Surcharge des Opérateurs (`==` et `!=`)

Par défaut, la comparaison de deux objets avec l'opérateur `==` compare leurs **adresses mémoire** (références). Pour comparer leur contenu métier (ex: deux articles sont égaux s'ils ont la même référence), il faut redéfinir `Equals()` et `GetHashCode()`.

Cependant, cela ne suffit pas pour que les opérateurs `==` et `!=` fonctionnent intuitivement sur des types référence. **Il faut explicitement les surcharger :**

```C#
public class Article
{
    private int reference;

    public override bool Equals(object? obj) => obj is Article article && reference == article.reference;

    public override int GetHashCode() => HashCode.Combine(reference);

    // Surcharge de l'opérateur ==
    public static bool operator ==(Article? a1, Article? a2)
    {
        if (ReferenceEquals(a1, a2)) return true;
        if (a1 is null || a2 is null) return false;
        return a1.Equals(a2);
    }

    // Surcharge de l'opérateur !=
    public static bool operator !=(Article? a1, Article? a2) => !(a1 == a2);
}
```

## 8. Tri des Objets : L'interface `IComparable<T>`

### Pourquoi faut-il hériter de l'interface `IComparable<T>` ?

Pour qu'une collection (comme une `List<Article>`) sache se trier automatiquement lors de l'appel de la méthode `.Sort()`, le framework .NET a besoin d'un contrat universel garantissant que chaque objet de la classe sait se comparer à un autre de son propre type.

En implémentant `IComparable<Article>` :

1. **On définit un tri naturel :** La classe embarque intrinsèquement sa propre règle de classement par défaut (ex: trier par référence croissante).
    
2. **Sécurité et standardisation :** Cela garantit la compatibilité avec tous les algorithmes de tri de la bibliothèque .NET.
    

```C#
public class Article : IComparable<Article>
{
    private int reference;

    public int CompareTo(Article? other)
    {
        if (other is null) return 1;
        return reference.CompareTo(other.reference);
    }
}
```
# 9. La Spécificité C# : Les Propriétés (`get` et `set`) et l'introduction des Exceptions

### Pourquoi les propriétés ont-elles été introduites en C# ?

L'utilisation de méthodes `getPrix()` et `setPrix()` alourdit la lecture du code (`monArticle.setPrix(monArticle.getPrix() + 5)`). Le C# introduit donc la **Propriété**, qui combine la syntaxe fluide d'un accès direct à un attribut (`monArticle.Prix = 50;`) tout en exécutant en arrière-plan la logique de contrôle.

### Le rôle fondamental des exceptions (`throw new ArgumentException`)

Contrairement à la méthode `setPrix()` qui retournait un booléen, **une affectation de propriété s'écrit sous forme d'une simple assignation (`monArticle.Prix = -10;`) et ne renvoie aucune valeur**. Il est donc impossible de signaler silencieusement un échec.

C'est ici qu'intervient l'exception (`throw`) :

- **Pédagogie de l'exception :** Une exception n'est pas qu'un « message d'erreur rouge ». C'est un mécanisme de rupture de sécurité qui **stoppe net l'exécution du programme** lorsqu'un état interdit est atteint. Elle empêche formellement un objet de se retrouver dans un état incohérent ou corrompu en mémoire.

```C#
public class Article
{
    private decimal prix;

    public decimal Prix
    {
        get => prix;
        set 
        {
            if (value >= 0) prix = value;
            else throw new ArgumentException("Le prix ne peut pas être négatif."); // Stoppe le programme en cas d'erreur critique
        }
    }
}
```

## 10. Les Propriétés Automatiques et Leurs Variantes (`init`, `private set`, `required`)

### A. Qu'est-ce qu'une propriété automatique ?

Lorsque vous écrivez une propriété qui se contente de lire et d'écrire une valeur **sans appliquer aucune règle de contrôle** (pas de vérification de borne, pas d'exception), il devient lourd de déclarer manuellement un attribut privé en plus de la propriété.

C'est là qu'interviennent les **propriétés automatiques** (`{ get; set; }`) :

- **Plus de champ privé à gérer explicitement :** Le champ privé sous-jacent existe toujours en mémoire, mais il est entièrement géré et pris en charge de manière transparente par le compilateur.
    
- **Avantages :** Le code est considérablement allégé, plus lisible, et s'intègre parfaitement dans l'architecture .NET.
    
- **La condition absolue pour sa mise en place :** Puisque le compilateur génère lui-même le code du `set`, **il est impossible d'y ajouter des règles de contrôle ou des exceptions** (comme vérifier si un prix est positif). Par conséquent, **on ne peut pas transformer tous les attributs en propriétés automatiques** : dès qu'une règle métier s'applique, vous _devez_ obligatoirement repasser par un attribut privé explicite et une propriété complète (comme vu au point 9).
    
### B. Les variantes avancées des propriétés automatiques

Le C# moderne permet de verrouiller la modification de ces propriétés grâce à des modificateurs précis :

- **`private set;` :** La propriété peut être lue librement de l'extérieur, mais sa modification est restreinte exclusivement à l'intérieur de la classe. Le code extérieur ne peut pas l'altérer directement.
    
- **`init;` (C# 9) :** C'est le pilier de l'**immutabilité contrôlée**. Une propriété en `init` peut recevoir une valeur **uniquement au moment de la création de l'objet** (soit dans le constructeur, soit via un initialiseur d'objet). Une fois l'objet instancié, la valeur est verrouillée en lecture seule et ne peut plus jamais être modifiée.
    
- **`required` (C# 11) :** Force le développeur à renseigner impérativement cette propriété lors de l'instanciation. Si elle est oubliée, le code refuse de compiler.
```C#
public class Article
{
    public string Designation { get; set; } = string.Empty; // Lecture/Écriture libre
    public int Reference { get.init; }                       // Immuable après création
    public decimal Prix { get; private set; }                // Modifiable uniquement en interne
    public required string Categorie { get; set; }           // Obligatoire à l'instanciation
}
```

# 11. Instanciation avec les accolades `{}` (Initialiseurs d'objets)

## A. Origine et Version (.NET / C#)

Introduits depuis **C# 3.0** (avec le .NET Framework 3.5 en **2007**), les **initialiseurs d'objets** permettent d'instancier une classe tout en assignant directement la valeur de ses propriétés publiques ou accessibles via des accolades `{}`, sans avoir à multiplier les surcharges de constructeurs.

## B. Intérêts et Avantages

1. **Lisibilité et clarté (code auto-descriptif) :** Contrairement à un constructeur lourd où l'on passe des paramètres positionnels (ex: `new Article(101, "Souris", 29.99m)` où l'on doit deviner l'ordre des arguments), l'initialiseur nomme explicitement chaque champ :
    
    ``` C#
    Article monArticle = new Article 
    {
        Reference = 101,
        Designation = "Souris sans fil",
        Categorie = "Informatique"
    };
    ```
    
2. **Évite l'explosion des constructeurs :** Plus besoin d'écrire 5 constructeurs différents pour gérer les combinaisons de paramètres optionnels. Un seul constructeur par défaut couplé à un initialiseur suffit.
    
3. **Flexibilité :** On peut renseigner uniquement les propriétés dont on a besoin à l'instant $t$.
## C. Inconvénients et Limites

1. **État transitoire et partiel :** L'objet est d'abord instancié vide (appel du constructeur par défaut), puis ses propriétés sont modifiées une à une de l'extérieur. Si une règle métier complexe exige que _tous_ les champs soient validés simultanément et de manière atomique à la création, l'initialiseur peut laisser l'objet dans un état incohérent ou temporairement incomplet.
    
2. **Contrainte d'accessibilité :** Les propriétés modifiées doivent impérativement posséder un accesseur en écriture accessible (`set` ou `init`), ce qui peut aller à l'encontre d'une encapsulation stricte si l'on ne maîtrise pas la visibilité.

### D. Le piège technique : Le Constructeur par Défaut et l'attribut `[SetsRequiredMembers]`

- **Comportement natif :** Si vous n'écrivez **aucun** constructeur, le compilateur génère automatiquement un constructeur sans paramètre (`public Article() { }`), rendant l'initialiseur d'objet possible. Dès que vous écrivez **votre propre constructeur** (ex: avec paramètres), ce constructeur par défaut disparaît. Si vous tentez d'utiliser `new Article { ... }` sans l'avoir réécrit explicitement, **le code ne compile plus**.
    
- **La gestion de `required` :** Pour obliger l'appelant à renseigner des champs clés tout en autorisant l'initialiseur, le constructeur personnalisé doit être décoré avec l'attribut **`[SetsRequiredMembers]`** (du namespace `System.Diagnostics.CodeAnalysis`) pour garantir au compilateur que les propriétés requises sont bien prises en charge.

## Exemple  :

```C#
using System.Diagnostics.CodeAnalysis;

namespace Metier
{
    public class Article
    {
        public required int Reference { get; init; }
        public required string Designation { get; set; }
        public decimal Prix { get; set; }

        // 1. Indispensable pour autoriser l'initialiseur d'objet new Article { ... }
        public Article() { }

        // 2. Constructeur personnalisé sécurisé par [SetsRequiredMembers]
        [SetsRequiredMembers]
        public Article(int reference, string designation, decimal prix)
        {
            Reference = reference;
            Designation = designation;
            Prix = prix;
        }
    }
}
```

## 12. Visualisation : Diagramme de Classes (Visual Studio)

Pour visualiser graphiquement l'architecture de vos classes sous Visual Studio :

1. Faites un clic droit sur le projet de bibliothèque de classes $\rightarrow$ **Ajouter** $\rightarrow$ **Nouvel élément** $\rightarrow$ **Diagramme de classe**.
    
2. Glissez-déposez les classes depuis l'Explorateur de solutions sur le canevas.
    
3. Pour afficher les associations ou relations de collection, faites un clic droit sur un membre de la classe $\rightarrow$ **Afficher en tant qu'association** (nécessite que le composant _Concepteur de classes_ soit installé via le programme d'installation de Visual Studio).