# Troubleshooting

> 🇫🇷 French: [Troubleshooting-fr.md](Troubleshooting-fr.md)

This guide covers common issues encountered after deploying Semantic MediaWiki from Azure Marketplace.

## Diagnostic Commands

Run these commands over SSH to gather information before investigating a specific issue:

```bash
# Check first-boot service status and logs
sudo journalctl -u smw-firstboot --no-pager

# Check the detailed installation log
sudo cat /var/log/smw-install.log

# Check Apache status
sudo systemctl status apache2

# Check Apache error log (last 50 lines)
sudo tail -50 /var/log/apache2/error.log

# Check MySQL status
sudo systemctl status mysql

# Confirm first-boot completed successfully (file must exist)
ls -la /var/lib/smw/.firstboot-done
```

---

## Issue 1 — Wiki Not Accessible

**Symptom:** Navigating to `https://<public-ip>/` times out or returns a connection error.

**Possible causes:**

| Cause | How to identify |
|---|---|
| First-boot still in progress | `sudo journalctl -u smw-firstboot -f` — service still running |
| Apache not running | `sudo systemctl status apache2` — not active |
| Network Security Group blocks port 443 | Azure portal → VM → Networking → verify port 443 inbound rule exists |

**Solutions:**

1. Wait 5–10 minutes and check again — first-boot takes time.
2. Follow the first-boot log: `sudo journalctl -u smw-firstboot -f`
3. Start Apache if stopped: `sudo systemctl start apache2`
4. Verify the NSG inbound rule allows port 443 from your IP.

---

## Issue 2 — Browser Certificate Warning

**Symptom:** Browser shows "Your connection is not private" or "Certificate not trusted."

**Cause:** The VM uses a self-signed TLS certificate by default.

**Solutions:**

- For evaluation or internal use: click **Advanced** → **Proceed** (Chrome/Edge) or **Accept the Risk** (Firefox).
- For production with a domain name: replace the certificate with Let's Encrypt. See [HTTPS-TLS-Certificate.md](HTTPS-TLS-Certificate.md).

---

## Issue 3 — Cannot Connect via SSH

**Symptom:** `ssh: connect to host <public-ip> port 22: Connection refused` or connection timeout.

**Possible causes:**

| Cause | How to identify |
|---|---|
| NSG rule limits SSH to a different IP | Azure portal → VM → Networking → check port 22 inbound rule |
| VM is stopped | Azure portal → VM → Overview → check Power state |
| Wrong SSH key | Ensure you are using the private key matching the public key provided during deployment |

**Solutions:**

1. In the Azure portal, navigate to the VM → **Networking** → **Inbound port rules**.
2. Find the rule for port 22 and update the source IP to your current IP address.
3. Start the VM from the Azure portal if it is deallocated.

---

## Issue 4 — First-Boot Failed

**Symptom:** `sudo journalctl -u smw-firstboot --no-pager` ends with `smw-firstboot.service: Failed.` or the sentinel file `/var/lib/smw/.firstboot-done` does not exist.

**Steps:**

1. Review the logs:
   ```bash
   sudo journalctl -u smw-firstboot --no-pager
   sudo cat /var/log/smw-install.log
   ```

2. Identify the failing step in the log.

3. Fix the underlying cause (for example, a wrong password causing MySQL initialization to fail).

4. Remove the sentinel file (if partially created) and restart the service:
   ```bash
   sudo rm -f /var/lib/smw/.firstboot-done
   sudo systemctl restart smw-firstboot
   sudo journalctl -u smw-firstboot -f
   ```

---

## Issue 5 — Database Connection Error

**Symptom:** Wiki shows "Database error" or "Cannot connect to database server."

**Cause:** MySQL is not running or credentials are incorrect.

**Solutions:**

```bash
# Check MySQL status
sudo systemctl status mysql

# Start MySQL if stopped
sudo systemctl start mysql

# Enable MySQL to start on boot
sudo systemctl enable mysql
```

If MySQL starts but the wiki still cannot connect, verify the database password:

```bash
sudo grep wgDBpassword /opt/mediawiki/LocalSettings.firstboot.php
```

Test the password manually:

```bash
mysql -u mediawiki -p mediawiki
```

---

## Issue 6 — White Screen or HTTP 500 Error

**Symptom:** Navigating to the wiki shows a blank white page or a generic error page.

**Cause:** PHP parse error or Apache misconfiguration.

**Steps:**

```bash
# Check Apache error log
sudo tail -50 /var/log/apache2/error.log

# Check for PHP syntax errors in the configuration file
sudo php -l /opt/mediawiki/LocalSettings.firstboot.php
```

If you recently edited `LocalSettings.firstboot.php`, check for syntax errors and then restart Apache:

```bash
sudo systemctl restart apache2
```

---

## Issue 7 — Semantic MediaWiki Not Listed in Special:Version

**Symptom:** SMW extensions are absent from `https://<public-ip>/wiki/Special:Version`.

**Cause:** Database migration for the extensions was not completed.

**Solution:**

```bash
cd /opt/mediawiki
sudo -u www-data php maintenance/update.php
```

Check for errors in the output. After the script completes, reload **Special:Version**.

---

## Issue 8 — #ask Query Returns No Results

**Symptom:** An `#ask` query on a page returns zero results, even though pages with matching properties exist.

**Cause:** The SMW data rebuild has not completed, or new pages have not yet been indexed.

**Solution:**

```bash
cd /opt/mediawiki
sudo -u www-data php extensions/SemanticMediaWiki/maintenance/rebuildData.php -v
```

This process can take several minutes depending on the number of pages. After it completes, refresh the query page.

---

## Getting Help

If you cannot resolve the issue using this guide:

1. Collect the following information:
   - The exact error message (from the browser or log)
   - Output of `sudo journalctl -u smw-firstboot --no-pager`
   - Last 50 lines of `/var/log/smw-install.log`
   - Output of `sudo tail -50 /var/log/apache2/error.log`

2. Open an issue at:
   **https://github.com/Cotechnoe/smw6-azure-marketplace-docs/issues**
