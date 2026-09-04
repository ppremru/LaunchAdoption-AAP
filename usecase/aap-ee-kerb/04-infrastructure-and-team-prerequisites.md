# Infrastructure Prerequisites

Standard Active Directory, WinRM, and network prerequisites required for remote domain management.

---

## 1. Infrastructure & Service Requirements

| Component | Scope | Technical Requirement |
| :--- | :--- | :--- |
| **Active Directory** | Accounts & SPNs | Standard unprivileged AD service account; target endpoints require registered WinRM Service Principal Names. |
| **Network & Firewall** | Port Access | Outbound connectivity on TCP/UDP 88, 464, 389/636, 5986, and 53. Target endpoints require valid A and PTR DNS records. |
| **PKI & Security** | Certificates | Base64 PEM-encoded Root/Intermediate CA certificates and unblocked HTTP access to CDP/CRL endpoints. |
| **Container Platform** | Registry & Time | Internal container registry repository and NTP clock drift maintained under 5 minutes. |

---

## 2. Network Firewall Matrix

Outbound traffic required from the execution subnet:

* **Port 88 (TCP/UDP):** Active Directory Domain Controllers (Kerberos ticket issuance)
* **Port 464 (TCP/UDP):** Active Directory Domain Controllers (Kerberos password management)
* **Port 389/636 (TCP/UDP):** Active Directory Domain Controllers (LDAP/LDAPS SPN discovery)
* **Port 53 (TCP/UDP):** Internal DNS Servers (Domain and host resolution)
* **Port 5986 (TCP):** Target Windows Endpoints (Encrypted WinRM HTTPS remoting)
* **Port 80/443 (TCP):** Internal PKI Web Servers (Certificate Revocation List validation)

> **DNS Resolution Mandate:** Target endpoints must be referenced by Fully Qualified Domain Name (e.g., `win01.yourdomain.com`). Kerberos authentication rejects IP-based connections. Internal DNS servers must resolve both forward (A) and reverse (PTR) records.

---

## 3. Active Directory & Kerberos Standards

* **Case-Sensitive Kerberos Realms:** AD domain names must be specified in **ALL CAPS** across configuration files (e.g., `YOURDOMAIN.COM`) per Kerberos RFC specifications.
* **Kerberos Time Synchronization:** Clock skew across Domain Controllers, execution nodes, and target endpoints must not exceed 5 minutes.
* **WinRM HTTPS Listeners:** Target endpoints must have active WinRM HTTPS listeners configured on port 5986.

---

## 4. PKI & Certificate Validation

* **Trusted Certificate Authorities:** Base64 PEM-encoded (`.crt`/`.pem`) issuing CA certificates are required for TLS trust verification.
* **Reachable CRL Endpoints:** Certificate Revocation List (CRL) distribution points embedded in target certificates must be reachable over HTTP to prevent connection timeouts.
