## 1. Présentation

**Dompdf** est une bibliothèque PHP permettant de générer des fichiers PDF à partir de contenu HTML et CSS.

Documentation officielle : https://github.com/dompdf/dompdf

## 2. Installation

Installer la bibliothèque avec Composer :

```
composer require dompdf/dompdf
```

La dépendance utilisée dans le projet est :

```
"dompdf/dompdf": "^3.1.6"
```

La version effectivement installée est déterminée par le fichier `composer.lock`.

## 3. Principe de fonctionnement

La génération d'un PDF suit généralement quatre étapes :

1. Instancier le moteur Dompdf.
2. Fournir le contenu HTML et éventuellement les styles CSS.
3. Générer le document avec `render()`.
4. Afficher le PDF dans le navigateur ou le télécharger.

### Exemple d'utilisation

```
<?php

declare(strict_types=1);

use Dompdf\Dompdf;
use Dompdf\Options;

$options = new Options();
$options->set('isRemoteEnabled', false);

$pdf = new Dompdf($options);

$html = <<<HTML
<html>
<head>
    <style>
        body {
            font-family: Helvetica, sans-serif;
            font-size: 12px;
        }

        h1 {
            text-align: center;
        }
    </style>
</head>
<body>
    <h1>Document administratif</h1>
    <p>Contenu du document.</p>
</body>
</html>
HTML;

$pdf->loadHtml($html, 'UTF-8');
$pdf->setPaper('A4', 'portrait');
$pdf->render();

$pdf->stream('document.pdf', [
    'Attachment' => false,
]);
```

Le paramètre `Attachment` détermine le comportement du navigateur :

- `false` : affichage du PDF dans le navigateur, si possible.
- `true` : téléchargement du PDF.

## 4. Gestion du HTML et du CSS

Dompdf accepte le HTML et une partie du CSS. Il ne reproduit toutefois pas complètement le comportement d'un navigateur moderne.

Les développeurs doivent privilégier :

- les mises en page simples ;
- les tableaux HTML pour les données tabulaires ;
- les styles CSS classiques ;
- les unités adaptées à l'impression ;
- les sauts de page explicites lorsque cela est nécessaire.

Les fonctionnalités CSS modernes, notamment Flexbox et CSS Grid, peuvent présenter des limitations. Un template qui fonctionne dans un navigateur ne garantit donc pas un rendu identique en PDF.

## 5. Images et ressources

Les images locales doivent être accessibles depuis le serveur et référencées avec un chemin valide.

Par défaut, il est recommandé de conserver `isRemoteEnabled` désactivé. Si le chargement de ressources distantes est nécessaire, il doit être activé uniquement après analyse des risques et restriction des sources autorisées.

Les chemins de fichiers doivent être construits de manière fiable à partir de répertoires connus de l'application.

## 6. Bonnes pratiques de développement

- Centraliser les templates PDF lorsqu'ils sont réutilisés.
- Séparer les données métier de la présentation HTML.
- Échapper les données textuelles dynamiques avec `htmlspecialchars()` avant leur insertion dans le HTML.
- Ne jamais injecter directement du HTML fourni par un utilisateur sans validation.
- Tester le rendu avec des données longues, des accents, des tableaux volumineux et plusieurs pages.
- Vérifier le comportement des sauts de page et des images.

Exemple d'échappement d'une valeur textuelle :

```
$nom = htmlspecialchars(
    $nom,
    ENT_QUOTES | ENT_SUBSTITUTE,
    'UTF-8'
);
```

## 7. Quand utiliser Dompdf ?

Dompdf convient aux documents simples, lorsque le HTML/CSS existant est suffisamment classique et que les besoins de pagination restent limités.

Pour les tableaux multipages, les en-têtes et pieds de page récurrents, ainsi que les documents administratifs nécessitant une présentation homogène, l'application Inscription privilégie la classe commune `ClasseMetier\Pdf` basée sur mPDF, décrite dans `MPDF.md`.