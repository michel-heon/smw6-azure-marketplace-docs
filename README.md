# Semantic MediaWiki — Azure Marketplace VM

[![SMW](https://img.shields.io/badge/SemanticMediaWiki-6.0.1-blue.svg)](https://www.semantic-mediawiki.org/)
[![MediaWiki](https://img.shields.io/badge/MediaWiki-1.43.0-blue.svg)](https://www.mediawiki.org/)
[![PHP](https://img.shields.io/badge/PHP-8.2-777BB4.svg?logo=php&logoColor=white)](https://www.php.net/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04_LTS-E95420.svg?logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![License](https://img.shields.io/badge/License-GPL--2.0-green.svg)](LICENSE)
[![Azure Marketplace](https://img.shields.io/badge/Azure-Marketplace-0078D4.svg?logo=microsoftazure&logoColor=white)](https://azuremarketplace.microsoft.com/marketplace/apps/cotechnoe.smw-knowledge-base)

> 🇫🇷 Cette page est également disponible en français : [README-fr.md](README-fr.md)

Semantic MediaWiki (SMW) is an open-source extension to MediaWiki that transforms a standard wiki into a powerful structured-data platform. It adds semantic annotations directly to wiki pages — allowing users to store, query, and export information as machine-readable data using `#ask` queries, SPARQL endpoints, and RDF/OWL exports.

This Azure Marketplace image provides a fully configured, production-ready SMW instance on a secure Ubuntu 22.04 LTS virtual machine. No manual installation required — deploy from the Azure portal and access your wiki within minutes.

## On Azure Marketplace

The **Semantic MediaWiki — Azure Marketplace VM** offer deploys a pre-configured virtual machine with:

- **MediaWiki 1.43.0** — the open-source wiki platform powering Wikipedia and thousands of organizations worldwide
- **Semantic MediaWiki 6.0.1** — structured data, semantic queries, and knowledge-graph capabilities
- **SemanticResultFormats** — additional output formats (charts, tables, calendars) for `#ask` queries
- **Maps** — geographic data visualization on wiki pages
- **Apache 2.4** on **Ubuntu 22.04 LTS** with HTTP → HTTPS redirect
- **PHP 8.2** via the official PHP PPA
- **MySQL 8.0** as the database backend
- **Self-signed HTTPS certificate** — auto-provisioned at first boot; replace with Let's Encrypt as needed
- **SSH key authentication** — password login is disabled
- **Automated first-boot provisioning** — MediaWiki and SMW are fully configured on the first VM start, with no manual intervention required

> **Pricing**: Free offer (BYOL) — you pay only for the Azure compute and storage resources.

## Documentation

| Topic | English | Français |
|---|---|---|
| Deploying from Marketplace | [Deploying-from-Marketplace.md](docs/Deploying-from-Marketplace.md) | [Deploying-from-Marketplace-fr.md](docs/Deploying-from-Marketplace-fr.md) |
| Post-Deployment Verification | [Post-Deployment-Verification.md](docs/Post-Deployment-Verification.md) | [Post-Deployment-Verification-fr.md](docs/Post-Deployment-Verification-fr.md) |
| HTTPS / TLS Certificate | [HTTPS-TLS-Certificate.md](docs/HTTPS-TLS-Certificate.md) | [HTTPS-TLS-Certificate-fr.md](docs/HTTPS-TLS-Certificate-fr.md) |
| Administering the Wiki | [Administering-the-Wiki.md](docs/Administering-the-Wiki.md) | [Administering-the-Wiki-fr.md](docs/Administering-the-Wiki-fr.md) |
| Semantic MediaWiki Basics | [Semantic-MediaWiki-Basics.md](docs/Semantic-MediaWiki-Basics.md) | [Semantic-MediaWiki-Basics-fr.md](docs/Semantic-MediaWiki-Basics-fr.md) |
| Troubleshooting | [Troubleshooting.md](docs/Troubleshooting.md) | [Troubleshooting-fr.md](docs/Troubleshooting-fr.md) |

## Release Notes

### SMW 6.0.1 / MediaWiki 1.43.0 (May 2026)

**Semantic MediaWiki 6.0.1**
- Requires PHP 8.1+ and MediaWiki 1.39+
- Improved SPARQL query performance and stability
- Updated RDF/OWL export compatibility
- Bundled extensions: SemanticResultFormats, Maps

**MediaWiki 1.43.0**
- PHP 8.1–8.2 officially supported
- Vector 2022 skin as default interface
- Enhanced REST API
- Security fixes included

## Resources

- [Semantic MediaWiki documentation](https://www.semantic-mediawiki.org/wiki/Help:Contents)
- [MediaWiki documentation](https://www.mediawiki.org/wiki/MediaWiki)
- [Report an issue](https://github.com/Cotechnoe/smw6-azure-marketplace-docs/issues)
- [Azure Marketplace listing](https://azuremarketplace.microsoft.com/marketplace/apps/cotechnoe.smw-knowledge-base)

---
*Published by [Cotechnoe](https://cotechnoe.com) — open-source academic software on Azure.*
