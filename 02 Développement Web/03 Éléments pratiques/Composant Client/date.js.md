## Objectif

Ce module fournit un ensemble de fonctions utilitaires pour :

- Manipuler des dates aux formats français (`jj/mm/aaaa`) et ISO/MySQL (`aaaa-mm-jj`).
- Effectuer des conversions entre différents formats de dates.
- Calculer des dates relatives.
- Déterminer l'âge d'une personne.
- Obtenir des informations liées aux saisons scolaires ou universitaires.
- Produire des affichages de dates adaptés aux utilisateurs francophones.

---

# Références rapides

| Fonction | Rôle |
|-----------|--------|
| [[#getDateCourante]] | Retourne la date du jour au format ISO (`aaaa-mm-jj`) |
| [[#isDateFR]] | Vérifie qu'une chaîne respecte le format français `jj/mm/aaaa` |
| [[#formatDateLong]] | Convertit une date ISO en date longue française |
| [[#encoderDate]] | Convertit une date française vers le format ISO |
| [[#decoderDate]] | Convertit une date ISO vers le format français |
| [[#convertirDateIsoEnDateFr]] | Alias de conversion ISO → FR |
| [[#convertirDateFrEnDateIso]] | Alias de conversion FR → ISO |
| [[#convertirDateHeureIsoEnDateHeureFr]] | Convertit une date/heure ISO vers un affichage français |
| [[#getDateRelative]] | Calcule une date relative à aujourd'hui |
| [[#getAge]] | Calcule l'âge à partir d'une date de naissance |
| [[#getSaisonCourante]] | Détermine la saison de référence courante |
| [[#getSaison]] | Détermine la saison de référence pour une date donnée |

---

# Détail des fonctions

## `getDateCourante()`

Retourne la date du jour au format ISO.

### Syntaxe

```
const date = getDateCourante();
```

### Retour

```
aaaa-mm-jj
```

### Exemple

```
getDateCourante();// "2025-10-04"
```

---

## `isDateFR()`

Vérifie qu'une chaîne respecte le format français `jj/mm/aaaa`.

### Syntaxe

```
isDateFR(chaine);
```

### Paramètres

|Paramètre|Type|Description|
|---|---|---|
|`chaine`|string|Date à vérifier|

### Retour

```
true
```

ou

```
false
```

### Exemple

```
isDateFR("15/09/2025");// trueisDateFR("2025-09-15");// false
```

> Cette fonction vérifie uniquement le format, pas la validité réelle de la date.

---

## `formatDateLong()`

Transforme une date ISO en date longue française.

### Syntaxe

```
formatDateLong(dateString);
```

### Paramètres

|Paramètre|Type|Format attendu|
|---|---|---|
|`dateString`|string|`aaaa-mm-jj`|

### Retour

```
Jour jj mois aaaa
```

### Exemple

```
formatDateLong("2025-10-04");// "Samedi 4 octobre 2025"
```

---

## `encoderDate()`

Convertit une date française vers le format ISO.

### Syntaxe

```
encoderDate(date);
```

### Paramètres

|Paramètre|Type|Format attendu|
|---|---|---|
|`date`|string|`jj/mm/aaaa`|

### Retour

```
aaaa-mm-jj
```

### Exemple

```
encoderDate("4/10/2025");// "2025-10-04"
```

---

## `decoderDate()`

Convertit une date ISO vers le format français.

### Syntaxe

```
decoderDate(date);
```

### Paramètres

|Paramètre|Type|Format attendu|
|---|---|---|
|`date`|string|`aaaa-mm-jj`|

### Retour

```
jj/mm/aaaa
```

### Exemple

```
decoderDate("2025-10-04");// "04/10/2025"
```

---

## `convertirDateIsoEnDateFr()`

Version compatible historique de la conversion ISO vers format français.

### Syntaxe

```
convertirDateIsoEnDateFr(dateIso);
```

### Exemple

```
convertirDateIsoEnDateFr("2025-10-04");// "04/10/2025"
```

### Équivalent

```
decoderDate();
```

---

## `convertirDateFrEnDateIso()`

Version compatible historique de la conversion format français vers ISO.

### Syntaxe

```
convertirDateFrEnDateIso(dateFr);
```

### Exemple

```
convertirDateFrEnDateIso("04/10/2025");// "2025-10-04"
```

### Équivalent

```
encoderDate();
```

---

## `convertirDateHeureIsoEnDateHeureFr()`

Convertit une date et une heure ISO en affichage français.

### Syntaxe

```
convertirDateHeureIsoEnDateHeureFr(dateHeureIso);
```

### Paramètres

|Paramètre|Type|Format attendu|
|---|---|---|
|`dateHeureIso`|string|`aaaa-mm-jjTHH:mm:ss`|

### Retour

```
jj/mm/aaaa à HH:mm
```

### Exemple

```
convertirDateHeureIsoEnDateHeureFr(    "2025-10-04T14:35:20");// "04/10/2025 à 14:35"
```

---

## `getDateRelative()`

Calcule une date relative à aujourd'hui.

### Syntaxe

```
getDateRelative(unite, delta);
```

### Paramètres

|Paramètre|Type|Description|
|---|---|---|
|`unite`|string|`jour`, `semaine`, `mois`, `annee`|
|`delta`|number|Décalage positif ou négatif|

### Retour

```
aaaa-mm-jj
```

### Exemples

#### Demain

```
getDateRelative("jour", 1);
```

#### Il y a une semaine

```
getDateRelative("semaine", -1);
```

#### Dans trois mois

```
getDateRelative("mois", 3);
```

#### L'année dernière

```
getDateRelative("annee", -1);
```

---

## `getAge()`

Calcule l'âge à partir d'une date de naissance.

### Formats acceptés

- Format français : `jj/mm/aaaa`
- Format MySQL : `aaaa-mm-jj`
- Format MySQL datetime : `aaaa-mm-jj HH:mm:ss`

### Syntaxe

```
getAge(dateNaissance);
```

### Paramètres

|Paramètre|Type|
|---|---|
|`dateNaissance`|string|

### Retour

```
number
```

### Exemples

```
getAge("15/06/2000");
```

```
getAge("2000-06-15");
```

```
getAge("2000-06-15 00:00:00");
```

Toutes ces écritures produisent le même résultat.

---

## `getSaisonCourante()`

Détermine l'année de référence d'une saison en fonction de la date actuelle.

Cette fonction est particulièrement utile pour :

- les années scolaires ;
- les années universitaires ;
- les saisons sportives ;
- les exercices comptables.

### Syntaxe

```
getSaisonCourante(mois);
```

### Paramètres

|Paramètre|Type|Valeur par défaut|
|---|---|---|
|`mois`|number|`9` (septembre)|

### Exemple

Si nous sommes en octobre 2025 :

```
getSaisonCourante();// 2026
```

Pour une saison débutant en janvier :

```
getSaisonCourante(1);
```

---

## `getSaison()`

Détermine l'année de référence d'une saison pour une date donnée.

### Syntaxe

```
getSaison(date, mois);
```

### Paramètres

|Paramètre|Type|Description|
|---|---|---|
|`date`|Date|Date de référence|
|`mois`|number|Mois de début de saison|

### Retour

```
number
```

### Exemple

```
const date = new Date("2025-10-15");getSaison(date);// 2026
```

Pour une date située avant septembre :

```
const date = new Date("2025-05-10");getSaison(date);// 2025
```

---

# Exemple d'importation du module

```
import {    getDateCourante,    encoderDate,    decoderDate,    getAge,    getDateRelative,    getSaisonCourante} from "./dates.js";
```

---

# Historique