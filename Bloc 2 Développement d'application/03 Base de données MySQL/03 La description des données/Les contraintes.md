# Introduction

Les contraintes permettent de garantir la **cohérence**, **l'intégrité** et la **qualité** des données enregistrées dans une base de données.

Elles sont définies au niveau :

- d'une colonne ;
- d'une table ;
- de plusieurs tables.

Les contraintes sont automatiquement vérifiées par le SGBD lors des opérations :

- `INSERT`
- `UPDATE`
- `DELETE`

Si une contrainte est violée, MySQL refuse l'opération.

# 1. Définition

Une contrainte est une règle que les données doivent obligatoirement respecter.

Elle permet notamment de garantir :

- la cohérence des valeurs ;
- l'unicité des identifiants ;
- les relations entre les tables ;
- le respect des règles métier.

Exemples :

- un âge ne peut pas être négatif ;
- un client possède un identifiant unique ;
- une commande doit appartenir à un client existant ;
- une note est comprise entre 0 et 20.

---

# 2. Les différents types de contraintes

On distingue principalement :

| Type | Portée |
|------|---------|
| Domaine (type) | Colonne |
| NOT NULL | Colonne |
| DEFAULT | Colonne |
| CHECK | Colonne ou table |
| PRIMARY KEY | Table |
| UNIQUE | Table |
| FOREIGN KEY | Plusieurs tables |
| Contraintes métier | Plusieurs tables |

---

# 3. Les contraintes de domaine

Chaque colonne possède un type.

Le type limite automatiquement les valeurs possibles.

Exemple :

```sql
nom VARCHAR(30)
```

Le champ ne peut contenir qu'une chaîne de caractères de 30 caractères maximum.

Autres exemples :

```sql
age INT
```

```sql
prix DECIMAL(8,2)
```

```sql
dateCommande DATE
```

Cette contrainte est obligatoire.

---

# 4. La contrainte NOT NULL

Par défaut, une colonne accepte la valeur `NULL`.

La contrainte `NOT NULL` interdit cette valeur.

Exemple :

```sql
nom VARCHAR(30) NOT NULL
```

Insertion interdite :

```sql
INSERT INTO client(nom)

VALUES(NULL);
```

---

# 5. La valeur par défaut (DEFAULT)

Une colonne peut recevoir automatiquement une valeur.

Syntaxe :

```sql
quantite INT NOT NULL DEFAULT 1
```

Insertion :

```sql
INSERT INTO produit(nom)

VALUES('Stylo');
```

Résultat :

```
quantite = 1
```

---

# 6. La contrainte CHECK

Elle limite les valeurs autorisées.

Syntaxe :

```sql
CHECK(condition)
```

---

## Exemple : note comprise entre 0 et 20

```sql
note DECIMAL(4,2)

CHECK(note BETWEEN 0 AND 20)
```

---

## Exemple : civilité

```sql
civilite CHAR(12)

DEFAULT 'Monsieur'

CHECK(

civilite IN(

'Monsieur',

'Madame',

'Mademoiselle'

)

)
```

---

# 7. CHECK utilisant plusieurs colonnes

Une contrainte peut comparer plusieurs colonnes.

Exemple :

Le prix de vente doit être supérieur ou égal au prix d'achat.

```sql
CREATE TABLE produit(

    id INT,

    prixAchat DECIMAL(8,2),

    prixVente DECIMAL(8,2),

    CONSTRAINT ck_prix

    CHECK(

        prixVente >= prixAchat

    )

);
```

Cette contrainte est définie au niveau de la table.

---

# 8. La clé primaire

La clé primaire identifie chaque ligne de façon unique.

Caractéristiques :

- unique ;
- obligatoire ;
- ne peut être NULL ;
- une seule clé primaire par table.

---

## Exemple

```sql
CREATE TABLE client(

    id INT,

    nom VARCHAR(30),

    CONSTRAINT pk_client

    PRIMARY KEY(id)

);
```

---

# 9. Clé primaire composée

Une clé primaire peut comporter plusieurs colonnes.

Exemple :

```sql
CREATE TABLE detailVente(

    idVente INT,

    idProduit INT,

    quantite INT,

    CONSTRAINT pk_detail

    PRIMARY KEY(

        idVente,

        idProduit

    )

);
```

Chaque produit ne peut apparaître qu'une seule fois dans une vente.

---

# 10. La contrainte UNIQUE

Elle garantit l'unicité d'une colonne.

Contrairement à la clé primaire :

- plusieurs contraintes UNIQUE sont possibles ;
- elles représentent des clés candidates.

Exemple :

```sql
CREATE TABLE produit(

    id INT,

    nom VARCHAR(50),

    PRIMARY KEY(id),

    CONSTRAINT u_nom

    UNIQUE(nom)

);
```

Deux produits ne pourront pas porter le même nom.

---

# 11. Clé primaire ou UNIQUE ?

| PRIMARY KEY | UNIQUE |
|--------------|---------|
|Une seule|Plusieurs possibles|
|Jamais NULL|Peut accepter NULL selon le SGBD|
|Identifie la ligne|Simple unicité|

---

# 12. Les clés étrangères

Une clé étrangère relie deux tables.

Elle garantit qu'une valeur existe dans la table référencée.

Exemple :

```
Client

id

↓

Commande

idClient
```

---

## Exemple

```sql
CREATE TABLE commande(

    id INT,

    idClient INT,

    CONSTRAINT fk_commande_client

    FOREIGN KEY(idClient)

    REFERENCES client(id)

);
```

---

# 13. Intégrité référentielle

La clé étrangère impose que :

- la valeur existe ;
- ou soit NULL si cela est autorisé.

Exemple interdit :

