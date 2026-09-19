
## 1. Présentation

**GitHub CLI (`gh`)** est l'interface en ligne de commande officielle de GitHub.

Elle permet d'effectuer de nombreuses opérations GitHub directement depuis un terminal comme **Cmder**, PowerShell ou l'invite de commandes, sans avoir à utiliser systématiquement le navigateur.

Avec `gh`, il est notamment possible de :

- gérer des dépôts GitHub ;
- créer, cloner ou supprimer des dépôts ;
- consulter et gérer les issues ;
- gérer les Pull Requests ;
- automatiser certaines tâches GitHub ;
- s'authentifier auprès de GitHub ;
- gérer les autorisations accordées à GitHub CLI.

> **Important :** `gh` complète `git`, mais ne le remplace pas.
> 
> - `git` gère principalement le dépôt **local**, les commits, les branches et les échanges avec un dépôt distant.
>     
> - `gh` permet d'interagir avec les **services GitHub** : création de dépôts, issues, Pull Requests, authentification, etc.
>     

# 2. Vérifier l'installation

Avant de commencer, vérifier que Git et GitHub CLI sont installés.
### Vérifier Git

```bash
git --version
```

Exemple :

```text
git version 2.x.x
```
### Vérifier GitHub CLI

```bash
gh --version
```

Exemple :

```text
gh version 2.x.x
```

Si ces commandes fonctionnent, les outils sont correctement installés.

# 3. Authentification avec `gh`

## 3.1 Se connecter à GitHub

La commande principale est :

```bash
gh auth login
```

L'assistant demande plusieurs informations.

### Plateforme

Choisir : (Valider simplement le choix par défaut avec la touche Entrée)

```text
GitHub.com
```

### Méthode de connexion

Choisir :  (Valider simplement le choix par défaut avec la touche Entrée)

```text
HTTPS
```

### Méthode d'authentification

Choisir : (Valider simplement le choix par défaut avec la touche Entrée)

```text
Login with a web browser
```

---

## 3.2 Connexion avec le navigateur

Lors de la connexion, `gh` affiche un code temporaire similaire à :

```text
! First copy your one-time code: E3E8-1006

Press Enter to open https://github.com/login/device in your browser...
```

La procédure est alors :

1. Copier le code affiché.
2. Appuyer sur `Entrée`.
3. Le navigateur s'ouvre sur GitHub.
4. Se connecter avec son compte GitHub.
5. Saisir le code temporaire.
6. Autoriser **GitHub CLI**.

GitHub crée alors une autorisation permettant à `gh` d'utiliser le compte.

Sur Windows, les informations d'authentification utilisées par GitHub CLI peuvent être stockées dans le **Gestionnaire d'identifiants Windows** (_Credential Manager_).

On peut y accéder via :

**Panneau de configuration → Comptes d'utilisateurs → Gestionnaire d'identifiants**

## 3.3 Vérifier l'authentification

Pour vérifier que GitHub CLI est correctement connecté :

```bash
gh auth status
```

Cette commande indique notamment le compte GitHub actuellement utilisé.

## 3.4 Changer de compte

Pour se déconnecter :

```bash
gh auth logout
```

Puis se reconnecter :

```bash
gh auth login
```

Cette procédure est utile notamment lorsqu'un poste est utilisé successivement par plusieurs étudiants.

# 4. Les autorisations (_scopes_)

Lorsqu'un étudiant connecte `gh` à son compte GitHub, certaines autorisations sont accordées à GitHub CLI.

Ces autorisations sont appelées **scopes**.

Elles déterminent les opérations que l'application est autorisée à effectuer.

Quelques exemples :

|Scope|Rôle|
|---|---|
|`repo`|Accès aux dépôts concernés par l'autorisation|
|`workflow`|Gestion des workflows GitHub Actions|
|`read:org`|Lecture des informations d'organisations|
|`delete_repo`|Autorisation de supprimer des dépôts|

