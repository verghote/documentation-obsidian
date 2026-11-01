# 1. Définition

Les tests unitaires permettent de vérifier que chaque unité de programme (instruction, méthode, classe, etc.) composant un module d'une application est bien conforme aux spécifications attendues.

- À partir de données de test, on vérifie que les données obtenues en sortie sont bien conformes aux attentes.
    
- Les tests unitaires permettent ainsi la mise en place d'une stratégie d'intégration continue rigoureuse.
# 2. Mise en œuvre sous Visual Studio

## 2.1 Création du projet de test

Pour ajouter un projet de test unitaire dans votre solution :

1. Faites un clic droit sur la solution.
    
2. Sélectionnez **Ajouter** > **Nouveau projet**.
    
3. Recherchez et choisissez **Projet de test unitaire** (basé sur le framework MSTest et la référence `Microsoft.VisualStudio.TestTools.UnitTesting`).
    
4. Nommez-le, par exemple `TestUnitaireArticle`.
5. Lier le projet de test au projet métier : clic droit > Références > Ajouter une référence de projet > cocher le projet Metier

Remarque : Il faut aussi que les packages NuGet nécessaires à MSTest soit installés

Pour les installer :

1. Dans l'**Explorateur de solutions**, faire un clic droit sur votre projet de test (`TestUnitaireArticle`) et choisissez **Gérer les packages NuGet...** (Manage NuGet Packages).
    
2. Aller dans l'onglet **Parcourir** (Browse) et recherchez : MSTest
3. Installer ou mettre à jour pour que le projet embarque la bibliothèque de test.

## 2.2 La classe d'assertions (`Assert`)

La bibliothèque met à disposition la classe statique `Assert` dont les méthodes permettent de valider les résultats de vos méthodes testées.

|**Méthode de la classe Assert**|**Rôle**|
|---|---|
|`AreEqual(attendu, réel)`|Vérifie si `attendu == réel`|
|`AreNotEqual(attendu, réel)`|Vérifie si `attendu != réel`|
|`IsFalse(condition)`|Vérifie que la condition est fausse|
|`IsTrue(condition)`|Vérifie que la condition est vraie|
|`IsNull(objet)`|Vérifie que l'objet spécifié est `null`|
|`IsNotNull(objet)`|Vérifie que l'objet spécifié n'est pas `null`|

> _Note :_ Toutes ces méthodes possèdent de nombreuses surcharges pour s'adapter à tous les types de données.

Quasiment toutes les méthodes d'assertion possèdent des surcharges (overloads) qui acceptent une chaîne de caractères (`string`) en fin de paramètre pour personnaliser le message d'échec.

### Exemples d'utilisation :

- **`Assert.IsTrue` :**
    
    ```C#
    Assert.IsTrue(compte2.estADecouvert(), "Erreur : Le compte devrait être à découvert à ce stade.");
    ```
    
- **`Assert.IsNull` :**
    
    ```C#
    Assert.IsNull(compteNull, "Erreur : L'objet compte devrait être null.");
    ```
    
- **`Assert.IsFalse` :**
    
    ```C#
    Assert.IsFalse(creditReussi, "Erreur : Un crédit avec un montant négatif n'aurait pas dû réussir.");
    ```

- **`Assert.AreEqual` :**
    
    ```C#
    Assert.AreEqual(1, compte1.getNumero(), "Erreur critique : Le numéro du compte 1 a été altéré !"); 
    Assert.AreEqual("Alice Dupont", compte1.getTitulaire(), "Le nom du titulaire ne correspond pas à la création.");
    ```
### Bonus : Formatage de message

Beaucoup de ces méthodes acceptent même des surcharges de type `string.Format` (avec des paramètres dynamiques) si vous avez besoin d'inclure des variables dans votre message d'erreur :

```C#
// Exemple de syntaxe avec formatage
Assert.AreEqual(valeurAttendue, valeurReelle, "Échec pour le compte ID {0} avec le titulaire {1}", idCompte, nomTitulaire);
```

C'est une excellente habitude  pour rendre les rapports de tests plus lisibles, en particulier lors de l'intégration continue ou sur des suites de tests comportant des dizaines de vérifications.

## 2.3 Structure type d'une classe de test

