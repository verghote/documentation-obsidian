# Objectif

Ce module fournit un ensemble de fonctions destinées à :

- afficher des messages utilisateur ;
- afficher des notifications temporaires ;
- gérer les confirmations utilisateur ;
- afficher les erreurs de validation ;
- faciliter les redirections ;
- afficher des indicateurs de chargement ;
- animer des compteurs numériques.

L'objectif est d'uniformiser les interactions utilisateur dans l'ensemble des applications.

---

# Référence rapide des fonctions

|Fonction|Rôle|
|---|---|
|[[#genererMessage]]|Génère un message Bootstrap prêt à afficher|
|[[#afficherToast]]|Affiche une notification temporaire|
|[[#afficherSousLeChamp]]|Affiche un message sous un champ de formulaire|
|[[#messageBox]]|Affiche une boîte de dialogue modale|
|[[#afficherErreur]]|Affiche proprement une erreur dans la console|
|[[#retournerVers]]|Affiche un message puis redirige automatiquement|
|[[#retournerVersApresConfirmation]]|Affiche un message puis attend une confirmation avant redirection|
|[[#corriger]]|Affiche une erreur et restaure la valeur précédente d'un champ|
|[[#confirmer]]|Demande une confirmation utilisateur|
|[[#afficherVeuillezPatienter]]|Affiche une fenêtre de chargement|
|[[#fermerVeuillezPatienter]]|Ferme la fenêtre de chargement|
|[[#afficherCompteur]]|Anime un compteur numérique|

---

# 1. Messages et notifications

## genererMessage : Génère un message Bootstrap prêt à être inséré dans une page.

| Paramètre | Description                            |
| --------- | -------------------------------------- |
| texte     | Message à afficher                     |
| couleur   | vert, **rouge** ou orange              |
| duree     | Fermeture automatique en millisecondes |
### Exemple

```javascript
msg.innerHTML = genererMessage('Enregistrement effectué','vert');
```

### Fermeture automatique

```javascript
msg.innerHTML = genererMessage('Opération réalisée', 'vert', 3000);
```

---

## afficherToast :  Affiche une notification temporaire flottante.

| Paramètre | Description                                                                                                                 |
| --------- | --------------------------------------------------------------------------------------------------------------------------- |
| message   | Message à afficher                                                                                                          |
| type      | **success**, warning, error ou info                                                                                         |
| position  | position sur l'écran : **bottom-center**, top-left, top-right, bottom-left, bottom-right, top-center, bottom-center, center |
| duration  | fermeture automatique en millisecondes                                                                                      |
### Exemple

```javascript
afficherToast('Enregistrement effectué');
```

### Notification d'erreur

```javascript
('Une erreur est survenue', 'error');
```

### Types disponibles

- success
- error
- warning
- info
### Positions disponibles

- top-left
- top-right
- top-center
- center
- bottom-left
- bottom-right
- bottom-center

---
## messageBox

Affiche une boîte de dialogue modale nécessitant une action de l'utilisateur.

### Exemple

```javascript
messageBox(
    'Le document a été enregistré',
    'success'
);
```

### Types disponibles

- success
- error
- warning
- info

---
# 2. Gestion des erreurs de formulaire

## afficherSousLeChamp

Affiche un message sous un champ de formulaire.

Si aucun message n'est transmis, le message HTML5 de validation est utilisé.

### Exemple

```javascript
afficherSousLeChamp(
    'nom',
    'Veuillez renseigner votre nom'
);
```

### Utilisation avec la validation HTML5

```javascript
const input = document.getElementById('nom');

if (!input.checkValidity()) {
    afficherSousLeChamp('nom');
}
```

---

## corriger

Affiche une erreur puis restaure la valeur précédente.

### Exemple

```javascript
corriger(
    input,
    'Valeur non autorisée'
);
```

### Cas pris en charge

- input texte ;
    
- select ;
    
- checkbox ;
    
- champ utilisant l'attribut `data-old`.
    

---

# 3. Gestion des erreurs techniques

## afficherErreur

Affiche proprement une erreur dans la console.

Fonction particulièrement utile pour les erreurs AJAX.

### Exemple

```javascript
catch(error => {
    afficherErreur(error);
});
```

### Exemple avec objet JSON

```javascript
afficherErreur({
    code: 500,
    message: 'Erreur serveur'
});
```

---

# 4. Confirmations utilisateur

## confirmer

Affiche une boîte de confirmation.

### Exemple

```javascript
confirmer(
    () => supprimer(),
    'Confirmer la suppression ?'
);
```

### Fonctionnement

Bouton :

```text
Oui
```

→ exécute le callback.

Bouton :

```text
Non
```

→ ferme simplement la fenêtre.

---

# 5. Redirections

## retournerVers

Affiche un message puis redirige automatiquement.

### Exemple

```javascript
retournerVers(
    'Opération réalisée',
    '/accueil.php'
);
```

### Délai personnalisé

```javascript
retournerVers(
    'Redirection...',
    '/accueil.php',
    5000
);
```

---

## retournerVersApresConfirmation

Affiche un message puis attend une confirmation utilisateur avant redirection.

### Exemple

```javascript
retournerVersApresConfirmation(
    'Votre compte a été créé',
    '/connexion.php'
);
```

---

# 6. Fenêtre de chargement

## afficherVeuillezPatienter

Affiche une fenêtre modale bloquante.

### Exemple

```javascript
const attente =
    await afficherVeuillezPatienter();
```

Utilisation typique avant un appel AJAX long.

---

## fermerVeuillezPatienter

Ferme la fenêtre affichée précédemment.

### Exemple

```javascript
await fermerVeuillezPatienter(attente);
```

### Exemple complet

```javascript
const attente =
    await afficherVeuillezPatienter();

await appelAjax(...);

await fermerVeuillezPatienter(attente);
```

---

# 7. Compteurs animés

## afficherCompteur

Anime progressivement une valeur numérique.

### Exemple simple

```javascript
afficherCompteur({
    conteneur : document.getElementById('nb'),
    valeurFinale : 250
});
```

---

### Exemple avec durée personnalisée

```javascript
afficherCompteur({
    conteneur : document.getElementById('nb'),
    valeurFinale : 250,
    duree : 3000
});
```

---

### Exemple avec formatage

```javascript
afficherCompteur({
    conteneur : document.getElementById('ca'),
    valeurFinale : 15000,
    format : valeur => valeur + ' €'
});
```

---

### Styles personnalisés

```javascript
afficherCompteur({
    conteneur : document.getElementById('ca'),
    valeurFinale : 15000,

    styles : {
        color : 'green',
        fontSize : '50px'
    }
});
```

---

# Bonnes pratiques

## Affichage d'une erreur utilisateur

```javascript
messageBox(
    'Veuillez corriger les erreurs',
    'error'
);
```

---

## Affichage d'une erreur de validation

```javascript
afficherSousLeChamp(
    'email',
    'Adresse email invalide'
);
```

---

## Confirmation avant suppression

```javascript
confirmer(
    () => supprimer(),
    'Confirmer la suppression ?'
);
```

---

## Traitement AJAX long

```javascript
const attente =
    await afficherVeuillezPatienter();

await appelAjax(...);

await fermerVeuillezPatienter(attente);
```

---

## Notification rapide

```javascript
afficherToast(
    'Modification enregistrée'
);
```

Les toasts sont à privilégier pour les messages courts ne nécessitant aucune action de l'utilisateur.