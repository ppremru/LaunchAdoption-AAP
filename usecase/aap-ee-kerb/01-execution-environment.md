# Use Case: Execution Environment Build Guide

This guide details the process of compiling a custom Execution Environment (EE) container image tailored for Active Directory-joined Windows automation using Kerberos over WinRM HTTPS.

---

## Table of Contents

* [1. Overview & Prerequisites](#1-overview--prerequisites)
* [2. Execution Environment Build Configuration](#2-execution-environment-build-configuration)
  * [Step 1: Configure Kerberos (`krb5.conf`)](#step-1-configure-kerberos-krb5conf)
  * [Step 2: Define Execution Environment (`execution-environment.yml`)](#step-2-define-execution-environment-execution-environmentyml)
  * [Step 3: Embed Enterprise Root Certificates (Air-Gapped / Strict TLS)](#step-3-embed-enterprise-root-certificates-air-gapped--strict-tls)
  * [Step 4: Build and Push Container Image](#step-4-build-and-push-container-image)
* [3. Air-Gapped / Disconnected Considerations](#3-air-gapped--disconnected-considerations)
* [4. Verification and Testing](#4-verification-and-testing)

---

## 1. Overview & Prerequisites

Ansible Automation Platform (AAP 2) executes jobs inside container images called Execution Environments. To authenticate against Active Directory and manage Windows hosts over WinRM HTTPS, the container must include specific OS binaries, Python libraries, and system configurations.

### Prerequisites

* Access to a RHEL/Fedora/CentOS host with `ansible-builder` (v3+) and `podman` or `docker` installed.
* Network access (or local mirror access) to Red Hat container registries (`registry.redhat.io`).
* Active Directory domain details (Realm Name in ALL CAPS and Domain Controller FQDNs).
* Enterprise Root/Intermediate CA certificates (for TLS validation).

---

## 2. Execution Environment Build Configuration

Create a dedicated build directory (e.g., `ee-build/`) on your workstation to store all build definitions.

### Step 1: Configure Kerberos (`krb5.conf`)

Create a file named `krb5.conf` inside your `ee-build/` folder. This file tells the container how to find your Active Directory Key Distribution Center (KDC) for ticket granting.

```ini
[libdefaults]
    default_realm = YOURDOMAIN.COM
    dns_lookup_realm = false
    dns_lookup_kdc = true
    ticket_lifetime = 24h
    renew_lifetime = 7d
    forwardable = true
    rdns = false
    pkinit_anchors = FILE:/etc/pki/tls/certs/ca-bundle.crt
    default_ccache_name = KEYRING:persistent:%{uid}

[realms]
    YOURDOMAIN.COM = {
        kdc = dc01.yourdomain.com
        kdc = dc02.yourdomain.com
        admin_server = dc01.yourdomain.com
        default_domain = yourdomain.com
    }

[domain_realm]
    .yourdomain.com = YOURDOMAIN.COM
    yourdomain.com = YOURDOMAIN.COM
```

> **Golden Rule:** The realm name (`YOURDOMAIN.COM`) **MUST** be written in ALL CAPS in both `[libdefaults]` and `[realms]`. Kerberos is case-sensitive and authentication will fail if lowercased.

### Step 2: Define Execution Environment (`execution-environment.yml`)

Create the main build specification file named `execution-environment.yml` inside `ee-build/`.

### Step 3: Embed Enterprise Root Certificates (Air-Gapped / Strict TLS)

In disconnected production environments where strict TLS validation (`ansible_winrm_server_cert_validation: validate`) is enforced, the EE container must trust your internal Active Directory Certificate Services (AD CS) or enterprise Root CA.

1. Export your domain's Root and Intermediate CAs in Base64 PEM format (`.crt` or `.pem`).
2. Place the certificate file (e.g., `internal-root-ca.crt`) in your `ee-build/` directory alongside `execution-environment.yml`.
3. Combine dependencies and build steps into `execution-environment.yml`:

```yaml
---
version: 3

build_arg_defaults:
  EE_BASE_IMAGE: 'registry.redhat.io/ansible-automation-platform-24/ee-supported-rhel9:latest'

dependencies:
  ansible_core:
    package_pip: ansible-core
  system:
    - krb5-workstation
    - krb5-libs
    - ca-certificates
  python:
    - pywinrm[kerberos]

additional_build_steps:
  prepend_base:
    # Copies your enterprise Root CA into the container trust store
    - COPY internal-root-ca.crt /etc/pki/ca-trust/source/anchors/internal-root-ca.crt
    # Updates the system CA bundle inside the container
    - RUN update-ca-trust
  append_final:
    # Copies the Kerberos configuration file into system location
    - COPY krb5.conf /etc/krb5.conf
```

> **Important:** If your internal Root CA is omitted from the container trust store, any playbook execution using `ansible_winrm_server_cert_validation: validate` will fail with `ssl.SSLCertVerificationError`.

### Step 4: Build and Push Container Image

Run `ansible-builder` to assemble the context directory and build the container image:

```bash
# 1. Build the container image
ansible-builder build --tag [my-registry.com/my-project/windows-kerberos-ee:latest](https://my-registry.com/my-project/windows-kerberos-ee:latest)

# 2. Log in to your container registry
podman login my-registry.com

# 3. Push the image to the private registry
podman push [my-registry.com/my-project/windows-kerberos-ee:latest](https://my-registry.com/my-project/windows-kerberos-ee:latest)
```

---

## 3. Air-Gapped / Disconnected Considerations

When building execution environments in disconnected networks where public access to PyPI or Red Hat registries is blocked:

* **Base Image:** Mirror `registry.redhat.io/ansible-automation-platform-24/ee-supported-rhel9:latest` into your internal private registry (e.g., Quay, Nexus, Harbor).
* **Python Packages:** Point `pip` to an internal PyPI mirror (like Nexus/JFrog) by adding an index URL or copying wheel files locally during container assembly:
  ```yaml
  additional_build_steps:
    prepend_builder:
      - RUN pip config set global.index-url [https://nexus.yourdomain.com/repository/pypi-group/simple](https://nexus.yourdomain.com/repository/pypi-group/simple)
  ```
* **System Packages:** Ensure RPM repositories (`krb5-workstation`, `krb5-libs`) are available via internal Satellite or local RHEL repos.

---

## 4. Verification and Testing

Verify the newly built container image locally before importing it into AAP:

```bash
# Launch interactive shell inside the EE image
podman run --rm -it [my-registry.com/my-project/windows-kerberos-ee:latest](https://my-registry.com/my-project/windows-kerberos-ee:latest) /bin/bash

# Test Kerberos binary installation
kinit --version

# Test manual ticket acquisition against Active Directory
kinit ansible_svc@YOURDOMAIN.COM

# View active Kerberos ticket cache
klist

# Verify pywinrm kerberos support in python
python3 -c "import winrm; print(winrm)"
```