
## 🧱 Syntaxe de base d’une requête SELECT

```
SELECT [DISTINCT] expression [AS nom], ...
FROM nomTable [alias], ...
WHERE condition
GROUP BY nomColonne, ...
HAVING condition
ORDER BY expression [DESC], ...
LIMIT n [, m]
```

---

## 📊 Fonctions de calcul statistiques

- `COUNT(expression)` : nombre de valeurs (ou `DISTINCT`)
    
    ```
    SELECT COUNT(*) FROM users;
    ```
    
- `SUM(expression)` : somme des valeurs
    
    ```
    SELECT SUM(prix) FROM ventes;
    ```
    
- `AVG(expression)` : moyenne
    
    ```
    SELECT AVG(age) FROM users;
    ```
    
- `MIN(expression)` : valeur minimale
    
    ```
    SELECT MIN(prix) FROM produits;
    ```
    
- `MAX(expression)` : valeur maximale
    
    ```
    SELECT MAX(prix) FROM produits;
    ```
    

---

## 🔎 Opérateurs de la clause WHERE

- `BETWEEN v1 AND v2` : intervalle inclus
    
    ```
    WHERE age BETWEEN 18 AND 30
    ```
    
- `IN (liste)` : appartient à une liste
    
    ```
    WHERE id IN (1, 2, 3)
    ```
    
- `LIKE` : recherche avec motifs
    
    ```
    WHERE nom LIKE 'A%'
    ```
    
    - `%` : n’importe quelle suite de caractères
    - `_` : un seul caractère
- `BINARY` : comparaison sensible à la casse
    
    ```
    WHERE BINARY nom = 'Alice'
    ```
    
- `REGEXP` / `REGEXP_LIKE` : expression régulière
    
    ```
    SELECT * FROM utilisateursWHERE nom REGEXP '^[AB]';SELECT * FROM utilisateursWHERE REGEXP_LIKE(nom, '^[AB]', 'i');
    ```
    

---

## 🔀 Conditionnel SQL

### CASE (standard SQL)

```
SELECT  CASE    WHEN place = 1 THEN 'or'    WHEN place = 2 THEN 'argent'    WHEN place = 3 THEN 'bronze'    ELSE 'aucune médaille'  END AS medailleFROM resultat;
```

### IF (MySQL uniquement)

```
SELECT IF(score > 10, 'OK', 'NON');
```

---

## 🧮 Autres fonctions utiles

- `COALESCE()` : remplace NULL par une valeur par défaut
    
    ```
    SELECT COALESCE(SUM(point), 0) FROM resultat;
    ```
    
- `IFNULL()` (MySQL) :
    
    ```
    SELECT IFNULL(score, 0);
    ```
    

👉 Exemple :

- `SUM()` peut retourner `NULL`
- `COALESCE(..., 0)` transforme `NULL` en `0`

---

## 📌 Remarque

- `CASE` est portable (SQL standard)
- `IF`, `IFNULL` sont spécifiques à MySQL
- `REGEXP_LIKE` est plus moderne et portable que `REGEXP` selon les SGBD