```C#
[TestClass]
[DoNotParallelize]
public class TestFonction
{
    [TestInitialize]
    public void InitialisationDesTests() 
    {
        // Actions exécutées avant chaque test
    }

    [TestCleanup]
    public void NettoyageDesTests() 
    {
        // Actions exécutées à la fin des tests
    }

    [TestMethod]
    public void TestEstDivisible() 
    {
        Assert.IsFalse(Fonction.EstDivisible(2, 5));
        // ...
    }
}
```

- **`[TestInitialize]`** et **`[TestCleanup]`** permettent d'initialiser et de restaurer l'environnement de test avant et après l'exécution de chaque méthode de test.
- `[TestClass]` : Indique à Visual Studio que cette classe contient des tests unitaires.
- `[DoNotParallelize]` : Empêche l'exécution simultanée des tests de cette classe (très utile si les tests modifient des attributs ou des états statiques partagés, évitant ainsi les conflits d'accès).

**Important : Le projet de test doit contenir une référence vers le projet contenant  la classe à tester**

## 3. Lancement des tests

Vous disposez de plusieurs solutions pour exécuter vos tests dans Visual Studio :

- **Solution 1 :** Cliquer sur l'icône de test directement dans l'éditeur de code (marge de gauche) puis sélectionner _Exécuter_ (ou _Déboguer_ si le test est en erreur).
    
- **Solution 2 :** Utiliser l'**Explorateur de tests** :
    
    1. Allez dans le menu supérieur : **Test** > **Explorateur de tests** (il est conseillé d'ancrer cette fenêtre).
        
    2. Ouvrez l'arborescence pour cibler le test souhaité.
        
    3. Faites un clic droit sur le test > _Exécuter_, ou utilisez le bouton principal **Exécuter tout** (triangle vert).
## 4. Exemples pratiques

### A. Test simple et utilisation de tableaux de valeurs

Soit une classe `Fonction` dans l'espace de noms `SIO` contenant une méthode `factoriel` :

```C#
namespace SIO
{
    public partial class Fonction 
    {
        public static long Factoriel(int n) 
        {
            // ...
        }
    }
}
```

Pour que le projet de test unitaire puisse accéder à cette classe, ajoutez une référence :

- Clic droit sur le projet de test > **Ajouter** > **Référence** > **Projets** > Cochez le projet contenant la classe `Fonction`.

```C#
using SIO;

namespace TestUnitaireFonction 
{
    [TestClass]
    public class TestFonctionMath 
    {
        [TestMethod]
        public void TestFactoriel() 
        {
            Assert.AreEqual(1, Fonction.Factoriel(1));
            Assert.AreEqual(3628800, Fonction.Factoriel(10));
        }

        // Alternative : Stocker l'ensemble des valeurs dans un tableau bidimensionnel
        [TestMethod]
        public void TestFactorielParTableau() 
        {
            int[,] lesTests = { 
                {0, 1}, {1, 1}, {2, 2}, {3, 6}, {4, 24}, 
                {5, 120}, {6, 720}, {7, 5040}, {8, 40320}, 
                {9, 362880}, {10, 3628800} 
            };

            int nbLigne = lesTests.GetLength(0);
            for (int i = 0; i < nbLigne; i++) 
            {
                Assert.AreEqual(lesTests[i, 1], Fonction.Factoriel(lesTests[i, 0]));
            }
        }
    }
}
```

_(Il est également possible d'utiliser d'autres conteneurs comme des collections ou des dictionnaires)._

### B. Test nécessitant l'initialisation d'objets (Classe `Transair`)

Certains tests nécessitent d'alimenter des objets avant de tester des méthodes de recherche (comme `indexOf` ou `getNom`).


```C#
[TestClass]
public class TestTransair 
{
    static Aeroport[] lesAeroports;

    [TestInitialize]
    public void InitialisationDesTests() 
    {
        lesAeroports = Transair.GetLesAeroports();
    }

    [TestMethod]
    public void TestIndexOfAeroport() 
    {
        Assert.AreEqual(0, Transair.IndexOf(lesAeroports, "CDG"));
        Assert.AreEqual(1, Transair.IndexOf(lesAeroports, "LLQ"));
        // ...
        Assert.AreEqual(-1, Transair.IndexOf(lesAeroports, "ZZZ"));
    }

    [TestMethod]
    public void TestNomAeroport() 
    {
        Assert.AreEqual("Toulouse Blagnac", Transair.GetNom(lesAeroports, "TLB"));
        Assert.AreEqual("Toulouse Blagnac", Transair.GetNom(lesAeroports, "tlb"));
        // ...
        Assert.AreEqual("Aéroport inexistant", Transair.GetNom(lesAeroports, "ZZZ"));
    }
}
```

## 5. Cas particuliers et bonnes pratiques

### Précision sur les types de données

Attention aux erreurs liées aux types stricts dans `Assert.AreEqual`.

- _Exemple :_ Tester la fonction `fibonacci()` qui retourne un `ulong` :
    
    ```C#
    // Générera une erreur de type (int vs ulong) malgré l'égalité de la valeur 0 :
    Assert.AreEqual(0, Fonction.Fibonacci(0)); 
    
    // Correction : Il faut caster explicitement la valeur attendue :
    Assert.AreEqual((ulong)0, Fonction.Fibonacci(0));
    ```

### Gestion des fichiers de ressources (Base de données, fichiers plats)

Si vos classes à tester s'appuient sur des fichiers externes (par exemple une base de données Access ou un fichier de configuration), **n'oubliez pas de copier ces ressources dans le répertoire de sortie (dossier `bin/Debug`) du projet de test unitaire**, de la même manière qu'elles le sont dans le projet principal.


# 6 Utilisation de la clause #nullable disable

Rappel sur ce qu'est le système de types "nullable" en C# 

Depuis C# 8.0, Microsoft a introduit une fonctionnalité de sécurité appelée **Nullable Reference Types (NRT)**. Par défaut, dans les projets modernes, le compilateur considère que **tous les types référence (comme `string`, `CompteBancaire`, etc.) ne peuvent jamais valoir `null`**, sauf si vous le précisez explicitement en ajoutant un point d'interrogation (`string?`).

Si vous déclarez un champ sans l'initialiser immédiatement, le compilateur lève des avertissements (ou des erreurs si l'option _Avertissements comme erreurs_ est active) pour vous forcer à sécuriser votre code.