```
Client

1

2

3

Commande

idClient = 8
```

Le client 8 n'existe pas.

MySQL refuse l'insertion.

---

# 14. Les actions automatiques

Une clé étrangère peut définir un comportement lors d'une suppression ou d'une modification.

Syntaxe :

```sql
ON DELETE ...

ON UPDATE ...
```

---

# 15. ON DELETE CASCADE

La suppression est propagée.

```
Client

↓

Commandes

↓

Lignes
```

Suppression du client :

↓

Suppression automatique des commandes.

Exemple :

```sql
FOREIGN KEY(idClient)

REFERENCES client(id)

ON DELETE CASCADE
```

---

# 16. ON UPDATE CASCADE

Une modification de clé primaire est répercutée.

```sql
FOREIGN KEY(idProduit)

REFERENCES produit(id)

ON UPDATE CASCADE
```

---

# 17. ON DELETE SET NULL

Lorsqu'un parent disparaît :

```
idClient

↓

NULL
```

Exemple :

```sql
ON DELETE SET NULL
```

La colonne doit accepter NULL.

---

# 18. ON DELETE SET DEFAULT

Le SGBD remplace la clé étrangère par sa valeur par défaut.

Tous les SGBD ne l'implémentent pas.

---

# 19. Exemple complet

```sql
CREATE TABLE detailVente(

    idVente INT,
    idProduit INT,
    
    PRIMARY KEY(idVente, idProduit),

    CONSTRAINT fk_vente FOREIGN KEY(idVente) REFERENCES vente(id) ON DELETE CASCADE,
    CONSTRAINT fk_produit FOREIGN KEY(idProduit)REFERENCES produit(id) ON UPDATE CASCADE
);
```

---
# 20. Les contraintes métier

Certaines règles ne peuvent pas être exprimées avec les contraintes classiques.

Elles nécessitent :

- des déclencheurs (Triggers) ;
- des procédures stockées ;
- des transactions.

---

# 21. Contrainte d'exclusion

Exemple :

Une vente provient :

- soit d'un client ;
- soit d'un service.

Jamais des deux.

```
idClient

idService
```

Une seule colonne peut être renseignée.

Cette règle est généralement implémentée à l'aide d'un Trigger.

---

# 22. Contrainte d'inclusion

Exemple :

Un salarié suit uniquement une formation qu'il a demandée.

```
Demander

↓

Suivre
```

Cette contrainte peut parfois être traduite avec une clé étrangère composée.

---

# 23. Contrainte de simultanéité

Exemple :

Une vente contient obligatoirement au moins un produit.

```
Vente

↓

DetailVente
```

La suppression du dernier produit doit être interdite.

Cette règle nécessite :

- une transaction ;
- un Trigger.

---

# 24. Les assertions

Une assertion contrôle un ensemble de lignes.

Exemple :

Un prospect ne peut pas être également client.

```sql
CREATE ASSERTION

a_unique

CHECK(

NOT EXISTS(

SELECT *

FROM prospect p

JOIN client c

ON p.id=c.id

)

);
```

Les assertions ne sont pas implémentées par MySQL.

On utilise donc des Triggers.

---

# 25. Les références circulaires

Deux tables peuvent se référencer mutuellement.

Exemple :

```
Document

↓

Version

↑
```

La création des deux clés étrangères dans le `CREATE TABLE` est impossible.

---

# 26. Solution

Créer les tables sans l'une des deux contraintes.

Puis ajouter la seconde avec :

```sql
ALTER TABLE

ADD CONSTRAINT
```

---

## Exemple

```sql
ALTER TABLE document

ADD CONSTRAINT

fk_document_version

FOREIGN KEY(

id,

numVersionPubliee

)

REFERENCES version(

idDocument,

numVersion

);
```

---

# Tableau récapitulatif

| Contrainte | Rôle |
|-------------|------|
|Type|Limiter les valeurs possibles|
|NOT NULL|Valeur obligatoire|
|DEFAULT|Valeur automatique|
|CHECK|Contrôle des valeurs|
|PRIMARY KEY|Identifiant unique|
|UNIQUE|Clé candidate|
|FOREIGN KEY|Intégrité référentielle|
|CASCADE|Propagation automatique|
|SET NULL|Remplacement par NULL|
|SET DEFAULT|Remplacement par la valeur par défaut|

---

# Bonnes pratiques

✔ Toujours définir une clé primaire.

✔ Nommer toutes les contraintes (`pk_client`, `fk_commande_client`, `ck_prix`...).

✔ Utiliser `CHECK` pour les contrôles simples.

✔ Utiliser les clés étrangères pour garantir l'intégrité référentielle.

✔ Éviter les suppressions en cascade lorsqu'elles peuvent entraîner des pertes importantes de données.

✔ Réserver les déclencheurs aux règles métier qui ne peuvent pas être exprimées avec des contraintes SQL.

✔ Vérifier les conséquences des actions `CASCADE`, `SET NULL` et `SET DEFAULT` avant leur utilisation.

---

# À retenir

Les contraintes constituent le premier niveau de protection des données. Elles permettent d'empêcher l'enregistrement de données incohérentes sans intervention de l'application.

Les contraintes les plus utilisées sont :

- **PRIMARY KEY** pour identifier les enregistrements ;
- **FOREIGN KEY** pour assurer les relations entre les tables ;
- **NOT NULL** pour imposer une valeur ;
- **CHECK** pour contrôler les données ;
- **UNIQUE** pour garantir l'unicité.

Lorsque les contraintes SQL ne suffisent plus à exprimer une règle métier complexe, il est nécessaire de recourir aux **déclencheurs**, aux **procédures stockées** ou aux **transactions**.