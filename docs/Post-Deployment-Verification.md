# Post-Deployment Verification

> 🇫🇷 French: [Post-Deployment-Verification-fr.md](Post-Deployment-Verification-fr.md)

After deploying Semantic MediaWiki from the Azure Marketplace, follow this checklist to confirm that the VM, wiki, and services are running correctly.

## Prerequisites

- Deployment completed in the Azure portal
- Public IP address of the VM (available in the Azure portal under the VM resource)
- SSH private key for the admin user configured during deployment

## Step 1 — Wait for First-Boot Provisioning

The first time the VM starts, a systemd service (`smw-firstboot`) runs automatically to:

1. Initialize the MySQL database and create the `mediawiki` user
2. Install and configure MediaWiki with the settings you provided
3. Generate a self-signed TLS certificate
4. Run MediaWiki database migrations (`maintenance/update.php`)
5. Configure Apache and restart it

This process takes **5–10 minutes**. The VM is accessible via SSH during this time, but the wiki may not yet respond on port 443.

## Step 2 — Connect via SSH

```bash
ssh <admin-username>@<public-ip>
```

Replace `<admin-username>` with the username entered in the Basics tab (e.g., `azureuser`) and `<public-ip>` with the VM's public IP address from the Azure portal.

## Step 3 — Check First-Boot Status

```bash
sudo journalctl -u smw-firstboot --no-pager
```

Expected output at the end of a successful run:

```
smw-firstboot.service: Succeeded.
```

If the service is still running, follow the log in real time:

```bash
sudo journalctl -u smw-firstboot -f
```

Press `Ctrl+C` to stop following. If first-boot failed, see [Troubleshooting.md](Troubleshooting.md).

## Step 4 — Verify Services

Check that Apache and MySQL are running:

```bash
sudo systemctl status apache2
sudo systemctl status mysql
```

Both should show `active (running)`.

Confirm that the first-boot sentinel file exists (its presence means first-boot completed):

```bash
ls -la /var/lib/smw/.firstboot-done
```

## Step 5 — Access the Wiki

1. Open a browser and navigate to `https://<public-ip>/`.
2. Accept the self-signed certificate warning:
   - **Chrome/Edge**: Click **Advanced** → **Proceed to \<ip\> (unsafe)**
   - **Firefox**: Click **Advanced** → **Accept the Risk and Continue**
3. The MediaWiki home page should load.
4. Log in at `https://<public-ip>/wiki/Special:UserLogin` using the wiki admin credentials you entered during deployment.

> To replace the self-signed certificate with a trusted Let's Encrypt certificate, see [HTTPS-TLS-Certificate.md](HTTPS-TLS-Certificate.md).

## Step 6 — Verify Semantic MediaWiki

1. Log in as the wiki administrator.
2. Navigate to **Special:Version**: `https://<public-ip>/wiki/Special:Version`
3. Confirm that the following extensions appear in the installed extensions list:
   - **SemanticMediaWiki** 6.0.1
   - **SemanticResultFormats**
   - **Maps**

## Step 7 — Review Installation Details

Key paths on the VM for future reference:

| Item | Path |
|---|---|
| MediaWiki installation | `/opt/mediawiki` |
| Active configuration | `/opt/mediawiki/LocalSettings.firstboot.php` |
| Apache configuration | `/etc/apache2/sites-enabled/mediawiki.conf` |
| MySQL database | `mediawiki` (user: `mediawiki`) |
| Installation log | `/var/log/smw-install.log` |
| First-boot completion | `/var/lib/smw/.firstboot-done` |

## Troubleshooting

| Issue | Solution |
|---|---|
| `smw-firstboot.service: Failed` | Run: `sudo journalctl -u smw-firstboot --no-pager` and `sudo cat /var/log/smw-install.log` |
| Apache not running | `sudo systemctl restart apache2` — check: `sudo journalctl -u apache2 --no-pager` |
| MySQL not running | `sudo systemctl restart mysql` — check: `sudo journalctl -u mysql --no-pager` |
| SMW not listed in Special:Version | Run: `cd /opt/mediawiki && sudo -u www-data php maintenance/update.php` |

See [Troubleshooting.md](Troubleshooting.md) for a full list of known issues.
