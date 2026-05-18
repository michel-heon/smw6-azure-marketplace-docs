# Administering the Wiki

> 🇫🇷 French: [Administering-the-Wiki-fr.md](Administering-the-Wiki-fr.md)

This guide covers common administration tasks for MediaWiki and Semantic MediaWiki running on the Azure Marketplace VM.

## Prerequisites

- SSH access to the VM
- Wiki administrator credentials (set during deployment)
- Basic familiarity with the Linux command line

## Key Paths

| Item | Path |
|---|---|
| MediaWiki installation | `/opt/mediawiki` |
| Active configuration file | `/opt/mediawiki/LocalSettings.firstboot.php` |
| Apache virtual host configuration | `/etc/apache2/sites-enabled/mediawiki.conf` |
| MySQL database name | `mediawiki` |
| MySQL user | `mediawiki` |
| Installation log | `/var/log/smw-install.log` |
| First-boot journal | `journalctl -u smw-firstboot` |

## Managing Users

### Add a new user

1. Log in as the wiki administrator at `https://<public-ip>/wiki/Special:UserLogin`.
2. Navigate to **Special:CreateAccount** and fill in the account details.
3. To assign roles, go to **Special:UserRights**, enter the username, and add the desired groups (`sysop`, `bureaucrat`, `interface-admin`).

### Reset a user password

Via the wiki interface (admin required):

1. Go to **Special:PasswordReset** (or ask the user to do so).

Via the command line:

```bash
cd /opt/mediawiki
sudo -u www-data php maintenance/changePassword.php --user="Username" --password="NewPassword"
```

> Always run MediaWiki maintenance scripts as the `www-data` user to avoid file permission issues.

## Managing Extensions

Extensions are managed with Composer. To list installed extensions:

```bash
cd /opt/mediawiki
sudo -u www-data composer show | grep mediawiki
```

To install a new extension from the MediaWiki extension registry:

```bash
cd /opt/mediawiki
sudo -u www-data composer require mediawiki/extension-name:*
sudo -u www-data php maintenance/update.php
```

After installation, add the `wfLoadExtension` call to `LocalSettings.firstboot.php` if it was not added automatically.

## Backing Up the Wiki

### Back up the database

```bash
sudo mysqldump -u mediawiki -p mediawiki > /tmp/mediawiki-backup-$(date +%Y%m%d).sql
```

Enter the database password when prompted (the password set during deployment).

### Back up uploaded files

```bash
sudo tar -czf /tmp/mediawiki-images-$(date +%Y%m%d).tar.gz /opt/mediawiki/images/
```

### Restore the database

```bash
sudo mysql -u mediawiki -p mediawiki < /tmp/mediawiki-backup-YYYYMMDD.sql
```

> Store backup files on an Azure Storage account or other external location. Do not rely solely on the data disk.

## Running Maintenance Scripts

MediaWiki maintenance scripts are located in `/opt/mediawiki/maintenance/`. Always run them as `www-data`:

| Task | Command |
|---|---|
| Apply database migrations | `sudo -u www-data php maintenance/update.php` |
| Process the job queue | `sudo -u www-data php maintenance/runJobs.php` |
| Rebuild all caches | `sudo -u www-data php maintenance/rebuildall.php` |
| Rebuild SMW property tables | `sudo -u www-data php extensions/SemanticMediaWiki/maintenance/rebuildData.php -v` |

## Updating Wiki Configuration

The active configuration file is `/opt/mediawiki/LocalSettings.firstboot.php`. Key settings:

| Setting | Description |
|---|---|
| `$wgSitename` | Wiki display name shown in the browser |
| `$wgServer` | Public URL of the wiki (update when adding a domain) |
| `$wgDBpassword` | MySQL password for the `mediawiki` user |
| `$wgEnableEmail` | `true` to enable wiki email features |
| `$wgSMTP` | SMTP settings for outgoing email |
| `$wgMaxUploadSize` | Maximum file upload size in bytes |

After editing `LocalSettings.firstboot.php`, restart Apache:

```bash
sudo systemctl restart apache2
```

## Verify

Log in to the wiki and navigate to **Special:Statistics** to confirm that pages, users, and edits are counted correctly.

## Troubleshooting

| Issue | Solution |
|---|---|
| White screen after config change | Check Apache error log: `sudo tail -50 /var/log/apache2/error.log` |
| Maintenance script fails with permission error | Run as `www-data`: `sudo -u www-data php maintenance/...` |
| Database connection error | Check MySQL: `sudo systemctl status mysql` |
| Composer fails with memory error | Add `--no-plugins` or increase PHP memory limit |

See [Troubleshooting.md](Troubleshooting.md) for a complete list.
