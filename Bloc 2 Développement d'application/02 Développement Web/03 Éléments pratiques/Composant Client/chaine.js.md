## Fonctions de la bibliothèque

### `enleverAccentEtMajuscule(str)`
**Rôle :** Supprime les accents et convertit en minuscules.  
**Exemple :**
```javascript
enleverAccentEtMajuscule("École") // "ecole"
```

---

### `comparerSansAccentEtSansCasse(str1, str2)`
**Rôle :** Compare deux chaînes sans accents ni casse.  
**Exemple :**
```javascript
comparerSansAccentEtSansCasse("École", "ecole") // true
```

---

### `comparerSansCasse(str1, str2)`
**Rôle :** Compare deux chaînes sans tenir compte des majuscules/minuscules.  
**Exemple :**
```javascript
comparerSansCasse("Bonjour", "bonjour") // true
```

---

### `contenir(texte, recherche)`
**Rôle :** Vérifie si une chaîne contient une autre (sans accents ni casse).  
**Exemple :**
```javascript
contenir("École de danse", "ecole") // true
```

---

### `supprimerEspace(valeur)`
**Rôle :** Supprime les espaces superflus (début, fin et multiples espaces internes).  
**Exemple :**
```javascript
supprimerEspace("  Jean   Dupont  ") // "Jean Dupont"
```

---

### `enleverAccent(valeur)`
**Rôle :** Supprime les accents d'une chaîne.  
**Exemple :**
```javascript
enleverAccent("àéîôù") // "aeiou"
```

---

### `normaliser(valeur)`
**Rôle :** Supprime accents, espaces superflus et met en minuscules.  
**Exemple :**
```javascript
normaliser(" ÉCOLE ") // "ecole"
```

---

### `ucWord(nom)`
**Rôle :** Met la première lettre de chaque mot en majuscule.  
**Exemple :**
```javascript
ucWord("jean-paul dupont") // "Jean-Paul Dupont"
```

---

### `ucFirst(nom)`
**Rôle :** Met la première lettre du texte en majuscule.  
**Exemple :**
```javascript
ucFirst("bonjour") // "Bonjour"
```

---

### `contient(texte, recherche)`
**Rôle :** Vérifie si une chaîne contient une autre (normalisée).  
**Exemple :**
```javascript
contient("Université", "vers") // true
```

---

### `commencePar(texte, prefixe)`
**Rôle :** Vérifie si une chaîne commence par une autre (sans accents ni casse).  
**Exemple :**
```javascript
commencePar("École", "eco") // true
```

---

### `terminePar(texte, suffixe)`
**Rôle :** Vérifie si une chaîne se termine par une autre (sans accents ni casse).  
**Exemple :**
```javascript
terminePar("Université", "site") // true
```

---

### `compterOccurrence(texte, recherche)`
**Rôle :** Compte le nombre d'occurrences d'une sous-chaîne.  
**Exemple :**
```javascript
compterOccurrence("abc abc abc", "abc") // 3
```

---

### `tronquer(texte, longueur)`
**Rôle :** Coupe une chaîne et ajoute `...` si nécessaire.  
**Exemple :**
```javascript
tronquer("Bonjour tout le monde", 7) // "Bonjour..."
```

---

### `repeter(texte, nombre)`
**Rôle :** Répète une chaîne plusieurs fois.  
**Exemple :**
```javascript
repeter("*", 5) // "*****"
```

---

### `supprimerTousLesEspaces(texte)`
**Rôle :** Supprime tous les espaces.  
**Exemple :**
```javascript
supprimerTousLesEspaces("Jean Dupont") // "JeanDupont"
```

---

### `chiffresSeulement(texte)`
**Rôle :** Ne garde que les chiffres.  
**Exemple :**
```javascript
chiffresSeulement("Tél : 06 12 34 56 78") // "0612345678"
```

---

### `lettresSeulement(texte)`
**Rôle :** Ne garde que les lettres.  
**Exemple :**
```javascript
lettresSeulement("Jean123!") // "Jean"
```

---

### `estVide(texte)`
**Rôle :** Vérifie si une chaîne est vide ou composée uniquement d'espaces.  
**Exemple :**
```javascript
estVide("   ") // true
```

---

### `inverser(texte)`
**Rôle :** Inverse les caractères d'une chaîne.  
**Exemple :**
```javascript
inverser("Bonjour") // "ruojnoB"
```