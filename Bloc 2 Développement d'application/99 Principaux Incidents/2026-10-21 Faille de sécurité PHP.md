Deux failles critiques publiées récemment, elles sont identifiées comme `CVE-2026-109618` et `CVE-2026-109619`. 
Elles peuvent permettre une exécution de code à distance. Le CERT-FR recommande de mettre à jour PHP vers une version corrigée

mesure temporaire : désactiver la fonction Session Upload Progress de PHP.

il s'agirait d'une corruption mémoire liée à cette fonction de l'extension session, activée par défaut.

De nombreux site ont été placés en maintenance le 21 et 22 octobre 2026 pour permettre l'installation du correctif
Chez  certains hébergeurs les sites restent accessibles mais l'accès à la base de données n'est plus possible temporairement

Chez l'hébergeur Ionos :  les bases de données n'ont pas été accessibles pendant plusieurs jours  : Pour le moment aucune donnée n’est disponible. Veuillez réessayer plus tard.


## Comment appliquer le correctif dans WampServer

Étape 1 — Mettre à jour WampServer si nécessaire

Sur le [site officiel WampServer](https://wampserver.aviatechno.net/?utm_source=chatgpt.com) ,  (https://wampserver.aviatechno.net/) rubrique Updates, télécharge la mise à jour Wampserver 3.4.2 si elle n'est pas déjà installée.

Cette mise à jour du gestionnaire est nécessaire pour installer les derniers modules complémentaires PHP proposés sur le site. Elle ne met pas elle-même à jour la version PHP active.

Étape 2 — Installer la bonne version de PHP

Sur la même page, dans Addons → PHP, télécharger l'installateur correspondant à la branche utilisée, par exemple PHP 8.5.12 64 bit x64 si ton application utilise PHP 8.5.

Vérifier l'architecture et les prérequis indiqués par le site. Ne pas remplacer manuellement les fichiers d'un ancien dossier PHP.

Étape 3 — Activer la version corrigée

Après installation, cliquer (gauche) sur l'icône WampServer dans la zone de notification, puis ouvrir PHP → Version et sélectionne la nouvelle version.

Étape 4 — Vérifier que le correctif est actif

Redémarrer les services WampServer et vérifier la version réellement utilisée avec une page temporaire contenant :


```php
<?php
echo PHP_VERSION;
```

Le passage sur la version  8.5.12 amène à régler deux problèmes :

+ les fonctions de fermeture de fichier sont maintenant déclarés obsolètes et déclenchent une  warning : Concerne dans nos projet la classe InpulFile.php
+ Le débogage côté serveur depuis l'IDE PhpStorm n'est plus fonctionnel puisque le fichier php.ini de la version 8.5.12 n'est pas configuré.
+ 
Il faut refaire les modifications dans le fichier C:\wamp64\bin\apache\apache2.4.65\bin\php.ini

```
[xdebug]
zend_extension="c:/wamp64/bin/php/php8.5.12/zend_ext/php_xdebug-3.5.3-8.5-ts-vs17-x86_64.dll"
;xdebug.mode allowed are : off develop coverage debug gcstats profile trace
xdebug.output_dir="c:/wamp64/tmp"
xdebug.show_local_vars=0
xdebug.log="c:/wamp64/logs/xdebug.log"
;xdebug.log_level : 0 Criticals, 1 Connection, 3 Warnings, 5 Communication, 7 Information, 10 Debug	Breakpoint
xdebug.log_level=7
xdebug.profiler_output_name=trace.%H.%t.%p.cgrind
xdebug.use_compression=false
xdebug.mode=develop,debug
xdebug.start_with_request=yes
xdebug.client_host=127.0.0.1
xdebug.client_port=9003
```

Complément 

Source : https://www.usine-digitale.fr/cybersecurite/failles-de-securite-php-pourquoi-de-nombreux-sites-web-ont-ete-mis-hors-ligne-de-maniere-si-soudaine.ST3BSC5LW5GZBGDD2C37RKA5KQ.html?utm_source=newsletter&utm_medium=email&utm_campaign=info_ud-week-end&email=2647873&_emu=be6e60472056c6c1c86b1d8c23ebcdf0ec1978e1f8662449e8fbe2b354468435&src_email_send_date=2026-10-24&user_id_nl=2647873&idbdd=10589139&_ope=eyJndWlkIjoiYmU2ZTYwNDcyMDU2YzZjMWM4NmIxZDhjMjNlYmNkZjBlYzE5NzhlMWY4NjYyNDQ5ZThmYmUyYjM1NDQ2ODQzNSJ9

Sites officiels des préfectures, de la [Commission nationale de l’informatique et des libertés](https://www.usine-digitale.fr/cybersecurite/data-protection/cnil/) (Cnil), de la Banque de France… À l’heure où nous écrivons ces lignes, de nombreux sites web affichent un message d’erreur ou de maintenance. En cause, la détection de deux [vulnérabilités](https://www.usine-digitale.fr/cybersecurite/) critiques affectant le langage de programmation PHP, contraignant les administrateurs à mettre leurs sites temporairement hors ligne afin de se protéger des attaques et d’appliquer un correctif. Ce dernier a été publié par la PHP Foundation le 21 octobre à 16 h.

## Deux vulnérabilités d’exécution de code à distance…

Servant principalement à créer des pages web dynamiques et des applications côté serveur, le langage open source PHP alimente près de 70% des sites web dans le monde. Les systèmes de gestion de contenu (CMS) [WordPress](https://www.usine-digitale.fr/article/une-faille-de-securite-s-immisce-dans-un-theme-de-wordpress-plus-de-25-000-sites-exposes.N2208606), Drupal ou Joomla tournent notamment sur PHP.

Baptisées PHP2Shell, ces deux vulnérabilités se caractérisent en premier lieu par une échelle de sévérité particulièrement élevée : 9,2 sur 10 (le seuil de criticité est atteint à partir de 9). La première (CVE-2026-109618) peut théoriquement permettre à un acteur malveillant d’exécuter du code à distance lorsqu’une requête atteint la limite mémoire de PHP au mauvais moment. Des données internes restent partiellement initialisées et peuvent être traitées comme valides, provoquant une corruption de mémoire insuffisante (OOM).

La seconde (CVE-2026-109619) est liée à une erreur fatale qui pourrait laisser l’état interne pointant vers une zone mémoire ayant appartenu à une requête précédente. De fait, une requête ultérieure traitée par le même processus pourrait utiliser cette mémoire libérée et permettre à un cybercriminel d’exécuter du code à distance sur Apache avec le module mod_php.

## … découvertes à l’aide de l’intelligence artificielle

Ensuite, ces vulnérabilités ont été découvertes à l’aide de… l’intelligence artificielle. Le 11 août, la société de cybersécurité américaine Calif signale une première faille en utilisant [GPT-5.6 Sol](https://www.usine-digitale.fr/intelligence-artificielle/openai/gpt-56-sol-openai-met-en-avant-un-modele-taille-pour-les-taches-longues-agentiques-et-sensibles-sous-surveillance-du-gouvernement-americain.RERNHGURJFHQJCTNG7BL4GGM5I.html) d’OpenAI. La découverte a été faite dans le cadre de l’initiative Patch the Planet de l’entreprise de Sam Altman, visant notamment à débusquer les vulnérabilités présentes dans les logiciels libres. Wiz a signalé de son côté la seconde faille une dizaine de jours plus tard, en utilisant GPT-5.6 Sol mais aussi [Gemini 4 Argon de Google](https://www.usine-digitale.fr/big-tech/google/google-deploie-gemini-4-argon-a-grande-echelle-mais-le-prix-de-lapi-va-doubler-le-1er-decembre.OWFAMXUYN5DDVCHVLJM2RA4LIM.html). Selon Abstract Security, les versions affectées remontent à PHP 5.4, traduisant une présence du code vulnérable depuis plus d’une décennie.

_“La divulgation coordonnée de ces vulnérabilités a bien fonctionné jusqu’à deux jours avant la date de publication prévue”_, dénonce la PHP Foundation dans un communiqué publié le 21 octobre. Elle explique que la veille vers 17 h, les identifiants CVE ont commencé à circuler avec un score CVSS erroné de 10 sans détails techniques, la contraignant à avancer la publication du correctif au 21 octobre à 16 h, soit 24 heures en avance. _“Les distributeurs étaient encore en train de distribuer le correctif_, précise la PHP Foundation. _Une publication immédiate aurait empêché de nombreux utilisateurs de mettre à jour facilement.”_

Sans détails sur les composants affectés et sur le chemin d’attaque, cette fuite pourrait sembler anecdotique : elle a néanmoins suffi à provoquer un emballement mondial en quelques heures et à donner aux cybercriminels des raisons de “chercher” la faille. D’autant que dans les heures qui ont suivi, de nouvelles fuites ont été repérées.

## Des détails sur les vulnérabilités mis en ligne avant la publication des correctifs

Calif a ainsi publié par erreur une vidéo mardi soir sur un PoC fonctionnant contre plusieurs cibles avec des détails limités sur les vulnérabilités. Bien qu’elle ait été supprimée, des extraits ont circulé sur les réseaux sociaux. _“Le 21 octobre à 9 h 41, soit plus de six heures avant la publication coordonnée, Chainguard a unilatéralement enfreint l’embargo et publié des images compilées contenant le correctif_, charge aussi la PHP Foundation. _Un attaquant pouvait désormais étudier le correctif sous forme binaire (…)._ _Cette action a bafoué des semaines de travail en vue d’une divulgation coordonnée.”_

Chainguard, spécialisée dans la sécurisation de la supply chain, a nié toute divulgation précoce via le canal coordonné. _“Nous avons appris que quelque chose se passait dans PHP vendredi par l’un des chercheurs qui avait découvert le problème_, écrit sur LinkedIn Dan Lorenc, CEO de Chainguard. _Ce soir-là, nous avons pu découvrir indépendamment le même problème en utilisant notre harnais et un frontier model avec le prompt basique ‘Trouver de mauvaises vulnérabilités dans PHP’”_.

La PHP Foundation a aussi sa part de responsabilité : un audit de sécurité réalisé par [Quarkslab](https://www.usine-digitale.fr/cybersecurite/detecter-les-cyberattaques-en-temps-reel-airbus-accelere-dans-la-cybersecurite-en-semparant-de-la-start-up-francaise-quarkslab.65B6AGIV5BFPZCWCKZXORXYTHE.html) en 2024 a fait ressortir de premiers signes de la vulnérabilité à l’origine de la faille CVE-2026-109618. _“En 2024, une exploitation potentielle de cette vulnérabilité a été envisagée, mais aucun vecteur concret n’a été trouvé et d’autres problèmes plus importants ont éclipsé ces préoccupations”_, justifie-t-elle.

## Amazon et Cloudflare ont observé de nombreuses tentatives d’exploitation

Quelques heures avant la publication des correctifs, Amazon Threat Intelligence dit avoir observé plusieurs tentatives d’exploitation de PHP2Shell, avec des charges utiles et choix d’infrastructure d’anonymisation différents suggérant l’implication de plusieurs acteurs. _“Au cours des 10 heures qui ont précédé l’annonce, nous avons constaté sur l’ensemble du réseau des milliers de tentatives d’exploitation de cette faille_, a déclaré sur X Matthew Prince, CEO de [Cloudflare](https://www.usine-digitale.fr/start-up/financement/acquisition/cloudflare-rachete-deno-pour-simplifier-lhebergement-dutilisation-de-ses-workers-hors-de-son-infrastructure.5KFZ6GN4CFAYXNA7K6HPTPUIMQ.html). _Ce chiffre est passé à plusieurs dizaines de milliers dans les 45 minutes qui ont suivi l’annonce et s’élève désormais à plusieurs millions.”_ Pour l’heure, aucune compromission n’a été confirmée.

Il est donc recommandé de mettre à jour au plus vite les équipements et logiciels concernés. Le support n’est cependant plus assuré pour les versions de PHP antérieures à 8.2. Dans son alerte publiée le 21 octobre, le CERT-FR recommande donc d’isoler les équipements affectés puis de procéder à la migration vers une version maintenue et avec les derniers correctifs de sécurité.

Prudence, également, à appliquer les correctifs de manière exhaustive. _“Une entreprise peut corriger son application web principale et se sentir protégée tout en passant à côté d’une application éditeur ou d’une image conteneur livrée avec une version vulnérable de PHP_, appuie Matt Psencik, Threat Research Engineer chez [Tanium](https://www.usine-digitale.fr/editorial/tanium-obtient-une-certification-par-l-anssi-un-atout-strategique-sur-le-marche-francais.N2234660). (…) _Les équipes doivent identifier les binaires et versions de PHP réellement présents plutôt que de supposer qu’une mise à jour du système d’exploitation couvre tout.”_

https://www.it-connect.fr/php2shell-correctifs-urgence-cve-2026-109618-cve-2026-109619/
