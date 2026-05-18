# Résolution de problèmes

> 🇬🇧 English: [Troubleshooting.md](Troubleshooting.md)

Ce guide couvre les problèmes courants rencontrés après le déploiement de Semantic MediaWiki depuis Azure Marketplace.

## Commandes de diagnostic

Exécutez ces commandes via SSH pour recueillir des informations avant d'investiguer un problème spécifique :

```bash
# Vérifier l'état et les journaux du service de premier démarrage
sudo journalctl -u smw-firstboot --no-pager

# Consulter le journal d'installation détaillé
sudo cat /var/log/smw-install.log

# Vérifier l'état d'Apache
sudo systemctl status apache2

# Consulter le journal d'erreurs Apache (50 dernières lignes)
sudo tail -50 /var/log/apache2/error.log

# Vérifier l'état de MySQL
sudo systemctl status mysql

# Confirmer que le premier démarrage s'est terminé avec succès (le fichier doit exister)
ls -la /var/lib/smw/.firstboot-done
```

---

## Problème 1 — Wiki inaccessible

**Symptôme :** Accéder à `https://<ip-publique>/` expire ou retourne une erreur de connexion.

**Causes possibles :**

| Cause | Comment l'identifier |
|---|---|
| Premier démarrage encore en cours | `sudo journalctl -u smw-firstboot -f` — service encore en cours d'exécution |
| Apache ne fonctionne pas | `sudo systemctl status apache2` — non actif |
| Le Groupe de Sécurité Réseau bloque le port 443 | Portail Azure → VM → Mise en réseau → vérifier que la règle entrante port 443 existe |

**Solutions :**

1. Attendez 5 à 10 minutes et réessayez — le premier démarrage prend du temps.
2. Suivez le journal du premier démarrage : `sudo journalctl -u smw-firstboot -f`
3. Démarrez Apache s'il est arrêté : `sudo systemctl start apache2`
4. Vérifiez que la règle entrante NSG autorise le port 443 depuis votre IP.

---

## Problème 2 — Avertissement de certificat dans le navigateur

**Symptôme :** Le navigateur affiche "Votre connexion n'est pas privée" ou "Certificat non approuvé".

**Cause :** La VM utilise un certificat TLS auto-signé par défaut.

**Solutions :**

- Pour une évaluation ou un usage interne : cliquez sur **Avancé** → **Continuer** (Chrome/Edge) ou **Accepter le risque** (Firefox).
- Pour une production avec un nom de domaine : remplacez le certificat par Let's Encrypt. Voir [HTTPS-TLS-Certificate-fr.md](HTTPS-TLS-Certificate-fr.md).

---

## Problème 3 — Impossible de se connecter via SSH

**Symptôme :** `ssh: connect to host <ip-publique> port 22: Connection refused` ou timeout de connexion.

**Causes possibles :**

| Cause | Comment l'identifier |
|---|---|
| La règle NSG limite SSH à une IP différente | Portail Azure → VM → Mise en réseau → vérifier la règle entrante port 22 |
| La VM est arrêtée | Portail Azure → VM → Vue d'ensemble → vérifier l'état de l'alimentation |
| Mauvaise clé SSH | Assurez-vous d'utiliser la clé privée correspondant à la clé publique fournie lors du déploiement |

**Solutions :**

1. Dans le portail Azure, accédez à la VM → **Mise en réseau** → **Règles de port entrant**.
2. Trouvez la règle pour le port 22 et mettez à jour l'IP source avec votre adresse IP actuelle.
3. Démarrez la VM depuis le portail Azure si elle est désallouée.

---

## Problème 4 — Échec du premier démarrage

**Symptôme :** `sudo journalctl -u smw-firstboot --no-pager` se termine par `smw-firstboot.service: Failed.` ou le fichier sentinelle `/var/lib/smw/.firstboot-done` n'existe pas.

**Étapes :**

1. Examinez les journaux :
   ```bash
   sudo journalctl -u smw-firstboot --no-pager
   sudo cat /var/log/smw-install.log
   ```

2. Identifiez l'étape en échec dans le journal.

3. Corrigez la cause sous-jacente (par exemple, un mauvais mot de passe causant l'échec de l'initialisation MySQL).

4. Supprimez le fichier sentinelle (s'il a été partiellement créé) et redémarrez le service :
   ```bash
   sudo rm -f /var/lib/smw/.firstboot-done
   sudo systemctl restart smw-firstboot
   sudo journalctl -u smw-firstboot -f
   ```

---

## Problème 5 — Erreur de connexion à la base de données

**Symptôme :** Le wiki affiche "Database error" ou "Cannot connect to database server."

**Cause :** MySQL ne fonctionne pas ou les identifiants sont incorrects.

**Solutions :**

```bash
# Vérifier l'état de MySQL
sudo systemctl status mysql

# Démarrer MySQL s'il est arrêté
sudo systemctl start mysql

# Activer MySQL au démarrage
sudo systemctl enable mysql
```

Si MySQL démarre mais que le wiki ne peut toujours pas se connecter, vérifiez le mot de passe de la base de données :

```bash
sudo grep wgDBpassword /opt/mediawiki/LocalSettings.firstboot.php
```

Testez le mot de passe manuellement :

```bash
mysql -u mediawiki -p mediawiki
```

---

## Problème 6 — Écran blanc ou erreur HTTP 500

**Symptôme :** Accéder au wiki affiche une page blanche ou une page d'erreur générique.

**Cause :** Erreur de syntaxe PHP ou mauvaise configuration Apache.

**Étapes :**

```bash
# Vérifier le journal d'erreurs Apache
sudo tail -50 /var/log/apache2/error.log

# Vérifier les erreurs de syntaxe PHP dans le fichier de configuration
sudo php -l /opt/mediawiki/LocalSettings.firstboot.php
```

Si vous avez récemment modifié `LocalSettings.firstboot.php`, vérifiez les erreurs de syntaxe puis redémarrez Apache :

```bash
sudo systemctl restart apache2
```

---

## Problème 7 — Semantic MediaWiki absent de Special:Version

**Symptôme :** Les extensions SMW sont absentes de `https://<ip-publique>/wiki/Special:Version`.

**Cause :** La migration de base de données pour les extensions n'a pas été complétée.

**Solution :**

```bash
cd /opt/mediawiki
sudo -u www-data php maintenance/update.php
```

Vérifiez les erreurs dans la sortie. Après la fin du script, rechargez **Special:Version**.

---

## Problème 8 — La requête #ask ne retourne aucun résultat

**Symptôme :** Une requête `#ask` sur une page retourne zéro résultat, même si des pages avec des propriétés correspondantes existent.

**Cause :** La reconstruction des données SMW n'a pas été effectuée, ou les nouvelles pages n'ont pas encore été indexées.

**Solution :**

```bash
cd /opt/mediawiki
sudo -u www-data php extensions/SemanticMediaWiki/maintenance/rebuildData.php -v
```

Ce processus peut prendre plusieurs minutes selon le nombre de pages. Après la fin, actualisez la page contenant la requête.

---

## Obtenir de l'aide

Si vous ne parvenez pas à résoudre le problème avec ce guide :

1. Collectez les informations suivantes :
   - Le message d'erreur exact (depuis le navigateur ou le journal)
   - Sortie de `sudo journalctl -u smw-firstboot --no-pager`
   - 50 dernières lignes de `/var/log/smw-install.log`
   - Sortie de `sudo tail -50 /var/log/apache2/error.log`

2. Ouvrez un ticket à :
   **https://github.com/Cotechnoe/smw6-azure-marketplace-docs/issues**
