## 1. Présentation

**mPDF** est une bibliothèque PHP permettant de générer des PDF à partir de HTML et de CSS.

Dans l'application Inscription, elle est utilisée pour produire des documents administratifs dont la présentation doit être homogène, notamment :

- les lettres ;
- les tableaux et listes de données ;
- les documents multipages ;
- les documents comportant un logo, des en-têtes, des pieds de page et une pagination.

Documentation officielle : https://mpdf.github.io/

## 2. Installation

Installer la bibliothèque avec Composer :

```
composer require mpdf/mpdf
```

La dépendance utilisée dans le projet est :

```
"mpdf/mpdf": "^8.2"
```

La version effectivement installée est déterminée par le fichier `composer.lock`.

## 3. Convention d'utilisation dans Inscription

Pour uniformiser la présentation des PDF, l'application dispose d'une classe commune :

```
ClasseMetier\Pdf
```

Cette classe centralise la mise en forme et la génération des documents. Les développeurs doivent privilégier ses méthodes plutôt que de recréer individuellement les en-têtes, les pieds de page, les marges ou les styles de tableaux.

Deux méthodes sont disponibles :

|Méthode|Utilisation|
|---|---|
|`Pdf::genererLettre()`|Générer une lettre avec l'en-tête de l'établissement, les coordonnées du destinataire et la signature du directeur.|
|`Pdf::genererTableau()`|Générer un tableau ou une liste avec un logo, un titre et une pagination automatique.|

La classe est définie dans le namespace `ClasseMetier`.

## 4. Utilisation de la classe commune

### 4.1. Générer une lettre

```php
use ClasseMetier\Pdf;

$corps = <<<'HTML'
<div class="objet">Objet : Inscription</div>

<div class="corps">
    <p>Madame, Monsieur,</p>
    <p>Votre dossier d'inscription a été pris en compte.</p>
</div>
HTML;

$destinataire = <<<'HTML'
Madame Dupont<br>
10 rue des Écoles<br>
80000 Amiens
HTML;

Pdf::genererLettre(
    $corps,
    $destinataire,
    'lettre-inscription.pdf',
    false
);
```

Paramètres :

- `$corps` : contenu HTML de la lettre.
- `$destinataire` : coordonnées du destinataire, au format HTML.
- `$nomFichier` : nom du fichier PDF, facultatif ; valeur par défaut `lettre.pdf`.
- `$telecharger` : `false` pour afficher le PDF dans le navigateur, `true` pour déclencher son téléchargement.

La classe ajoute automatiquement les éléments communs aux lettres : logo, coordonnées de l'établissement, formule de politesse et signature.

Le contenu fourni dans `$corps` doit contenir uniquement la partie spécifique à la lettre. Il ne doit pas recréer les éléments déjà ajoutés par la classe.

### 4.2. Générer un tableau

```
use ClasseMetier\Pdf;

$corps = <<<'HTML'
<table>
    <thead>
        <tr>
            <th>Nom</th>
            <th>Prénom</th>
            <th>Formation</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Dupont</td>
            <td>Marie</td>
            <td>BTS</td>
        </tr>
        <tr>
            <td>Martin</td>
            <td>Paul</td>
            <td>BAC PRO</td>
        </tr>
    </tbody>
</table>
HTML;

Pdf::genererTableau(
    $corps,
    'Liste des inscrits',
    'liste-inscrits.pdf',
    false
);
```

Paramètres :

- `$corps` : tableau HTML ou liste HTML à afficher.
- `$titre` : titre du document, affiché dans l'en-tête.
- `$nomFichier` : nom du fichier PDF, facultatif ; valeur par défaut `document.pdf`.
- `$telecharger` : `false` pour afficher le PDF dans le navigateur, `true` pour le télécharger.

La classe ajoute automatiquement le logo, le titre dans l'en-tête de chaque page et la pagination au format « Page X / Y ».

Pour que la présentation reste cohérente, il faut fournir uniquement le contenu du tableau ou de la liste, sans recréer l'en-tête ni le pied de page.

## 5. Classe commune `ClasseMetier\Pdf`

Le code suivant constitue l'implémentation de référence fournie pour l'application Inscription.

Fichier recommandé : `src/ClasseMetier/Pdf.php`, ou le répertoire correspondant à l'organisation actuelle du projet et à son autoload Composer.

