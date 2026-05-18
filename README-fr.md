# Semantic MediaWiki — VM Azure Marketplace

[![SMW](https://img.shields.io/badge/SemanticMediaWiki-6.0.1-blue.svg)](https://www.semantic-mediawiki.org/)
[![MediaWiki](https://img.shields.io/badge/MediaWiki-1.43.0-blue.svg)](https://www.mediawiki.org/)
[![PHP](https://img.shields.io/badge/PHP-8.2-777BB4.svg?logo=php&logoColor=white)](https://www.php.net/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04_LTS-E95420.svg?logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![License](https://img.shields.io/badge/License-GPL--2.0-green.svg)](LICENSE)
[![Azure Marketplace](https://img.shields.io/badge/Azure-Marketplace-0078D4.svg?logo=microsoftazure&logoColor=white)](https://azuremarketplace.microsoft.com/marketplace/apps/cotechnoe.smw-knowledge-base)

> 🇬🇧 This page is also available in English: [README.md](README.md)

Semantic MediaWiki (SMW) est une extension open source de MediaWiki qui transforme un wiki classique en une puissante plateforme de données structurées. Elle ajoute des annotations sémantiques directement dans les pages wiki — permettant aux utilisateurs de stocker, interroger et exporter l'information comme données lisibles par les machines via des requêtes `#ask`, des points de terminaison SPARQL et des exports RDF/OWL.

Cette image Azure Marketplace fournit une instance SMW entièrement configurée et prête à la production sur une machine virtuelle Ubuntu 22.04 LTS sécurisée. Aucune installation manuelle requise — déployez depuis le portail Azure et accédez à votre wiki en quelques minutes.

## Sur Azure Marketplace

L'offre **Semantic MediaWiki — VM Azure Marketplace** déploie une machine virtuelle préconfigurée avec :

- **MediaWiki 1.43.0** — la plateforme wiki open source qui propulse Wikipédia et des milliers d'organisations dans le monde
- **Semantic MediaWiki 6.0.1** — données structurées, requêtes sémantiques et capacités de graphe de connaissances
- **SemanticResultFormats** — formats de sortie supplémentaires (graphiques, tableaux, calendriers) pour les requêtes `#ask`
- **Maps** — visualisation de données géographiques dans les pages wiki
- **Apache 2.4** sur **Ubuntu 22.04 LTS** avec redirection HTTP → HTTPS
- **PHP 8.2** via le PPA PHP officiel
- **MySQL 8.0** comme moteur de base de données
- **Certificat HTTPS auto-signé** — provisionné automatiquement au premier démarrage ; remplaçable par Let's Encrypt selon vos besoins
- **Authentification par clé SSH** — la connexion par mot de passe est désactivée
- **Provisionnement automatique au premier démarrage** — MediaWiki et SMW sont entièrement configurés dès le premier démarrage de la VM, sans intervention manuelle

> **Tarification** : Offre gratuite (BYOL) — vous ne payez que les ressources de calcul et de stockage Azure.

## Documentation

| Sujet | English | Français |
|---|---|---|
| Déploiement depuis Marketplace | [Deploying-from-Marketplace.md](docs/Deploying-from-Marketplace.md) | [Deploying-from-Marketplace-fr.md](docs/Deploying-from-Marketplace-fr.md) |
| Vérification post-déploiement | [Post-Deployment-Verification.md](docs/Post-Deployment-Verification.md) | [Post-Deployment-Verification-fr.md](docs/Post-Deployment-Verification-fr.md) |
| Certificat HTTPS / TLS | [HTTPS-TLS-Certificate.md](docs/HTTPS-TLS-Certificate.md) | [HTTPS-TLS-Certificate-fr.md](docs/HTTPS-TLS-Certificate-fr.md) |
| Administration du wiki | [Administering-the-Wiki.md](docs/Administering-the-Wiki.md) | [Administering-the-Wiki-fr.md](docs/Administering-the-Wiki-fr.md) |
| Notions de base de Semantic MediaWiki | [Semantic-MediaWiki-Basics.md](docs/Semantic-MediaWiki-Basics.md) | [Semantic-MediaWiki-Basics-fr.md](docs/Semantic-MediaWiki-Basics-fr.md) |
| Résolution de problèmes | [Troubleshooting.md](docs/Troubleshooting.md) | [Troubleshooting-fr.md](docs/Troubleshooting-fr.md) |

## Notes de version

### SMW 6.0.1 / MediaWiki 1.43.0 (mai 2026)

**Semantic MediaWiki 6.0.1**
- Nécessite PHP 8.1+ et MediaWiki 1.39+
- Amélioration des performances et de la stabilité des requêtes SPARQL
- Compatibilité d'export RDF/OWL mise à jour
- Extensions incluses : SemanticResultFormats, Maps

**MediaWiki 1.43.0**
- PHP 8.1–8.2 officiellement supporté
- Interface Vector 2022 par défaut
- REST API améliorée
- Correctifs de sécurité inclus

## Ressources

- [Documentation officielle Semantic MediaWiki](https://www.semantic-mediawiki.org/wiki/Help:Contents)
- [Documentation MediaWiki](https://www.mediawiki.org/wiki/MediaWiki)
- [Signaler un problème](https://github.com/Cotechnoe/smw6-azure-marketplace-docs/issues)
- [Offre Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/cotechnoe.smw-knowledge-base)

---
*Publié par [Cotechnoe](https://cotechnoe.com) — logiciels académiques open source sur Azure.*
