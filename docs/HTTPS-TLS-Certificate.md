# HTTPS / TLS Certificate

> 🇫🇷 French: [HTTPS-TLS-Certificate-fr.md](HTTPS-TLS-Certificate-fr.md)

The VM includes a self-signed TLS certificate generated at first boot. This certificate enables HTTPS immediately but triggers a browser security warning because it is not signed by a trusted Certificate Authority (CA). This guide explains how to replace it with a certificate from Let's Encrypt (free, browser-trusted CA).

## Understanding the Self-Signed Certificate

The self-signed certificate is generated at first boot and stored at:

| File | Path |
|---|---|
| Certificate | `/etc/ssl/certs/mediawiki-selfsigned.crt` |
| Private key | `/etc/ssl/private/mediawiki-selfsigned.key` |
| Apache configuration | `/etc/apache2/sites-enabled/smw.conf` |

The certificate is valid for **10 years** and is sufficient for internal use or evaluation. For production deployments accessible to end users via a public domain name, replace it with a Let's Encrypt certificate.

## Prerequisites

- The wiki is accessible at `https://<public-ip>/`
- A domain name with a DNS A record pointing to the VM's public IP address (required by Let's Encrypt)
- SSH access to the VM
- Port 80 open in the Network Security Group (it is open by default)

## Step 1 — Point a Domain Name to the VM

1. In your DNS provider, create an **A record** for your domain pointing to the VM's public IP address.

   Example:
   ```
   wiki.example.com.  IN  A  <public-ip>
   ```

2. Wait for DNS propagation (typically 5–60 minutes). Verify with:

   ```bash
   dig wiki.example.com
   ```

   The response should return the VM's public IP address.

## Step 2 — Install Certbot

Connect to the VM via SSH and install Certbot:

```bash
sudo apt update
sudo apt install -y certbot python3-certbot-apache
```

## Step 3 — Obtain a Let's Encrypt Certificate

```bash
sudo certbot --apache -d wiki.example.com
```

Certbot will:

1. Verify domain ownership by serving a challenge file on port 80
2. Issue a certificate signed by Let's Encrypt
3. Update the Apache virtual host configuration automatically

Follow the on-screen prompts. When asked about HTTP redirects, select option **2 (Redirect)** to force HTTPS.

## Step 4 — Update the Wiki Hostname

If you deployed with `wikiHostname = localhost` (auto-detect from IMDS), the wiki's `$wgServer` setting was set to the public IP. Update it to use your domain:

```bash
sudo nano /opt/mediawiki/LocalSettings.firstboot.php
```

Find the line:

```php
$wgServer = 'https://<public-ip>';
```

Replace it with:

```php
$wgServer = 'https://wiki.example.com';
```

Restart Apache to apply the change:

```bash
sudo systemctl restart apache2
```

## Step 5 — Enable Automatic Renewal

Certbot installs a systemd timer that renews the certificate automatically before it expires. Verify it is active:

```bash
sudo systemctl status certbot.timer
```

Test the renewal process:

```bash
sudo certbot renew --dry-run
```

A successful dry run confirms that renewal will work automatically.

## Verify

Open `https://wiki.example.com/` in a browser. The padlock icon should appear without any security warning.

## Troubleshooting

| Issue | Solution |
|---|---|
| Certbot: "Could not bind to port 80" | Another service is using port 80: `sudo ss -tlnp \| grep :80`. Stop the service and retry. |
| Certbot: "DNS problem — SERVFAIL" | DNS has not propagated yet. Wait and verify with `dig wiki.example.com`. |
| Wiki links still show the IP address | Update `$wgServer` in `LocalSettings.firstboot.php` and restart Apache. |
| Certificate expired warning | Certbot auto-renewal may not be running: `sudo systemctl enable --now certbot.timer` |

See [Troubleshooting.md](Troubleshooting.md) for additional help.