```php
<?php

declare(strict_types=1);

namespace ClasseMetier;

use Mpdf\HTMLParserMode;
use Mpdf\Mpdf;
use Mpdf\Output\Destination;

class Pdf
{
    /**
     * Génère une lettre PDF avec entête + coordonnées + signature
     */
    public static function genererLettre(
        string $corps,
        string $destinataire,
        string $nomFichier = 'lettre.pdf',
        bool $telecharger = false
    ): void {
        $mpdf = new Mpdf([
            'margin_top' => 15,
            'margin_bottom' => 20,
            'margin_left' => 25,
            'margin_right' => 25,
        ]);

        // Partie commune à toutes les lettres
        $html = <<<HTML
<html>
<head>
    <style>
        body        { font-family: helvetica, sans-serif; font-size: 12pt; }
        .coordonnees, .objet, .corps, .signature { margin-bottom: 15px; }
        .coordonnees, .signature { margin-top: 20px; margin-left: 10cm; }
        .objet       { font-weight: bold; margin-top: 40px; }
        .corps       { margin-top: 20px; }
        p { text-indent: 2em; text-align: justify; line-height: 35px; margin: 10px 0; }
        .footer      { margin-top: 30px; font-size: 10pt; }
    </style>
</head>
<body>

    <!-- Entête avec logo et coordonnées de l’établissement -->
    <img src="/data/logo.png" style="width: 100px;" alt="Logo"><br>

    <div>
        Lycée Sacré Cœur<br>
        2, rue des Augustins<br>
        BP 40178<br>
        80007 Amiens cedex 1
    </div>

    <!-- Coordonnées du destinataire -->
    <div class="coordonnees">
    $destinataire
    </div>
HTML;

        // Ajout du corps de la lettre
        $html .= $corps;

        // Pied : signature du directeur
        $pied = <<<HTML
            <p>
            Veuillez croire, Madame, Monsieur, en l’expression de nos salutations distinguées.
            </p>
        </div>
        <div class="signature" style="margin-top: 20px;">
                <p>
                    Le Directeur<br><br>
                    <img src="/data/signature.png" style="width: 150px;" alt="Signature">
                </p>
        </div>
HTML;

        // Ajout du pied
        $html .= $pied;

        // Génération du document avec sécurisation du nom du fichier
        $mpdf->WriteHTML($html);
        $nomFichierSecurise = rawurlencode($nomFichier);

        $mpdf->Output(
            $nomFichierSecurise,
            $telecharger
                ? Destination::DOWNLOAD
                : Destination::INLINE
        );
    }

    /**
     * Génère un tableau ou une liste PDF avec entête (logo + titre)
     * et pied (pagination)
     */
    public static function genererTableau(
        string $corps,
        string $titre,
        string $nomFichier = 'document.pdf',
        bool $telecharger = false
    ): void {
        $mpdf = new Mpdf([
            'margin_top' => 40,
            'margin_bottom' => 20,
            'margin_left' => 15,
            'margin_right' => 15,
        ]);

        // Chemin absolu du logo
        $cheminLogo = $_SERVER['DOCUMENT_ROOT'] . '/data/logo.png';

        // 1. Styles CSS spécifiques aux tableaux
        $styles = <<<CSS
        <style>
            body {
                font-family: helvetica, sans-serif;
                font-size: 11pt;
            }
            table {
                width: 100%;
                border-collapse: collapse;
                font-size: 10pt;
                margin-top: 10px;
            }
            th, td {
                border: 1px solid #000;
                padding: 6px 4px;
                text-align: left;
            }
            th {
                background-color: #dde;
            }
            td {
                vertical-align: top;
            }
            .center {
                text-align: center;
            }
        </style>
CSS;

        $mpdf->WriteHTML($styles, HTMLParserMode::HEADER_CSS);

        // 2. En-tête : logo + titre (affiché sur chaque page)
        $entete = <<<HTML
    <table style="width: 100%; border-collapse: collapse; border-bottom: 1px solid #aaa;">
        <tr>
            <td style="width: 20%; vertical-align: middle; border: none;">
                <img src="{$cheminLogo}" style="max-height: 70px;" alt="Logo">
            </td>
            <td style="text-align: left; font-size: 14pt; vertical-align: middle; border: none;">
                <strong>{$titre}</strong>
            </td>
        </tr>
    </table>
HTML;

        $mpdf->SetHTMLHeader($entete);

        // 3. Pied : pagination automatique
        $pied = <<<HTML
        <div style="text-align: right; font-size: 10pt; border-top: 1px solid #aaa; padding-top: 5px;">
            Page {PAGENO} / {nb}
        </div>
HTML;

        $mpdf->SetHTMLFooter($pied);

        // 4. Contenu principal (tableau ou liste)
        $mpdf->WriteHTML($corps);

        // 5. Génération du fichier avec nom sécurisé
        $nomFichierSecurise = rawurlencode($nomFichier);

        $mpdf->Output(
            $nomFichierSecurise,
            $telecharger
                ? Destination::DOWNLOAD
                : Destination::INLINE
        );
    }
}
```

## 6. Règles à respecter

### Contenu HTML et sécurité

Les paramètres `$corps`, `$destinataire` et `$titre` sont injectés dans du HTML.

- Pour les données textuelles provenant de la base de données ou d'un utilisateur, utiliser `htmlspecialchars()` avec l'encodage UTF-8.
- Ne conserver du HTML brut que lorsque cela est nécessaire et que sa structure a été maîtrisée ou assainie.
- Dans `$titre`, échapper le texte avant de l'insérer dans le HTML de l'en-tête.
- Ne pas considérer `rawurlencode()` sur le nom du fichier comme une protection du contenu HTML.

### Chemins des images

Les fichiers suivants sont utilisés par la classe :

