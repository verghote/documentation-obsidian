# Suppression de plusieurs enregistrements : quelles stratégies adopter ?

Lorsqu'une opération supprime plusieurs enregistrements de la base de données (par exemple toutes les annonces périmées), une question se pose immédiatement :

> **Comment mettre à jour l'interface utilisateur afin qu'elle reflète exactement l'état de la base de données ?**

Plusieurs approches sont possibles. Chacune présente des avantages et des inconvénients.

---

# Solution 1 : le serveur supprime, le client supprime également

## Principe

Le serveur réalise uniquement la suppression dans la base de données :

```sql
DELETE
FROM annonce
WHERE date <= CURDATE();
```

Il retourne simplement le nombre d'enregistrements supprimés.

Le navigateur applique ensuite **la même règle métier** pour retirer les lignes correspondantes de l'interface.

Exemple :

```javascript
const aujourdHui = new Date().toISOString().slice(0,10);

const lesLignes = document.querySelectorAll("tr[data-date]");

for (const ligne of lesLignes) {
    if (ligne.dataset.date <= aujourdHui) {
        ligne.remove();
    }
}
```

Lors de l'affichage des lignes chaque balise tr comporte un attribut date contenant la date  au format aaaa-mm-jj

```javascript
tr.dataset.date = date;
```

---

## Avantages

- une seule requête SQL ;
- peu de données échangées ;
- interface mise à jour immédiatement ;
- aucune reconstruction complète de la page.

---

## Inconvénients

Le navigateur reproduit une règle métier qui appartient normalement au serveur.

Des incohérences peuvent apparaître :

- différence de fuseau horaire ;
- modification des données entre le chargement de la page et la suppression ;
- évolution future de la règle métier ;
- erreur de programmation dans l'algorithme JavaScript.

Le client et le serveur prennent alors chacun leur décision.

---

## Bilan

Cette solution est rapide mais fragile.

Elle est acceptable uniquement lorsque la règle métier est extrêmement simple et que le risque d'incohérence est faible.

---

# Solution 2 : le serveur supprime puis la page est rechargée

## Principe

Le serveur réalise simplement :

```sql
DELETE
FROM annonce
WHERE date <= CURDATE();
```

Il retourne uniquement le nombre d'enregistrements supprimés.

Si au moins une annonce a été supprimée, le navigateur recharge complètement la page :

```javascript
location.reload();
```

---

## Avantages

- code extrêmement simple ;
- aucune logique métier dans le navigateur ;
- cohérence parfaite entre la base et l'interface ;
- tous les compteurs, statistiques et tris sont automatiquement recalculés ;
- maintenance très facile.

---

## Inconvénients

- rechargement complet de la page ;
- légère perte de fluidité ;
- toutes les données sont relues depuis le serveur.

---

## Bilan

Cette solution est souvent la plus robuste.

Le rechargement d'une page d'administration est généralement très rapide et la simplicité obtenue compense largement ce coût.

---

# Solution 3 : le serveur renvoie les identifiants supprimés

## Principe

Le serveur commence par récupérer les identifiants :

```sql
SELECT id
FROM annonce
WHERE date <= CURDATE();
```

Puis il effectue la suppression :

```sql
DELETE
FROM annonce
WHERE date <= CURDATE();
```

Il retourne ensuite :

```json
[
    12,
    15,
    27
]
```

Méthode deletedOld
```php
/**
 * Suppression des anciennes annonces dont la date est dépassée.
 *
 * @return array Tableau contenant les identifiants des annonces supprimées.
 */
public static function deleteOld(): array
{
    $db = Database::getInstance();

    $db->beginTransaction();

    try {

        // Récupération des identifiants avant suppression
        $sql = <<<SQL
            SELECT id
            FROM annonce
            WHERE date <= CURDATE()
        SQL;

        $select = new Select();
        $lesIdSupprimes = array_column(
            $select->getRows($sql),
            'id'
        );

        // Suppression uniquement si nécessaire
        if (!empty($lesIdSupprimes)) {

            $sql = <<<SQL
                DELETE FROM annonce
                WHERE date <= CURDATE()
            SQL;

            $db->exec($sql);
        }

        $db->commit();

        return $lesIdSupprimes;

    } catch (Exception $e) {

        $db->rollBack();

        throw $e;
    }
}
```

Le navigateur supprime uniquement les lignes correspondantes :

```javascript
function supprimerAnciennesAnnonces() {

    msg.innerText = "";

    appelAjax({
        url: 'ajax/deleteold.php',

        success: (lesIdsSupprimes) => {
            for (const id of lesIdsSupprimes) {
                document.getElementById(id)?.remove();
            }
            if (lesIdsSupprimes.length === 0) {
                msg.innerHTML = genererMessage('Aucune annonce à supprimer','orange',2000);

            } else if (lesIdsSupprimes.length === 1) {
                msg.innerHTML = genererMessage('Une annonce supprimée', 'vert',2000);
            } else {
                msg.innerHTML = genererMessage(lesIdsSupprimes.length + ' annonces supprimées', 'vert', 2000);
            }
        }
    });
}
```

---

## Avantages

- aucune logique métier côté client ;
- interface parfaitement synchronisée avec le serveur ;
- pas de rechargement complet de la page ;
- mise à jour très rapide.

---

## Inconvénients

- deux requêtes SQL sont nécessaires ;
- une transaction est souhaitable afin de garantir la cohérence entre le `SELECT` et le `DELETE` ;
- code plus complexe côté serveur.

---

## Bilan

C'est la solution la plus élégante lorsque l'on souhaite conserver une interface entièrement dynamique.

Elle demande toutefois davantage de développement.

---

# Comparaison

| Critère | Solution 1 | Solution 2 | Solution 3 |
|----------|------------|------------|------------|
| Nombre de requêtes SQL | 1 | 1 | 2 |
| Rechargement de la page | Non | Oui | Non |
| Logique métier côté client | Oui | Non | Non |
| Risque d'incohérence | Moyen | Aucun | Aucun |
| Complexité du code | Faible | Très faible | Plus élevée |
| Performances | Excellentes | Bonnes | Très bonnes |
| Maintenance | Moyenne | Excellente | Bonne |

---

# Quelle solution choisir ?

Le choix dépend essentiellement du contexte.

### Pour une interface d'administration

La **solution 2** est souvent la plus pertinente.

Elle privilégie :

- la simplicité ;
- la robustesse ;
- la facilité de maintenance.

Le coût d'un rechargement de page est généralement négligeable.

---

### Pour une interface fortement interactive

La **solution 3** est préférable.

Elle permet une mise à jour immédiate de l'interface sans rechargement tout en laissant au serveur l'entière responsabilité de la logique métier.

---

### La solution 1

Elle peut sembler séduisante car elle ne nécessite qu'une seule requête SQL.

Cependant, elle introduit une duplication de la logique métier entre le serveur et le navigateur.

Cette duplication augmente le risque de divergence et complique l'évolution future de l'application.

En règle générale, on retiendra le principe suivant :

> **Le serveur décide ; le client affiche.**