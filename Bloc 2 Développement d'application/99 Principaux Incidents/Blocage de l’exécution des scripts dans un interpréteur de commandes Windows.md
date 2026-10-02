## 1. Contexte

Un utilisateur disposant de **droits d’administration** sur son poste Windows rencontrait un problème lors de l’exécution de scripts depuis un interpréteur de commandes.

Malgré ses privilèges administrateur, l'utilisateur ne pouvait pas lancer correctement des scripts à l'aide de l'interpréteur de commandes Windows (`cmd.exe`).

> **Point important :** les droits d'administration de l'utilisateur n'étaient pas en cause. Le blocage provenait d'une **stratégie Windows appliquée au niveau de l'utilisateur**.

## 2. Symptôme

Lorsqu'une nouvelle fenêtre était ouverte depuis Cmder, le message suivant apparaissait :

> **« L’administrateur a désactivé l’invite de commandes. »**

Ce message indiquait que l'utilisation de l'interpréteur de commandes Windows était interdite par une stratégie système.

Le problème empêchait notamment :

- l'ouverture normale d'un terminal `cmd.exe` depuis Cmder ;
- l'exécution de commandes nécessitant `cmd.exe` ;
- l'exécution de scripts `.cmd` et `.bat` ;
- l'utilisation de certaines fonctionnalités de Cmder reposant sur l'interpréteur de commandes Windows.
## 3. Analyse

En questionnant l'IA la raison est expliquée et une solution est proposée :

Ce message indique que l'invite de commandes (cmd.exe) a été bloquée dans le Registre Windows. Comme **Cmder** s'appuie sur `cmd.exe` par défaut pour démarrer, il se retrouve bloqué lui aussi.

Pour débloquer la situation, il faut réactiver l'invite de commandes.

### Solution 1 : Via l'Éditeur du Registre (Recommandé)

**1.Ouvrir l'éditeur du Registre :**Prerequisite.

Appuie sur les touches **Windows + R**, tape `regedit` et appuie sur **Entrée**. Si une fenêtre demande une autorisation d'administrateur, valide.

**2.Accéder à la clé System :**

Dans la barre d'adresse en haut, colle ce chemin :

`HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\System`

**3.Modifier la valeur DisableCMD :**

Dans le panneau de droite, cherche la valeur nomme **DisableCMD**.

- Double-clique dessus et change la donnée de la valeur de **1** à **0** (ou supprime complètement la valeur **DisableCMD**).
    

**4.Vérifier la modification :**

Relance Cmder ou ouvre une fenêtre `cmd.exe` classique pour vérifier que le message n'apparaît plus.

### Solution 2 : Via l'Éditeur de stratégie de groupe (Windows Pro / Enterprise)

Si tu utilises Windows Pro, Éducation ou Enterprise, tu peux passer par l'outil de stratégie de groupe :

1. Appuie sur **Windows + R**, tape `gpedit.msc` et valide.
    
2. Navigue dans l'arborescence : **Configuration utilisateur** > **Modèles d'administration** > **Système**.
    
3. Dans le panneau de droite, double-clique sur **Empêcher l'accès à l'invite de commandes**.
    
4. Sélectionne **Non configuré** ou **Désactivé**, puis clique sur **Appliquer** et **OK**.
    

Informations importantes

Si ton PC appartient à une entreprise, une école ou une organisation, cette restriction a peut-être été appliquée par l'administrateur réseau. Dans ce cas, la modification sera annulée automatiquement au redémarrage ou à la prochaine mise à jour des stratégies réseau.

```text
HKEY_CURRENT_USER\SOFTWARE\Policies\Microsoft\Windows\System
```

avec la valeur :

```text
DisableCMD
```

Cette valeur contrôle la désactivation de l'invite de commandes Windows pour l'utilisateur concerné.

Dans le cas rencontré, la valeur empêchait l'utilisation de l'interpréteur de commandes malgré les droits administrateur de l'utilisateur.

### Configuration problématique

La configuration de la clé de registre était incompatible avec l'utilisation de `cmd.exe`.

