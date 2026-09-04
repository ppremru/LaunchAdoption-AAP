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