- `/data/logo.png` pour le logo des lettres ;
- `/data/signature.png` pour la signature ;
- `$_SERVER['DOCUMENT_ROOT'] . '/data/logo.png'` pour le logo des tableaux.

Vérifier que ces chemins correspondent bien à l'installation de l'application et que le processus PHP peut lire les fichiers. Le chemin `/data/logo.png` utilisé dans les lettres doit être vérifié avec la configuration des ressources de mPDF ; un chemin absolu sur le système de fichiers est généralement plus fiable.

### Présentation et pagination

- Conserver les styles communs dans la classe `Pdf`.
- Adapter le HTML transmis aux contraintes de mise en page de mPDF.
- Utiliser les balises `<thead>`, `<tbody>`, `<th>` et `<td>` pour les tableaux.
- Tester les documents contenant suffisamment de lignes pour provoquer plusieurs pages.
- Vérifier les marges, les retours à la ligne, les images et les caractères accentués.

### Cohérence du rendu

Toute évolution des styles communs doit être testée sur les deux types de documents : lettres et tableaux. Cela évite que des corrections destinées à un type de PDF dégradent l'autre.

## 7. Points techniques à vérifier dans l'implémentation fournie

Avant de considérer cette classe comme définitive, les développeurs doivent notamment vérifier les points suivants :

1. **Structure HTML des lettres :** le pied ajoute une balise `</div>` sans ouverture correspondante dans la structure initiale présentée. Il est préférable de corriger cette structure pour obtenir un HTML valide.
2. **Chemins des ressources :** uniformiser la résolution des chemins du logo et de la signature, idéalement avec des chemins absolus du système de fichiers.
3. **Encodage des valeurs dynamiques :** échapper les valeurs textuelles et assainir les contenus HTML lorsque nécessaire.
4. **Dépendances et environnement :** vérifier la compatibilité de la version installée de mPDF avec la version PHP du projet et prévoir un répertoire temporaire accessible en écriture si nécessaire.
5. **Nom de fichier :** valider le nom de fichier selon les conventions de l'application et tester son comportement avec des espaces et des caractères accentués.

## 8. Choix entre mPDF et Dompdf

Les deux bibliothèques PHP, `dompdf/dompdf` et `mpdf/mpdf`, permettent de générer des fichiers PDF à partir de HTML et de CSS. Elles remplissent le même objectif, mais leur moteur de rendu, leur compatibilité CSS et leur gestion des documents diffèrent.
### Comparaison rapide

| Critère                            | `dompdf/dompdf` `^3.1.6`         | `mpdf/mpdf` `^8.2`                                        |
| ---------------------------------- | -------------------------------- | --------------------------------------------------------- |
| Langage                            | PHP                              | PHP                                                       |
| Entrée principale                  | HTML + CSS                       | HTML + CSS                                                |
| Moteur de rendu                    | DOM et moteur de rendu propre    | Moteur de rendu propre à mPDF                             |
| Compatibilité CSS                  | Partielle, avec des limitations  | Partielle, avec des limitations                           |
| Flexbox / Grid                     | Support limité                   | Support limité, notamment pour les mises en page modernes |
| Tableaux HTML                      | Bon support                      | Très bon support, notamment pour les documents multipages |
| En-têtes et pieds de page          | Possibles                        | Très adaptés aux documents paginés                        |
| Polices personnalisées             | Oui                              | Oui                                                       |
| Langues RTL (arabe, hébreu)        | Support limité selon les besoins | Point fort de mPDF                                        |
| Génération de factures et rapports | Adapté                           | Très adapté                                               |
| Installation                       | Via Composer                     | Via Composer                                              |

### Les principales différences

### `dompdf/dompdf` : simple et léger à utiliser

Documentation : [Dompdf sur GitHub](https://github.com/dompdf/dompdf) 

Dompdf est intéressant lorsque tu disposes déjà de templates HTML/CSS et que tu souhaites les convertir en PDF sans trop de complexité.

- Facile à intégrer dans une application PHP ou Symfony.
- Adapté aux factures simples, attestations et documents HTML classiques.
- Son moteur ne prend pas en charge toutes les fonctionnalités CSS modernes : il ne faut pas s'attendre à un rendu identique à Chrome.
- Il peut nécessiter des ajustements pour les tableaux complexes, les sauts de page et les mises en page élaborées.

### `mpdf/mpdf` : davantage orienté documents imprimables

Documentation : [Manuel officiel mPDF](https://mpdf.github.io/) 

mPDF est particulièrement adapté aux documents professionnels avec une structure paginée.

- Gestion pratique des en-têtes, pieds de page et numéros de page.
- Bonnes fonctionnalités pour les tableaux multipages, les rapports et les documents longs.
- Prise en charge avancée de l'Unicode, notamment pour l'arabe et l'hébreu (écriture de droite à gauche).
- Fonctionnalités supplémentaires comme les filigranes, les signets et la génération de sommaires.
- Il possède lui aussi des limitations CSS : ce n'est pas un navigateur web complet.
    