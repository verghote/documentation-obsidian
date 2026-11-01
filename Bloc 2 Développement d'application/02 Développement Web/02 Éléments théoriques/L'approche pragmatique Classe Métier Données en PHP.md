Dans de nombreux projets web pédagogiques ou de taille modeste développés en PHP avec PDO, on adopte une organisation où les classes (comme la classe `Club`) fusionnent la logique de gestion, les requêtes SQL, et parfois même la manipulation du système de fichiers.

Cette documentation dresse un panorama complet de ce modèle en croisant **ses avantages pragmatiques** (pourquoi il séduit tant) et **ses limites architecturales** (les critiques qu'on lui adresse d'un point de vue strict).

## 1. Qu'est-ce que l'approche hybride en PHP ?

Dans ce modèle, une classe unique représente à la fois l'entité conceptuelle (le Club) et son propre mécanisme d'accès aux données.

### Exemple de référence :

PHP

```
namespace ClasseMetier;

class Club {
    public const string DOSSIER_LOGO_CLUB = DOSSIER_WWW . '/data/club/';

    public static function getAll(): array {
        $sql = "select id, nom, fichier, (select count(*) from coureur where coureur.idClub = club.id) as nb from club order by nom;";
        $select = new Select();
        $lesLignes = $select->getRows($sql);

        // Traitement orienté vue/affichage directement intégré
        foreach ($lesLignes as &$ligne) {
            $ligne['present'] = isset($ligne['fichier']) && is_file(self::DOSSIER_LOGO_CLUB . $ligne['fichier']);
        }
        return $lesLignes;
    }
}
```

- **Le principe :** La méthode exécute sa requête SQL, récupère un tableau associatif brut, y ajoute des métadonnées (comme l'existence physique du fichier logo sur le disque via `is_file`) et le renvoie directement prêt à l'emploi pour la page web.
    

## 2. Les Avantages : Pourquoi cette approche est-elle si populaire et redoutable d'efficacité ?

Dans le développement web orienté "pages" (où chaque écran du site affiche des données sur-mesure), cette organisation présente des atouts pragmatiques indéniables :

### A. Éviter la sur-ingénierie (Object Over-engineering)

Dans une architecture d'entreprise ultra-stricte (comme en C# multi-couches), si l'on voulait appliquer un découpage académique pur pour chaque page web :

1. Il faudrait créer une entité `Club` (objet).
    
2. Un mapping complexe (ORM ou transformateur) pour convertir les lignes SQL en objets.
    
3. Gérer des projections spécifiques (par exemple, le sous-compte du nombre de licenciés) en créant des objets "DTO" (_Data Transfer Object_) ad hoc pour chaque page.
    

Pour une application web classique, **cette rigueur devient totalement improductive**. On passerait plus de temps à écrire du code de plomberie (mapping, classes intermédiaires, passerelles) qu'à développer les fonctionnalités du site.

### B. Le pragmatisme du tableau associatif (`array`)

Dans l'écosystème web PHP, le format natif manipulé par les moteurs de template ou les tableaux HTML est le tableau associatif.

- La méthode `Club::getAll()` renvoie directement un tableau de lignes prêtes à être bouclées en HTML (`foreach ($lesClubs as $club)`).
    
- Le chemin entre la base de données et l'écran est le plus court possible, ce qui garantit une vitesse de développement maximale.
    

## 3. Les Critères et Limites : Pourquoi cette approche est-elle critiquée par les architectes ?

D'un point de vue strict des design patterns et des architectures logicielles professionnelles, cette classe `Club` souffre de plusieurs défauts majeurs :

### A. La violation du Principe de Responsabilité Unique (SRP)

Si l'on analyse en détail la classe `Club`, elle endosse **trop de casquettes à la fois** :

1. **La couche Données (Data) :** Elle contient du code technique brut (les requêtes SQL, l'utilisation de `Select` et de `Database`).
    
2. **La couche Système / Infrastructure :** Elle gère des vérifications physiques sur le disque du serveur web (`is_file` sur un chemin de fichiers).
    
3. **La couche Métier (Domain) :** Elle est censée représenter l'entité et ses règles de gestion.
    

_Conséquence :_ La classe fait "trop de choses". Si la structure de la base change, ou si l'emplacement des fichiers sur le serveur change, ou si la règle de calcul évolue, on modifie toujours le même fichier.

### B. Un couplage fort et rigide

Le code SQL est directement écrit à l'intérieur des méthodes de la classe métier.

- Si demain l'application doit évoluer pour remplacer MySQL par PostgreSQL, ou pour consommer une API REST externe au lieu d'une base de données locale, **il faut réécrire le code interne de la classe `Club`**.
    
- Dans une architecture découplée (comme en C# avec la `Passerelle`), le code métier ignore totalement d'où viennent les données, ce qui isole les changements.
    

### C. L'absence de véritable Modèle Objet

En manipulant principalement des tableaux associatifs (`$ligne['nom']`) au lieu d'instances d'objets dotées de comportements (`$club->getNom()`), on se prive de la puissance de la programmation orientée objet (encapsulation, polymorphisme, typage fort). Le risque d'erreurs (faute de frappe dans le nom d'une clé de tableau, par exemple) n'est détecté qu'à l'exécution, et non par le compilateur.

## Conclusion : Le bon outil au bon service du bon contexte

L'approche observée en PHP n'est ni "bonne" ni "mauvaise" dans l'absolu : c'est un **choix de compromis pragmatique**.

- **Dans un projet pédagogique, un petit site ou une application web rapide**, elle évite de s'embourber dans une machinerie lourde et inadaptée (la couche métier objet rigide) là où chaque page web réclame des données sur-mesure. C'est l'application directe du principe _KISS (Keep It Simple, Stupid)_.
    
- **Dans une application d'entreprise critique, de grande taille ou maintenue par une large équipe**, le balancier penche vers le C# multi-couches (séparation stricte `Donnee` / `Metier`), où la rigueur architecturale prime sur la vitesse de prototypage pour garantir la maintenabilité à long terme.