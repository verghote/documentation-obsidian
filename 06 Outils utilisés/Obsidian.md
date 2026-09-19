## 1. Qu'est-ce qu'Obsidian ?

**Obsidian** est un logiciel de prise de notes basé sur des fichiers **Markdown (.md)** stockés localement sur votre ordinateur. Il permet de créer une base de connaissances personnelle (PKM – Personal Knowledge Management) grâce aux liens entre les notes.

### Points forts

- ✔ Les notes vous appartiennent (stockage local).
- ✔ Format Markdown simple et pérenne.
- ✔ Liens entre les notes (`[[Nom de la note]]`).
- ✔ Recherche très rapide.
- ✔ Nombreux plugins et thèmes.
- ✔ Fonctionne sur Windows, macOS, Linux, Android et iOS.

# 2. Vocabulaire

|Terme|Définition|
|---|---|
|Vault|Dossier contenant toutes vos notes|
|Note|Fichier Markdown (.md)|
|Markdown|Langage de mise en forme léger|
|Backlinks|Notes qui pointent vers une note|
|Tags|Mots-clés précédés de #|
|Graph View|Carte des liens entre les notes|
|Canvas|Tableau visuel pour organiser des idées|

# 3. Créer une note

Nouvelle note :  Ctrl + N  ou Cliquez sur **Nouvelle note**.

# 4. Markdown essentiel

## Titres

```markdown
# Titre 1

## Titre 2

### Titre 3
```

## Gras

```markdown
**texte**
```

→ **texte**

## Italique

```markdown
*texte*
```

→ _texte_

## Barré

```markdown
~~texte~~
```

→ ~~texte~~

## Liste

```markdown
- élément
- élément
    - sous élément
```

## Liste numérotée

```markdown
1. premier
2. deuxième
3. troisième
```

## Case à cocher

```markdown
- [ ] À faire
- [x] Terminé
```

## Citation

```markdown
> Une citation
```

## Code

Inline :

```markdown
`code`
```

Bloc :

````markdown
```python
print("Bonjour")
```
````

# 5. Les liens

## Vers une autre note

```markdown
[[Projet Maison]]
```

Si la note n'existe pas, Obsidian peut la créer.

## Vers un titre

```markdown
[[Projet Maison#Budget]]
```

## Vers un paragraphe

```markdown
[[Projet Maison^abc123]]
```

## Alias

```markdown
[[Projet Maison|Mon projet]]
```

Affiche :  Mon projet

# 6. Les tags

```markdown
#travail

#lecture

#formation
```

Ou plusieurs :

```markdown
#obsidian #markdown #notes
```

# 7. Les backlinks

Chaque note affiche automatiquement :

> Toutes les notes qui la citent.

Très utile pour naviguer dans votre base de connaissances.

# 8. Les propriétés (Frontmatter)

Au début d'une note :

```yaml
---
titre: Projet Maison
etat: En cours
date: 2026-07-23
tags:
  - maison
  - travaux
---
```

Permet d'organiser les notes avec des métadonnées.

# 9. Les recherches

- **Rechercher dans le fichier courant** : `Ctrl + F`
- **Remplacer dans le fichier courant** : `Ctrl + H`
- **Rechercher dans tous les fichiers du coffre** : `Ctrl + Shift + F`

Recherche simple :

```
budget
```

Recherche par tag :

```
tag:#lecture
```

Recherche par fichier :

```
file:Maison
```

Recherche par contenu :

```
content:python
```


# 10. Les liens externes

```markdown
[OpenAI](https://openai.com)
```

# 11. Les images

Depuis le disque :

```markdown
![[photo.png]]
```

ou

```markdown
![](photo.png)
```

# 12. Les tableaux

```markdown
| Nom | Âge |
|------|-----|
| Paul | 30 |
| Léa  | 28 |
```

# 13. Les callouts

```markdown
> [!NOTE]
> Une information.
```

Autres types :

- NOTE
- TIP
- WARNING
- IMPORTANT
- QUESTION
- SUCCESS
- FAILURE

Exemple :

```markdown
> [!WARNING]
> Sauvegarder avant modification.
```

# 14. Les tâches

```markdown
- [ ] Acheter du bois
- [ ] Appeler le plombier
- [x] Commander la peinture
```

Avec le plugin **Tasks**, on peut ajouter des dates, priorités et filtres.

# 15. Les modèles (Templates)

Exemple :

```markdown
# {{title}}

Date : {{date}}

## Objectif

## Notes

## Actions
```

# 16. Les raccourcis utiles

|Action|Raccourci|
|---|---|
|Nouvelle note|Ctrl + N|
|Rechercher|Ctrl + O|
|Recherche globale|Ctrl + Shift + F|
|Palette de commandes|Ctrl + P|
|Paramètres|Ctrl + ,|
|Basculer aperçu/édition|Ctrl + E|
|Ouvrir Graph|Ctrl + G (si configuré)|

# 17. Les plugins d'Obsidian

Les plugins permettent d'ajouter des fonctionnalités à Obsidian. Il en existe deux catégories :

- **Les plugins intégrés (Core Plugins)** : développés et maintenus par l'équipe d'Obsidian. Ils sont installés par défaut et il suffit de les activer.
- **Les plugins communautaires (Community Plugins)** : développés par la communauté. Ils doivent être installés avant de pouvoir être utilisés.

## Activer un plugin intégré

1. Ouvrez **Paramètres** (icône ⚙️ ou **Ctrl + ,**).
2. Sélectionnez **Plugins intégrés (Core Plugins)**.
3. Activez le plugin souhaité en basculant son interrupteur.

