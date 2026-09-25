# Les déclencheurs (Triggers) MySQL

## 1. Définition et rôle d'un déclencheur

Un **trigger** (ou déclencheur) est un traitement SQL exécuté automatiquement lorsqu'une opération de modification est réalisée sur une table.

Contrairement à une procédure stockée qui doit être appelée explicitement avec l'instruction `CALL`, un déclencheur est exécuté automatiquement par le système de gestion de base de données.

Les opérations pouvant déclencher un trigger sont :

- `INSERT` : ajout d'un enregistrement
- `UPDATE` : modification d'un enregistrement
- `DELETE` : suppression d'un enregistrement

Un déclencheur permet notamment de :

- contrôler les données avant leur enregistrement ;
- appliquer des règles métier complexes ;
- maintenir automatiquement des données calculées ;
- historiser les modifications ;
- empêcher certaines opérations interdites.

Exemple :

Lorsqu'un client est supprimé, un trigger peut automatiquement déplacer ses informations dans une table d'archive avant la suppression.

Il est important de respecter la règle suivante :
**La base de données utilise les contraintes natives pour garantir l'intégrité structurelle et les triggers uniquement pour les règles qui nécessitent un comportement dynamique ou une comparaison avec l'ancienne ligne.**

Un déclencheur doit donc éviter de contrôler ce que les contraintes (clé primaire, unicité,  clé étrangère, check, domaine) ou tout simplement le type de la données font déjà
Rappel : avec MySQL : les expressions `CHECK` doivent utiliser des fonctions déterministes, et `CURRENT_DATE` est dépendant du moment où l'expression est évaluée.



---

# 2. Caractéristiques des triggers MySQL

Sous MySQL, un déclencheur possède les caractéristiques suivantes :

|Caractéristique|Description|
|---|---|
|Association|Un trigger est associé à une seule table|
|Opération|Un trigger concerne une seule opération (`INSERT`, `UPDATE` ou `DELETE`)|
|Moment d'exécution|Il peut être exécuté avant (`BEFORE`) ou après (`AFTER`) l'opération|
|Traitement|Il traite les lignes une par une avec `FOR EACH ROW`|
|Transaction|Il est exécuté dans la même transaction que l'opération déclencheuse|
|Transaction SQL|Il ne peut pas utiliser `COMMIT`, `ROLLBACK` ou `SAVEPOINT`|
|Accès à la table|Il ne peut pas modifier directement la table qui l'a déclenché|

---

# 3. Création et gestion d'un déclencheur

## 3.1 Syntaxe générale

La création d'un déclencheur utilise l'instruction `CREATE TRIGGER`.

```
CREATE TRIGGER nomTrigger
BEFORE | AFTER
INSERT | UPDATE | DELETE
ON nomTable
FOR EACH ROW
BEGIN
    instruction SQL;
END;
```

## Signification des éléments

|Élément|Rôle|
|---|---|
|`nomTrigger`|Nom du déclencheur|
|`BEFORE`|Exécution avant l'opération|
|`AFTER`|Exécution après l'opération|
|`INSERT`|Déclenché lors d'un ajout|
|`UPDATE`|Déclenché lors d'une modification|
|`DELETE`|Déclenché lors d'une suppression|
|`FOR EACH ROW`|Traitement de chaque ligne concernée|
|`BEGIN END`|Bloc d'instructions multiples|

---

## 3.2 Utilisation du DELIMITER

Comme un trigger contient souvent plusieurs instructions SQL terminées par `;`, il faut modifier temporairement le séparateur d'instructions.

Par convention, on utilise généralement `$$`.

```
DELIMITER $$

CREATE TRIGGER avantAjoutClient
BEFORE INSERT ON client
FOR EACH ROW
BEGIN
    SET NEW.dateCreation = NOW();
END $$

DELIMITER ;
```

Dans certains environnements comme **DataGrip** ou **PhpStorm**, l'utilisation de `DELIMITER` n'est pas nécessaire.

---

# 4. Suppression et consultation des déclencheurs

## 4.1 Suppression d'un trigger

Syntaxe :

```
DROP TRIGGER [IF EXISTS] nomTrigger;
```

Exemple :

```
DROP TRIGGER IF EXISTS avantAjoutClient;
```

---

## 4.2 Liste des triggers

Pour afficher les déclencheurs existants :

```
SHOW TRIGGERS;
```

