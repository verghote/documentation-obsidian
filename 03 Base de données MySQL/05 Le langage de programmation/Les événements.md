
# Introduction

Les **événements MySQL** (*MySQL Events*) permettent d'exécuter automatiquement une requête SQL ou une procédure stockée à une date donnée ou de manière périodique.

Ils constituent un **planificateur de tâches intégré** à MySQL et sont particulièrement utiles pour automatiser des opérations de maintenance telles que :

- suppression de données obsolètes ;
- archivage d'informations ;
- mise à jour de statistiques ;
- sauvegarde de certaines données ;
- exécution régulière de procédures stockées.

---

# 1. Pourquoi utiliser les événements ?

Il est fréquent d'avoir besoin d'exécuter automatiquement une requête à intervalles réguliers.

Sans les événements MySQL, cette automatisation peut être réalisée à l'aide d'un planificateur de tâches du système d'exploitation :

- **Cron** sous Linux ;
- **Planificateur de tâches** sous Windows.

Par exemple, sous Linux, la commande suivante supprime chaque jour à **1 heure du matin** certaines données de la base :

```bash
0 1 * * * mysql -u utilisateur -pmotDePasse nomBase -e "DELETE FROM ..."
```

Les événements MySQL permettent de réaliser ce type d'automatisation directement dans le serveur de bases de données, sans dépendre du système d'exploitation.

---

# 2. Le planificateur d'événements

Les événements sont exécutés par un composant interne appelé **Event Scheduler**.

Avant de créer des événements, il faut vérifier qu'il est activé.

## Activer le planificateur

```sql
SET @@GLOBAL.event_scheduler = 1;
```

---

## Vérifier son état

```sql
SELECT @@GLOBAL.event_scheduler;
```

Valeurs possibles :

| Valeur | Signification |
|---------|---------------|
| ON (1) | Planificateur activé |
| OFF (0) | Planificateur désactivé |

---

## Remarque

Sur certains serveurs mutualisés ou hébergés, le planificateur est désactivé.

Son activation nécessite généralement le privilège **SUPER** ou **SYSTEM_VARIABLES_ADMIN**.

Même si le planificateur est désactivé, il est souvent possible de créer les événements ; ils seront simplement exécutés uniquement lorsque le planificateur sera activé.

---

# 3. Créer un événement

La création d'un événement s'effectue avec l'instruction `CREATE EVENT`.

## Syntaxe générale

```sql
CREATE EVENT [IF NOT EXISTS] nomEvenement

ON SCHEDULE
    EVERY intervalle
    STARTS dateHeure

DO

    instruction_SQL;
```

L'instruction exécutée peut être :

- une requête SQL ;
- plusieurs requêtes (dans un bloc `BEGIN ... END`) ;
- un appel à une procédure stockée.

---

# 4. Les intervalles

L'option `EVERY` indique la fréquence d'exécution.

## Exemples

Toutes les minutes :

```sql
EVERY 1 MINUTE
```

Toutes les 5 minutes :

```sql
EVERY 5 MINUTE
```

Toutes les heures :

```sql
EVERY 1 HOUR
```

Toutes les 6 heures :

```sql
EVERY 6 HOUR
```

Tous les jours :

```sql
EVERY 1 DAY
```

Toutes les semaines :

```sql
EVERY 1 WEEK
```

Tous les mois :

```sql
EVERY 1 MONTH
```

Tous les ans :

```sql
EVERY 1 YEAR
```

---

# 5. Définir la date de début

La clause `STARTS` permet de choisir la date de première exécution.

## Exécution immédiate

```sql
STARTS NOW()
```

---

## Exécution dans 5 minutes

```sql
STARTS NOW() + INTERVAL 5 MINUTE
```

---

## Exécution à une date précise

```sql
STARTS '2026-12-01 08:00:00'
```

---

# 6. Exemple complet

Suppression automatique des messages dont la date limite d'affichage est dépassée.

```sql
CREATE EVENT effacerMessage

ON SCHEDULE
    EVERY 1 DAY

STARTS
    NOW() + INTERVAL 5 MINUTE

DO

DELETE FROM message

WHERE dateLimiteAffichage < NOW();
```

## Fonctionnement

L'événement :

- démarre cinq minutes après sa création ;
- s'exécute ensuite tous les jours ;
- supprime tous les messages dont la date d'affichage est dépassée.

---

# 7. Exécuter une procédure stockée

Un événement peut appeler directement une procédure.

```sql
CREATE EVENT majStatistiques

ON SCHEDULE
EVERY 1 DAY

DO

CALL calculerStatistiques();
```

Cette solution est recommandée lorsque le traitement est complexe.

---

# 8. Modifier un événement

L'instruction `ALTER EVENT` permet de modifier un événement existant.

## Modifier la fréquence

```sql
ALTER EVENT effacerMessage

ON SCHEDULE
EVERY 6 HOUR;
```

L'événement sera désormais exécuté toutes les six heures.

---

# 9. Activer ou désactiver un événement

## Activer

```sql
ALTER EVENT effacerMessage
ENABLE;
```

---

## Désactiver

```sql
ALTER EVENT effacerMessage
DISABLE;
```

Lorsqu'un événement est désactivé, il reste enregistré dans la base mais n'est plus exécuté.

---

# 10. Supprimer un événement

Pour supprimer définitivement un événement :

```sql
DROP EVENT effacerMessage;
```

---

## Éviter une erreur si l'événement n'existe pas

```sql
DROP EVENT IF EXISTS effacerMessage;
```

---

# 11. Afficher les événements

Afficher les événements de la base courante :

```sql
SHOW EVENTS;
```

Afficher les événements d'une base spécifique :

```sql
SHOW EVENTS FROM nomBase;
```

---

# 12. Voir le code d'un événement

Comme pour les procédures stockées, il est possible d'afficher le code SQL d'un événement.

```sql
SHOW CREATE EVENT effacerMessage;
```

---

# 13. Bonnes pratiques

Il est recommandé de :

- donner un nom explicite aux événements ;
- placer les traitements complexes dans des procédures stockées ;
- éviter les événements exécutés trop fréquemment ;
- tester les requêtes avant leur automatisation ;
- surveiller régulièrement leur bon fonctionnement.

---

# 14. Cas d'utilisation

Les événements MySQL sont particulièrement adaptés pour :

- supprimer les anciennes données ;
- archiver des informations ;
- nettoyer les tables temporaires ;
- recalculer des statistiques ;
- générer des rapports ;
- mettre à jour des indicateurs ;
- envoyer des notifications via une procédure.

---

# 15. Tableau récapitulatif

| Instruction | Description |
|-------------|-------------|
| CREATE EVENT | Crée un événement |
| ALTER EVENT | Modifie un événement |
| ENABLE | Active un événement |
| DISABLE | Désactive un événement |
| DROP EVENT | Supprime un événement |
| SHOW EVENTS | Liste les événements |
| SHOW CREATE EVENT | Affiche le code d'un événement |
| SET @@GLOBAL.event_scheduler = 1 | Active le planificateur |
| SELECT @@GLOBAL.event_scheduler | Vérifie l'état du planificateur |

---

# Conclusion

Les événements MySQL permettent d'automatiser directement dans le serveur de bases de données l'exécution de requêtes SQL ou de procédures stockées.

Ils remplacent avantageusement les planificateurs de tâches externes pour les traitements liés à la base de données. Grâce au **planificateur d'événements (Event Scheduler)**, il devient possible de programmer des opérations répétitives comme la maintenance, l'archivage ou le nettoyage des données de manière simple et centralisée.