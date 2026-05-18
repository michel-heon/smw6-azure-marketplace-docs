# Semantic MediaWiki Basics

> 🇫🇷 French: [Semantic-MediaWiki-Basics-fr.md](Semantic-MediaWiki-Basics-fr.md)

This guide introduces the core features of Semantic MediaWiki (SMW): semantic annotations, properties, `#ask` queries, and data export.

## Prerequisites

- A running SMW instance (see [Post-Deployment-Verification.md](Post-Deployment-Verification.md))
- Wiki administrator or editor account

## Semantic Annotations

SMW adds semantic meaning to wiki content using **in-line annotations**. Any fact on a wiki page can be annotated with a typed property by wrapping it in double square brackets with a double colon separator:

```
[[Property name::Value]]
```

Example — on a page about a researcher:

```
Jane Smith is a professor.
She [[Works at::University of Montreal]] and specializes in [[Research area::Linked Data]].
She has published [[Publication count::42]] articles.
```

The annotated values appear normally on the page. SMW also stores them as structured data that can be queried across any number of pages.

## Creating Properties

SMW properties are wiki pages in the `Property:` namespace. Properties are created automatically when first used in an annotation, but you should define their type explicitly for reliable querying.

1. Navigate to `Property:Works at` (create the page if it does not exist).
2. Add the property type declaration:

```
[[Has type::Page]]
```

Common property types:

| Type | Description | Example value |
|---|---|---|
| `Page` | Links to another wiki page | `University of Montreal` |
| `Text` | Free-form text string | `Semantic Web researcher` |
| `Number` | Integer or decimal number | `42` |
| `Date` | Date or date range | `2024-03-15` |
| `URL` | External web address | `https://example.com` |
| `Boolean` | True/false value | `true` |
| `Geographic coordinates` | Latitude/longitude pair (requires Maps extension) | `45.5017° N, 73.5673° W` |

## Querying with #ask

The `#ask` parser function queries the semantic store and displays results inline on any wiki page.

### Basic syntax

```
{{#ask: [[query conditions]]
 | ?Property1
 | ?Property2
 | format=table
}}
```

### Example query

List all researchers at the University of Montreal with their research area and publication count:

```
{{#ask: [[Category:Researcher]] [[Works at::University of Montreal]]
 | ?Research area
 | ?Publication count
 | format=table
 | limit=25
 | sort=Publication count
 | order=desc
}}
```

### Common parameters

| Parameter | Description |
|---|---|
| `format=table` | Display results as a table (default) |
| `format=list` | Display as a bulleted list |
| `format=count` | Return only the number of results |
| `limit=N` | Maximum number of results to return |
| `offset=N` | Skip the first N results (useful for pagination) |
| `sort=Property` | Sort results by this property |
| `order=asc` / `order=desc` | Sort direction |
| `link=none` | Do not link property values to their pages |
| `headers=show` | Show or hide column headers |

## SemanticResultFormats

The **SemanticResultFormats** extension (pre-installed) adds additional output formats for `#ask` results:

| Format | Description |
|---|---|
| `format=timeline` | Display events on an interactive timeline |
| `format=calendar` | Calendar view (requires a Date property) |
| `format=gallery` | Image gallery (requires an image file property) |
| `format=graph` | Relationship graph between wiki pages |
| `format=csv` | Export results as a CSV file |
| `format=json` | Export results as JSON |
| `format=rss` | Export results as an RSS feed |

Example — calendar of events:

```
{{#ask: [[Category:Event]] [[Has date::+]]
 | ?Has date
 | ?Location
 | format=calendar
}}
```

## Templates and Semantic Annotations

Templates allow consistent property annotation across many pages without requiring editors to remember the annotation syntax.

Example template `Template:Researcher`:

```
<noinclude>Use this template to describe a researcher.</noinclude>
<includeonly>
{{#set: Works at={{{works_at|}}} | Research area={{{research_area|}}} | Publication count={{{publications|}}} }}
== {{{name|{{PAGENAME}}}}} ==
* '''Works at:''' [[Works at::{{{works_at|}}}]]
* '''Research area:''' [[Research area::{{{research_area|}}}]]
* '''Publications:''' {{{publications|}}}
</includeonly>
```

Use the template on a page:

```
{{Researcher
 | name = Jane Smith
 | works_at = University of Montreal
 | research_area = Linked Data
 | publications = 42
}}
```

## SPARQL and RDF Export

SMW stores all property values as RDF triples. You can access this data through the built-in export features.

### Export a single page as RDF

```
https://<public-ip>/wiki/Special:ExportRDF/Page_Name
```

### Browse all properties and concepts

```
https://<public-ip>/wiki/Special:Properties
https://<public-ip>/wiki/Special:Concepts
```

### Run a SPARQL query (if SPARQL endpoint is enabled)

```
https://<public-ip>/wiki/Special:SPARQLEndpoint
```

## Verify

1. Create a test page with the following content:

   ```
   [[Category:Test]]
   [[My property::Hello World]]
   ```

2. Create a second page with the following query:

   ```
   {{#ask: [[Category:Test]] | ?My property | format=table }}
   ```

3. The query page should return a table with your test page and the value `Hello World`.

## Resources

- [SMW Help contents](https://www.semantic-mediawiki.org/wiki/Help:Contents)
- [#ask query language](https://www.semantic-mediawiki.org/wiki/Help:Selecting_pages)
- [Property types reference](https://www.semantic-mediawiki.org/wiki/Help:Property_types)
- [SemanticResultFormats documentation](https://github.com/SemanticMediaWiki/SemanticResultFormats)
- [Maps extension documentation](https://www.mediawiki.org/wiki/Extension:Maps)
