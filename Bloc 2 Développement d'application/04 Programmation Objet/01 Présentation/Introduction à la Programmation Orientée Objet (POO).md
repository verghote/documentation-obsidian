# 1. Présentation générale

La programmation orientée objet (POO) a pour but de créer des **composants réutilisables**. La conception des applications se limite ensuite à un assemblage de ces composants (ex: une fenêtre, un bouton, une zone de texte).

- **Programmation structurée :** Sépare d'un côté le code et de l'autre les données manipulées par ce code.
- **Programmation orientée objet :** Réunit dans une même structure les données (aspect statique) et les procédures ou fonctions (aspect dynamique).
### Exemple conceptuel : La classe Pile
```text
Class Pile
    -- Attributs privés
    lesValeurs : Tableau de caractères
    leSommet : entier

    -- Méthodes publiques
    Pile() // Constructeur
    procédure empiler(c) // Ajoute le caractère c au sommet de la pile
    fonction depiler() : caractère // Retire et retourne le caractère au sommet
    function estVide() : booléen // Retourne vrai si la pile est vide
Fin Classe
````

### Les trois principes fondamentaux

1. **La Classe :** Structure qui regroupe les attributs (aspect statique) et les méthodes (aspect dynamique) agissant sur ces objets. Elle possède une partie publique (visible de l'extérieur / interface) et une partie privée.
    
2. **L'Encapsulation :** Mécanisme permettant de cacher l'information contenue dans un objet et de ne proposer que des méthodes de manipulation. L'accès direct aux données est verrouillé ; il faut passer par l'interface publique obligatoire.
    
3. **L'Héritage :** Mécanisme permettant de définir une nouvelle classe à partir d'une classe existante. La classe dérivée hérite de l'ensemble des attributs et des méthodes de la classe de base.
 
## Avantages

- **Réduction du temps de développement** (développer = assembler des composants).
    
- **Maintenance facilitée** (possibilité de modifier un composant sans impacter toute l'application).

# 2. Définition des concepts de base

### Classe

- Regroupe des attributs et des méthodes dans une même structure.
    
- Représente le **modèle (le moule)** à partir duquel on fabrique des objets.

#### Tableau d'accessibilité (Visibilité)

|**Accessibilité**|**Programme utilisant les objets**|**Méthodes de la classe**|**Méthodes d'une classe dérivée**|
|---|---|---|---|
|**`+` Public** (Interface) : Visible par tous|Oui|Oui|Oui|
|**`-` Privé** : Visible uniquement par la classe|Non|Oui|Non|
|**`#` Protégé** : Visible par la classe et les classes dérivées|Non|Oui|Oui|

### Attribut

- Caractéristique propre à chaque objet de la classe.
- Défini par un nom unique, un type de données et une visibilité (privée, publique ou protégée).

### Attribut à portée classe (Attribut statique / `static`)

- Attribut attaché à la classe elle-même et non à une instance particulière.
- Il existe indépendamment de l'existence d'objets et est partagé par tous.
- _Exemple :_ Le code TVA d'un article s'il est unique.
- _Accès :_ `NomClasse.nomAttribut`

## Méthode

- Fonction ou procédure qui s'applique sur les objets de la classe.
### Méthode à portée classe (Méthode statique / `static`)

- S'applique sur la classe et non sur une instance.
- Une méthode de classe ne peut manipuler que des attributs de classe et ses propres paramètres (pas d'accès aux autres attributs de la classe).
- _Exemples :_ `getTauxTva() : réel`, `setTauxTva(unTauxTva : réel)`
- _Exemple de classe entièrement statique :_ La classe `Math` contenant les fonctions mathématiques.

### Accesseurs (`Getters` / `Setters`)

- Méthodes permettant de lire ou de modifier la valeur d'un attribut.
    
- _Règles de nommage :_
    
    - `getNomAttribut()` pour récupérer la valeur.
        
    - `setNomAttribut(valeur)` pour modifier la valeur.

### Propriété (Spécifique C#)

- Membre offrant un mécanisme d'accès simplifié à un attribut privé de la classe.
    
- Exemple :
    
    ```C#
    public int Quantite {
        get { return quantite; }
        set { quantite = value; }
    }
    ```
    
- Permet d'interagir directement en lecture/écriture :
    
    ```C#
    unArticle.Quantite -= uneQuantite;
    Console.WriteLine("Nouveau stock : {0}", unArticle.Quantite);
    ```
    
- _Remarque :_ Si on retire l'accesseur `set`, la propriété devient en lecture seule.
    
## Constructeur

- Méthode particulière qui porte le même nom que la classe et s'exécute automatiquement lors de l'instanciation (`new`).
- Permet d'initialiser les attributs de l'objet.

### Surcharge d'une méthode (Overloading)

- Principe consistant à fournir dans une classe plusieurs méthodes portant le même nom mais avec un nombre ou un type d'argument différent (très utilisé pour les constructeurs).
    
## Objet & Instanciation

- Un **objet** est une instance (occurrence) d'une classe.
    
- **L'instanciation** est l'opération qui consiste à créer un objet en mémoire et à renvoyer une référence vers cet objet.
    
- _Exemple de syntaxe :_
    
    ```
    unArticle <- new Article(1, 'clé usb', 10);
    ```
    
## Envoi de message

- Opération consistant à appeler un membre accessible d'un objet.
    
- _Exemples :_
    
    ```C#
    unArticle.Acheter(10)
    unArticle.Vendre(5)
    Afficher unArticle.getQuantite()
    ```
    

## 3. Représentation graphique (UML)

La représentation graphique correspond à la modélisation conceptuelle de la classe (diagramme de classes).

![[Pasted image 20261028102154.png|600]]
## Légende des visibilités

- **`+`** : Public
- **`-`** : Privé
- **`#`** : Protégé
    

> **Remarque :** Les constructeurs, accesseurs et paramètres des méthodes ne sont pas forcément présentés dans les schémas simplifiés.

![[Pasted image 20261028102707.png|300]]