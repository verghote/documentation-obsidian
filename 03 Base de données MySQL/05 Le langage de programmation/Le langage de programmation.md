# Le langage de programmation MySQL

## Introduction

En plus du langage SQL classique, MySQL propose un **langage procédural** permettant de créer des :

- procédures stockées (*Stored Procedures*)
- fonctions (*Stored Functions*)
- déclencheurs (*Triggers*)
- événements (*Events*)

Ce langage ajoute les principales notions d'un langage de programmation :

- Variables
- Structures conditionnelles
- Structures itératives (boucles)
- Procédures
- Fonctions

---

# 1. Les variables

MySQL propose trois catégories de variables.

## 1.1 Variables locales

Les variables locales doivent être **déclarées**.

Leur portée est limitée au bloc `BEGIN ... END` dans lequel elles sont déclarées.

### Déclaration

```sql
DECLARE nomClient CHAR(30);
```

### Déclaration avec une valeur par défaut

```sql
DECLARE nomClient CHAR(30) DEFAULT 'Legrand';
```

### Affectation

```sql
SET nomClient = 'Durand';
```

### Affectation à partir d'une requête

```sql
DECLARE nomClient CHAR(30);

SELECT nom
INTO nomClient
FROM client
WHERE id = 2;
```

---

## 1.2 Variables utilisateur

Les variables utilisateur :

- ne nécessitent aucune déclaration ;
- sont accessibles durant toute la session ;
- commencent par le caractère `@`.

### Exemples

```sql
SET @nom = 'Legrand';
```

```sql
SET @dateMax = CURDATE() - INTERVAL 17 YEAR;
```

### Récupérer plusieurs valeurs

```sql
SELECT nom, prenom
INTO @nom, @prenom
FROM client
WHERE id = 2;
```

---

## 1.3 Variables système

Les variables système sont préfixées par `@@`.

Exemple :

```sql
SELECT @@version;
```

Afficher toutes les variables :

```sql
SHOW VARIABLES;
```

---

# 2. Les structures de programmation

Le langage procédural MySQL possède des structures proches des langages classiques.

- IF
- CASE
- WHILE
- LOOP
- REPEAT

Toutes les structures peuvent être utilisées dans :

- une procédure stockée
- une fonction
- un déclencheur (Trigger)

Les structures **IF** et **CASE** peuvent également être utilisées dans une requête SQL.

---

# 3. Structure conditionnelle IF

## Syntaxe

```sql
IF condition THEN
    instruction;
ELSEIF condition THEN
    instruction;
ELSE
    instruction;
END IF;
```

## Exemple

```sql
IF salaire > 50000 THEN
    SET categorie = 'Élevé';
ELSE
    SET categorie = 'Standard';
END IF;
```

---

# 4. Structure CASE

Deux syntaxes sont possibles.

## CASE selon une valeur

```sql
CASE variable

    WHEN valeur1 THEN
        instruction;

    WHEN valeur2 THEN
        instruction;

    ELSE
        instruction;

END CASE;
```

### Exemple

```sql
CASE departement

    WHEN 'ventes' THEN
        SET poste = 'Commercial';

    WHEN 'it' THEN
        SET poste = 'Technique';

    ELSE
        SET poste = 'Autre';

END CASE;
```

---

## CASE selon une condition

```sql
CASE

    WHEN salaire > 50000 THEN
        SET categorie = 'Élevé';

    ELSE
        SET categorie = 'Standard';

END CASE;
```

---

## Attention

Si la clause `ELSE` est absente et qu'aucun `WHEN` n'est vrai, MySQL génère l'erreur :

```
Error Code: 1339
Case not found for CASE statement
```

---

# 5. Les boucles

MySQL possède trois structures itératives.

- WHILE
- LOOP
- REPEAT

Il est possible d'ajouter une **étiquette** (*label*) devant la boucle.

Cette étiquette est obligatoire pour utiliser :

- `LEAVE`
- `ITERATE`

---

# 6. Boucle WHILE

## Syntaxe

```sql
WHILE condition DO

    instruction;

END WHILE;
```

## Exemple

```sql
CREATE PROCEDURE exWhile()

BEGIN

    DECLARE i INT DEFAULT 1;

    WHILE i <= 10 DO

        SELECT i;

        SET i = i + 1;

    END WHILE;

END;
```

Résultat :

```
1
2
3
...
10
```

---

# 7. Boucle REPEAT

La boucle `REPEAT` exécute toujours le bloc au moins une fois.

## Syntaxe

