
La bonne séparation des responsabilités est la suivante.

|Couche|Ce qu'elle contrôle|Ce qu'elle ne doit pas contrôler|
|---|---|---|
|**Contrôleur**|Présence des paramètres, type attendu (string, int, array…), format HTTP, validation de formulaire|Règles métier, cohérence de la base|
|**Classe métier**|Orchestration d'un traitement métier, transaction, enchaînement des opérations, règles métier qui ne peuvent pas être garanties par la base|Validation HTTP, contraintes déjà garanties par la base|
|**Base de données (contraintes, clés étrangères, CHECK, triggers)**|Intégrité des données, règles qui doivent être vraies quelle que soit l'application qui modifie la base|Validation des paramètres HTTP, présentation des erreurs|

## 1. Le contrôleur

Le contrôleur vérifie uniquement que la requête reçue est correcte.

Exemples :

- paramètres obligatoires présents ;
- types attendus (`string`, `array`, `int`...) ;
- format d'une date ;
- format d'un email ;
- tableau non vide si l'écran l'impose.

Exemple :

```
$nom = Requete::postString('nom');
$lesCompetences = Requete::postArray('lesCompetences');

if ($nom === '') {
    ReponseJson::envoyerLesErreurs(...);
}

if (count($lesCompetences) === 0) {
    ReponseJson::envoyerLesErreurs(...);
}
```



Si le contrôleur reçoit un tableau, il peut le parcourir pour vérifier le format des données

```
$idCompetence = filter_var($idCompetence, FILTER_VALIDATE_INT);
```

Sauf exeption, le contrôleur **ne vérifie pas** si une données existe.  Ce n'est pas son métier.

## 2. La classe métier

Les méthodes qui réalisent des opérations de mise à jour sur les données de la base peuvent effectuer quelques contrôles de cohérence qui ne relèvent pas de la base.

En revanche elle ne doit pas refaire les contrôles déjà garantis dans la base

## 3. Le trigger

Le trigger protège la base de données.

Son rôle est de garantir que **quelle que soit l'application** qui modifie la base :

- PHP ;
- Postman ;
- phpMyAdmin ;
- un script SQL ;
- une autre application Java, Python...

les données restent valides.

Il contrôle donc :

- les règles métier liées aux données ;
- les normalisations ;
- les contraintes impossibles à exprimer autrement.

Exemples :

```
nom entre 10 et 150 caractères
```

```
suppression des espaces multiples
```

```
une compétence d'un projet doit appartenir au bloc 1
```

Ce sont d'excellents exemples de règles de trigger.

## 4. Les contraintes SQL

Avant même le trigger, il faut utiliser tout ce que la base sait faire nativement.

- PRIMARY KEY
- FOREIGN KEY
- UNIQUE
- NOT NULL
- CHECK

Un trigger ne doit pas remplacer une contrainte SQL.


## 6. Exception 

Il est possible d'ajouter un contrôle applicatif s'il améliore la compréhension de l'erreur ou le parcours utilisateur.
Par exemple : 
+ ajouter une vérification de l'existence d'un valeur peut éviter d'envoyer une requête de suppression ou de modification
+ ajouter une vérification de la non existence d'un valeur peut éviter d'envoyer une requête d'ajout 
# En résumé

- **Contrôleur** : valide la requête HTTP (présence, type, format des données).
- **Classe métier** : orchestre le traitement, gère les transactions et applique les règles métier qui dépassent la simple intégrité des données.
- **Base de données (contraintes + triggers)** : garantit définitivement l'intégrité des données et les règles métier attachées aux tables, sans duplication dans le code PHP.

Cette répartition est celle que l'on retrouve généralement dans les architectures professionnelles : chaque règle est implémentée **une seule fois**, dans la couche la plus légitime pour l'appliquer.