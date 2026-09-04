# Disconnected Supply Chain & Air-Gapped Build Guide

Procedures for packaging, staging, and building custom Execution Environments in air-gapped or network-isolated enterprise environments.

---

## 1. Base Container Image Mirroring

Transfer the Red Hat Ansible Automation Platform base image from an internet-connected staging host to your internal enterprise registry using `skopeo`.

```bash
# 1. Login to Red Hat Registry and Internal Registry
skopeo login registry.redhat.io
skopeo login my-registry.com

# 2. Mirror base image directly between registries
skopeo copy \
  docker://registry.redhat.io/ansible-automation-platform-24/ee-supported-rhel9:latest \
  docker://[my-registry.com/automation/ee-supported-rhel9:latest](https://my-registry.com/automation/ee-supported-rhel9:latest)
```

> **Tarball Transfer Alternative:** If direct network transfer between registries is blocked, save the image as a tar archive to offline media:
> ```bash
> podman pull registry.redhat.io/ansible-automation-platform-24/ee-supported-rhel9:latest
> podman save -o ee-supported-rhel9.tar registry.redhat.io/ansible-automation-platform-24/ee-supported-rhel9:latest
> # Import on internal network:
> podman load -i ee-supported-rhel9.tar
> podman tag registry.redhat.io/ansible-automation-platform-24/ee-supported-rhel9:latest [my-registry.com/automation/ee-supported-rhel9:latest](https://my-registry.com/automation/ee-supported-rhel9:latest)
> podman push [my-registry.com/automation/ee-supported-rhel9:latest](https://my-registry.com/automation/ee-supported-rhel9:latest)
> ```

---

## 2. Offline Python Wheel Packaging

Download `pywinrm[kerberos]` and associated compiled binaries on an internet-connected host, then stage the wheel files for internal build access.

```bash
# On Internet-Connected Staging Host:
mkdir -p ./python-wheels
pip download \
  --dest ./python-wheels \
  --only-binary=:all: \
  --platform manylinux2014_x86_64 \
  --python-version 3.11 \
  "pywinrm[kerberos]"
```

---

## 3. Disconnected Build Definition (`execution-environment.yml`)

Configure `execution-environment.yml` to utilize internal container registries, local yum repositories, and offline Python wheel archives.

```yaml
---
version: 3

build_arg_defaults:
  EE_BASE_IMAGE: '[my-registry.com/automation/ee-supported-rhel9:latest](https://my-registry.com/automation/ee-supported-rhel9:latest)'

dependencies:
  ansible_core:
    package_pip: ansible-core
  system:
    - krb5-workstation
    - krb5-libs
    - ca-certificates
  python:
    - ./python-wheels/pywinrm-*.whl
    - ./python-wheels/requests_kerberos-*.whl

additional_build_steps:
  prepend_base:
    - COPY internal-root-ca.crt /etc/pki/ca-trust/source/anchors/internal-root-ca.crt
    - RUN update-ca-trust
    - COPY ./python-wheels /tmp/python-wheels
  append_final:
    - COPY krb5.conf /etc/krb5.conf
    - RUN rm -rf /tmp/python-wheels
```

---

## 4. Air-Gapped Network Diagnostics

Execute these CLI diagnostics from the execution node prior to running playbooks to confirm network pathways and internal repository access.

```bash
# 1. Test DNS FQDN resolution for Domain Controllers
dig +short dc01.yourdomain.com

# 2. Test reverse PTR record lookup
dig +short -x <DC_IP_ADDRESS>

# 3. Test egress connectivity to WinRM HTTPS port
nc -zvw3 win01.yourdomain.com 5986

# 4. Test Kerberos port egress to Domain Controller
nc -zvw3 dc01.yourdomain.com 88

# 5. Validate internal HTTP CRL distribution endpoint
curl -I [http://crl.yourdomain.com/pki/internal-ca.crl](http://crl.yourdomain.com/pki/internal-ca.crl)
```
