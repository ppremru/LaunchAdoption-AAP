# Use Case:  Master Index & Active Directory Integration Overview

This suite provides an implementation framework for integrating **Active Directory (AD)** and **Identity Providers (IdP)** with **Ansible Automation Platform 2 (AAP 2)**. It follows a progressive **Crawl, Walk, Run** adoption model—moving from read-only LDAPS authentication to automated group mapping and federated Single Sign-On (SSO).

---

## Table of Contents

| Document | Focus / Summary |
| :--- | :--- |
| **[01 - Phase 1: Crawl (LDAPS Baseline)](01-ldaps-baseline-auth.md)** | Firewall matrix, Root CA trust injection, unprivileged bind account, and user authentication. |
| **[02 - Phase 2: Walk (RBAC & Group Mapping)](02-rbac-and-group-mapping.md)** | Mapping AD Security Groups to AAP Organizations, Teams, and System Admins with auto-revocation. |
| **[03 - Phase 3: Run (SAML / SSO Federation)](03-saml-oidc-sso-federation.md)** | Enterprise SSO setup (Entra ID/Okta), SAML claims mapping, JIT provisioning, and MFA. |
| **[04 - Pre-Flight Diagnostics & Debugging](04-troubleshooting-and-diagnostics.md)** | Terminal tests (`ldapsearch`, `openssl`), debug log configuration, and common error fixes. |
| **[05 - Configuration-as-Code Automation](05-controller-as-code-automation.md)** | Automating identity settings using `infra.controller_configuration` and GitOps pipelines. |
| **[06 - Entra ID SAML App Manifest](06-entra-id-app-manifest-template.md)** | Pre-configured Microsoft Entra ID JSON manifest and Microsoft Graph PowerShell scripts. |
| **[07 - Execution Environment Kerberos](07-ee-sssd-kerberos-integration.md)** | Container image build (`ansible-builder`), `krb5.conf`, Keytabs, and passwordless WinRM/SSH auth. |
| **[08 - Executive Briefing Slides](08-executive-briefing-slides.md)** | Executive architecture deck, adoption risk matrix, and implementation timeline. |

---

## Adoption Phases

* **Phase 1: Crawl (Direct LDAPS)** — Establish encrypted connectivity and user lookup using an unprivileged service account.
* **Phase 2: Walk (Group RBAC)** — Delegate access management to Active Directory Security Groups.
* **Phase 3: Run (Federated SSO)** — Offload authentication to a central IdP for Single Sign-On and Multi-Factor Auth.

---

## Core Guidelines

1. **Unprivileged Bind Account:** Requires only standard `Read` and `List Contents` permissions in AD. No domain admin rights or schema changes needed.
2. **System-Level TLS Trust:** Domain Controller Root CAs must be installed in RHEL's `/etc/pki/ca-trust/source/anchors/` store on all AAP nodes.
3. **Automate Access Revocation:** Always enforce `"remove": true` in group mappings so offboarded users lose access instantly upon logout.
4. **Multi-Domain Routing:** Use Port `636` for single domains and Global Catalog Port `3269` for multi-domain Active Directory forests.