Ou filtrer sur une table :

```
SHOW TRIGGERS LIKE 'client';
```

---

# 5. Accès aux valeurs avec NEW et OLD

Lorsqu'un trigger est exécuté, MySQL met à disposition deux structures particulières :

- `NEW`
- `OLD`

Elles permettent d'accéder aux valeurs de l'enregistrement concerné.

---

# 5.1 La structure NEW

`NEW` représente les nouvelles valeurs.

Elle est utilisable pour :

- `INSERT`
- `UPDATE`

Exemple :

```
CREATE TRIGGER avantAjoutProduit
BEFORE INSERT ON produit
FOR EACH ROW
BEGIN
    SET NEW.dateCreation = NOW();
END;
```

Avant l'ajout du produit, la date est automatiquement renseignée.

---

## Modification d'une valeur avec NEW

Dans un trigger `BEFORE INSERT` ou `BEFORE UPDATE`, il est possible de modifier une valeur :

```
SET NEW.nom = UPPER(NEW.nom);
```

Exemple :

```
CREATE TRIGGER majNomClient
BEFORE INSERT ON client
FOR EACH ROW
BEGIN
    SET NEW.nom = UPPER(NEW.nom);
END;
```

Le nom sera enregistré en majuscule.

---

# 5.2 La structure OLD

`OLD` représente les anciennes valeurs.

Elle est utilisable pour :

- `DELETE`
- `UPDATE`

Elle est accessible uniquement en lecture.

Exemple :

```
CREATE TRIGGER archiveSuppressionClient
BEFORE DELETE ON client
FOR EACH ROW
BEGIN
    INSERT INTO client_archive(id, nom)
    VALUES(OLD.id, OLD.nom);
END;
```

L'ancien enregistrement est sauvegardé avant suppression.

---

# 6. L'instruction SIGNAL

L'instruction `SIGNAL` permet de provoquer volontairement une erreur et d'annuler l'opération en cours.

Syntaxe :

```
SIGNAL SQLSTATE '45000'
SET MESSAGE_TEXT = 'Message d''erreur';
```

Exemple :

```
CREATE TRIGGER controleAge
BEFORE INSERT ON client
FOR EACH ROW
BEGIN
    IF NEW.age < 18 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Le client doit être majeur';
    END IF;
END;
```

Résultat :

- l'ajout est annulé ;
- une erreur est renvoyée à l'application.

---

# 7. Ordre d'exécution des traitements

## 7.1 Lors d'un INSERT

Lorsqu'un ajout est réalisé :

```
INSERT
   |
   V
Trigger BEFORE INSERT
   |
   V
Vérification des contraintes
(NULL, PRIMARY KEY, UNIQUE, FOREIGN KEY, CHECK)
   |
   V
Ajout dans la table
   |
   V
Trigger AFTER INSERT
```

---

## 7.2 Lors d'un DELETE

```
DELETE
   |
   V
Trigger BEFORE DELETE
   |
   V
Vérification des contraintes
   |
   V
Suppression
   |
   V
Trigger AFTER DELETE
```

---

## 7.3 Lors d'un UPDATE

```
UPDATE
   |
   V
Trigger BEFORE UPDATE
   |
   V
Vérification des contraintes
   |
   V
Modification
   |
   V
Trigger AFTER UPDATE
```

---

# 8. Choisir entre BEFORE et AFTER

## BEFORE

Un trigger `BEFORE` est utilisé pour :

- contrôler une donnée ;
- modifier une valeur avant insertion ;
- empêcher une opération.

Exemple :

```
CREATE TRIGGER controlePrix
BEFORE INSERT ON produit
FOR EACH ROW
BEGIN
    IF NEW.prix < 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT='Prix incorrect';
    END IF;
END;
```

---

## AFTER

Un trigger `AFTER` est utilisé pour :

- mettre à jour une autre table ;
- enregistrer une trace ;
- maintenir des compteurs.

Exemple :

```
CREATE TRIGGER apresAjoutCommande
AFTER INSERT ON commande
FOR EACH ROW
BEGIN
    UPDATE client
    SET nombreCommande = nombreCommande + 1
    WHERE id = NEW.idClient;
END;
```

---

# 9. Système de traçabilité

Les triggers sont souvent utilisés pour conserver un historique des opérations.

## 9.1 Historisation des modifications

Exemple :