> **Principe de sécurité :** il est préférable de n'accorder que les autorisations nécessaires.

## 4.1 Autorisation supplémentaire : `delete_repo`

Par défaut, l'autorisation permettant de supprimer des dépôts (`delete_repo`) n'est pas nécessaire pour une utilisation courante.

Si elle est réellement nécessaire :

```bash
gh auth refresh --scopes delete_repo
```

GitHub CLI demandera alors une nouvelle autorisation via le navigateur.

> ⚠️ Le scope `delete_repo` est particulièrement sensible : il permet de supprimer des dépôts. Ne l'ajoutez que si vous en avez réellement besoin.

## 4.2 Modifier les autorisations

Les autorisations accordées à GitHub CLI peuvent être consultées dans les paramètres du compte GitHub :

[GitHub — Authorized OAuth Apps](https://github.com/settings/applications?utm_source=chatgpt.com)

L'authentification de GitHub CLI ne permet pas à l'utilisateur de définir directement une durée d'expiration personnalisée lors de `gh auth login`.

L'accès accordé à GitHub CLI peut cependant être **révoqué à tout moment** depuis les paramètres du compte GitHub, dans la section des applications autorisées.

Si une autorisation supplémentaire a été ajoutée et que l'on souhaite repartir d'une configuration propre, on peut :

```bash
gh auth logout
```

puis se reconnecter :

```bash
gh auth login
```

# 5. Authentification avec un token

Il est également possible d'utiliser un **Personal Access Token (PAT)**.

```bash
gh auth login --with-token
```

GitHub CLI demande alors de fournir le token.

> **Attention :** un token est une information secrète. Il ne doit jamais être publié dans un dépôt Git, un fichier source, une capture d'écran ou un message.

Pour les étudiants, la connexion via navigateur est généralement préférable car elle évite d'avoir à manipuler directement un token.

### Avantages de la connexion via navigateur

- pas besoin de copier manuellement un token ;
- procédure simple ;
- moins de risques de divulgation accidentelle du token ;
- autorisations gérées lors de la connexion ;
- possibilité de réautoriser facilement GitHub CLI.
    
# 6. Consulter ses dépôts GitHub

Pour afficher les dépôts du compte actuellement connecté :

```bash
gh repo list
```

Pour afficher les informations d'un dépôt précis :

```bash
gh repo view utilisateur/mon-projet
```

Par exemple :

```bash
gh repo view verghote/evaluation
```

Pour ouvrir le dépôt directement dans le navigateur :

```bash
gh repo view verghote/evaluation --web
```

# 7. Créer un dépôt GitHub

La commande principale est :

```bash
gh repo create
```

Elle peut être utilisée de manière interactive ou avec des options.

## 7.1 Création interactive

```bash
gh repo create
```

L'assistant demande notamment :

- le nom du dépôt ;
- sa visibilité ;
- l'ajout éventuel d'un README ;
- l'ajout éventuel d'un `.gitignore` ;
- l'ajout éventuel d'une licence.

Cette méthode est intéressante lorsque l'on découvre GitHub CLI.

## 7.2 Créer un dépôt privé

```bash
gh repo create mon-projet --private
```

## 7.3 Créer un dépôt public

```bash
gh repo create mon-projet --public
```

Par exemple :

```bash
gh repo create evaluation --public
```

Le dépôt sera alors créé sur le compte GitHub actuellement connecté.

# 8. Créer un dépôt à partir d'un projet local existant

C'est une situation **très fréquente lors des travaux pratiques**.
### Situation

Le professeur fournit un projet GitHub.
L'étudiant le clone sur son ordinateur :

```bash
git clone https://github.com/professeur/evaluation.git
```

Le projet local possède alors déjà un dépôt Git et un historique.

L'objectif est maintenant de créer :

```text
https://github.com/ETUDIANT/evaluation
```

et d'y envoyer le projet.

## 8.1 Vérifier le dépôt local

Se placer dans le dossier du projet :

```bash
cd chemin/vers/evaluation
```

Puis :

```bash
git status
```

Si Git reconnaît le dépôt, il n'est **pas nécessaire de faire `git init`**.

C'est important : le `git clone` effectué précédemment a déjà créé le dépôt Git local.

## 8.2 Vérifier le dépôt distant actuel

Avant de modifier quoi que ce soit :

```bash
git remote -v
```

On peut obtenir par exemple :

```text
origin  https://github.com/professeur/evaluation.git (fetch)
origin  https://github.com/professeur/evaluation.git (push)
```

Cela signifie que le dépôt local est actuellement relié au dépôt du professeur.

# 9. Méthode recommandée : `gh repo create --source=. --push`

Lorsque le dépôt local existe déjà, GitHub CLI permet de simplifier considérablement la procédure.

Depuis le dossier du projet :

```bash
gh repo create evaluation --public --source=. --push
```

Cette commande permet de :

1. créer le dépôt GitHub `evaluation` ;
2. le rendre public ;
3. utiliser le dépôt Git local actuel comme source ;
4. configurer le dépôt GitHub comme dépôt distant ;
5. envoyer les commits existants vers GitHub.

### Exemple

Si l'étudiant est connecté avec le compte :

```text
verghote
```

la commande :

```bash
gh repo create evaluation --public --source=. --push
```

créera :

```text
https://github.com/verghote/evaluation
```

# 10. Attention au dépôt `origin`

Avant d'utiliser `--push`, il est important de comprendre le rôle de `origin`.

Après un clone :

```bash
git clone https://github.com/professeur/evaluation.git
```

Git configure généralement :

```text
origin → dépôt du professeur
```

On peut vérifier avec :

```bash
git remote -v
```

### Situation souhaitée

Après la création de son propre dépôt, l'étudiant doit avoir :

```text
origin → son propre dépôt GitHub
```

et non :

```text
origin → dépôt du professeur
```

## 10.1 Si `origin` existe déjà

Selon la situation, GitHub CLI peut demander comment gérer le dépôt distant existant.

Pour éviter toute ambiguïté, il est possible de supprimer l'ancien `origin` avant de créer le nouveau dépôt :

```bash
git remote remove origin
```

Puis :

```bash
gh repo create evaluation --public --source=. --push
```

Enfin, vérifier :

```bash
git remote -v
```

On doit obtenir quelque chose ressemblant à :

```text
origin  https://github.com/verghote/evaluation.git (fetch)
origin  https://github.com/verghote/evaluation.git (push)
```

# 11. Risque de conflit lors du `git push`

Un `git push` peut être refusé si le dépôt distant contient déjà des commits que le dépôt local ne possède pas.

Par exemple :

```text
Dépôt local
A ─── B ─── C

Dépôt GitHub
A ─── B ─── D
```

Git ne peut pas simplement remplacer `D` par `C`.

Cela peut produire une erreur de type :

```text
! [rejected] main -> main (non-fast-forward)
```

## Cas habituel dans un nouveau dépôt

Si le dépôt GitHub vient juste d'être créé et qu'il est **vide**, il n'y a normalement pas de conflit d'historique.

C'est pourquoi, lorsqu'on utilise :

```bash
gh repo create evaluation --public --source=. --push
```

il est préférable de laisser GitHub CLI créer le dépôt sans lui demander de générer préalablement un README ou d'autres fichiers.

> ⚠️ Ne pas utiliser `git push --force` pour résoudre un problème de push sans avoir compris la cause. `--force` peut écraser l'historique distant.

# 12. Procédure complète : projet du professeur → dépôt étudiant

Voici la procédure recommandée pour un travail pratique.
### Étape 1 — Se placer dans le projet

```bash
cd chemin/vers/evaluation
```

### Étape 2 — Vérifier Git

```bash
git status
```

### Étape 3 — Vérifier le dépôt distant

```bash
git remote -v
```

### Étape 4 — Vérifier le compte GitHub utilisé par `gh`

```bash
gh auth status
```

### Étape 5 — Si nécessaire, supprimer l'ancien `origin`

Si `origin` pointe vers le dépôt du professeur :

```bash
git remote remove origin
```

### Étape 6 — Créer le dépôt étudiant et envoyer le projet

```bash
gh repo create evaluation --public --source=. --push
```

### Étape 7 — Vérifier le résultat

```bash
git remote -v
```

Puis :

```bash
gh repo view --web
```

Le navigateur doit afficher le dépôt personnel de l'étudiant.

# 13. Différence entre `git` et `gh`

Il est important de ne pas confondre les deux outils.

|Commande|Rôle|
|---|---|
|`git clone`|Récupérer un dépôt Git|
|`git status`|Consulter l'état du dépôt local|
|`git add`|Préparer des fichiers pour un commit|
|`git commit`|Créer un commit|
|`git push`|Envoyer des commits vers un dépôt distant|
|`git pull`|Récupérer et intégrer des modifications|
|`git remote`|Gérer les dépôts distants|
|`gh auth`|Gérer l'authentification GitHub CLI|
|`gh repo`|Gérer les dépôts GitHub|
|`gh issue`|Gérer les issues|
|`gh pr`|Gérer les Pull Requests|

On peut donc utiliser les deux outils dans un même projet.

Exemple :

```bash
git add .
git commit -m "Ajout de la fonctionnalité X"
git push
```

et utiliser `gh` pour gérer le dépôt GitHub :

```bash
gh repo view --web
```

---

# 14. Supprimer un dépôt GitHub

La commande :

```bash
gh repo delete
```

permet de supprimer un dépôt GitHub.

> ⚠️ **Attention : la suppression d'un dépôt est une opération destructive.**

Pour supprimer un dépôt précis :

```bash
gh repo delete utilisateur/mon-projet
```

GitHub CLI demande normalement une confirmation.

Pour supprimer sans confirmation :

```bash
gh repo delete utilisateur/mon-projet --yes
```

### Exemple

```bash
gh repo delete verghote/evaluation
```

ou :

```bash
gh repo delete verghote/evaluation --yes
```

> **Attention :** `--yes` supprime la confirmation interactive. Cette option doit être utilisée avec beaucoup de prudence.


# 15. Commandes essentielles à retenir

Pour les travaux pratiques, les commandes les plus importantes sont :

### Authentification

```bash
gh auth login
gh auth status
gh auth logout
```

### Dépôts

```bash
gh repo list
gh repo view utilisateur/mon-projet
gh repo create mon-projet --public
```

### Créer un dépôt à partir du projet local

```bash
gh repo create mon-projet --public --source=. --push
```

### Git local

```bash
git status
git add .
git commit -m "Message du commit"
git push
git pull
git remote -v
```

# 16. Exemple complet

Un étudiant récupère le projet du professeur :

```bash
git clone https://github.com/professeur/evaluation.git
```

Il entre dans le projet :

```bash
cd evaluation
```

Il vérifie son état :

```bash
git status
```

Il vérifie son compte GitHub :

```bash
gh auth status
```

Il vérifie le dépôt distant :

```bash
git remote -v
```

Si `origin` correspond au dépôt du professeur :

```bash
git remote remove origin
```

Il crée alors son propre dépôt public et y pousse le projet :

```bash
gh repo create evaluation --public --source=. --push
```

Il vérifie finalement le dépôt distant :

```bash
git remote -v
```

et peut ouvrir le dépôt GitHub :

```bash
gh repo view --web
```

### Résultat

Le projet qui était initialement :

```text
GitHub du professeur
        ↓
     git clone
        ↓
   Projet local
        ↓
gh repo create --source=. --push
        ↓
GitHub de l'étudiant
```

L'étudiant dispose alors de son **propre dépôt GitHub**, contenant l'historique Git du projet local.