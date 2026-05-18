# Notions de base de Semantic MediaWiki

> 🇬🇧 English: [Semantic-MediaWiki-Basics.md](Semantic-MediaWiki-Basics.md)

Ce guide présente les fonctionnalités essentielles de Semantic MediaWiki (SMW) : les annotations sémantiques, les propriétés, les requêtes `#ask` et l'export de données.

## Prérequis

- Une instance SMW fonctionnelle (voir [Post-Deployment-Verification-fr.md](Post-Deployment-Verification-fr.md))
- Un compte administrateur ou éditeur du wiki

## Annotations sémantiques

SMW ajoute un sens sémantique au contenu du wiki grâce aux **annotations en ligne**. Tout fait sur une page wiki peut être annoté avec une propriété typée en l'encadrant de doubles crochets avec un double deux-points :

```
[[Nom de la propriété::Valeur]]
```

Exemple — sur la page d'un chercheur :

```
Jane Smith est professeure.
Elle [[Travaille à::Université de Montréal]] et se spécialise en [[Domaine de recherche::Données liées]].
Elle a publié [[Nombre de publications::42]] articles.
```

Les valeurs annotées s'affichent normalement sur la page. SMW les stocke également en tant que données structurées qui peuvent être interrogées sur l'ensemble des pages.

## Créer des propriétés

Les propriétés SMW sont des pages wiki dans l'espace de noms `Property:`. Les propriétés sont créées automatiquement lors de leur première utilisation dans une annotation, mais vous devez définir leur type explicitement pour des requêtes fiables.

1. Accédez à `Property:Travaille à` (créez la page si elle n'existe pas).
2. Ajoutez la déclaration de type de propriété :

```
[[Has type::Page]]
```

Types de propriétés courants :

| Type | Description | Exemple de valeur |
|---|---|---|
| `Page` | Lien vers une autre page wiki | `Université de Montréal` |
| `Text` | Chaîne de texte libre | `Chercheur en Web sémantique` |
| `Number` | Nombre entier ou décimal | `42` |
| `Date` | Date ou plage de dates | `2024-03-15` |
| `URL` | Adresse web externe | `https://exemple.com` |
| `Boolean` | Valeur vrai/faux | `true` |
| `Geographic coordinates` | Paire latitude/longitude (nécessite l'extension Maps) | `45.5017° N, 73.5673° O` |

## Requêtes avec #ask

La fonction parseur `#ask` interroge le magasin sémantique et affiche les résultats en ligne sur n'importe quelle page wiki.

### Syntaxe de base

```
{{#ask: [[conditions de la requête]]
 | ?Propriété1
 | ?Propriété2
 | format=table
}}
```

### Exemple de requête

Lister tous les chercheurs de l'Université de Montréal avec leur domaine de recherche et leur nombre de publications :

```
{{#ask: [[Category:Chercheur]] [[Travaille à::Université de Montréal]]
 | ?Domaine de recherche
 | ?Nombre de publications
 | format=table
 | limit=25
 | sort=Nombre de publications
 | order=desc
}}
```

### Paramètres courants

| Paramètre | Description |
|---|---|
| `format=table` | Affiche les résultats sous forme de tableau (par défaut) |
| `format=list` | Affiche sous forme de liste à puces |
| `format=count` | Retourne uniquement le nombre de résultats |
| `limit=N` | Nombre maximum de résultats à retourner |
| `offset=N` | Ignorer les N premiers résultats (utile pour la pagination) |
| `sort=Propriété` | Trier les résultats selon cette propriété |
| `order=asc` / `order=desc` | Ordre de tri |
| `link=none` | Ne pas lier les valeurs de propriété à leurs pages |
| `headers=show` | Afficher ou masquer les en-têtes de colonnes |

## SemanticResultFormats

L'extension **SemanticResultFormats** (pré-installée) ajoute des formats de sortie supplémentaires pour les résultats `#ask` :

| Format | Description |
|---|---|
| `format=timeline` | Affiche les événements sur une chronologie interactive |
| `format=calendar` | Vue calendrier (nécessite une propriété Date) |
| `format=gallery` | Galerie d'images (nécessite une propriété de fichier image) |
| `format=graph` | Graphe de relations entre les pages wiki |
| `format=csv` | Exporte les résultats en fichier CSV |
| `format=json` | Exporte les résultats en JSON |
| `format=rss` | Exporte les résultats en flux RSS |

Exemple — calendrier d'événements :

```
{{#ask: [[Category:Événement]] [[A une date::+]]
 | ?A une date
 | ?Lieu
 | format=calendar
}}
```

## Modèles et annotations sémantiques

Les modèles permettent une annotation cohérente des propriétés sur de nombreuses pages sans que les éditeurs aient à mémoriser la syntaxe d'annotation.

Exemple de modèle `Template:Chercheur` :

```
<noinclude>Utilisez ce modèle pour décrire un chercheur.</noinclude>
<includeonly>
{{#set: Travaille à={{{travaille_a|}}} | Domaine de recherche={{{domaine_recherche|}}} | Nombre de publications={{{publications|}}} }}
== {{{nom|{{PAGENAME}}}}} ==
* '''Travaille à :''' [[Travaille à::{{{travaille_a|}}}]]
* '''Domaine de recherche :''' [[Domaine de recherche::{{{domaine_recherche|}}}]]
* '''Publications :''' {{{publications|}}}
</includeonly>
```

Utiliser le modèle sur une page :

```
{{Chercheur
 | nom = Jane Smith
 | travaille_a = Université de Montréal
 | domaine_recherche = Données liées
 | publications = 42
}}
```

## Export SPARQL et RDF

SMW stocke toutes les valeurs de propriétés sous forme de triplets RDF. Vous pouvez accéder à ces données via les fonctionnalités d'export intégrées.

### Exporter une seule page en RDF

```
https://<ip-publique>/wiki/Special:ExportRDF/Nom_de_la_page
```

### Parcourir toutes les propriétés et concepts

```
https://<ip-publique>/wiki/Special:Properties
https://<ip-publique>/wiki/Special:Concepts
```

### Exécuter une requête SPARQL (si le point d'accès SPARQL est activé)

```
https://<ip-publique>/wiki/Special:SPARQLEndpoint
```

## Vérification

1. Créez une page de test avec le contenu suivant :

   ```
   [[Category:Test]]
   [[Ma propriété::Bonjour le monde]]
   ```

2. Créez une deuxième page avec la requête suivante :

   ```
   {{#ask: [[Category:Test]] | ?Ma propriété | format=table }}
   ```

3. La page de requête devrait afficher un tableau avec votre page de test et la valeur `Bonjour le monde`.

## Ressources

- [Aide SMW — Table des matières](https://www.semantic-mediawiki.org/wiki/Help:Contents)
- [Langage de requête #ask](https://www.semantic-mediawiki.org/wiki/Help:Selecting_pages)
- [Référence des types de propriétés](https://www.semantic-mediawiki.org/wiki/Help:Property_types)
- [Documentation SemanticResultFormats](https://github.com/SemanticMediaWiki/SemanticResultFormats)
- [Documentation de l'extension Maps](https://www.mediawiki.org/wiki/Extension:Maps)