## Installer un plugin communautaire

1. Ouvrez **Paramètres**.
2. Cliquez sur **Plugins communautaires (Community Plugins)**.
3. Désactivez le **Mode restreint (Safe Mode)** si nécessaire.
4. Cliquez sur **Parcourir (Browse)**.
5. Recherchez le plugin.
6. Cliquez sur **Installer**, puis sur **Activer**.

## Les principaux plugins intégrés

### Daily Notes

Crée automatiquement une note quotidienne (journal) à chaque nouvelle journée. Idéal pour tenir un carnet de bord, prendre des notes de réunion ou organiser ses tâches.

### Templates

Permet d'insérer des modèles de notes prédéfinis afin de gagner du temps et de conserver une structure homogène.

### Canvas

Offre un espace de travail visuel où l'on peut disposer librement des notes, images, liens et fichiers pour réaliser des cartes mentales, des schémas ou organiser un projet.

### Backlinks

Affiche toutes les notes qui font référence à la note actuelle. Il facilite la navigation et met en évidence les relations entre les informations.

### Graph View

Représente graphiquement les liens entre les notes sous la forme d'un réseau interactif, permettant d'explorer facilement les connexions de votre base de connaissances.

### Outline

Affiche automatiquement le plan de la note à partir des titres (`#`, `##`, `###`). Très pratique pour naviguer rapidement dans les documents longs.

### Bookmarks

Permet d'enregistrer des notes, dossiers, recherches ou vues fréquemment utilisées afin d'y accéder rapidement.

### Workspaces

Enregistre la disposition des fenêtres, des panneaux et des notes ouvertes. On peut ainsi retrouver instantanément un environnement de travail adapté à une activité précise (rédaction, lecture, développement, etc.).

### Command Palette

Donne accès à toutes les commandes d'Obsidian grâce à une zone de recherche (raccourci **Ctrl + P**). C'est l'un des moyens les plus rapides d'utiliser toutes les fonctionnalités du logiciel.

# 18. Les principaux plugins communautaires

### Dataview

Transforme les notes en une véritable base de données. Il permet d'afficher automatiquement des listes, tableaux ou rapports à partir des propriétés (métadonnées) des notes.
### Tasks

Améliore considérablement la gestion des tâches : dates d'échéance, priorités, récurrence, filtres, tableaux de suivi et requêtes de recherche.
### Calendar

Ajoute un calendrier interactif permettant d'accéder rapidement aux notes quotidiennes ou périodiques.
### Periodic Notes

Complète le plugin _Daily Notes_ en créant automatiquement des notes hebdomadaires, mensuelles, trimestrielles ou annuelles.
### Excalidraw

Intègre un tableau blanc de dessin dans Obsidian pour réaliser des schémas, diagrammes, cartes mentales ou croquis reliés à vos notes.
### Kanban

Permet de gérer des projets sous forme de tableaux Kanban (colonnes « À faire », « En cours », « Terminé »), très utiles pour le suivi des tâches.
### Omnisearch

Remplace le moteur de recherche standard par une recherche plus rapide, plus complète et souvent plus pertinente, avec des résultats affichés en temps réel.
### Templater

Étend les possibilités du plugin _Templates_. Il permet de créer des modèles dynamiques intégrant des variables, des scripts JavaScript, des dates automatiques ou des contenus personnalisés.
### QuickAdd

Automatise la création de notes, l'ajout d'informations et l'exécution de commandes. Très utile pour créer des raccourcis et accélérer les tâches répétitives.
### Style Settings

Permet de personnaliser facilement l'apparence d'Obsidian (couleurs, polices, espacements, thèmes...) sans modifier directement les fichiers CSS. Il est souvent utilisé avec des thèmes communautaires.

# 19. Bonnes pratiques

- Une idée = une note.
- Donner des titres explicites.
- Créer des liens entre les notes.
- Utiliser des tags avec modération.
- Ajouter des métadonnées si nécessaire.
- Effectuer des sauvegardes régulières.
- Privilégier des notes courtes et reliées entre elles.

# 20. Flux de travail type

1. Créer une note.
2. Écrire en Markdown.
3. Ajouter des liens (`[[...]]`).
4. Ajouter quelques tags si besoin.
5. Classer ou laisser Obsidian retrouver la note par les liens et la recherche.
6. Réutiliser les informations dans d'autres notes.

# 21. Astuces

- Tapez `[[` pour rechercher une note existante.
- Utilisez `#` pour insérer un titre d'une autre note.
- Tapez `![[` pour intégrer une image ou une note.
- Ouvrez la **Palette de commandes** (`Ctrl + P`) pour accéder rapidement à toutes les fonctions.
- Utilisez le **Graph View** pour visualiser les connexions entre vos notes.

# Résumé express

|Fonction|Syntaxe|
|---|---|
|Titre|`# Titre`|
|Gras|`**texte**`|
|Italique|`*texte*`|
|Lien interne|`[[Note]]`|
|Alias|`[[Note\|Alias]]`|
|Image|`![[image.png]]`|
|Tag|`#tag`|
|Tâche|`- [ ]`|
|Citation|`>`|
|Code|`` `code` ``|
|Tableau|`\| Col1 \| Col2 \|`|

Ce mémento couvre les fonctionnalités essentielles d'Obsidian. Une fois ces bases maîtrisées, vous pourrez explorer des usages plus avancés comme les requêtes avec Dataview, l'automatisation avec Templater, les tableaux Kanban ou les cartes mentales avec Canvas.