# Certificat HTTPS / TLS

> 🇬🇧 English: [HTTPS-TLS-Certificate.md](HTTPS-TLS-Certificate.md)

La VM inclut un certificat TLS auto-signé généré au premier démarrage. Ce certificat active HTTPS immédiatement mais déclenche un avertissement de sécurité dans le navigateur car il n'est pas signé par une Autorité de Certification (CA) reconnue. Ce guide explique comment le remplacer par un certificat Let's Encrypt (gratuit, reconnu par les navigateurs).

## Comprendre le certificat auto-signé

Le certificat auto-signé est généré au premier démarrage et stocké à :

| Fichier | Chemin |
|---|---|
| Certificat | `/etc/ssl/certs/mediawiki-selfsigned.crt` |
| Clé privée | `/etc/ssl/private/mediawiki-selfsigned.key` |
| Configuration Apache | `/etc/apache2/sites-enabled/mediawiki.conf` |

Le certificat est valable **10 ans** et est suffisant pour une utilisation interne ou d'évaluation. Pour les déploiements en production accessibles aux utilisateurs via un nom de domaine public, remplacez-le par un certificat Let's Encrypt.

## Prérequis

- Le wiki est accessible à `https://<ip-publique>/`
- Un nom de domaine avec un enregistrement DNS A pointant vers l'adresse IP publique de la VM (requis par Let's Encrypt)
- Accès SSH à la VM
- Le port 80 ouvert dans le Groupe de Sécurité Réseau (ouvert par défaut)

## Étape 1 — Pointer un nom de domaine vers la VM

1. Chez votre fournisseur DNS, créez un **enregistrement A** pour votre domaine pointant vers l'adresse IP publique de la VM.

   Exemple :
   ```
   wiki.exemple.com.  IN  A  <ip-publique>
   ```

2. Attendez la propagation DNS (généralement 5 à 60 minutes). Vérifiez avec :

   ```bash
   dig wiki.exemple.com
   ```

   La réponse devrait retourner l'adresse IP publique de la VM.

## Étape 2 — Installer Certbot

Connectez-vous à la VM via SSH et installez Certbot :

```bash
sudo apt update
sudo apt install -y certbot python3-certbot-apache
```

## Étape 3 — Obtenir un certificat Let's Encrypt

```bash
sudo certbot --apache -d wiki.exemple.com
```

Certbot va :

1. Vérifier la propriété du domaine en servant un fichier de défi sur le port 80
2. Émettre un certificat signé par Let's Encrypt
3. Mettre à jour automatiquement la configuration du virtual host Apache

Suivez les instructions à l'écran. Lorsqu'on vous demande les redirections HTTP, sélectionnez l'option **2 (Rediriger)** pour forcer HTTPS.

## Étape 4 — Mettre à jour le nom d'hôte du wiki

Si vous avez déployé avec `wikiHostname = localhost` (détection automatique via IMDS), le paramètre `$wgServer` du wiki a été défini à l'adresse IP publique. Mettez-le à jour pour utiliser votre domaine :

```bash
sudo nano /opt/mediawiki/LocalSettings.firstboot.php
```

Trouvez la ligne :

```php
$wgServer = 'https://<ip-publique>';
```

Remplacez-la par :

```php
$wgServer = 'https://wiki.exemple.com';
```

Redémarrez Apache pour appliquer la modification :

```bash
sudo systemctl restart apache2
```

## Étape 5 — Activer le renouvellement automatique

Certbot installe un timer systemd qui renouvelle automatiquement le certificat avant son expiration. Vérifiez qu'il est actif :

```bash
sudo systemctl status certbot.timer
```

Testez le processus de renouvellement :

```bash
sudo certbot renew --dry-run
```

Un test à sec réussi confirme que le renouvellement fonctionnera automatiquement.

## Vérification

Ouvrez `https://wiki.exemple.com/` dans un navigateur. L'icône de cadenas devrait apparaître sans aucun avertissement de sécurité.

## Résolution de problèmes

| Problème | Solution |
|---|---|
| Certbot : "Could not bind to port 80" | Un autre service utilise le port 80 : `sudo ss -tlnp \| grep :80`. Arrêtez le service et réessayez. |
| Certbot : "DNS problem — SERVFAIL" | La propagation DNS n'est pas encore terminée. Attendez et vérifiez avec `dig wiki.exemple.com`. |
| Les liens du wiki affichent encore l'IP | Mettez à jour `$wgServer` dans `LocalSettings.firstboot.php` et redémarrez Apache. |
| Avertissement de certificat expiré | Le renouvellement automatique ne fonctionne pas : `sudo systemctl enable --now certbot.timer` |

Consultez [Troubleshooting-fr.md](Troubleshooting-fr.md) pour de l'aide supplémentaire.
