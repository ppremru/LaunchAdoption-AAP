# Active Directory Windows Automation: Execution Environment Framework

Execution environment specifications for automating Active Directory-joined Windows endpoints using Ansible Automation Platform (AAP 2) via native Kerberos over WinRM HTTPS (Port 5986).

---

## Deployment Sequence

Modules must be executed sequentially. Infrastructure prerequisites and container build steps must be locked before configuring assets inside AAP.

### 1. Prerequisites: Infrastructure & Team Alignment
* **File:** [04-infrastructure-and-team-prerequisites.md](04-infrastructure-and-team-prerequisites.md)
* **Scope:** Cross-team technical alignment, network firewall requirements (Ports 88, 464, 389/636, 5986, 53), Active Directory prerequisites, NTP clock sync rules, and enterprise PKI/CRL trust configurations.

### 2. Execution Environment Build Guide
* **File:** [01-execution-environment.md](01-execution-environment.md)
* **Scope:** Compiling a custom Execution Environment container image with `ansible-builder`, system Kerberos binaries (`krb5-workstation`), Python dependencies (`pywinrm[kerberos]`), `krb5.conf` formatting rules, and container registry publishing.

### 3. Inventory & Playbooks Guide
* **File:** [02-inventory-and-playbooks.md](02-inventory-and-playbooks.md)
* **Scope:** Structuring inventory connection variables for Kerberos authentication (`ansible_winrm_transport: kerberos`), target host domain prerequisites, host FQDN mandates, and ready-to-use sample playbooks.

### 4. AAP 2 UI Integration & Execution Guide
* **File:** [03-aap-integration.md](03-aap-integration.md)
* **Scope:** Integration inside the AAP 2 Controller web console (Execution Environments, Projects, Inventories, Credentials), Job Templates, interactive job execution, and automated `kinit` ticket acquisition.

### 5. Disconnected Supply Chain & Air-Gapped Build Guide
* **File:** [05-airgapped-supply-chain-guide.md](05-airgapped-supply-chain-guide.md)
* **Scope:** Offline supply chain management, base image mirroring using `skopeo`, offline Python wheel bundling for `pywinrm[kerberos]`, local Ansible Collection packaging, and offline RHEL RPM repos.

---

## Onboarding Workflow

```mermaid
flowchart LR
    P["1. Infrastructure Prerequisites"] --> M1["2. EE Container Build"]
    M1 --> M2["3. Inventory & Playbooks"]
    M2 --> M3["4. AAP 2 Integration"]
