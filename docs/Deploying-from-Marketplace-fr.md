# Déploiement de Semantic MediaWiki depuis Azure Marketplace

> 🇬🇧 English: [Deploying-from-Marketplace.md](Deploying-from-Marketplace.md)

Ce guide vous accompagne pas à pas dans le déploiement de Semantic MediaWiki depuis Azure Marketplace. Le déploiement utilise un modèle Azure Resource Manager (ARM) et prend environ 5 à 10 minutes.

## Prérequis

- Un abonnement Azure actif avec les autorisations nécessaires pour créer des machines virtuelles
- Une paire de clés SSH (clé publique au format OpenSSH) — voir [Créer et utiliser une paire de clés SSH pour des machines virtuelles Linux dans Azure](https://learn.microsoft.com/fr-fr/azure/virtual-machines/linux/mac-create-ssh-keys)
- Un groupe de ressources dans votre abonnement Azure (ou laissez l'assistant en créer un)

## Étape 1 — Trouver l'offre et démarrer le déploiement

1. Accédez à la [fiche Semantic MediaWiki sur Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/cotechnoe.smw-knowledge-base).
2. Cliquez sur **Obtenir maintenant**, puis confirmez vos coordonnées si demandé.
3. Cliquez sur **Créer** pour ouvrir l'assistant de déploiement dans le portail Azure.

## Étape 2 — Paramètres de base

Remplissez l'onglet **Paramètres de base** :

| Champ | Description | Exemple |
|---|---|---|
| **Abonnement** | Votre abonnement Azure | — |
| **Groupe de ressources** | Créez-en un nouveau ou sélectionnez un existant | `rg-smw-wiki` |
| **Région** | Région Azure pour la VM | `Canada Centre` |
| **Nom de la VM** | 1 à 15 caractères, lettres et traits d'union uniquement | `smw-wiki` |
| **Nom d'utilisateur admin** | Nom de connexion SSH pour la VM | `azureuser` |
| **Clé publique SSH** | Votre clé publique OpenSSH | `ssh-rsa AAAA…` |

Cliquez sur **Suivant : Paramètres du wiki**.

## Étape 3 — Paramètres du wiki

Remplissez l'onglet **Paramètres du wiki** :

| Champ | Description | Exemple |
|---|---|---|
| **Nom du wiki** | Nom affiché dans le titre du navigateur | `Base de connaissances de recherche` |
| **Nom d'hôte du wiki** | Laissez `localhost` pour utiliser l'IP publique automatiquement. Entrez un FQDN uniquement si le DNS est déjà configuré. | `localhost` |
| **Nom d'utilisateur admin du wiki** | Compte administrateur pour MediaWiki | `WikiAdmin` |
| **Courriel admin du wiki** | Adresse courriel de l'administrateur du wiki | `admin@exemple.com` |
| **Mot de passe admin du wiki** | Mot de passe fort pour le compte administrateur | — |
| **Mot de passe de la base de données** | Mot de passe fort pour l'utilisateur MySQL `mediawiki` | — |

> **Conseil — Nom d'hôte du wiki** : Si vous laissez ce champ à `localhost`, le système détecte automatiquement l'adresse IP publique de la VM au premier démarrage via l'Azure Instance Metadata Service (IMDS). Vous pouvez modifier le nom d'hôte ultérieurement en éditant le fichier `LocalSettings.firstboot.php` sur la VM. N'entrez un nom de domaine complet (ex. : `wiki.exemple.com`) que si votre enregistrement DNS pointe déjà vers l'IP de la VM.

Cliquez sur **Suivant : Paramètres de la VM**.

## Étape 4 — Paramètres de la VM

Remplissez l'onglet **Paramètres de la VM** :

| Champ | Description | Recommandé |
|---|---|---|
| **Taille de la VM** | Taille de calcul pour la VM | `Standard_D2s_v3` (petits/moyens wikis) |
| **Taille du disque de données (Go)** | Disque supplémentaire pour la base de données et les fichiers téléversés (16–1024 Go) | `32` Go minimum |
| **IP source SSH** | Adresse IP ou plage CIDR autorisée à se connecter via SSH | L'IP de votre organisation |

> **Note de sécurité** : Restreindre l'**IP source SSH** à une plage d'adresses spécifique réduit l'exposition. Évitez d'utiliser `*` (toute IP) dans un environnement de production.

Cliquez sur **Vérifier + créer**.

## Étape 5 — Vérifier et déployer

1. Vérifiez le récapitulatif de configuration affiché par le portail Azure.
2. Cliquez sur **Créer** pour lancer le déploiement.
3. Azure déploie la VM et exécute le provisionnement au premier démarrage automatiquement. La durée totale est d'environ **5 à 10 minutes**.

Vous pouvez suivre la progression du déploiement dans le panneau **Notifications** (icône de cloche) du portail Azure.

## Vérification

Une fois le déploiement terminé :

1. Accédez à votre groupe de ressources dans le portail Azure.
2. Ouvrez la ressource VM et copiez l'**adresse IP publique**.
3. Ouvrez un navigateur et accédez à `https://<ip-publique>/`.
4. Acceptez l'avertissement concernant le certificat auto-signé (comportement attendu — voir [HTTPS-TLS-Certificate-fr.md](HTTPS-TLS-Certificate-fr.md)).
5. La page d'accueil de MediaWiki devrait s'afficher.

Consultez [Post-Deployment-Verification-fr.md](Post-Deployment-Verification-fr.md) pour une liste de vérification complète après déploiement.

## Résolution de problèmes

| Problème | Solution |
|---|---|
| Page inaccessible après 10 minutes | Le provisionnement au premier démarrage est peut-être encore en cours. Connectez-vous via SSH et exécutez : `sudo journalctl -u smw-firstboot -f` |
| Connexion SSH refusée | Vérifiez que votre IP source SSH correspond à la règle NSG. Consultez VM → **Mise en réseau** dans le portail Azure. |
| Le wiki affiche une erreur de base de données | Le premier démarrage n'est peut-être pas terminé. Consultez [Troubleshooting-fr.md](Troubleshooting-fr.md). |

Consultez [Troubleshooting-fr.md](Troubleshooting-fr.md) pour la liste complète des problèmes connus.
