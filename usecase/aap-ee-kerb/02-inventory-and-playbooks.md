# Inventory & Playbook Standards

Standardized inventory connection variables and verification playbooks for managing Active Directory-joined Windows endpoints via Kerberos over WinRM HTTPS (Port 5986).

---

## 1. WinRM Connection Variables

Configure baseline connection settings inside `group_vars/windows.yml` to enforce encrypted Kerberos transport and TLS certificate validation.

```yaml
---
# Connection Protocol & Port
ansible_connection: winrm
ansible_port: 5986

# Authentication & Encryption
ansible_winrm_transport: kerberos
ansible_winrm_realm: YOURDOMAIN.COM
ansible_winrm_server_cert_validation: validate

# Service Account Principal (Omit domain prefix when using Kerberos)
ansible_user: ansible_svc
```

> **Mandatory Host Address Standard:** All inventory host definitions must use target Fully Qualified Domain Names (e.g., `win01.yourdomain.com`). Kerberos authentication resolves Service Principal Names (SPNs) bound strictly to DNS hostnames; specifying IP addresses causes authentication failures.

---

## 2. Inventory Definition

Example YAML inventory structure (`hosts.yml`):

```yaml
---
all:
  children:
    windows:
      hosts:
        dc01.yourdomain.com:
        app01.yourdomain.com:
        db01.yourdomain.com:
      vars:
        ansible_connection: winrm
        ansible_port: 5986
        ansible_winrm_transport: kerberos
        ansible_winrm_realm: YOURDOMAIN.COM
        ansible_winrm_server_cert_validation: validate
```

---

## 3. Baseline Verification Playbooks

Execute these standard playbooks sequentially to validate WinRM HTTPS connectivity, Kerberos ticket delegation, and Active Directory module integration.

### Playbook 1: WinRM Connectivity Check (`01_winrm_ping.yml`)

Verifies port 5986 access, Kerberos ticket exchange, and PowerShell execution context:

```yaml
---
- name: Verify Windows WinRM Connectivity
  hosts: windows
  gather_facts: false
  tasks:
    - name: Execute WinRM ping
      ansible.windows.win_ping:

    - name: Query executing identity
      ansible.windows.win_whoami:
      register: whoami_out

    - name: Display authenticated identity
      ansible.builtin.debug:
        var: whoami_out.account.sid
```

### Playbook 2: Active Directory Information Retrieval (`02_ad_info.yml`)

Verifies domain query permissions using the `microsoft.ad` collection:

```yaml
---
- name: Query Active Directory Infrastructure
  hosts: windows
  gather_facts: false
  tasks:
    - name: Retrieve Domain Information
      microsoft.ad.domain_info:
      register: ad_info

    - name: Display Active Directory Domain Details
      ansible.builtin.debug:
        msg:
          - "Domain Name: {{ ad_info.domain.name }}"
          - "Domain Functional Level: {{ ad_info.domain.domain_mode }}"
          - "Primary Forest: {{ ad_info.domain.forest }}"
```

---

## 4. Common Execution Errors & Root Causes

* **`HTTP 401 Unauthorized`:** Kerberos realm mismatch or lowercase domain name in configuration files. Ensure realm is **ALL CAPS** (`YOURDOMAIN.COM`).
* **`Cannot find SPN`:** Target endpoint defined via IP address or missing WinRM SPN registration in Active Directory.
* **`ssl.SSLCertVerificationError`:** Target host TLS certificate issued by an untrusted internal CA. Ensure issuing CAs are embedded into the container trust store.
* **`Kerberos clock skew too great`:** Time difference between execution node, Domain Controller, and target endpoint exceeds 5 minutes.
