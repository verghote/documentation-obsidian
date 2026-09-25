# Objectif

Ce module fournit un ensemble de fonctions destinées à :

- valider les formulaires ;
- afficher les erreurs utilisateur ;
- manipuler les chaînes de caractères ;
- construire des objets `FormData` destinés aux appels AJAX.

L'objectif est d'uniformiser le comportement des interfaces et de réduire la quantité de code JavaScript à écrire dans les applications.

---
# Référence rapide des fonctions

| Fonction                      | Rôle                                                                         |
| ----------------------------- | ---------------------------------------------------------------------------- |
| configurerFormulaire          | Prépare automatiquement les formulaires et les zones d'affichage des erreurs |
| configurerDate                | Configure un champ date avec contraintes min/max                             |
| filtrerLaSaisie               | Restreint les caractères autorisés pendant la saisie                         |
| verifier                      | Valide un champ à l'aide des contraintes HTML5                               |
| donneesValides                | Contrôle l'ensemble des champs d'un formulaire                               |
| dateValide                    | Vérifie une date au format jj/mm/aaaa                                        |
| effacerLesChamps              | Vide les champs d'un formulaire                                              |
| effacerLesErreurs             | Supprime les messages d'erreur affichés                                      |
| supprimerEspace               | Supprime les espaces superflus                                               |
| enleverAccent                 | Supprime les accents d'une chaîne                                            |
| enleverAccentEtMajuscule      | Supprime accents et majuscules                                               |
| comparerSansAccentEtSansCasse | Compare deux chaînes sans accent ni casse                                    |
| comparerSansCasse             | Compare deux chaînes sans tenir compte de la casse                           |
| contenir                      | Recherche une chaîne sans tenir compte des accents ni de la casse            |
| construireFormData            | Construit un objet FormData normalisé                                        |

---

## `configurerFormulaire`

Prépare automatiquement tous les champs : input, select, textarea.

Une zone d'affichage des erreurs est ajoutée après chaque champ.
 Exemple

```javascript
configurerFormulaire();
```
 Avant

```html
<input id="nom">
```
 Après

```html
<input id="nom">
<div class="messageErreur"></div>
```

---

## `configurerDate`

Configure dynamiquement un champ :

```html
<input type="date">
```

avec :

- une date minimale ;
- une date maximale ;
- une valeur par défaut.

Un commentaire explicatif est automatiquement ajouté au label associé.

 Exemple

```javascript
configurerDate(
    document.getElementById('dateNaissance'),
    {
        min: '2000-01-01',
        max: '2030-12-31'
    }
);
```

Résultat :

```text
Date de naissance
(La date doit être comprise entre le 01/01/2000 et le 31/12/2030)
```

---

## `filtrerLaSaisie`

Empêche l'utilisateur de saisir des caractères non autorisés.
 Chiffres uniquement

```javascript
filtrerLaSaisie('age', /[0-9]/);
```
 Lettres uniquement

```javascript
filtrerLaSaisie('nom', /[A-Za-z]/);
```

---

## `verifier`

Valide un champ en utilisant les contraintes HTML5 : required, minlength, maxlength,  pattern, min, max.

  Exemple

```javascript
if (verifier(document.getElementById('email'))) {
    // champ valide
}
```

En cas d'erreur : bordure rouge et affichage du message utilisateur.
    
---

## `donneesValides`

Contrôle automatiquement tous les champs du formulaire.
 Exemple

```javascript
if (!donneesValides()) {
    return;
}
```

 Contrôle d'une zone particulière

```javascript
const zone = document.getElementById('formulaire');

if (donneesValides(zone)) {
    enregistrer();
}
```

---

## `dateValide`

Contrôle une date saisie au format : jj/mm/aaaa

 Exemple

```javascript
if (dateValide('dateNaissance')) {
    enregistrer();
}
```

Contrôles réalisés : format, jour, mois, année et cohérence réelle de la date.

---

## `effacerLesChamps`

Vide tous les champs d'une zone.

```javascript
effacerLesChamps();
```

ou

```javascript
effacerLesChamps(document.getElementById('formulaire'));
```

---

## `effacerLesErreurs`

Supprime tous les messages d'erreur.

```javascript
effacerLesErreurs();
```

ou

```javascript
effacerLesErreurs(document.getElementById('formulaire'));
```

---

##  `supprimerEspace`

Supprime : les espaces en début, les espaces en fin et les espaces multiples.

 Exemple

```javascript
supprimerEspace('   Jean     Dupont   ');
```

Résultat : Jean Dupont

---
## `enleverAccent`

```javascript
enleverAccent('Événement');
```

Résultat : Evenement

---
##  `enleverAccentEtMajuscule`

```javascript
enleverAccentEtMajuscule('ÉVÉNEMENT');
```

Résultat : evenement

---
## `comparerSansAccentEtSansCasse`

```javascript
comparerSansAccentEtSansCasse('École', 'ecole');
```

Résultat : true

---
##  `comparerSansCasse`

```javascript
comparerSansCasse('Paris', 'PARIS');
```

Résultat : text

---
## `contenir`

Recherche un texte dans une chaîne sans tenir compte des accents ni des majuscules.

 Exemple

```javascript
contenir('École Française', 'francaise');
```

Résultat : true

---

## `construireFormData`

Construit un objet `FormData` normalisé à transmettre lors d'un appel AJAX.

 Paramètres

| Groupe       | Description                               |
| ------------ | ----------------------------------------- |
| obligatoires | Toujours transmis                         |
| optionnels   | Transmis uniquement s'ils sont renseignés |
| booleens     | Convertis automatiquement en 1 ou 0       |

 Exemple

```javascript
const formData = construireFormData({
    obligatoires: {
        id: 12,
        nom: 'Dupont'
    },

    optionnels: {
        telephone: '',
        email: 'contact@test.fr'
    },

    booleens: {
        actif: true
    }
});
```

 Données produites

```text
id=12
nom=Dupont
email=contact@test.fr
actif=1
```

Le champ `telephone` n'est pas transmis car il est vide.

---

# Bonnes pratiques

## Initialisation d'un formulaire

```javascript
configurerFormulaire();
```

À exécuter après le chargement du DOM.

---

## Validation avant appel AJAX

```javascript
if (!donneesValides()) {
    return;
}
```

---

## Nettoyage après succès

```javascript
effacerLesErreurs();
effacerLesChamps();
```

---

## Construction des données AJAX

Toujours privilégier :

```javascript
const formData = construireFormData(...);
```

plutôt que plusieurs appels successifs à :

```javascript
formData.append(...);
```

afin de conserver un comportement homogène dans toute l'application.