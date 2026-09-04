# AAP 2 Controller Integration & Execution Guide

Configuration standards for registering custom Execution Environments, credentials, projects, and job templates in Ansible Automation Platform (AAP 2) Controller for Active Directory automation.

---

## 1. Asset Provisioning Sequence

To prevent execution failures, assets must be created in the Controller UI in this strict order:

```
1. Execution Environment  -->  2. Credentials  -->  3. Project  -->  4. Inventory  -->  5. Job Template
```

---

## 2. Controller Asset Configuration

### Step 1: Register Execution Environment

Navigate to **Execution Environments** > **Add**:

* **Name:** `Windows Active Directory EE`
* **Image:** `my-registry.com/my-project/windows-kerberos-ee:latest`
* **Pull:** `Always` (or `Only if not present` based on registry access)
* **Organization:** Select designated organization

### Step 2: Configure Machine Credentials

Navigate to **Credentials** > **Add**:

* **Name:** `AD Service Account - WinRM`
* **Credential Type:** `Machine`
* **Username:** `ansible_svc` *(Do not include domain prefix or `@YOURDOMAIN.COM` suffix here)*
* **Password:** `<Service Account Password>`
* **Privilege Escalation:** None *(WinRM operates under the authenticated token scope)*

> **Security Mandate:** Never hardcode service account credentials or passwords in playbooks, role defaults, or inventory files. AAP securely injects credentials at runtime.

### Step 3: Link Source Control (Project)

Navigate to **Projects** > **Add**:

* **Name:** `Windows Automation Playbooks`
* **Execution Environment:** `Windows Active Directory EE`
* **Source Control Type:** `Git`
* **Source Control URL:** `https://git.yourdomain.com/automation/windows-ad.git`
* **Source Control Branch/Tag:** `main`

### Step 4: Define Inventory & Group Variables

Navigate to **Inventories** > **Add** > **Add Standard Inventory**:

1. **Details:** Name the inventory (e.g., `Active Directory Managed Endpoints`).
2. **Hosts:** Add target systems using **FQDN only** (e.g., `dc01.yourdomain.com`).
3. **Variables:** Paste connection settings into the inventory YAML variables editor:

```yaml
---
ansible_connection: winrm
ansible_port: 5986
ansible_winrm_transport: kerberos
ansible_winrm_realm: YOURDOMAIN.COM
ansible_winrm_server_cert_validation: validate
```

### Step 5: Construct Job Template

Navigate to **Templates** > **Add** > **Add Job Template**:

* **Name:** `WinRM Verification & Health Check`
* **Job Type:** `Run`
* **Inventory:** `Active Directory Managed Endpoints`
* **Project:** `Windows Automation Playbooks`
* **Execution Environment:** `Windows Active Directory EE`
* **Playbook:** `01_winrm_ping.yml`
* **Credentials:** `AD Service Account - WinRM`

---

## 3. Automated Runtime Ticket Execution

When a Job Template launches:

1. AAP spins up an isolated container instance using the designated **Execution Environment**.
2. AAP injects the **Machine Credential** into the container context.
3. System Kerberos libraries read `/etc/krb5.conf` inside the EE and request a Kerberos Ticket-Granting Ticket (TGT) from the Domain Controller.
4. `pywinrm` establishes an encrypted HTTPS session to port 5986 on the target host using the active Kerberos ticket.
5. Upon job completion, the temporary container instance and active ticket cache are destroyed.

---

## 4. Execution Diagnostics & Troubleshooting

| UI Error Output | Root Cause | Remediation |
| :--- | :--- | :--- |
| `Kerberos module not found` | Job launched using default EE instead of custom Kerberos EE | Edit Job Template and explicitly select `Windows Active Directory EE`. |
| `kinit: Cannot find KDC for realm` | Lowercase realm or missing DNS record in `krb5.conf` | Ensure `ansible_winrm_realm` is **ALL CAPS** and DNS resolves DC hostnames. |
| `401 Unauthorized` | Invalid password or incorrect username formatting | Verify Machine Credential password; ensure username is bare (e.g., `ansible_svc`). |
| `Connection timed out` | Port 5986 blocked or host unreachable | Verify network egress rules from AAP execution nodes to target host on port 5986. |
