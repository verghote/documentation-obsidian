

Markdown un **langage de balisage léger** (_lightweight markup language_). permettant d’écrire du texte **formaté** facilement, sans logiciel compliqué.
Il est utilisé pour :
- Les fichiers README (GitHub)
- La documentation
- Les prises de notes
- Les rapports simples

Avantage : Le texte reste **lisible même sans mise en forme**.

Un fichier Markdown est un simple fichier texte avec l'extension `.md`. Contrairement à un fichier `.docx` ou `.pages` qui nécessite un logiciel spécifique et payant, un fichier `.md` s'ouvrira toujours, sur n'importe quel ordinateur, smartphone ou système d'exploitation. 

C'est un format très utilisé pour la prise de notes à long terme, notamment dans les méthodes d'organisation des connaissances. On peut parler en quelque sorte d'un « second cerveau ».

Les outils d'IA générative (ChatGPT, Claude, Gemini, Mistral, etc.) répondent généralement en Markdown (titres, listes, tableaux, blocs de code...). Savoir lire et écrire le Markdown permet de mieux exploiter leurs réponses et de les copier-coller en conservant la mise en forme.
# Les titres

On utilise le symbole '#'

```
# Titre principal
## Sous-titre
### Sous-sous-titre
```

Plus il y a de `#`, plus le titre est de niveau inférieur.
# Mettre du texte en forme

```
*italique*
**gras**
~~barré~~
```

Résultat :

- _italique_
- **gras**
- ~~barré~~

# Faire des listes

## Liste simple

```
- Pomme
- Banane
- Orange
```
## Liste numérotée

```
1. Introduction
2. Développement
3. Conclusion
```
# Ajouter un lien externe

```
[Aller sur Google](https://www.google.com)
```
# Ajouter un lien vers une autre note (Obsidian)

Dans Obsidian, il est possible de créer un lien vers une autre note :

```
[[nom de la note]]
```
# Ajouter une image

```
![Description de l’image](image.png)
```
# Écrire du code

Il faut délimiter le bloc contenant le code par trois caractères `` ` `` (touche **AltGr + 7** sur un clavier AZERTY).

Il est possible, et même conseillé, de préciser le nom du langage (`php`, `bash`, `text`, etc.) après les trois caractères `` ` `` afin de bénéficier d'une coloration syntaxique.

```php
echo "Bonjour";
```

ou

```bash
git status
```
# Faire un tableau simple

```
| Nom  | Âge |
|------|----:|
| Paul | 20  |
| Léa  | 22  |
```
# Faire une citation

```
> Ceci est une citation.
```
# Liste de tâches

```
- [x] Exercice fait
- [ ] Exercice à faire
```
# Les séparateurs horizontaux

Très utiles dans la documentation.

```
---
```
# Récapitulatif

| Élément      | Syntaxe                              |
| ------------ | ------------------------------------ |
| Titre        | `# Titre`                            |
| Gras         | `**texte**`                          |
| Italique     | `*texte*`                            |
| Code         | `` `code` ``                         |
| Bloc de code | Trois `` ` `` puis le nom du langage |
| Lien         | `[texte](url)`                       |
| Image        | `![alt](image.png)`                  |
| Liste        | `- élément`                          |
| Citation     | `> citation`                         |
| Barré        | `~~texte~~`                          |
# Conclusion

Markdown n'est pas un langage « de développeur », mais un **format d'échange universel**.
Avec la pratique, sa syntaxe devient rapidement naturelle.
Vous serez amenés à utiliser ce langage pour :
- rédiger des `README.md` sur GitHub ;
- documenter les projets ;
- prendre des notes dans Obsidian ;
- réutiliser les réponses des IA génératives.
