## Message d’erreur

```text
unmerged files because of an error.

detected dubious ownership in repository at '//STR-ELEVES/mboularbi$/Documents/F1-New'

"//STR-ELEVES/mboularbi$/Documents/F1-New' is owned by: 'S-1-5-21-1202660629-651377827-682003330-43862"

but the current user is: 'S-1-5-21-1202660629-651377827-682003330-48575'

To add an exception for this directory, call:

git config --global --add safe.directory '%(prefix)///STR-ELEVES/mboularbi$/Documents/F1-New'
```

## Origine de l'erreur

Git détecte que le dépôt :

```text
//STR-ELEVES/mboularbi$/Documents/F1-New
```

appartient à un utilisateur différent de l'utilisateur actuellement connecté.

- **Propriétaire du dépôt :**  
    `S-1-5-21-1202660629-651377827-682003330-43862`
- **Utilisateur courant :**  
    `S-1-5-21-1202660629-651377827-682003330-48575`

Git considère donc le dépôt comme potentiellement non sûr et bloque certaines opérations.

## Commande à l'origine de l'erreur

```bash
git pull
```

## Commande proposée par Git pour corriger le problème

Git fournit directement la commande permettant d'ajouter ce dépôt aux répertoires considérés comme sûrs :

```bash
git config --global --add safe.directory '//STR-ELEVES/mboularbi$/Documents/F1-New'
```
### Variante exacte affichée par Git

Git affiche également :

```bash
git config --global --add safe.directory '%(prefix)///STR-ELEVES/mboularbi$/Documents/F1-New'
```

La version avec le chemin UNC explicite est généralement plus lisible :

```bash
git config --global --add safe.directory '//STR-ELEVES/mboularbi$/Documents/F1-New'
```

## Vérifier la configuration

Pour vérifier que le répertoire a bien été ajouté :

```bash
git config --global --get-all safe.directory
```

Vous devriez retrouver :

```text
//STR-ELEVES/mboularbi$/Documents/F1-New
```

## Résumé

|Élément|Valeur|
|---|---|
|Dépôt|`//STR-ELEVES/mboularbi$/Documents/F1-New`|
|Propriétaire|`...-43862`|
|Utilisateur courant|`...-48575`|
|Problème|`detected dubious ownership`|
|Correction|Ajouter le dépôt à `safe.directory`|
|Commande|`git config --global --add safe.directory '//STR-ELEVES/mboularbi$/Documents/F1-N`|