Table :

```
client(
 id,
 nom,
 modifieLe,
 modifiePar
)
```

Trigger :

```
CREATE TRIGGER avantModificationClient
BEFORE UPDATE ON client
FOR EACH ROW
BEGIN
    SET NEW.modifieLe = NOW();
    SET NEW.modifiePar = CURRENT_USER();
END;
```

Chaque modification est automatiquement datée.

---

## 9.2 Archivage des suppressions

Création d'une table d'archive :

```
CREATE TABLE clientArchive(
 id INT,
 nom VARCHAR(30),
 supprimeLe DATETIME,
 supprimePar VARCHAR(100)
);
```

Trigger :

```
CREATE TRIGGER archiveClient
BEFORE DELETE ON client
FOR EACH ROW
BEGIN
    INSERT INTO clientArchive
    VALUES(
        OLD.id,
        OLD.nom,
        NOW(),
        CURRENT_USER()
    );
END;
```

# 10. Utilisation des triggers pour interdire certaines opérations

Un déclencheur `BEFORE` peut empêcher une opération en générant volontairement une erreur avec `SIGNAL`.

---

## 10.1 Interdire toute modification d'une table

Exemple : une table historique ne doit jamais être modifiée.

```
CREATE TRIGGER interdictionModificationHistorique
BEFORE UPDATE ON historique
FOR EACH ROW
BEGIN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'Modification interdite sur cette table';
END;
```

Toute tentative de modification provoquera une erreur.

---

## 10.2 Interdire la modification d'une colonne

Certaines colonnes ne doivent jamais être modifiées, comme un identifiant.

Exemple :

```
CREATE TRIGGER controleModificationId
BEFORE UPDATE ON client
FOR EACH ROW
BEGIN
    IF OLD.id <> NEW.id THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'L''identifiant ne peut pas être modifié';
    END IF;
END;
```

---

# 11. Contrôle de format et contrôle d'intégrité

Les triggers permettent d'appliquer des règles métier qui ne peuvent pas toujours être exprimées avec des contraintes classiques.

Exemple :

Une compétence doit respecter un format numérique et exister dans une table de référence.

```
CREATE TRIGGER controleCompetence
BEFORE INSERT ON employe_competence
FOR EACH ROW
BEGIN

    IF NEW.idCompetence NOT REGEXP '^[0-9]{1,2}$' THEN

        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Format de compétence invalide';

    END IF;


    IF NOT EXISTS(
        SELECT 1
        FROM competence
        WHERE id = NEW.idCompetence
    ) THEN

        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Cette compétence n''existe pas';

    END IF;

END;
```

---

# 12. Contrôles courants avec les expressions régulières

Les expressions régulières permettent de contrôler la forme d'une donnée.

## Exemple de contrôle d'une date

```
IF NEW.dateNaissance REGEXP '^0000-00-00$' THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT='Date invalide';
END IF;
```

---

## Contrôle d'un nom

Vérifier que le nom possède entre 3 et 30 caractères :

```
IF CHAR_LENGTH(NEW.nom) NOT BETWEEN 3 AND 30 THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT='Nom incorrect';
END IF;
```

---

## Contrôle d'un email

```
IF NEW.email NOT REGEXP
'^[0-9a-zA-Z._-]+@[0-9a-zA-Z.-]+\\.[a-zA-Z]{2,4}$'
THEN

SIGNAL SQLSTATE '45000'
SET MESSAGE_TEXT='Email incorrect';

END IF;
```

---

## Attention aux valeurs NULL

En MySQL :

```
NULL = NULL
```

ne retourne pas vrai.

Une comparaison avec `NULL` retourne toujours `NULL`.

Il faut utiliser :

```
IS NULL
```

ou :

```
IS NOT NULL
```

Exemple :

Incorrect :

```
IF NEW.nom = NULL THEN
```

Correct :

```
IF NEW.nom IS NULL THEN
```

---

# 13. Mise en œuvre d'une contrainte d'exclusion entre deux colonnes

Une contrainte d'exclusion impose qu'une seule colonne parmi plusieurs soit renseignée.

Exemple :

Une vente provient soit :

- d'un client ;
- d'un service.

Mais jamais des deux.

Table :

```
Vente(
 id,
 idClient,
 idService
)
```

Règle :

- `idClient` renseigné → `idService` vide
- `idService` renseigné → `idClient` vide

