# Phase 1: Crawl — Direct LDAPS Read-Only Integration

## Executive Summary

This guide details the implementation of **Phase 1 (Crawl)** for Active Directory integration in **Ansible Automation Platform 2 (AAP 2)**. 

The objective of this phase is to establish basic, secure user authentication over encrypted LDAPS using an unprivileged, read-only bind service account. This step requires zero Active Directory schema modifications, zero elevated domain privileges, and zero write permissions, making it the lowest-risk approach for initial deployment.

---

## Technical Prerequisites & Network Requirements

Before configuring AAP Controller, ensure the following network access, service account attributes, and PKI certificates are staged by the respective infrastructure teams.

### 1. Network Firewall Requirements

The AAP Controller execution and control nodes must have direct egress access to Active Directory Domain Controllers:

| Source | Destination | Port | Protocol | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **AAP Controller Nodes** | Active Directory DCs | **636** | TCP | Encrypted LDAPS (Single Domain Search) |
| **AAP Controller Nodes** | Global Catalog Servers | **3269** | TCP | Encrypted LDAPS Global Catalog (Multi-Domain Forest Search) |
| **AAP Controller Nodes** | Internal DNS Servers | **53** | TCP/UDP | Domain Controller Hostname Resolution |

---

### 2. Bind Service Account Specifications

Request an Active Directory service account from the AD Operations team with the following constraints:

* **Account Type:** Standard User Account.
* **Privileges:** Unprivileged (`Domain Users` group only). Needs `Read` and `List Contents` permissions on user OUs.
* **Password Policy:** Password set to never expire (or managed via automated rotation), with **"User cannot change password"** enabled.
* **Target OU Location:** Stored in a dedicated Service Accounts Organizational Unit (e.g., `OU=ServiceAccounts,DC=yourdomain,DC=com`).

---

## Operating System PKI CA Trust Injection

LDAPS connection attempts will fail with `SERVER_DOWN` errors if the underlying Red Hat Enterprise Linux (RHEL) operating system hosting AAP Controller does not trust the issuing Certificate Authority (CA) of the Active Directory Domain Controller certificate.

Execute the following commands on **all AAP Controller nodes** as `root`:

### Step 1: Copy Enterprise Root & Intermediate CA Certificates
Transfer the Base64 PEM-encoded CA certificate files (`.crt` or `.pem`) to the anchor directory:

```bash
cp corp_root_ca.crt /etc/pki/ca-trust/source/anchors/
cp corp_intermediate_ca.crt /etc/pki/ca-trust/source/anchors/
```

### Step 2: Update System CA Trust Store
Force RHEL to rebuild its system-wide trust store:

```bash
update-ca-trust extract
```

### Step 3: Test OpenSSL Handshake
Verify that the OS can establish a trusted TLS connection to the Domain Controller without SSL errors:

```bash
openssl s_client -connect dc01.yourdomain.com:636 -CAfile /etc/pki/tls/certs/ca-bundle.crt
```
*Look for `Verify return code: 0 (ok)` at the end of the output.*

---

## AAP Controller Configuration

Configure LDAP settings in the AAP Controller Web UI under **Settings $\rightarrow$ Authentication $\rightarrow$ LDAP** or inject via JSON payload.

### 1. Server Connection & Bind Settings

```json
{
  "LDAP_SERVER_URI": "ldaps://dc01.yourdomain.com:636 ldaps://dc02.yourdomain.com:636",
  "LDAP_BIND_DN": "CN=svc_aap_bind,OU=ServiceAccounts,DC=yourdomain,DC=com",
  "LDAP_BIND_PASSWORD": "<service_account_password>",
  "LDAP_START_TLS": false,
  "LDAP_REQUIRE_GROUP_TYPE": "ActiveDirectoryGroupType"
}
```

> **Note on Multi-Domain Forests:** If user accounts are spread across multiple domain partitions in an Active Directory forest, change the port from `636` to Global Catalog Port `3269`:  
> `"LDAP_SERVER_URI": "ldaps://dc01.yourdomain.com:3269 ldaps://dc02.yourdomain.com:3269"`

---

### 2. User Search Filter Configuration

This configuration instructs AAP how to locate user objects upon login. The search filter below uses an `OR` condition (`|`) to allow users to log in using either their legacy short username (`sAMAccountName`) or their full UPN format (`userPrincipalName`).

```json
{
  "LDAP_USER_SEARCH": [
    "OU=Users,DC=yourdomain,DC=com",
    "SCOPE_SUBTREE",
    "(|(sAMAccountName=%(user)s)(userPrincipalName=%(user)s))"
  ]
}
```

---

### 3. User Attribute Mapping

Map Active Directory object attributes to AAP internal user profile attributes:

```json
{
  "LDAP_USER_ATTR_MAP": {
    "first_name": "givenName",
    "last_name": "sn",
    "email": "mail"
  }
}
```

---

## Phase 1 Post-Implementation Verification Checklist

Perform the following operational checks to validate that Phase 1 is complete before proceeding to Phase 2 (Group Mapping):

1. **Service Account Bind Test:** Verify that AAP successfully binds to Active Directory without logging `INVALID_CREDENTIALS` errors in `/var/log/tower/tower.log`.
2. **User Profile Population:** Log into AAP with a standard AD user account. Verify that the user's **First Name**, **Last Name**, and **Email Address** are automatically populated in the top-right profile menu.
3. **Password Rejection Test:** Attempt login with an incorrect domain password to confirm that access is denied and logged appropriately.
4. **Local Admin Access Preserved:** Ensure local AAP admin accounts (e.g., `admin`) can still log in normally alongside AD users.
