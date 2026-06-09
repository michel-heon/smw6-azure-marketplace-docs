# Vérification post-déploiement

> 🇬🇧 English: [Post-Deployment-Verification.md](Post-Deployment-Verification.md)

Après avoir déployé Semantic MediaWiki depuis Azure Marketplace, suivez cette liste de vérification pour confirmer que la VM, le wiki et les services fonctionnent correctement.

## Prérequis

- Déploiement terminé dans le portail Azure
- Adresse IP publique de la VM (disponible dans le portail Azure sous la ressource VM)
- Clé SSH privée correspondant à l'utilisateur admin configuré lors du déploiement

## Étape 1 — Attendre le provisionnement au premier démarrage

Au premier démarrage de la VM, un service systemd (`smw-firstboot`) s'exécute automatiquement pour :

1. Initialiser la base de données MySQL et créer l'utilisateur `mediawiki`
2. Installer et configurer MediaWiki avec les paramètres que vous avez fournis
3. Générer un certificat TLS auto-signé
4. Exécuter les migrations de base de données MediaWiki (`maintenance/update.php`)
5. Configurer Apache et le redémarrer

Ce processus prend **5 à 10 minutes**. La VM est accessible via SSH pendant ce temps, mais le wiki peut ne pas encore répondre sur le port 443.

## Étape 2 — Se connecter via SSH

```bash
ssh <nom-utilisateur-admin>@<ip-publique>
```

Remplacez `<nom-utilisateur-admin>` par le nom d'utilisateur saisi dans l'onglet Paramètres de base (ex. : `azureuser`) et `<ip-publique>` par l'adresse IP publique de la VM.

## Étape 3 — Vérifier l'état du premier démarrage

```bash
sudo journalctl -u smw-firstboot --no-pager
```

Sortie attendue à la fin d'une exécution réussie :

```
Finished SMW Marketplace — Configuration au premier démarrage.
```

ou de façon équivalente :

```
smw-firstboot[...]: [smw-firstboot] Terminé avec succès
```

Si le service est encore en cours d'exécution, suivez le journal en temps réel :

```bash
sudo journalctl -u smw-firstboot -f
```

Appuyez sur `Ctrl+C` pour arrêter le suivi. En cas d'échec, consultez [Troubleshooting-fr.md](Troubleshooting-fr.md).

## Étape 4 — Vérifier les services

Vérifiez qu'Apache et MySQL sont en cours d'exécution :

```bash
sudo systemctl status apache2
sudo systemctl status mysql
```

Les deux doivent afficher `active (running)`.

Confirmez que le fichier sentinelle du premier démarrage existe (sa présence indique que le premier démarrage s'est terminé avec succès) :

```bash
ls -la /var/lib/smw/.firstboot-done
```

## Étape 5 — Accéder au wiki

1. Ouvrez un navigateur et accédez à `https://<ip-publique>/`.
2. Acceptez l'avertissement concernant le certificat auto-signé :
   - **Chrome/Edge** : Cliquez sur **Avancé** → **Continuer vers \<ip\> (non sécurisé)**
   - **Firefox** : Cliquez sur **Avancé** → **Accepter le risque et continuer**
3. La page d'accueil de MediaWiki devrait s'afficher.
4. Connectez-vous à `https://<ip-publique>/wiki/Special:UserLogin` avec les identifiants de l'administrateur du wiki saisis lors du déploiement.

> Pour remplacer le certificat auto-signé par un certificat Let's Encrypt de confiance, consultez [HTTPS-TLS-Certificate-fr.md](HTTPS-TLS-Certificate-fr.md).

## Étape 6 — Vérifier Semantic MediaWiki

1. Connectez-vous en tant qu'administrateur du wiki.
2. Accédez à **Special:Version** : `https://<ip-publique>/wiki/Special:Version`
3. Confirmez que les extensions suivantes apparaissent dans la liste des extensions installées :
   - **SemanticMediaWiki** 6.0.1
   - **SemanticResultFormats**
   - **Maps**

## Étape 7 — Consulter les détails d'installation

Chemins importants sur la VM pour référence future :

| Élément | Chemin |
|---|---|
| Installation MediaWiki | `/opt/mediawiki` |
| Configuration active | `/opt/mediawiki/LocalSettings.firstboot.php` |
| Configuration Apache | `/etc/apache2/sites-enabled/smw.conf` |
| Base de données MySQL | `mediawiki` (utilisateur : `mediawiki`) |
| Journal d'installation | `/var/log/smw-install.log` |
| Journal applicatif premier démarrage | `/var/log/smw-firstboot.log` |
| Fin du premier démarrage | `/var/lib/smw/.firstboot-done` |

## Résolution de problèmes

| Problème | Solution |
|---|---|
| `smw-firstboot.service: Failed` | Exécutez : `sudo journalctl -u smw-firstboot --no-pager` et `sudo cat /var/log/smw-install.log` |
| Apache ne fonctionne pas | `sudo systemctl restart apache2` — vérification : `sudo journalctl -u apache2 --no-pager` |
| MySQL ne fonctionne pas | `sudo systemctl restart mysql` — vérification : `sudo journalctl -u mysql --no-pager` |
| SMW absent de Special:Version | Exécutez : `cd /opt/mediawiki && sudo -u www-data php maintenance/update.php` |

Consultez [Troubleshooting-fr.md](Troubleshooting-fr.md) pour la liste complète des problèmes connus.