Trigger :

```
CREATE TRIGGER controleOrigineVente
BEFORE INSERT ON vente
FOR EACH ROW
BEGIN

    IF 
    (NEW.idClient IS NOT NULL AND NEW.idService IS NOT NULL)
    OR
    (NEW.idClient IS NULL AND NEW.idService IS NULL)

    THEN

        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT='Une vente doit provenir d''une seule origine';

    END IF;

END;
```

Le même principe doit être appliqué lors d'une modification :

```
BEFORE UPDATE
```

---

# 14. Contrainte d'exclusion entre deux tables

Certaines règles métiers nécessitent de vérifier deux tables.

Exemple :

Un salarié peut être :

- soit un contremaître ;
- soit un installateur.

Il ne peut pas appartenir aux deux tables.

Tables :

```
Contremaitre(id)
Installateur(id)
```

Trigger sur Contremaitre :

```
CREATE TRIGGER controleContremaitre
BEFORE INSERT ON contremaitre
FOR EACH ROW
BEGIN

    IF EXISTS(
        SELECT 1
        FROM installateur
        WHERE id = NEW.id
    )
    THEN

        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT='Violation de la contrainte d''exclusion';

    END IF;

END;
```

Un trigger équivalent doit être créé sur `installateur`.

---

# 15. Remplacement des contraintes CHECK

Dans les anciennes versions de MySQL (< 8.0.16), les contraintes `CHECK` étaient ignorées.

Les triggers permettaient alors de les remplacer.

Exemple :

Table :

```
Produit(
 id,
 prixAchat,
 prixVente
)
```

Règle :

```
prixVente >= prixAchat
```

Trigger :

```
CREATE TRIGGER controlePrix
BEFORE INSERT ON produit
FOR EACH ROW
BEGIN

    IF NEW.prixVente < NEW.prixAchat THEN

        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT='Le prix de vente doit être supérieur au prix d''achat';

    END IF;

END;
```

Un trigger similaire doit être créé pour `UPDATE`.

---

# 16. Contraintes portant sur plusieurs tables

Certaines règles métier impliquent plusieurs tables.

Exemple :

Tables :

```
Commande(
 id,
 idClient,
 montant
)

Client(
 id,
 plafond
)
```

Règle :

```
Commande.montant <= Client.plafond
```

Trigger :

```
CREATE TRIGGER controlePlafond
BEFORE INSERT ON commande
FOR EACH ROW
BEGIN

    IF EXISTS(
        SELECT 1
        FROM client
        WHERE client.id = NEW.idClient
        AND NEW.montant > client.plafond
    )
    THEN

        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT='Plafond dépassé';

    END IF;

END;
```

---

## Modification de la table référencée

La règle doit également être contrôlée lorsque le plafond du client change.

Exemple :

```
CREATE TRIGGER controleModificationPlafond
BEFORE UPDATE ON client
FOR EACH ROW
BEGIN

    IF EXISTS(
        SELECT 1
        FROM commande
        WHERE commande.idClient = NEW.id
        AND commande.montant > NEW.plafond
    )
    THEN

        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT='Nouveau plafond insuffisant';

    END IF;

END;
```

---

# 17. Maintien automatique des champs calculés

Les triggers permettent de maintenir automatiquement des valeurs calculées.

Exemples :

- nombre d'éléments associés (`COUNT`) ;
- total d'une somme (`SUM`).

---

# 17.1 Champ calculé de type COUNT

Exemple :

Tables :

```
Classe(
 id,
 libelle,
 nbEleves
)

Eleve(
 id,
 nom,
 idClasse
)
```

Le champ `nbEleves` contient le nombre d'élèves d'une classe.

---

## Ajout d'un élève

```
CREATE TRIGGER apresAjoutEleve
AFTER INSERT ON eleve
FOR EACH ROW
BEGIN

    UPDATE classe
    SET nbEleves = nbEleves + 1
    WHERE id = NEW.idClasse;

END;
```

---

## Suppression d'un élève

```
CREATE TRIGGER apresSuppressionEleve
AFTER DELETE ON eleve
FOR EACH ROW
BEGIN

    UPDATE classe
    SET nbEleves = nbEleves - 1
    WHERE id = OLD.idClasse;

END;
```

---

## Modification d'un élève

Si l'élève change de classe :

