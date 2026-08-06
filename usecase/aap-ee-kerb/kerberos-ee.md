# How to Set Up Kerberos Authentication for Windows in AAP (Including Disconnected Environments)

Welcome! If you are a Windows administrator new to Ansible Automation Platform (AAP), configuring your platform to communicate with Windows servers securely is a crucial first step. By default, Windows environments heavily rely on Active Directory and Kerberos for authentication.

This guide walks you through creating a custom **Execution Environment (EE)**—a containerized environment holding all required tools—that natively authenticates to Windows endpoints using Kerberos.

---

<a id="table-of-contents"></a>
## Table of Contents

* [Prerequisites & Requirements](#prerequisites)
  * [Standard Requirements](#standard-requirements)
  * [Disconnected (Air-Gapped) Environment Requirements](#disconnected-requirements)
* [Step 1: Create the Kerberos Configuration File (`krb5.conf`)](#step-1)
* [Step 2: Define the Execution Environment (`execution-environment.yml`)](#step-2)
* [Step 3: Build and Publish the Container Image](#step-3)
* [Step 4: Configure AAP Controller UI](#step-4)
* [Step 5: Set Up Inventory Connection Variables](#step-5)
* [Important Tips & Disconnected Operational Notes](#tips-and-notes)
* [References & Further Reading](#references)

---

<a id="prerequisites"></a>
## Prerequisites & Requirements

Before starting, ensure all required components are in place.

<a id="standard-requirements"></a>
### Standard Requirements
* **Ansible Automation Platform:** A working installation of Red Hat AAP 2.x.
* **Build Workstation:** Access to a Linux host or workstation with `ansible-builder` and `podman` installed.
* **Active Directory Service Account:** A domain account created in Active Directory with access permissions on target Windows hosts.
* **Target Windows Hosts:** Endpoints configured with WinRM enabled over HTTPS (port 5986).

<a id="disconnected-requirements"></a>
### Disconnected (Air-Gapped) Environment Requirements
In environments without direct internet access, ensure the following local infrastructure is configured before building:
* **Internal Container Registry:** Red Hat Quay or Private Automation Hub accessible by both your build workstation and AAP execution nodes.
* **Mirrored Base Container Image:** The base EE image (`ee-supported-rhel9`) must be mirrored from `registry.redhat.io` into your internal container registry.
* **Internal Content Sources:**
  * **Ansible Collections:** `ansible.windows` and `microsoft.ad` must be hosted on your local Private Automation Hub or downloaded as offline `.tar.gz` tarballs.
  * **Python Packages:** `pywinrm[kerberos]` and `requests-kerberos` must be hosted on an internal PyPI mirror (e.g., Nexus or Artifactory).
  * **System RPMs:** RHEL repositories (`krb5-workstation`, `krb5-libs`, `krb5-devel`, `gcc`, `python3-devel`) must be available via local Red Hat Satellite or internal yum mirrors.

---

<a id="step-1"></a>
## Step 1: Create the Kerberos Configuration File (`krb5.conf`)

Kerberos requires explicit domain architecture details provided in `krb5.conf`. Domain names in Kerberos configurations must **always be written in ALL CAPS**.

Create a file named `krb5.conf` in your project folder:

```ini
[libdefaults]
  default_realm = YOURDOMAIN.COM
  dns_lookup_realm = false
  dns_lookup_kdc = true
  rdns = false
  ticket_lifetime = 24h
  renew_lifetime = 7d
  forwardable = true

[realms]
  YOURDOMAIN.COM = {
    kdc = dc01.yourdomain.com
    admin_server = dc01.yourdomain.com
  }

[domain_realm]
  .yourdomain.com = YOURDOMAIN.COM
  yourdomain.com = YOURDOMAIN.COM
```

---

<a id="step-2"></a>
## Step 2: Define the Execution Environment (`execution-environment.yml`)

The `execution-environment.yml` file instructs `ansible-builder` which collections, Python libraries, and system RPMs to install.

In the same directory as `krb5.conf`, create `execution-environment.yml`:

```yaml
version: 3
images:
  base_image:
    # Disconnected Note: Replace with your internal registry URL if air-gapped
    name: registry.redhat.io/ansible-automation-platform-24/ee-supported-rhel9:latest

dependencies:
  galaxy:
    collections:
      - name: ansible.windows
      - name: microsoft.ad
  python:
    - "pywinrm[kerberos]>=0.4.0"
    - "requests-kerberos"
  system:
    - krb5-workstation
    - krb5-libs
    - krb5-devel
    - gcc
    - python3-devel

additional_build_steps:
  append_base:
    - COPY krb5.conf /etc/krb5.conf
```

> **Disconnected Note:** If building without internet access, configure `ansible-builder` to point to your internal PyPI mirror and internal Galaxy/Automation Hub using a custom `ansible.cfg` and `pip.conf` during the build process.

---

<a id="step-3"></a>
## Step 3: Build and Publish the Container Image

Run `ansible-builder` to assemble the execution environment image, then push it to your registry.

```bash
# Build the custom EE container
ansible-builder build -t [my-registry.com/my-project/windows-kerberos-ee:latest](https://my-registry.com/my-project/windows-kerberos-ee:latest)

# Push to your internal registry
podman push [my-registry.com/my-project/windows-kerberos-ee:latest](https://my-registry.com/my-project/windows-kerberos-ee:latest)
```

---

<a id="step-4"></a>
## Step 4: Configure AAP Controller UI

Log into the AAP Controller web UI to register the execution environment and configure credentials:

1. **Add Execution Environment:**
   * Navigate to **Administration** -> **Execution Environments** -> **Add**.
   * Enter the image URL (`my-registry.com/my-project/windows-kerberos-ee:latest`).
2. **Create Machine Credential:**
   * Navigate to **Resources** -> **Credentials** -> **Add** and select **Machine** as the type.
   * **Username:** Enter in User Principal Name (UPN) format with an ALL-CAPS domain (e.g., `ansible_svc@YOURDOMAIN.COM`).
   * **Password:** Enter the Active Directory service account password.
   * *Note:* AAP automatically runs `kinit` in the background during job execution using these credentials.

---

<a id="step-5"></a>
## Step 5: Set Up Inventory Connection Variables

Assign connection variables in your AAP Inventory at either the Group or Host level:

| Variable Name | Required Value | Purpose |
| :--- | :--- | :--- |
| `ansible_connection` | `winrm` | Sets the connection plugin to WinRM. |
| `ansible_winrm_transport` | `kerberos` | Specifies Kerberos as the transport mechanism. |
| `ansible_port` | `5986` | Sets WinRM HTTPS port. |
| `ansible_winrm_server_cert_validation` | `ignore` | Bypasses self-signed certificate validation (use `validate` for CA certs). |

---

<a id="tips-and-notes"></a>
## Important Tips & Disconnected Operational Notes

* **FQDN Mandate:** Inventory target entries must use Fully Qualified Domain Names (e.g., `server01.yourdomain.com`), not IP addresses. Kerberos requires FQDNs to generate Service Principal Name (SPN) tickets.
* **Time Synchronization (NTP):** System time across AAP execution nodes and Active Directory Domain Controllers must be synchronized via NTP within 5 minutes. Time drift will cause Kerberos tickets to be immediately rejected.
* **Internal DNS Resolution:** In disconnected networks, ensure AAP execution containers have access to local DNS servers capable of resolving Domain Controllers (KDCs) and target host FQDNs.
* **Registry Authentication:** Ensure AAP execution nodes are logged into your internal container registry (`podman login my-registry.com`) or that Registry Credentials are configured in AAP.

---

<a id="references"></a>
## References & Further Reading

* **Red Hat Technical Blog:** [Using Kerberos for Windows in Ansible Automation Platform 2](https://www.redhat.com/en/blog/using-kerberos-for-windows-in-ansible-automation-platform-2)
* **Ansible Community Documentation:** [Ansible Windows Kerberos Authentication Guide](https://docs.ansible.com/projects/ansible/latest/os_guide/windows_winrm_kerberos.html)
* **Ansible Core Documentation:** [Windows Remote Management (WinRM) Setup](https://docs.ansible.com/projects/ansible-core/2.15/os_guide/windows_winrm.html)
* **Red Hat Official Course:** [Red Hat Enterprise Automation with Python and Ansible (DO417)](https://www.redhat.com/en/services/training/do417-red-hat-enterprise-automation-python-ansible)
* **Video Walkthrough:** [Ansible Execution Environments Crash Course](https://www.youtube.com/watch?v=p8w4lz0tfRw)