La présence de cette stratégie au niveau :

```text
HKEY_CURRENT_USER
```

signifie qu'elle s'applique au **profil de l'utilisateur courant**.

## 4. Solution apportée

Pour rétablir l'utilisation de l'interpréteur de commandes, la valeur `DisableCMD` a été configurée à `0`.

Le fichier `.reg` utilisé pour appliquer la correction était le suivant :

```reg
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\SOFTWARE\Policies\Microsoft\Windows\System]
"DisableCMD"=dword:00000000
```

### Interprétation

La valeur :

```text
DisableCMD = 0
```

permet de réactiver l'utilisation de l'invite de commandes.

Après application de cette modification, l'utilisateur a pu de nouveau utiliser l'interpréteur de commandes et lancer ses scripts.

## 5. Procédure de correction

### Méthode 1 — Import du fichier `.reg`

1. Créer un fichier, par exemple :
    
    ```text
    correction_disablecmd.reg
    ```
    
2. Y ajouter le contenu suivant :
    
    ```reg
    Windows Registry Editor Version 5.00
    
    [HKEY_CURRENT_USER\SOFTWARE\Policies\Microsoft\Windows\System]
    "DisableCMD"=dword:00000000
    ```
    
3. Exécuter le fichier `.reg` avec le **compte utilisateur concerné**.
    
4. Valider l'importation dans le Registre Windows.
    
5. Fermer puis rouvrir l'interpréteur de commandes, ou ouvrir une nouvelle session Windows si nécessaire.
    
6. Tester l'exécution d'un script `.bat` ou `.cmd`.
    
## 6. Vérification

La présence et la valeur de la clé peuvent être vérifiées avec `regedit.exe` :

```text
HKEY_CURRENT_USER
└── SOFTWARE
    └── Policies
        └── Microsoft
            └── Windows
                └── System
                    └── DisableCMD = 0
```

Il est également possible de vérifier la valeur depuis une invite de commandes avec :

```cmd
reg query "HKCU\SOFTWARE\Policies\Microsoft\Windows\System" /v DisableCMD
```

La configuration attendue est :

```text
DisableCMD    REG_DWORD    0x0
```

## 7. Points d'attention

Avant d'appliquer cette correction sur d'autres postes, il convient de vérifier l'origine de la valeur `DisableCMD`.

Si celle-ci est définie par une **GPO (Group Policy)**, une solution de gestion centralisée ou un outil de sécurité, une modification locale du registre peut être temporaire : la stratégie peut réappliquer la valeur lors du prochain rafraîchissement des stratégies.

Dans ce cas, il est préférable d'identifier et de corriger la stratégie à l'origine du paramétrage plutôt que de modifier individuellement le registre sur chaque poste.

## 8. Résumé de l'incident

|Élément|Description|
|---|---|
|**Problème**|Impossible pour un utilisateur administrateur de lancer des scripts via l'interpréteur de commandes|
|**Composant concerné**|`cmd.exe` / scripts `.cmd` et `.bat`|
|**Cause identifiée**|Stratégie `DisableCMD` appliquée au niveau utilisateur|
|**Clé concernée**|`HKCU\SOFTWARE\Policies\Microsoft\Windows\System`|
|**Valeur problématique**|`DisableCMD`|
|**Correction**|Définition de `DisableCMD` à `0`|
|**Résultat**|Réactivation de l'utilisation de l'interpréteur de commandes et exécution des scripts|


L'incident n'était pas lié aux privilèges administrateur du compte mais à une **restriction configurée dans le Registre Windows au niveau utilisateur**.

La modification de :

```text
HKCU\SOFTWARE\Policies\Microsoft\Windows\System\DisableCMD
```

avec la valeur :

```text
0
```

a permis de rétablir le fonctionnement normal de l'interpréteur de commandes.

Pour éviter la réapparition du problème, il est recommandé d'identifier la source de cette stratégie (`GPO`, configuration de sécurité ou autre mécanisme de gestion) et de la corriger à la source lorsque cela est possible.