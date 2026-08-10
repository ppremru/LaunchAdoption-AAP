# Prerequisites. Disconnected Supply Chain & Air-Gapped Build Guide

When building an Execution Environment inside an air-gapped network, `ansible-builder` cannot pull images from `registry.redhat.io`, Python packages from `pypi.org`, or collections from `galaxy.ansible.com`.

This guide details how to stage all build dependencies inside a disconnected supply chain.

---

## 1. Base Image Synchronization (`skopeo` / `podman`)

Mirror the Red Hat base image from an internet-connected host into your internal container registry (e.g., Quay, Harbor, or Nexus).

* **Step 1: On an Internet-Connected Staging Host**
  ```bash
  # Log in to Red Hat Registry
  podman login registry.redhat.io

  # Export the base image to a local directory archive
  skopeo copy \
    docker://registry.redhat.io/ansible-automation-platform-24/ee-supported-rhel9:latest \
    dir:/tmp/ee-base-image
  ```

* **Step 2: Transfer Files Across the Air Gap**
  Copy the `/tmp/ee-base-image` folder via approved file-transfer mechanisms (such as a bastion host or secure media) to the disconnected network.

* **Step 3: On the Disconnected Build Host**
  ```bash
  # Push the local directory archive to your internal registry
  skopeo copy \
    dir:/tmp/ee-base-image \
    docker://[my-registry.com/my-project/ee-supported-rhel9:latest](https://my-registry.com/my-project/ee-supported-rhel9:latest)
  ```

* **Step 4: Update `execution-environment.yml`**
  Point the base image default to your internal registry URL:
  ```yaml
  build_arg_defaults:
    EE_BASE_IMAGE: '[my-registry.com/my-project/ee-supported-rhel9:latest](https://my-registry.com/my-project/ee-supported-rhel9:latest)'
  ```

---

## 2. Python Dependencies (`pywinrm[kerberos]`)

Choose one of two strategies depending on whether your organization operates an internal PyPI proxy server.

### Option A: Internal PyPI Mirror (Nexus / Artifactory)
If an internal PyPI repository exists, inject a custom `pip.conf` into the container context:

1. Create a `pip.conf` file inside your local `ee-build/` folder:
   ```ini
   [global]
   index-url = [https://nexus.yourdomain.com/repository/pypi-group/simple](https://nexus.yourdomain.com/repository/pypi-group/simple)
   trusted-host = nexus.yourdomain.com
   ```
2. Copy `pip.conf` into the image using `execution-environment.yml`:
   ```yaml
   additional_build_steps:
     prepend_builder:
       - COPY pip.conf /etc/pip.conf
     prepend_final:
       - COPY pip.conf /etc/pip.conf
   ```

### Option B: Bundled Wheels Directory (No PyPI Server Required)
If no PyPI server exists, download compiled `.whl` files on a connected host and bundle them directly into the build folder:

1. **On an Internet-Connected Host:**
   ```bash
   pip download -d ./wheels "pywinrm[kerberos]"
   ```
2. **Transfer:** Copy the `wheels/` directory into your disconnected `ee-build/` folder.
3. **In `execution-environment.yml`:** Instruct `pip` to install offline from local files:
   ```yaml
   additional_build_steps:
     prepend_final:
       - COPY wheels /tmp/wheels
       - RUN pip install --no-index --find-links=/tmp/wheels pywinrm[kerberos]
   ```

---

## 3. Offline Ansible Collections (`ansible.windows` / `microsoft.ad`)

Instead of reaching out to Ansible Galaxy, package collection tarballs locally:

1. **On an Internet-Connected Host:**
   ```bash
   ansible-galaxy collection download ansible.windows -p ./collections
   ansible-galaxy collection download microsoft.ad -p ./collections
   ```
2. **Transfer:** Copy the `collections/` directory into your `ee-build/` folder.
3. **Update `requirements.yml`:** Reference local archive paths instead of Galaxy names:
   ```yaml
   ---
   collections:
     - name: ./collections/ansible-windows-2.1.0.tar.gz
     - name: ./collections/microsoft-ad-1.3.0.tar.gz
   ```

---

## 4. Offline RHEL RPM Repositories (`krb5-workstation`)

Red Hat containers attempt to reach Red Hat CDN repos by default. In air-gapped networks, these checks will fail.

* **Satellite Registration:** If the host server running `ansible-builder` is registered to an internal Red Hat Satellite server, the build process inherits local system subscriptions automatically.
* **Custom RPM Repositories:** If using local HTTP yum mirrors, replace default UBI repositories inside `execution-environment.yml`:
  ```yaml
  additional_build_steps:
    prepend_base:
      - RUN rm -f /etc/yum.repos.d/ubi.repo
      - COPY internal-rhel9.repo /etc/yum.repos.d/internal-rhel9.repo
  ```

---

## 5. Offline Pre-Flight Verification Checklist

Run these commands inside a test container on the disconnected build host to verify infrastructure readiness **before** launching full AAP jobs.

### Test 1: Verify Base Image and DNS Access
```bash
# Verify internal registry base image accessibility
skopeo inspect docker://[my-registry.com/my-project/ee-supported-rhel9:latest](https://my-registry.com/my-project/ee-supported-rhel9:latest)

# Verify DNS resolution and TCP port access to AD and Target Hosts
dig +short dc01.yourdomain.com
nc -zv dc01.yourdomain.com 88
nc -zv win01.yourdomain.com 5986
```

### Test 2: Verify Offline Python Wheel Installation
```bash
# Test dry-run installation of local wheel files without network connectivity
python3 -m pip install --no-index --find-links=./wheels pywinrm[kerberos] --dry-run
```

### Test 3: Verify Kerberos Ticket & CRL Fetch
```bash
# Test ticket acquisition against Active Directory
kinit ansible_svc@YOURDOMAIN.COM
klist

# Test HTTP access to PKI Certificate Revocation List
curl -I [http://pki.yourdomain.com/crl/corp_root.crl](http://pki.yourdomain.com/crl/corp_root.crl)
```

### Test 4: Verify WinRM Python Connection
```python
# Test WinRM HTTPS handshake inside Python shell
import winrm

s = winrm.Session(
    '[https://win01.yourdomain.com:5986/wsman](https://win01.yourdomain.com:5986/wsman)',
    transport='kerberos',
    server_cert_validation='validate'
)
r = s.run_cmd('ipconfig', ['/all'])
print("Status Code:", r.status_code)
```

---

## 6. Official Reference Documentation & Learning Material

For further reading on disconnected enterprise deployments:

* **Red Hat Official Docs:**
  * [Red Hat AAP Installation Guide: Disconnected Installation](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.4/html/red_hat_ansible_automation_platform_installation_guide/disconnected-installation)
  * [Red Hat AAP Guide: Building Ansible Execution Environments](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.4/html/building_ansible_execution_environments/index)
* **Technical Industry Guides:**
  * *The Anatomy of Automation Execution Environments* (Red Hat Blog)
  * *Ansible-Builder in a Disconnected Environment* (Pat Harrison Blog)
* **Recommended Video Search Terms:**
  * "Building Custom Execution Environments for Air-Gapped Networks"
  * "Ansible Builder v3 Schema Deep Dive"
