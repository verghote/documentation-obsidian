
Ce mini-framework transforme les erreurs techniques PDO en messages compréhensibles pour l’utilisateur, tout en journalisant les détails pour le développeur.

Il repose sur 3 niveaux complémentaires :

1. La classe `Erreur` (gestion centralisée et fallback).
2. Le fichier `config/contrainte.php` (messages métier clairs par contrainte SQL).
3. Les validations précoces (méthodes 'before*' définies dans les classe métier ) pour certaines règles métier applicatives.

# 1) Rôle de la classe `Erreur`

Fichier : `src/ClasseTechnique/Erreur.php`

### Ce qu’elle fait

- Intercepte les exceptions globalement (`set_exception_handler`).
- Journalise les erreurs techniques (`Journal::enregistrer`).
- Retourne une réponse adaptée :
  - JSON pour AJAX/API (`ReponseJson::envoyerErreur`)
  - HTML sinon (redirection `/erreur`).

## Cas `PDOException`

Quand une exception PDO est levée :

1. Elle est journalisée.
2. Le framework tente de produire un message lisible via `resoudreMessageSQL()` :
   - message venant d’un `SIGNAL SQL` (`SQLSTATE 45000`),
   - message configuré par nom de contrainte (`config/contrainte.php`),
   - fallback sur code SQL connu (ex. 1062, 1451, 1452…),
   - sinon message système générique.

### Important

Même **sans** fichier `config/contrainte.php`, la classe `Erreur` renvoie un message :
- soit générique,
- soit “semi-générique” basé sur le code SQL.

Le système continue donc à fonctionner, mais le message peut être moins précis métier.

## 2) Rôle du fichier `config/contrainte.php`

Fichier : `config/contrainte.php`

Ce fichier associe le **nom d’une contrainte SQL** à un **message utilisateur clair**.

Exemple :

```php
return [
    'uk_categorie_nom' => "Une catégorie portant ce nom existe déjà.",
    'ck_categorie_age' => "L'âge minimum doit être inférieur à l'âge maximum.",
    'fk_coureur_categorie' => "Suppression impossible : cette catégorie possède des coureurs.",
];
```

**Principe**
Quand MySQL renvoie une erreur contenant fk_coureur_categorie, Erreur::resoudreMessageSQL() remplace le message technique par un message pédagogique.

Exemple concret : suppression d’une catégorie utilisée

**Contrainte SQL**
	Dans sql/Coureur/create.sql : constraint fk_coureur_categorie  foreign key (idCategorie)   references categorie (id)
 
**Comportement**
Si on supprime une catégorie liée à des coureurs :
+ MySQL bloque (1451),
+ Erreur intercepte,

si fk_coureur_categorie est configurée, on affiche : **Suppression impossible : cette catégorie possède des coureurs.**
Sinon, la réponse sera : **Suppression impossible : donnée utilisée.**
 
Une même FK peut couvrir plusieurs erreurs
Exemple FK coureur.idCategorie -> categorie.id :
+ 1452 : ajout/modification d’un coureur avec catégorie inexistante (enfant invalide).
+ 1451 : suppression (ou modif PK parent) d’une catégorie référencée.

Le framework peut déjà distinguer grossièrement ces cas avec les codes SQL (1451 vs 1452).

Si l'on souhaite obtenir un message plus clair, il faut placer la règle métier dans une méthode **"before*"** de la classe métier 

**Cas particulier des contraintes d'intégrité (fk_)**

Normalement, il n’est pas possible d’associer une contrainte d’intégrité référentielle à un seul message d’erreur, car elle contrôle théoriquement 4 traitements : 
+ 1) ajouter un enregistrement avec une clé étrangère qui n'existe pas
+ 2) supprimer un enregistrement dont la clé primaire est encore utilisée dans une clé étrangère
+ 3) modifier la clé primaire si cette clé primaire est utilisée dans une clé étrangère
+ 4) modifier la clé étrangère d'un enregistrement avec un valeur qui n'existe pas

Concrètement, avec la relation club (parent) / coureur (enfant) cela correspond à : 
+ 1) ajout d’un coureur avec idClub inexistant
+ 4) modification d’un coureur avec un idClub inexistant
+ 2) suppression d’un club encore référencé, 
+ 3) modification de l’identifiant d’un club référencé ; 

Dans nos projets, 
+ Les cas 1) et 4) est  traité par l'utilisation d'un objet ColumList pour définir la colonne de clé étrangère et limiter ainsi sa valeur aux seules valeurs possible

```php
$this->addColumn('idClub', new ColumnList(  
    values: Club::getLesId()  
));
```

+ Le cas 3) est neutralisé soit par un trigger interdisant la modification de la clé primaire soit simplement par le fait que l'application ne propose pas cette opération

``` sql
create trigger avantMajCategorie before update  on categorie  
for each row  
begin  
    if new.id <> old.id then  
        signal sqlstate '45000'  set message_text =  'Le code de la catégorie ne peut pas être modifié.';  
    end if;
    ...
```

Donc il ne reste que le cas 2), la contrainte 'fk' peut donc être intégrée dans le fichier de configuration config/contrainte.php

# 3) Rôle des méthodes before* et after* définies dans les classes métier

Elles permettent d'exprimer des règles métiers qui ne sont pas forcément exprimables ou souhaitables au niveau SQL

Exemples :
+ Interdire la suppression d’une catégorie “système” (M10, M11) même sans coureurs.
+ Bloquer une action selon le rôle utilisateur courant.

Elles peuvent aussi reprendre une contrainte définie en SQL pour
+ Donner un message plus contextualisé immédiatement,
+ Eviter une requête SQL vouée à l’échec et ainsi gagner en performance en évitant un aller-retour vers le serveur MySQL

Il faut cependant nuancer ce gain de performance qui de l'ordre de 5 à 30 ms selon charge/réseau. Donc l’intérêt principal de la validation précoce est souvent la qualité du message et l’UX, plus que la performance brute, sauf traitements en masse. 
# Synthèse

1. La base protège l’intégrité (source de vérité).
2. Le framework traduit les erreurs techniques en messages lisibles.
3. config/contrainte.php apporte la qualité métier du message.
4. before sert aux règles applicatives et à la validation précoce.

# Résumé visuel du flux
1. Action utilisateur (insert/update/delete).
2. Règle before éventuelle (validation précoce).
3. Requête SQL.
4. Si erreur PDO :
	+ journalisation technique,
	+ résolution message (SIGNAL / config/contrainte.php / fallback code SQL / message générique),
	+ réponse JSON ou HTML.