```
CREATE TRIGGER apresModificationEleve
AFTER UPDATE ON eleve
FOR EACH ROW
BEGIN

    IF NEW.idClasse <> OLD.idClasse THEN

        UPDATE classe
        SET nbEleves = nbEleves + 1
        WHERE id = NEW.idClasse;


        UPDATE classe
        SET nbEleves = nbEleves - 1
        WHERE id = OLD.idClasse;

    END IF;

END;
```

# 18. Maintien automatique des champs calculés de type SUM

Les triggers peuvent également maintenir automatiquement des valeurs correspondant à une somme.

Exemple :

On souhaite conserver le total d'une facture sans recalculer systématiquement :

```
SELECT SUM(montant)
FROM detailFacture
WHERE idFacture = 1;
```

On utilise alors un champ calculé :

```
Facture(
    id,
    dateFacture,
    total
)

DetailFacture(
    idFacture,
    idProduit,
    quantite,
    montant
)
```

Le champ `Facture.total` représente :

```
Somme des montants des détails associés
```

---

# 18.1 Après l'ajout d'un détail

Lorsqu'une ligne est ajoutée dans `DetailFacture`, le total doit être augmenté.

```
CREATE TRIGGER ajoutDetailFacture
AFTER INSERT ON detailFacture
FOR EACH ROW
BEGIN

    UPDATE facture
    SET total = total + NEW.montant
    WHERE id = NEW.idFacture;

END;
```

---

# 18.2 Après la suppression d'un détail

Lorsqu'une ligne est supprimée, son montant doit être retiré.

```
CREATE TRIGGER suppressionDetailFacture
AFTER DELETE ON detailFacture
FOR EACH ROW
BEGIN

    UPDATE facture
    SET total = total - OLD.montant
    WHERE id = OLD.idFacture;

END;
```

---

# 18.3 Après la modification d'un détail

Lors d'une modification, plusieurs cas sont possibles :

- le détail reste sur la même facture ;
- le détail change de facture.

```
CREATE TRIGGER modificationDetailFacture
AFTER UPDATE ON detailFacture
FOR EACH ROW
BEGIN

    IF NEW.idFacture <> OLD.idFacture THEN

        UPDATE facture
        SET total = total + NEW.montant
        WHERE id = NEW.idFacture;


        UPDATE facture
        SET total = total - OLD.montant
        WHERE id = OLD.idFacture;

    ELSE

        UPDATE facture
        SET total = total + NEW.montant - OLD.montant
        WHERE id = NEW.idFacture;

    END IF;

END;
```

---

# 19. Protection des champs calculés

Un champ calculé maintenu par trigger ne devrait normalement jamais être modifié directement par un utilisateur.

Exemple :

```
Classe(
 id,
 libelle,
 nbEleves
)
```

Le champ :

```
nbEleves
```

est calculé automatiquement.

Il faut empêcher :

```
UPDATE classe
SET nbEleves = 100;
```

---

# 19.1 Restriction des droits

La première solution consiste à limiter les droits SQL.

Exemple :

```
GRANT UPDATE(id, libelle)
ON maBase.classe
TO utilisateur@serveur;
```

L'utilisateur peut modifier :

- `id`
- `libelle`

mais pas :

- `nbEleves`

---

# 19.2 Protection par variable utilisateur

Cette méthode permet d'autoriser uniquement les modifications réalisées par les triggers.

Lorsqu'un trigger modifie le champ :

```
SET @autorisationTrigger = 1;
```

Puis :

```
SET @autorisationTrigger = NULL;
```

---

Trigger de protection :

```
CREATE TRIGGER protectionNbEleves
BEFORE UPDATE ON classe
FOR EACH ROW
BEGIN

    IF NEW.nbEleves <> OLD.nbEleves
       AND (@autorisationTrigger IS NULL
       OR @autorisationTrigger <> 1)

    THEN

        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT='Modification interdite du champ calculé';

    END IF;

END;
```

---

# 20. Gestion des erreurs dans une application cliente

Lorsqu'un trigger utilise :

```
SIGNAL SQLSTATE '45000'
```

une erreur est envoyée au programme appelant.

Cette erreur doit être récupérée avec le mécanisme de gestion des exceptions du langage utilisé.

---

# 20.1 Exemple en PHP

Insertion :

