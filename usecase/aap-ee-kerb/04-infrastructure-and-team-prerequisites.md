# Prerequisites. Enterprise Infrastructure & Team Alignment Guide

This guide translates Ansible Automation Platform (AAP) requirements into standard IT infrastructure terms. Use these sections to request firewall openings, Active Directory accounts, and PKI assets from enterprise domain teams—even if those teams have zero prior experience with AAP or containerized automation.

---

## 1. Non-Technical Stakeholder Alignment Matrix

When presenting requirements to system administrators, network engineers, or security officers, use standard enterprise terminology instead of AAP-specific jargon.

| AAP Technical Term | Standard IT Equivalent | What to Request from the Respective Team |
| :--- | :--- | :--- |
| **AAP Execution Node** | RHEL Linux Server / Worker Node | Network egress rules to Active Directory Domain Controllers and Windows endpoints. |
| **Execution Environment (EE)** | Standalone Podman/Docker Container | An internal container registry repository (e.g., Quay, Harbor, Nexus) to store custom images. |
| **Machine Credential** | Domain Service Account | A standard Active Directory user account with permissions to manage target Windows services. |
| **Kerberos Transport** | Encrypted WinRM over Port 5986 | Target Windows endpoints configured with WinRM HTTPS listeners and port 5986 open in host firewalls. |

---

## 2. Domain A: Network & Firewall Matrix

The AAP Execution Nodes (Linux servers running container jobs) must have line-of-sight network access to both Active Directory Domain Controllers and the target Windows endpoints.

### Required Firewall Rules

Request the following network egress rules from the **AAP Execution Subnet**:

* **Port 88 (TCP/UDP) — Kerberos:** AAP Nodes → Domain Controllers (Ticket acquisition)
* **Port 464 (TCP/UDP) — Kerberos Password:** AAP Nodes → Domain Controllers (Credential management)
* **Port 389/636 (TCP/UDP) — LDAP/LDAPS:** AAP Nodes → Domain Controllers (Service Principal Name discovery)
* **Port 53 (TCP/UDP) — Internal DNS:** AAP Nodes → Internal DNS Servers (Host and DC resolution)
* **Port 5986 (TCP) — WinRM HTTPS:** AAP Nodes → Target Windows Endpoints (Encrypted playbook execution)
* **Port 80/443 (TCP) — HTTP/HTTPS:** AAP Nodes → Internal PKI Web Servers (Certificate Revocation Lists)

> **The DNS Mandate:** Kerberos authentication **will fail** if target hosts are specified by IP address. The DNS servers assigned to the AAP Execution Nodes must resolve target Windows Fully Qualified Domain Names (e.g., `win01.yourdomain.com`) and return matching Reverse DNS (PTR) records.

---

## 3. Domain B: Active Directory & Kerberos Requirements

Active Directory domain administrators must ensure the service account and domain endpoints meet three strict Kerberos conditions:

1. **ALL-CAPS Realm Formatting:** Kerberos is strictly case-sensitive. Active Directory domain names must be entered in **ALL CAPS** in all configuration files (e.g., `YOURDOMAIN.COM`).
2. **NTP Clock Skew (< 5 Minutes):** Time drift between the AAP Linux nodes, Active Directory Domain Controllers, and target Windows servers **must be under 5 minutes**. Kerberos immediately rejects tickets if system times do not match.
3. **WinRM Service Principal Names (SPNs):** Target Windows servers must have their WinRM Service Principal Names registered in Active Directory (typically created automatically when joining the domain or configuring WinRM listeners).

---

## 4. Domain C: Enterprise PKI & Certificate Revocation

Security teams enforcing strict TLS validation (`ansible_winrm_server_cert_validation: validate`) must supply the necessary certificate authority files.

### Two Hard PKI Requirements

1. **Root & Intermediate CA Certificates:** The PKI team must provide Base64 PEM-formatted (`.crt` or `.pem`) files for all internal issuing Certificate Authorities. These will be baked into the Execution Environment container image during compilation.
2. **Reachable Certificate Revocation Lists (CRLs):**
   * Target Windows certificates contain a **CRL Distribution Point (CDP)** URL (typically an internal HTTP web server).
   * During the WinRM HTTPS connection, the Linux container attempts to fetch this list to verify the certificate has not been revoked.
   * **The Air-Gap Trap:** If the CDP URL points to an unreachable external server or an unrouted internal subnet, the connection will hang for **30–60 seconds** before timing out. Ensure the CDP HTTP/LDAP URLs are reachable from the AAP execution subnet.

> **Where Certificate Validation is Enforced:**
> * **Execution Environment (`01-execution-environment.md`):** Stores the Root CA files so the container *has the capability* to trust the domain.
> * **Inventory (`02-inventory-and-playbooks.md` / `03-aap-integration.md`):** Defines `ansible_winrm_server_cert_validation: validate` in inventory variables to instruct Ansible to *enforce* that trust check during job execution.
