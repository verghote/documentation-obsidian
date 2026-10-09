## Message d’erreur

Installing dependencies from lock file (including require-dev)  
Verifying lock file contents can be installed on current platform.  
Warning: The lock file is not up to date with the latest changes in composer.json. You may be getting outdated dependencies. It is recommended that you run `composer update` or `composer update <package name>`.  
- Required package "gumlet/php-image-resize" is not present in the lock file.  
- Required package "ezyang/htmlpurifier" is not present in the lock file.  
This usually happens when composer files are incorrectly merged or the composer.json file is manually edited.  
Read more about correctly resolving merge conflicts https://getcomposer.org/doc/articles/resolving-merge-conflicts.md  
and prefer using the "require" command over editing the composer.json file directly https://getcomposer.org/doc/03-cli.md#require-r

## Origine de l'erreur

Ce message indique que le fichier **`composer.json`** et le fichier **`composer.lock`** ne sont plus synchronisés. 
Les dépendances `gumlet/php-image-resize` et `ezyang/htmlpurifier` ont été ajoutées manuellement dans le fichier JSON, mais le fichier lock ne les a pas encore enregistrées.


## La solution :

synchroniser le fichier `.lock`

```batch
composer update --lock
```


## Bonnes pratiques

Pour éviter ce message il faut ajouter les dépendances via la ligne de commande et pas directement en modifiant le fichier `composer.json` à la main :


```bash 
composer require gumlet/php-image-resize ezyang/htmlpurifier
```

Cette commande met automatiquement à jour le `composer.json`, télécharge les paquets et régénère le `composer.lock` en une seule opération.