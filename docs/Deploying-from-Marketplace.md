# Deploying Semantic MediaWiki from Azure Marketplace

> 🇫🇷 French: [Deploying-from-Marketplace-fr.md](Deploying-from-Marketplace-fr.md)

This guide walks you through deploying Semantic MediaWiki from the Azure Marketplace. The deployment uses an Azure Resource Manager (ARM) template and takes approximately 5–10 minutes to complete.

## Prerequisites

- An active Azure subscription with permissions to create virtual machines
- An SSH key pair (public key in OpenSSH format) — see [Create and use an SSH key pair for Linux VMs in Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/mac-create-ssh-keys)
- A resource group in your Azure subscription (or let the wizard create one)

## Step 1 — Find the Offer and Start Deployment

1. Go to the [Semantic MediaWiki listing on Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/cotechnoe.smw-knowledge-base).
2. Click **Get It Now**, then confirm your contact information if prompted.
3. Click **Create** to open the deployment wizard in the Azure portal.

## Step 2 — Basics

Fill in the **Basics** tab:

| Field | Description | Example |
|---|---|---|
| **Subscription** | Your Azure subscription | — |
| **Resource group** | Create new or select existing | `rg-smw-wiki` |
| **Region** | Azure region for the VM | `Canada Central` |
| **VM name** | 1–15 characters, letters and hyphens only | `smw-wiki` |
| **Admin username** | SSH login name for the VM | `azureuser` |
| **SSH public key** | Your OpenSSH public key | `ssh-rsa AAAA…` |

Click **Next: Wiki Settings**.

## Step 3 — Wiki Settings

Fill in the **Wiki Settings** tab:

| Field | Description | Example |
|---|---|---|
| **Wiki name** | Display name shown in the browser title | `Research Knowledge Base` |
| **Wiki hostname** | Leave as `localhost` to use the public IP automatically. Enter a FQDN only if DNS is already configured. | `localhost` |
| **Wiki admin username** | Administrator account for MediaWiki | `WikiAdmin` |
| **Wiki admin email** | Email address for the wiki administrator | `admin@example.com` |
| **Wiki admin password** | Strong password for the wiki admin account | — |
| **Database password** | Strong password for the MySQL `mediawiki` user | — |

> **Tip — Wiki hostname**: If you leave this field as `localhost`, the system automatically detects the VM's public IP address at first boot using the Azure Instance Metadata Service (IMDS). You can update the hostname later by editing `LocalSettings.firstboot.php` on the VM. Enter a fully qualified domain name (e.g., `wiki.example.com`) only if your DNS record is already pointing to the VM's IP.

Click **Next: VM Settings**.

## Step 4 — VM Settings

Fill in the **VM Settings** tab:

| Field | Description | Recommended |
|---|---|---|
| **VM size** | Compute size for the VM | `Standard_D2s_v3` (small/medium wikis) |
| **Data disk size (GB)** | Additional disk for the database and uploaded files (16–1024 GB) | `32` GB minimum |
| **SSH source IP** | IP address or CIDR range allowed to connect via SSH | Your organization's IP |

> **Security note**: Restricting **SSH source IP** to a specific IP range reduces exposure. Avoid using `*` (any IP) in production environments.

Click **Review + create**.

## Step 5 — Review and Deploy

1. Review the configuration summary displayed by the Azure portal.
2. Click **Create** to start the deployment.
3. Azure deploys the VM and runs first-boot provisioning automatically. The total time is approximately **5–10 minutes**.

You can monitor deployment progress in the **Notifications** panel (bell icon) in the Azure portal.

## Verify

Once the deployment completes:

1. Navigate to your resource group in the Azure portal.
2. Open the VM resource and copy the **Public IP address**.
3. Open a browser and go to `https://<public-ip>/`.
4. Accept the self-signed certificate warning (expected — see [HTTPS-TLS-Certificate.md](HTTPS-TLS-Certificate.md)).
5. The MediaWiki home page should load.

See [Post-Deployment-Verification.md](Post-Deployment-Verification.md) for a complete post-deployment checklist.

## Troubleshooting

| Issue | Solution |
|---|---|
| Page not loading after 10 minutes | First-boot provisioning may still be running. Connect via SSH and run: `sudo journalctl -u smw-firstboot -f` |
| SSH connection refused | Verify your SSH source IP matches the NSG rule. Check VM → **Networking** in the Azure portal. |
| Wiki shows a database error | First-boot may not have completed. See [Troubleshooting.md](Troubleshooting.md). |

See [Troubleshooting.md](Troubleshooting.md) for a complete list of known issues.
