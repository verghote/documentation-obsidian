
En PHP, une classe peut être conçue de deux façons :

- comme une **classe statique** ;
- comme une **classe instanciable** (classe objet).

Le choix dépend avant tout de la responsabilité de la classe.

# La classe statique

Une classe statique ne possède pas d'état.

Toutes ses méthodes sont déclarées `static` et sont appelées directement sur la classe.

Exemple :

```php
$coureurs = Coureur::getAll();

$coureur = Coureur::getByLicence($licence);
```

Aucun objet n'est créé.

# Quand utiliser une classe statique ?

Une classe statatique est adaptée lorsqu'elle :

- ne mémorise aucune information ;
- produit toujours le même résultat à partir des paramètres reçus ;
- ne possède aucun état interne.

C'est exactement le cas des classes :

- Coureur
- Club
- Catégorie
- ReponseJson
- Jeton
- Requete

Par exemple :

```php
Coureur::getByCategorie("JU");
```

Cette méthode :

- reçoit un paramètre ;
- exécute une requête SQL ;
- retourne le résultat.

Elle ne conserve rien en mémoire.

# Avantages

### Simplicité

Aucune instanciation.

```php
Coureur::getAll();
```

au lieu de

```php
$coureur = new Coureur();

$coureur->getAll();
```

### Lisibilité

On comprend immédiatement que la méthode :

- ne dépend d'aucun objet ;
- ne modifie aucun état.

## Performances

Aucun objet n'est créé.

Même si le gain est faible avec les versions récentes de PHP, cela reste légèrement plus léger.

## Idéal pour les classes utilitaires

Par exemple :

```php
ReponseJson::envoyerLesDonnees(...);

Jeton::verifier();

Requete::postString(...);
```

Ces classes ne font qu'exécuter une action.

## Inconvénients

Une classe statique ne peut pas conserver d'informations.

Par exemple, ceci est impossible proprement :

```php
Validation::validerNom(...);

Validation::validerPrenom(...);

Validation::validerTelephone(...);
```

Comment récupérer toutes les erreurs ?

Il faudrait :

- retourner un tableau à chaque méthode ;
- utiliser des variables statiques (à éviter).

La classe devient rapidement moins agréable à utiliser.

# La classe objet

Une classe objet est instanciée.

```php
$validateur = new CoureurValidateur();
```

Chaque objet possède son propre état.

## Quand utiliser une classe objet ?

Une classe objet est adaptée lorsqu'elle doit :

- mémoriser des informations ;
- évoluer au cours de son utilisation ;
- représenter un véritable objet métier ou technique.

# Exemple avec le validateur

Ton validateur contient :

```php
private array $erreurs = [];
```

Chaque appel :

```php
$validateur
    ->validerSexe(...)
    ->validerClub(...)
    ->validerCategorie(...);
```

complète progressivement ce tableau.

À la fin :

```php
$validateur->getErreurs();
```

retourne toutes les erreurs rencontrées.

Le validateur possède donc un état.

Il est donc naturel qu'il soit un objet.

## Avantages

### Conservation d'un état

Chaque objet mémorise ses propres informations.

Exemple :

```php
private array $erreurs = [];
```

### Écriture fluide

Grâce au retour de `self` :

```php
(new CoureurValidateur())
    ->validerSexe($sexe)
    ->validerClub($club)
    ->validerCategorie($categorie);
```

Le code est très lisible.

### Plusieurs validations simultanées

On peut créer plusieurs validateurs indépendants.

```php
$v1 = new CoureurValidateur();

$v2 = new CoureurValidateur();
```

Chaque objet possède son propre tableau d'erreurs.

### Évolution facile

Demain on pourra ajouter :

```php
private int $nbErreurs;

private bool $stopPremiereErreur;

private array $valeursNettoyees;
```

sans modifier l'interface publique.

## Inconvénients

### Instanciation

Il faut créer un objet.

```php
$v = new CoureurValidateur();
```

au lieu d'un simple appel statique.

### Légèrement plus complexe

Une classe objet nécessite :

- un constructeur éventuel ;
- des propriétés ;
- la gestion de l'état.

Pour une simple méthode utilitaire, cela serait inutile.

# Comparaison

| Critère | Classe statique | Classe objet |
|----------|-----------------|--------------|
| Instanciation | Non | Oui |
| Possède un état | Non | Oui |
| Mémorise des informations | Non | Oui |
| Méthodes fluentes | Peu adaptées | Très adaptées |
| Plusieurs instances indépendantes | Non | Oui |
| Simplicité | Excellente | Bonne |
| Adaptée aux utilitaires | Oui | Non |
| Adaptée aux objets métier | Non | Oui |

---

# Application au mini-framework

## Classes statiques

Les classes suivantes sont naturellement statiques :

```text
Requete
Jeton
ReponseJson
Ajax
Coureur
Club
Categorie
```

Leur rôle est uniquement :

- lire une requête ;
- envoyer une réponse ;
- accéder aux données ;
- vérifier un jeton.

Elles ne mémorisent aucune information.

---

## Classes objet

Les classes suivantes sont naturellement des objets :

```text
Page
CoureurValidateur
```

Pourquoi ?

### Page

Une page possède un état.

Au fur et à mesure de sa construction :

```php
$page
    ->setTitre(...)
    ->addScript(...)
    ->setDonnee(...)
    ->avecJeton();
```

l'objet mémorise :

- son titre ;
- ses scripts ;
- ses feuilles de style ;
- ses données ;
- la présence d'un jeton.

Lors de l'appel :

```php
$page->afficher();
```

tout cet état est utilisé pour générer le document HTML.

Une classe statique serait ici totalement inadaptée.

---

### CoureurValidateur

Le validateur mémorise progressivement les erreurs rencontrées.

Il représente une validation en cours.

Chaque objet possède son propre tableau d'erreurs.

L'utilisation d'une classe objet est donc le choix le plus naturel.

---

# Règle pratique

Une question permet généralement de choisir entre une classe statique et une classe objet :

> **La classe doit-elle mémoriser des informations entre deux appels de méthodes ?**

- **Non** → une classe statique est souvent le meilleur choix.
- **Oui** → il est préférable d'utiliser une classe objet.

C'est cette règle qui a guidé la conception des différentes classes du mini-framework.