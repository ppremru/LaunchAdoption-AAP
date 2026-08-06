# How to Set Up Kerberos Authentication for Windows in AAP

Welcome! If you are a Windows administrator new to Ansible Automation Platform (AAP), configuring your platform to communicate with Windows servers securely is a crucial first step[cite: 1]. By default, Windows environments heavily rely on Active Directory and Kerberos for authentication[cite: 1].

This guide will walk you through creating a custom **Execution Environment (EE)**—a containerized environment holding all required tools—that natively authenticates to Windows endpoints using Kerberos[cite: 1].

## Prerequisites

* A working installation of Red Hat Ansible Automation Platform (AAP 2.x)[cite: 1].
* Access to a machine with `ansible-builder` installed to build your container image[cite: 1].
* A container registry to store your custom image, such as Private Automation Hub or Quay[cite: 1].
* A service account created in Active Directory for Ansible to use[cite: 1].

## Step 1: Create the Kerberos Configuration File (`krb5.conf`)

Kerberos requires explicit domain architecture details provided in `krb5.conf`[cite: 1]. Domain names in Kerberos configurations must **always be written in ALL CAPS**[cite: 1].

Create a file named `krb5.conf` in your project directory[cite: 1]:

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

## Step 2: Define the Execution Environment (`execution-environment.yml`)

The `execution-environment.yml` file instructs `ansible-builder` to install system Kerberos tools and Python WinRM libraries[cite: 1].

In the same directory as `krb5.conf`, create `execution-environment.yml`[cite: 1]:

```yaml
version: 3
images:
  base_image:
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

> **Note:** The `COPY` command bakes `krb5.conf` directly into `/etc/krb5.conf` within the container[cite: 1].

## Step 3: Build and Publish the Environment

Build the container image using `ansible-builder`[cite: 1]:

```bash
ansible-builder build -t [my-registry.com/my-project/windows-kerberos-ee:latest](https://my-registry.com/my-project/windows-kerberos-ee:latest)
```

Push the image to your container registry[cite: 1]:

```bash
podman push [my-registry.com/my-project/windows-kerberos-ee:latest](https://my-registry.com/my-project/windows-kerberos-ee:latest)
```

## Step 4: Configure AAP Settings

Log into the AAP Controller UI to register the image and machine credentials[cite: 1]:

1. **Add Execution Environment:** Go to **Execution Environments** -> **Add** and enter your image registry URL[cite: 1].
2. **Create Machine Credential:** Go to **Credentials** -> **Add** and select **Machine**[cite: 1].
   * **Username:** Use User Principal Name (UPN) format with an ALL-CAPS domain (e.g., `ansible_svc@YOURDOMAIN.COM`)[cite: 1].
   * **Password:** Enter the Active Directory service account password[cite: 1].

## Step 5: Set Up Inventory Variables

Set connection variables in your AAP Inventory at the Group or Host level[cite: 1]:

| Variable Name | Required Value | Purpose |
| :--- | :--- | :--- |
| `ansible_connection` | `winrm` | Sets the connection plugin to WinRM[cite: 1]. |
| `ansible_winrm_transport` | `kerberos` | Specifies Kerberos authentication[cite: 1]. |
| `ansible_port` | `5986` | Sets WinRM HTTPS port[cite: 1]. |
| `ansible_winrm_server_cert_validation` | `ignore` | Ignores self-signed certificates (use `validate` for CA certs)[cite: 1]. |

## Important Tips for Success

* **Fully Qualified Domain Names (FQDNs):** Host inventory entries must use FQDNs (e.g., `server01.yourdomain.com`) instead of IP addresses to allow SPN ticket resolution[cite: 1].
* **Time Synchronization:** Clocks between AAP execution nodes and Active Directory Domain Controllers must be synchronized via NTP within 5 minutes[cite: 1].
* **DNS Resolution:** Execution nodes must be able to resolve Domain Controller and host FQDNs[cite: 1].

## References & Further Reading

* **Red Hat Technical Blog:** [Using Kerberos for Windows in Ansible Automation Platform 2](https://www.redhat.com/en/blog/using-kerberos-for-windows-in-ansible-automation-platform-2)
* **Ansible Community Documentation:** [Ansible Windows Kerberos Authentication Guide](https://docs.ansible.com/projects/ansible/latest/os_guide/windows_winrm_kerberos.html)
* **Ansible Core Documentation:** [Windows Remote Management (WinRM) Setup](https://docs.ansible.com/projects/ansible-core/2.15/os_guide/windows_winrm.html)
* **Red Hat Official Course:** [Red Hat Enterprise Automation with Python and Ansible (DO417)](https://www.redhat.com/en/services/training/do417-red-hat-enterprise-automation-python-ansible)
* **Video Walkthrough:** [Ansible Execution Environments Crash Course](https://www.youtube.com/watch?v=p8w4lz0tfRw)
