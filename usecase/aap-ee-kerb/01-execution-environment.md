# Execution Environment Build Guide

Guide for building a custom Execution Environment (EE) container image to automate Active Directory-joined Windows endpoints using Kerberos over WinRM HTTPS.

---

## 1. Prerequisites

* RHEL 9 / Fedora build host with `ansible-builder` (v3+) and `podman` installed.
* Access to an enterprise container registry (Quay, Harbor, Nexus, or `registry.redhat.io`).
* Active Directory domain details (Domain Realm Name in ALL CAPS and Domain Controller FQDNs).
* Base64 PEM-encoded (`.crt`/`.pem`) Root and Intermediate CA certificates.

---

## 2. Container Build Files

Create a dedicated build directory (e.g., `ee-build/`) containing the following build definitions.

### Step 1: Kerberos Configuration (`krb5.conf`)

Create `krb5.conf` in the build directory:

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

> **Kerberos Protocol Standard:** Realm names (`YOURDOMAIN.COM`) must be specified in **ALL CAPS** across all configuration blocks. Per Kerberos RFC specifications, domain realms are case-sensitive; lowercased entries will cause ticket request failures.

### Step 2: Build Definition (`execution-environment.yml`)

Create `execution-environment.yml` to package system dependencies, Python libraries, CA certificates, and Kerberos settings:

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
    - COPY internal-root-ca.crt /etc/pki/ca-trust/source/anchors/internal-root-ca.crt
    - RUN update-ca-trust
  append_final:
    - COPY krb5.conf /etc/krb5.conf
```

> **TLS Certificate Requirement:** The internal Root CA (`internal-root-ca.crt`) must be placed in the build folder prior to compilation. Omitting internal issuing CAs causes `ssl.SSLCertVerificationError` when certificate validation is enforced over WinRM HTTPS.

### Step 3: Build & Publish Container Image

Execute `ansible-builder` to assemble context and publish the image to your container registry:

```bash
# 1. Build the container image
ansible-builder build --tag [my-registry.com/my-project/windows-kerberos-ee:latest](https://my-registry.com/my-project/windows-kerberos-ee:latest)

# 2. Authenticate to registry
podman login my-registry.com

# 3. Push image to registry
podman push [my-registry.com/my-project/windows-kerberos-ee:latest](https://my-registry.com/my-project/windows-kerberos-ee:latest)
```

---

## 3. Disconnected & Air-Gapped Build Adjustments

For build hosts without direct internet access:

* **Base Image:** Mirror `registry.redhat.io/ansible-automation-platform-24/ee-supported-rhel9:latest` to your private registry.
* **Python Dependencies:** Direct `pip` to an internal PyPI mirror (Nexus/Artifactory) via `pip.conf` or utilize pre-packaged wheel files.
* **System Packages:** Ensure `krb5-workstation` and `krb5-libs` are available via internal Red Hat Satellite or local RPM repositories.

---

## 4. Pre-Flight Container Validation

Validate ticket acquisition and driver support inside the compiled container prior to registering inside AAP:

```bash
# 1. Run interactive container shell
podman run --rm -it [my-registry.com/my-project/windows-kerberos-ee:latest](https://my-registry.com/my-project/windows-kerberos-ee:latest) /bin/bash

# 2. Confirm Kerberos binary installation
kinit --version

# 3. Test ticket acquisition against Active Directory
kinit ansible_svc@YOURDOMAIN.COM

# 4. Inspect active Kerberos ticket cache
klist

# 5. Verify pywinrm Kerberos module loading
python3 -c "import winrm; print(winrm)"
```
