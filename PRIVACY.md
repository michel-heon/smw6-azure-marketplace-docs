# Privacy Policy — Semantic MediaWiki on Azure Marketplace

**Publisher:** Cotechnoe  
**Offer:** Semantic MediaWiki — Azure VM  
**Effective date:** May 18, 2026  

---

## 1. Overview

This Privacy Policy describes how Cotechnoe handles data in relation to the
**Semantic MediaWiki — Azure VM** offer published on Microsoft Azure Marketplace.

Cotechnoe is the publisher of this virtual machine (VM) image. The offer deploys
an open-source software stack (MediaWiki + Semantic MediaWiki) into the customer's
own Azure subscription. Cotechnoe does **not** operate or host this software on
behalf of the customer.

---

## 2. Data processed by this offer

### 2.1 Data that stays in your Azure subscription

All data created, stored, or queried within your Semantic MediaWiki instance
(wiki pages, user accounts, semantic annotations, uploaded files, and query results)
resides exclusively within your Azure virtual machine and the associated Azure
storage resources in your subscription. Cotechnoe has no access to this data.

### 2.2 Data collected by Cotechnoe

Cotechnoe does **not** collect, transmit, or process any personal data from your
Semantic MediaWiki instance or from end users of that instance.

When you purchase or deploy this offer through Azure Marketplace, Microsoft collects
transactional and billing information as described in the
[Microsoft Privacy Statement](https://privacy.microsoft.com/privacystatement).
Cotechnoe may receive aggregated, non-identifiable usage signals from Microsoft
for billing and marketplace compliance purposes only.

### 2.3 Data collected by third-party open-source components

This VM image includes the following open-source software:

| Component | Version | Privacy reference |
|-----------|---------|-------------------|
| MediaWiki | 1.43.x | [mediawiki.org/wiki/Privacy_policy](https://www.mediawiki.org/wiki/Privacy_policy) |
| Semantic MediaWiki | 6.0.x | [semantic-mediawiki.org](https://www.semantic-mediawiki.org) |
| PHP | 8.2 | [php.net/privacy.php](https://www.php.net/privacy.php) |
| MySQL Community | 8.0 | [mysql.com/about/legal/privacy.html](https://www.mysql.com/about/legal/privacy.html) |
| Ubuntu | 22.04 LTS | [ubuntu.com/legal/data-privacy](https://ubuntu.com/legal/data-privacy) |

None of these components send data to Cotechnoe.

---

## 3. Your responsibilities as the VM operator

When you deploy this VM, you become the **data controller** for all personal data
processed by your Semantic MediaWiki instance. This includes:

- Wiki user accounts, profiles, and contribution history
- Any personal information entered into wiki pages or semantic forms
- Server access logs (Apache, SSH)

You are responsible for configuring your instance in compliance with applicable
data protection regulations, including but not limited to GDPR (EU 2016/679),
where applicable.

Recommended steps:

- Configure your own privacy policy page within MediaWiki (`MediaWiki:Privacy`)
- Enable HTTPS using a certificate from a trusted authority (e.g., Let's Encrypt)
- Restrict SSH access to trusted IP ranges via Azure Network Security Groups
- Review and configure MediaWiki's built-in user rights and privacy settings

---

## 4. Azure platform privacy

This offer runs on Microsoft Azure infrastructure. Azure's privacy and security
practices are described in the [Microsoft Trust Center](https://www.microsoft.com/trust-center)
and the [Microsoft Online Services Privacy Statement](https://privacy.microsoft.com/privacystatement).

---

## 5. Contact

For questions regarding this privacy policy or the Semantic MediaWiki Azure
Marketplace offer, contact Cotechnoe at:

**GitHub:** [github.com/Cotechnoe/server-azure-marketplace-docs](https://github.com/Cotechnoe/server-azure-marketplace-docs)

---

## 6. Changes to this policy

Cotechnoe may update this policy to reflect changes in the offer or applicable
regulations. The effective date at the top of this page will be updated accordingly.
The latest version is always available at:

`https://github.com/Cotechnoe/server-azure-marketplace-docs/blob/main/PRIVACY.md`