**Pourquoi le compilateur râle-t-il dans une classe de test unitaire ?**

Regardez comment sont déclarés les objets au début de la classe de test :


```C#
[TestClass]
public class TestCompteBancaire {
    CompteBancaire compte1; // <--- Pas initialisé ici !
    CompteBancaire compte2; // <--- Pas initialisé ici !
}
```

Dans la logique classique du C#, le compilateur regarde le constructeur de la classe. Comme il n'y a pas de constructeur (ou qu'il est implicite), il se dit : _« Attention, ces variables vont valoir `null` au démarrage, c'est interdit ! »_.

Il lève alors l'avertissement **CS8618** :

> _« Le champ non nullable 'compte1' doit contenir une valeur non null lors de la sortie du constructeur. »_

Pourtant, dans un framework de test comme MSTest, ces variables **ne seront jamais nulles** au moment d'exécuter les tests, car elles sont initialisées juste avant via la méthode `[TestInitialize]` (`Init()`). Le compilateur, lui, ne comprend pas le cycle de vie magique de MSTest et s'alarme à tort.

Dans la documentation sur les types de référence nullable, nous avons vu plusieurs solutions pour éviter l'avertissement, la soluttion au niveau des tests unitaires réside dans l'utilisation de la classe  # nullable disable

**Pourquoi `#nullable disable` est la meilleure solution ici ?**

Sans `#nullable disable`, pour faire taire le compilateur, vous auriez dû utiliser des solutions lourdes et pénibles pour des étudiants :

1. **Rendre les champs "nullables" (`CompteBancaire? compte1;`) :** Le compilateur exigerait ensuite que vous mettiez des points d'exclamation partout dans vos tests (`compte1!.getNumero()`), rendant le code illisible et pollué par des symboles techniques.
    
2. **Initialiser avec un artifice (`= null!;`) :** Écrire `CompteBancaire compte1 = null!;` pour tromper le compilateur alourdit inutilement la syntaxe d'exercice.
    

En plaçant **`#nullable disable`** tout en haut du fichier de test, vous demandez simplement au compilateur de désactiver temporairement cette vérification stricte des `null` **pour ce fichier précis**.

### Les avantages :

- **Zéro pollution visuelle :** Les étudiants écrivent leurs tests de façon naturelle, exactement comme dans la documentation officielle, sans être bloqués par des alertes de compilation incompréhensibles à leur niveau.
    
- **Confort pédagogique :** On évite de perdre les étudiants sur des subtilités de configuration du compilateur C#, pour leur permettre de se concentrer exclusivement sur la logique des tests unitaires et des assertions (`Assert`).

