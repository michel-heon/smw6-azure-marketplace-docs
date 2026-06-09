# Administration du wiki

> 🇬🇧 English: [Administering-the-Wiki.md](Administering-the-Wiki.md)

Ce guide couvre les tâches d'administration courantes pour MediaWiki et Semantic MediaWiki fonctionnant sur la VM Azure Marketplace.

## Prérequis

- Accès SSH à la VM
- Identifiants d'administrateur du wiki (définis lors du déploiement)
- Familiarité de base avec la ligne de commande Linux

## Chemins importants

| Élément | Chemin |
|---|---|
| Installation MediaWiki | `/opt/mediawiki` |
| Fichier de configuration actif | `/opt/mediawiki/LocalSettings.firstboot.php` |
| Configuration Apache virtual host | `/etc/apache2/sites-enabled/smw.conf` |
| Nom de la base de données MySQL | `mediawiki` |
| Utilisateur MySQL | `mediawiki` |
| Journal d'installation | `/var/log/smw-install.log` |
| Journal applicatif premier démarrage | `/var/log/smw-firstboot.log` |
| Journal système premier démarrage | `journalctl -u smw-firstboot` |

## Gestion des utilisateurs

### Ajouter un nouvel utilisateur

1. Connectez-vous en tant qu'administrateur du wiki à `https://<ip-publique>/wiki/Special:UserLogin`.
2. Accédez à **Special:CreateAccount** et remplissez les informations du compte.
3. Pour attribuer des rôles, allez à **Special:UserRights**, saisissez le nom d'utilisateur et ajoutez les groupes souhaités (`sysop`, `bureaucrat`, `interface-admin`).

### Réinitialiser le mot de passe d'un utilisateur

Via l'interface du wiki (droits admin requis) :

1. Accédez à **Special:PasswordReset** (ou demandez à l'utilisateur de le faire).

Via la ligne de commande :

```bash
cd /opt/mediawiki
sudo -u www-data php maintenance/changePassword.php --user="NomUtilisateur" --password="NouveauMotDePasse"
```

> Exécutez toujours les scripts de maintenance MediaWiki en tant qu'utilisateur `www-data` pour éviter les problèmes de permissions sur les fichiers.

### Récupérer le mot de passe de l'administrateur wiki

Si vous n'avez pas défini de **mot de passe administrateur wiki** lors du déploiement
Azure, ou si vous l'avez perdu, le mot de passe a été généré automatiquement au
premier démarrage et n'est stocké nulle part sur le disque. Utilisez SSH pour le
réinitialiser :

```bash
ssh <nom-utilisateur-admin>@<ip-publique>
cd /opt/mediawiki
sudo -u www-data php maintenance/changePassword.php --user="WikiAdmin" --password="<nouveau-mot-de-passe-fort>"
```

Remplacez `<nouveau-mot-de-passe-fort>` par un mot de passe d'au moins 12 caractères.
Après cette commande, vous pouvez vous connecter au wiki avec le nouveau mot de passe.

> **Note de sécurité :** Choisissez un mot de passe fort et unique, et conservez-le
> dans un gestionnaire de mots de passe. Le compte administrateur wiki (`WikiAdmin`
> par défaut) dispose de tous les privilèges d'administration sur le wiki.

## Gestion des extensions

Les extensions sont gérées avec Composer. Pour lister les extensions installées :

```bash
cd /opt/mediawiki
sudo -u www-data composer show | grep mediawiki
```

Pour installer une nouvelle extension depuis le registre d'extensions MediaWiki :

```bash
cd /opt/mediawiki
sudo -u www-data composer require mediawiki/nom-extension:"^1.0"
sudo -u www-data php maintenance/update.php
```

> **Important — spécifiez la version :** Utilisez une contrainte de version (ex. `^1.0`
> ou `~1.2`) plutôt que `*`. L'utilisation de `*` peut installer une version
> incompatible qui brise le premier démarrage ou les extensions existantes.
> Par exemple, Maps 12.x nécessite que SMW soit complètement initialisé avant
> l'exécution de son hook ; une version plus récente pourrait modifier ce
> comportement et casser `maintenance/update.php`.

Après l'installation, ajoutez l'appel `wfLoadExtension` dans `LocalSettings.firstboot.php` s'il n'a pas été ajouté automatiquement.

## Sauvegarde du wiki

### Sauvegarder la base de données

```bash
sudo mysqldump -u mediawiki -p mediawiki > /tmp/mediawiki-backup-$(date +%Y%m%d).sql
```

Entrez le mot de passe de la base de données lorsqu'il est demandé (le mot de passe défini lors du déploiement).

### Sauvegarder les fichiers téléversés

```bash
sudo tar -czf /tmp/mediawiki-images-$(date +%Y%m%d).tar.gz /opt/mediawiki/images/
```

### Restaurer la base de données

```bash
sudo mysql -u mediawiki -p mediawiki < /tmp/mediawiki-backup-AAAAMMJJ.sql
```

> Stockez les fichiers de sauvegarde sur un compte Azure Storage ou autre emplacement externe. Ne comptez pas uniquement sur le disque de données.

## Exécuter les scripts de maintenance

Les scripts de maintenance MediaWiki se trouvent dans `/opt/mediawiki/maintenance/`. Exécutez-les toujours en tant que `www-data` :

| Tâche | Commande |
|---|---|
| Appliquer les migrations de base de données | `sudo -u www-data php maintenance/update.php` |
| Traiter la file de tâches | `sudo -u www-data php maintenance/runJobs.php` |
| Reconstruire tous les caches | `sudo -u www-data php maintenance/rebuildall.php` |
| Reconstruire les tables de propriétés SMW | `sudo -u www-data php extensions/SemanticMediaWiki/maintenance/rebuildData.php -v` |

## Mettre à jour la configuration du wiki

Le fichier de configuration actif est `/opt/mediawiki/LocalSettings.firstboot.php`. Paramètres importants :

| Paramètre | Description |
|---|---|
| `$wgSitename` | Nom d'affichage du wiki dans le navigateur |
| `$wgServer` | URL publique du wiki (à mettre à jour lors de l'ajout d'un domaine) |
| `$wgDBpassword` | Mot de passe MySQL pour l'utilisateur `mediawiki` |
| `$wgEnableEmail` | `true` pour activer les fonctionnalités de courriel |
| `$wgSMTP` | Paramètres SMTP pour les courriels sortants |
| `$wgMaxUploadSize` | Taille maximale de téléversement en octets |

Après avoir modifié `LocalSettings.firstboot.php`, redémarrez Apache :

```bash
sudo systemctl restart apache2
```

## Vérification

Connectez-vous au wiki et accédez à **Special:Statistics** pour confirmer que les pages, utilisateurs et modifications sont correctement comptabilisés.

## Résolution de problèmes

| Problème | Solution |
|---|---|
| Écran blanc après modification de la config | Vérifiez le journal d'erreurs Apache : `sudo tail -50 /var/log/apache2/error.log` |
| Erreur de permission dans un script de maintenance | Exécutez en tant que `www-data` : `sudo -u www-data php maintenance/...` |
| Erreur de connexion à la base de données | Vérifiez MySQL : `sudo systemctl status mysql` |
| Échec de Composer avec erreur mémoire | Ajoutez `--no-plugins` ou augmentez la limite mémoire PHP |

Consultez [Troubleshooting-fr.md](Troubleshooting-fr.md) pour la liste complète.