```sql
REPEAT

    instruction;

UNTIL condition

END REPEAT;
```

## Exemple

```sql
CREATE PROCEDURE exRepeat()

BEGIN

    DECLARE i INT DEFAULT 1;

    REPEAT

        SELECT i;

        SET i = i + 1;

    UNTIL i > 10

    END REPEAT;

END;
```

---

# 8. Boucle LOOP

La boucle `LOOP` est une boucle infinie.

Pour en sortir, on utilise `LEAVE`.

## Syntaxe

```sql
Etiquette: LOOP

    instruction;

END LOOP Etiquette;
```

## Exemple

```sql
CREATE PROCEDURE exLoop()

BEGIN

    DECLARE i INT DEFAULT 0;

    Et1: LOOP

        SET i = i + 1;

        SELECT i;

        IF i >= 10 THEN
            LEAVE Et1;
        END IF;

    END LOOP;

END;
```

---

# 9. LEAVE et ITERATE

Ces instructions sont l'équivalent de celles que l'on retrouve dans de nombreux langages.

| MySQL | Équivalent |
|--------|------------|
| LEAVE | break |
| ITERATE | continue |

Ces instructions nécessitent une **étiquette**.

---

## LEAVE

Quitte immédiatement la boucle.

```sql
LEAVE Et1;
```

---

## ITERATE

Passe directement à l'itération suivante.

```sql
ITERATE Et1;
```

---

# 10. Exemple avec deux boucles imbriquées

```sql
CREATE PROCEDURE exLoop2()

BEGIN

    DECLARE i INT DEFAULT 1;
    DECLARE j INT;

    Et1: LOOP

        SET j = 1;

        Et2: LOOP

            IF j > 5 THEN
                LEAVE Et2;
            END IF;

            IF i * j > 20 THEN
                LEAVE Et2;
            END IF;

            SELECT i, j;

            SET j = j + 1;

        END LOOP;

        SET i = i + 1;

        IF i > 5 THEN
            LEAVE Et1;
        END IF;

    END LOOP;

END;
```

## Fonctionnement

La boucle principale :

- initialise `j` à 1 ;
- incrémente `i` ;
- s'arrête lorsque `i > 5`.

La boucle interne :

- s'arrête lorsque `j > 5` ;
- ou lorsque `i × j > 20`.

L'exécution se termine donc avec :

```
i = 5
j = 4
```

---

# 11. Utilisation de IF dans une requête SQL

La fonction `IF()` peut être utilisée directement dans un `SELECT`.

## Syntaxe

```sql
IF(condition, valeur_si_vrai, valeur_si_faux)
```

## Exemple

```sql
SELECT
    nom,
    salaire,

    IF(
        salaire > 50000,
        'Élevé',
        'Standard'
    ) AS categorie_salaire,

    IF(
        departement = 'ventes',
        'Commercial',
        IF(
            departement = 'it',
            'Technique',
            'Autre'
        )
    ) AS type_poste

FROM employe;
```

---

# 12. Utilisation de CASE dans une requête SQL

```sql
SELECT

    nom,

    salaire,

    CASE

        WHEN salaire > 50000 THEN 'Élevé'
        ELSE 'Standard'

    END AS categorie_salaire,

    CASE departement

        WHEN 'ventes' THEN 'Commercial'
        WHEN 'it' THEN 'Technique'
        ELSE 'Autre'

    END AS type_poste

FROM employe;
```

---

# 13. Tableau récapitulatif

| Élément | Description |
|----------|-------------|
| DECLARE | Déclare une variable locale |
| SET | Affecte une valeur |
| SELECT ... INTO | Stocke le résultat d'une requête |
| IF | Condition |
| CASE | Sélection multiple |
| WHILE | Boucle avec test au début |
| REPEAT | Boucle avec test à la fin |
| LOOP | Boucle infinie |
| LEAVE | Quitte une boucle |
| ITERATE | Passe à l'itération suivante |
| @variable | Variable utilisateur |
| @@variable | Variable système |

---

# Conclusion

Le langage procédural MySQL permet d'étendre les possibilités du SQL grâce aux mécanismes classiques des langages de programmation :

- gestion des variables ;
- structures conditionnelles (`IF`, `CASE`) ;
- boucles (`WHILE`, `LOOP`, `REPEAT`) ;
- contrôle d'exécution (`LEAVE`, `ITERATE`) ;
- procédures, fonctions et déclencheurs.

Ces fonctionnalités permettent d'implémenter directement dans la base de données des traitements complexes tout en limitant le code applicatif.