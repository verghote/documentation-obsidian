## 🧩 Présentation générale

La classe abstraite `Input` sert de **base pour la validation des données d’entrée** dans une application PHP.

Elle fournit un socle commun pour :

- représenter une valeur utilisateur,
- définir si un champ est obligatoire,
- vérifier la validité minimale des données,
- stocker un message de validation en cas d’erreur.

👉 Elle est conçue pour être **étendue par des classes spécialisées** (ex : `InputEmail`, `InputText`, `InputNumber`, etc.).

---

## ⚙️ Caractéristiques techniques

|Propriété|Valeur|
|---|---|
|Type|Classe abstraite|
|Accès|Public / Protected|
|Usage|Validation de données|
|Extensibilité|Oui (héritage obligatoire)|
|PHP recommandé|≥ 8.0 (`strict_types=1`)|

---

## 🧱 Structure de la classe

```
abstract class Input
```

La classe ne peut pas être instanciée directement.

---

## 🧾 Propriétés

### 🔹 `public mixed $Value`

- Contient la valeur du champ.
- Peut être de n’importe quel type (`string`, `int`, `null`, etc.).

---

### 🔹 `public bool $Require`

- Indique si le champ est obligatoire.
- Valeur par défaut : `true`.

|Valeur|Signification|
|---|---|
|`true`|Champ obligatoire|
|`false`|Champ optionnel|

---

### 🔹 `protected string $validationMessage`

- Stocke le message d’erreur lié à la validation.
- Accessible uniquement dans la classe et ses classes filles.

---

## 🏗️ Constructeur

### 📌 Signature

```
public function __construct()
```

### 🎯 Rôle

Initialise les propriétés par défaut :

|Propriété|Valeur initiale|
|---|---|
|`$Value`|`null`|
|`$Require`|`true`|
|`$validationMessage`|`''`|

---

## 🔍 Méthode : `getValidationMessage`

### 📌 Signature

```
public function getValidationMessage(): string
```

---

### 🎯 Objectif

Retourner le message d’erreur de validation associé à l’input.

---

### 📤 Retour

|Type|Description|
|---|---|
|string|Message d’erreur (vide si aucun problème)|

---

### 💡 Exemple

```
echo $input->getValidationMessage();
```

---

## 🧪 Méthode : `checkValidity`

### 📌 Signature

```
public function checkValidity(): bool
```

---

### 🎯 Objectif

Vérifier si la valeur de l’input respecte la règle de base :

- si le champ est obligatoire (`Require = true`)
- alors il ne doit pas être :
    - `null`
    - vide
    - ou composé uniquement d’espaces

---

### 🔄 Logique de validation

La vérification s’effectue ainsi :

```
if ($this->Require && ($this->Value === null || strlen(trim((string)$this->Value)) === 0))
```

### ✔️ Cas valide

- champ non requis (`Require = false`)
- ou valeur non vide

---

### ❌ Cas invalide

- champ requis
- valeur `null`
- chaîne vide `" "`, `""`, ou espaces uniquement

---

### ⚠️ Effet en cas d’erreur

- un message est stocké dans `$validationMessage`
- la méthode retourne `false`

```
$this->validationMessage = "Veuillez renseigner ce champ " . $this->Value;
```

---

### 📤 Retour

|Type|Signification|
|---|---|
|bool|`true` si valide, `false` sinon|

---

## 💡 Exemple d’utilisation

### ✔️ Cas valide

```
$input = new class extends Input {};$input->Value = "Jean Dupont";if ($input->checkValidity()) {    echo "OK";}
```

---

### ❌ Cas invalide

```
$input = new class extends Input {};$input->Value = "   ";if (!$input->checkValidity()) {    echo $input->getValidationMessage();}
```

Résultat :

```
Veuillez renseigner ce champ
```

---

## 🧠 Rôle dans une architecture

La classe `Input` est typiquement utilisée dans :

- formulaires web
- API REST
- DTO (Data Transfer Objects)
- couches de validation métier

---

## 🔒 Avantages

- standardisation de la validation des champs
- réutilisation via héritage
- centralisation du comportement
- simplification des contrôleurs

---

## 📌 Limites actuelles

- validation très basique (uniquement présence)
- message d’erreur peu configurable
- concaténation du message avec `$Value` parfois inutile
- pas de typage métier (email, int, date…)

---

## 🚀 Améliorations possibles

- ajout de règles de validation avancées :
    - email
    - longueur min/max
    - regex
- internationalisation des messages (i18n)
- exceptions au lieu de booléens
- système de validation en chaîne (pipeline)
- séparation valeur / état de validation

---

## 📎 Exemple d’extension

```
class InputEmail extends Input{    public function checkValidity(): bool    {        if (!parent::checkValidity()) {            return false;        }        if (!filter_var($this->Value, FILTER_VALIDATE_EMAIL)) {            $this->validationMessage = "Email invalide";            return false;        }        return true;    }}
```