```
$sql = "
INSERT INTO course(dateCourse, nomCourse, distance)
VALUES(:dateCourse, :nomCourse, :distance)
";

$curseur = $db->prepare($sql);

try {

    $curseur->execute();

}
catch(Exception $e)
{

    echo $e->getMessage();

}
```

Si un trigger bloque l'ajout :

```
SQLSTATE[45000]: Le format de la donnée est incorrect
```

sera retourné.

---

# 20.2 Exemple en C#

L'appel SQL doit être placé dans un bloc `try / catch`.

```
try
{
    MySqlCommand cmd = new MySqlCommand(
        requete,
        connexion
    );

    cmd.ExecuteNonQuery();
}
catch(MySqlException e)
{
    Console.WriteLine(e.Message);
}
```

Le message envoyé par :

```
SIGNAL SQLSTATE '45000'
```

sera récupéré dans l'exception.

---

# 21. Bonnes pratiques avec les triggers

## 21.1 Donner des noms explicites

Éviter :

```
trigger1
```

Préférer :

```
avantAjoutClient
controlePrixProduit
apresSuppressionFacture
```

---

## 21.2 Un trigger doit rester simple

Un déclencheur ne doit pas contenir une logique applicative complexe.

À éviter :

- calculs importants ;
- appels multiples ;
- traitements longs.

Préférer :

- procédures stockées ;
- fonctions ;
- traitements dans l'application.

---

## 21.3 Documenter les effets secondaires

Un trigger agit automatiquement.

Un développeur qui réalise :

```
INSERT INTO client(...)
```

doit savoir qu'un trigger peut :

- modifier une valeur ;
- créer une archive ;
- mettre à jour une autre table ;
- provoquer une erreur.

---

# 22. Avantages des triggers

## Automatisation

Les règles sont exécutées automatiquement.

Exemple :

```
Ajout d'une facture
        |
        V
Trigger
        |
        V
Mise à jour automatique du total
```

---

## Centralisation des règles métier

Les contrôles sont stockés dans la base.

Avantages :

- toutes les applications utilisent les mêmes règles ;
- moins de duplication de code.

---

## Sécurité renforcée

Un trigger peut empêcher :

- des modifications interdites ;
- des incohérences ;
- des suppressions dangereuses.

---

## Cohérence des données

Les données restent valides même si plusieurs applications utilisent la base.

---

# 23. Limites des triggers

## Manque de visibilité

Le développeur peut oublier qu'une action déclenche automatiquement d'autres traitements.

Exemple :

```
DELETE FROM client;
```

peut provoquer :

- suppression d'autres données ;
- archivage ;
- mise à jour de compteurs.

---

## Difficulté de maintenance

Une grande quantité de triggers peut rendre une base difficile à comprendre.

---

## Risque de boucle infinie

Exemple :

```
Trigger A modifie table B
        |
        V
Trigger B modifie table A
        |
        V
Trigger A...
```

Il faut donc éviter les dépendances circulaires.

---

## Dépendance au SGBD

Les triggers ne sont pas totalement portables.

Un trigger MySQL peut nécessiter une adaptation sous :

- PostgreSQL ;
- SQL Server ;
- Oracle.

---

# 24. Comparaison : contrainte, procédure et trigger

|Élément|Déclenchement|Utilisation|
|---|---|---|
|Contrainte|Automatique par le SGBD|Règles simples d'intégrité|
|Procédure stockée|Appel explicite|Traitements complexes|
|Fonction stockée|Appel dans une requête|Calcul d'une valeur|
|Trigger|Automatique après événement|Contrôles et synchronisations|

---

# 25. Synthèse

Les déclencheurs MySQL permettent d'ajouter une logique automatique directement dans la base de données.

Ils sont particulièrement adaptés pour :

✅ contrôler les données avant insertion ou modification ;  
✅ empêcher certaines opérations interdites ;  
✅ appliquer des règles métier complexes ;  
✅ conserver un historique des modifications ;  
✅ maintenir automatiquement des champs calculés.

Cependant, ils doivent être utilisés avec précaution car ils rendent le comportement de la base moins visible et peuvent compliquer la maintenance.

Une bonne architecture consiste généralement à répartir les responsabilités :

- **Contraintes SQL** → règles simples d'intégrité ;
- **Triggers** → contrôles automatiques liés aux données ;
- **Procédures stockées** → traitements métier complexes ;
- **Application** → logique fonctionnelle de l'utilisateur.