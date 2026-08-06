# How to Set Up Kerberos Authentication for Windows in AAP

Welcome! If you are a Windows administrator new to Ansible Automation Platform (AAP), configuring your platform to communicate with Windows servers securely is a crucial first step[cite: 1]. By default, Windows environments heavily rely on Active Directory and Kerberos for authentication[cite: 1].

This guide will walk you through creating a custom **Execution Environment (EE)**—which is essentially a containerized environment holding all the necessary tools—that can natively authenticate to your Windows endpoints using Kerberos[cite: 1].

## Prerequisites

* A working installation of Red Hat Ansible Automation Platform (AAP 2.x)[cite: 1].
* Access to a machine with `ansible-builder` installed to build your container image[cite: 1].
* A container registry to store your custom image (like Private Automation Hub, Quay, or similar)[cite: 1].
* A service account created in your Active Directory for Ansible to use[cite: 1].

## Step 1: Create the Kerberos Configuration File (`krb5.conf`)

Kerberos needs to know about your Active Directory domain architecture[cite: 1]. We provide this information using a file named `krb5.conf`[cite: 1]. **Crucial Rule:** Domain names in Kerberos configurations must *always* be written in ALL CAPS[cite: 1].

Create a new file named `krb5.conf` in your project directory and configure it for your domain[cite: 1]:

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
```[cite: 1]

## Step 2: Define the Execution Environment (`execution-environment.yml`)

Next, we need to instruct `ansible-builder` on how to build our container[cite: 1]. We do this with a file named `execution-environment.yml`[cite: 1]. This file tells the builder to install the system-level Kerberos tools and the Python libraries required to speak the Kerberos protocol over WinRM[cite: 1].

In the same directory as your `krb5.conf`, create `execution-environment.yml`[cite: 1]:

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
```[cite: 1]

> **Note:** The `COPY` command at the bottom takes the `krb5.conf` file you made in Step 1 and bakes it directly into the execution container so that Ansible automatically has access to it[cite: 1].

## Step 3: Build and Publish the Environment

Open your terminal, navigate to the folder containing your two files, and run the following command to build the container image[cite: 1]:

```bash
ansible-builder build -t [my-registry.com/my-project/windows-kerberos-ee:latest](https://my-registry.com/my-project/windows-kerberos-ee:latest)
```[cite: 1]

Once the build is completely finished, push it to your registry[cite: 1]:

```bash
podman push [my-registry.com/my-project/windows-kerberos-ee:latest](https://my-registry.com/my-project/windows-kerberos-ee:latest)
```[cite: 1]

## Step 4: Configure AAP Settings

Now that the container is ready and hosted on your registry, log into your AAP Controller Web UI and perform the following tasks[cite: 1]:

1. **Add the Execution Environment:** Navigate to *Execution Environments* -> *Add*[cite: 1]. Point it to the registry URL where you pushed your image[cite: 1].
2. **Create the Credential:** Go to *Credentials* -> *Add*[cite: 1]. Choose "Machine" as the credential type[cite: 1].
   * **Username:** Must be in the User Principal Name (UPN) format with an ALL CAPS domain (e.g., `ansible_svc@YOURDOMAIN.COM`)[cite: 1].
   * **Password:** Provide the password for the service account (AAP will securely handle the `kinit` ticket-granting process in the background using this password)[cite: 1].

## Step 5: Set Up Your Inventory Variables

Ansible needs specific instructions on how to connect to the Windows machines[cite: 1]. You will define these variables in your AAP Inventory, either at the Group level (recommended) or the Host level[cite: 1].

| Variable Name | Required Value | Purpose / Explanation |
| :--- | :--- | :--- |
| `ansible_connection` | `winrm` | Tells Ansible to use the Windows Remote Management connection plugin[cite: 1]. |
| `ansible_winrm_transport` | `kerberos` | Explicitly specifies Kerberos as the required authentication method[cite: 1]. |
| `ansible_port` | `5986` | Standard port for WinRM over HTTPS[cite: 1]. |
| `ansible_winrm_server_cert_validation` | `ignore` | Helpful if using self-signed certificates on Windows endpoints (set to `validate` for production CA)[cite: 1]. |

## Important Tips for Success

* **Fully Qualified Domain Names (FQDNs):** You *cannot* use IP addresses in your AAP inventory when using Kerberos[cite: 1]. You must use the full computer name (e.g., `server01.yourdomain.com`)[cite: 1]. Kerberos relies on this name to securely generate the correct Service Principal Name (SPN) ticket[cite: 1].
* **Time Synchronization:** Kerberos is highly sensitive to time differences to prevent replay attacks[cite: 1]. Ensure your AAP environment and your Windows servers have their clocks synchronized via NTP (within 5 minutes)[cite: 1].
* **DNS Resolution:** Your AAP execution nodes must be able to accurately resolve the FQDNs of your Windows servers and the Active Directory Domain Controllers[cite: 1].

## References & Further Reading

* **Red Hat Technical Blog:** [Using Kerberos for Windows in Ansible Automation Platform 2](https://www.redhat.com/en/blog/using-kerberos-for-windows-in-ansible-automation-platform-2)
* **Ansible Community Documentation:** [Ansible Windows Kerberos Authentication Guide](https://docs.ansible.com/projects/ansible/latest/os_guide/windows_winrm_kerberos.html)
* **Ansible Core Documentation:** [Windows Remote Management (WinRM) Setup](https://docs.ansible.com/projects/ansible-core/2.15/os_guide/windows_winrm.html)
* **Red Hat Official Course:** [Red Hat Enterprise Automation with Python and Ansible (DO417)](https://www.redhat.com/en/services/training/do417-red-hat-enterprise-automation-python-ansible)
* **Video Walkthrough:** [Ansible Execution Environments Crash Course](https://www.youtube.com/watch?v=p8w4lz0tfRw)
