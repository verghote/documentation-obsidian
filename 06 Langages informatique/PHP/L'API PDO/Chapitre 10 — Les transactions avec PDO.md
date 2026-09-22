## Objectifs

À la fin de ce chapitre, vous serez capable de :

- comprendre le rôle d'une transaction SQL ;
- regrouper plusieurs opérations dans une seule unité logique ;
- utiliser `beginTransaction()`, `commit()` et `rollBack()` ;
- garantir la cohérence des données lors d'opérations complexes ;
- appliquer les transactions dans une classe métier.

---

# 10.1 Introduction

Une transaction permet de regrouper plusieurs requêtes SQL afin de garantir qu'elles seront :

- **toutes validées** ;
- ou **toutes annulées**.

Le principe est :

> Une opération complexe doit être entièrement réalisée ou ne pas être réalisée du tout.

---

# 10.2 Pourquoi utiliser une transaction ?

Certaines opérations nécessitent plusieurs modifications liées.

Exemple :

Lors de la création d'un projet :

1. création du projet ;
2. récupération de son identifiant ;
3. ajout des compétences associées.

On obtient :

```
Projet
   |
   |-- Compétence 1
   |
   |-- Compétence 2
   |
   |-- Compétence 3
```

Si l'ajout de la troisième compétence échoue, le projet ne doit pas rester créé seul.

Sans transaction :

```
Projet créé       ✅
Compétence 1      ✅
Compétence 2      ✅
Compétence 3      ❌
```

La base devient incohérente.

Avec une transaction :

```
Projet créé       ✅
Compétence 1      ✅
Compétence 2      ✅
Compétence 3      ❌

=> Annulation complète
```

Résultat :

```
Projet supprimé
Compétence 1 supprimée
Compétence 2 supprimée
```

---

# 10.3 Les trois étapes d'une transaction

Une transaction PDO utilise trois méthodes principales.

## 1. Démarrer la transaction

```
$db->beginTransaction();
```

À partir de ce moment, les modifications ne sont pas encore définitivement enregistrées.

---

## 2. Valider la transaction

```
$db->commit();
```

Toutes les modifications deviennent définitives.

---

## 3. Annuler la transaction

```
$db->rollBack();
```

Toutes les modifications effectuées depuis `beginTransaction()` sont annulées.

---

# 10.4 Structure générale

Le modèle classique est :

```
$db = Database::getInstance();


$db->beginTransaction();


try {

    // requêtes SQL

    $db->commit();

}
catch(Exception $e) {

    $db->rollBack();

}
```

---

# 10.5 Exemple simple

Supposons une application bancaire.

On doit :

1. retirer de l'argent d'un compte ;
2. ajouter cet argent sur un autre compte.

Sans transaction :

```
Compte A : -100 €
Erreur avant le crédit du compte B
```

L'argent disparaît.

---

Avec transaction :

```
$db->beginTransaction();


try {

    $sql = "
    UPDATE compte
    SET solde = solde - 100
    WHERE id = 1
    ";

    $db->exec($sql);



    $sql = "
    UPDATE compte
    SET solde = solde + 100
    WHERE id = 2
    ";

    $db->exec($sql);



    $db->commit();

}
catch(Exception $e) {

    $db->rollBack();

}
```

Soit les deux opérations réussissent, soit aucune n'est appliquée.

---

# 10.6 Transaction avec des requêtes préparées

Les transactions sont très souvent utilisées avec `prepare()`.

Exemple :

Ajout d'un projet et de ses compétences.

```
public static function ajouterProjet(
    string $nom,
    array $competences
): int
{

    $db = Database::getInstance();


    $db->beginTransaction();


    try {

        // Création du projet

        $sql = <<<SQL
        INSERT INTO projet(nom)
        VALUES(:nom)
        SQL;


        $stmt = $db->prepare($sql);


        $stmt->execute([
            "nom" => $nom
        ]);



        $idProjet = $db->lastInsertId();



        // Ajout des compétences

        $sql = <<<SQL
        INSERT INTO competenceProjet(
            idProjet,
            idCompetence
        )
        VALUES(
            :idProjet,
            :idCompetence
        )
        SQL;


        $stmt = $db->prepare($sql);



        foreach($competences as $competence)
        {

            $stmt->execute([
                "idProjet" => $idProjet,
                "idCompetence" => $competence
            ]);

        }



        $db->commit();


        return $idProjet;


    }
    catch(Exception $e)
    {

        $db->rollBack();

        throw $e;

    }

}
```

---

# 10.7 Transactions et clés étrangères

Les transactions sont particulièrement utiles lorsque plusieurs tables sont liées par des clés étrangères.

Exemple :

Tables :

```
CLIENT
 |
 |
COMMANDE
 |
 |
LIGNE_COMMANDE
```

Création d'une commande :

1. insertion du client ;
2. création de la commande ;
3. ajout des lignes.

Une erreur dans une ligne de commande doit annuler toute la création.

---

# 10.8 Transactions et auto-incrémentation

Une question fréquente concerne les identifiants générés :

```
id AUTO_INCREMENT
```

Exemple :

```
$id = $db->lastInsertId();
```

Si la transaction est annulée :

```
$db->rollBack();
```

l'identifiant peut avoir été consommé.

Exemple :

```
Projet créé
id = 25

Erreur
rollback

Projet supprimé
```

La prochaine insertion peut obtenir :

```
id = 26
```

Il peut donc exister des trous dans les numéros d'identifiant.

C'est normal.

---

# 10.9 Vérifier l'état d'une transaction

PDO permet de savoir si une transaction est active :

```
$db->inTransaction();
```

Exemple :

```
if ($db->inTransaction()) {

    $db->commit();

}
```

---

# 10.10 Transactions et niveau d'isolation

Lorsqu'une base est utilisée par plusieurs utilisateurs simultanément, plusieurs transactions peuvent s'exécuter en même temps.

Le SGBDR utilise un niveau d'isolation pour éviter certains problèmes.

Les niveaux courants sont :

|Niveau|Description|
|---|---|
|READ UNCOMMITTED|lecture de données non validées|
|READ COMMITTED|lecture uniquement des données validées|
|REPEATABLE READ|lecture stable pendant la transaction|
|SERIALIZABLE|isolation maximale|

Le niveau utilisé dépend du SGBDR.

Dans la majorité des applications Web classiques, le niveau par défaut est suffisant.

---

# 10.11 Quand utiliser une transaction ?

Utiliser une transaction lorsque :

✅ plusieurs tables doivent être modifiées ensemble ;

✅ une opération nécessite plusieurs requêtes dépendantes ;

✅ une erreur intermédiaire rendrait les données incohérentes ;

✅ une action métier doit être atomique.

Exemples :

- création d'une commande avec ses lignes ;
- inscription d'un utilisateur avec son profil ;
- réservation d'une ressource ;
- transfert bancaire ;
- création d'un projet avec ses associations.

---

# 10.12 Quand ne pas utiliser une transaction ?

Une transaction n'est pas nécessaire pour :

- une simple consultation (`SELECT`) ;
- un ajout isolé sans dépendance ;
- une modification unique indépendante.

Exemple :

```
UPDATE utilisateur
SET derniereConnexion = NOW()
WHERE id = 5;
```

Une transaction supplémentaire n'apporte généralement rien.

---

# 10.13 Bonnes pratiques

- Démarrer la transaction juste avant les modifications.
- Garder les transactions courtes.
- Toujours prévoir un `rollBack()` en cas d'échec.
- Toujours appeler `commit()` lorsque tout est réussi.
- Ne pas effectuer d'opération longue pendant une transaction.
- Utiliser les transactions pour protéger la cohérence métier, pas pour toutes les requêtes.

---

# À retenir

- Une transaction garantit l'intégrité d'une opération composée de plusieurs requêtes.
- `beginTransaction()` démarre la transaction.
- `commit()` valide définitivement les modifications.
- `rollBack()` annule toutes les modifications réalisées depuis le début.
- Les transactions sont indispensables pour les opérations métier complexes impliquant plusieurs tables.
- Une transaction doit rester courte et regrouper uniquement les opérations qui doivent réussir ensemble.