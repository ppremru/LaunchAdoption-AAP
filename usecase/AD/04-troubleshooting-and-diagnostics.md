# Pre-Flight Diagnostics, `ldapsearch` & Debugging

## Executive Summary

This guide provides an operational diagnostic framework for troubleshooting Active Directory (AD) and Single Sign-On (SSO) integrations within **Ansible Automation Platform 2 (AAP 2)**. 

To prevent misconfigurations in the AAP UI or Controller-as-Code automation, this document details **pre-flight terminal tests**, step-by-step **CLI verification commands**, **AAP debug logging setup**, and remediation patterns for common authentication failure modes.

---

## Pre-Flight Terminal Diagnostics

Before entering configuration settings into the AAP Controller UI, perform these three CLI checks from an AAP control/execution node as `root` or `awx` user.

### 1. Network Egress & Port Verification
Verify network line-of-sight to Domain Controllers over standard LDAPS (Port 636) or Global Catalog LDAPS (Port 3269):

```bash
nc -zvw3 dc01.yourdomain.com 636
nc -zvw3 dc01.yourdomain.com 3269
```

---

### 2. TLS Certificate & Handshake Validation
Test whether the RHEL system trust store trusts the server certificate chain presented by the Domain Controller:

```bash
openssl s_client -connect dc01.yourdomain.com:636 -showcerts < /dev/null
```
* **Success Indicator:** `Verify return code: 0 (ok)`
* **Failure Indicator:** `Verify return code: 21 (unable to verify the first certificate)`. *Remediation:* Re-import the Root/Intermediate CAs into `/etc/pki/ca-trust/source/anchors/` and execute `update-ca-trust`.

---

### 3. Service Account Bind & Query Validation (`ldapsearch`)
Test the bind credentials and user search filter using `ldapsearch` directly from the command line:

```bash
ldapsearch -x -H ldaps://dc01.yourdomain.com:636 \
  -D "CN=svc_aap_bind,OU=ServiceAccounts,DC=yourdomain,DC=com" \
  -w 'YourServiceAccountPassword' \
  -b "OU=Users,DC=yourdomain,DC=com" \
  "(|(sAMAccountName=jdoe)(userPrincipalName=jdoe@yourdomain.com))" \
  givenName sn mail memberOf
```

---

## Enabling AAP LDAP & SAML Debug Logging

When authentication fails silently or group mappings do not apply, temporarily increase log verbosity inside AAP to capture raw `django-auth-ldap` or `social_core` tracebacks.

### 1. Enable Verbose Logging via UI / API
In the AAP Controller UI, navigate to **Settings $\rightarrow$ System $\rightarrow$ Logging** (or apply via API setting):

* **Log Level (`LOG_AGGREGATOR_LEVEL`):** `DEBUG`
* **Enabled Loggers (`LOG_AGGREGATOR_LOGGERS`):** Ensure `django_auth_ldap` and `social` are present.

Alternatively, set python log levels directly in `/etc/tower/settings.py` (or custom setting file) on single-node deployments:

```python
LOGGING['loggers']['django_auth_ldap'] = {
    'handlers': ['console'],
    'level': 'DEBUG',
}
```

### 2. Inspecting Live Authentication Logs
Run the following commands on the control node while attempting a user login attempt in the web UI:

```bash
# Containerized AAP 2 (Podman)
podman logs -f automation-controller-task

# Systemd / RPM-based Controller Deployment
journalctl -u automation-controller -f -g "django_auth_ldap"
```

---

## Failure Modes & Remediation Matrix

| Symptom / Error Log | Root Cause | Remediation Procedure |
| :--- | :--- | :--- |
| **`SERVER_DOWN: {'desc': "Can't contact LDAP server"}`** | Firewall drop, invalid DNS, or TLS CA certificate verification failure. | 1. Test port with `nc -zv <DC_IP> 636`. <br>2. Run `openssl s_client` to check CA trust. <br>3. Verify hostname in `LDAP_SERVER_URI` matches the DC SSL cert Subject Alternative Name (SAN). |
| **`INVALID_CREDENTIALS: {'desc': 'Invalid credentials'}`** | Bind DN syntax incorrect or bind account password wrong/expired. | 1. Confirm `LDAP_BIND_DN` uses exact Distinguished Name syntax. <br>2. Escaping special characters in password. <br>3. Unlock account in AD. |
| **`OPERATIONAL_ERROR: {'desc': 'LdapReferralError'}`** | AD Domain Controller returned a referral to another domain partition that AAP cannot resolve. | Disable LDAP referral chasing by adding `"LDAP_OPT_REFERRALS": 0` or `false` to AAP extra settings. |
| **User logs in successfully, but Team/Org maps are empty.** | 1. Case-sensitivity mismatch in Group DNs.<br>2. Incorrect `LDAP_GROUP_SEARCH` base.<br>3. User missing from group. | 1. Verify group DNs match AD case exactly.<br>2. Ensure `LDAP_GROUP_TYPE_PARAMS` uses `{"name_attr": "cn"}`.<br>3. Execute `ldapsearch` for the user's `memberOf` attribute. |
| **`SAML login failed: Signature validation failed`** | IdP signing certificate expired or mismatch between IdP and AAP SP certificate. | 1. Export fresh IdP signing certificate from Entra ID / Okta.<br>2. Update `x509cert` field in `SOCIAL_AUTH_SAML_ENABLED_STRATEGIES`. |

---

## LDAP Referral Chasing Fix (`LDAP_OPT_REFERRALS`)

In multi-domain Active Directory forests, standard LDAP queries often trigger referral responses (`LDAPReferralError`) when a query hits a domain boundary, causing authentication timeouts.

To disable referral chasing explicitly, inject the following setting in **Settings $\rightarrow$ Authentication $\rightarrow$ LDAP $\rightarrow$ LDAP Extra Settings**:

```json
{
  "LDAP_OPT_REFERRALS": 0
}
```

---

## Post-Troubleshooting Cleanup

Once authentication issue resolution is verified:

1. **Revert Log Level:** Reset `LOG_AGGREGATOR_LEVEL` back to `INFO` or `WARNING` in AAP settings to prevent log inflation and preserve disk I/O performance.
2. **Rotate Test Credentials:** If service account passwords were used in raw CLI tests (`ldapsearch`), clear command history (`history -c`) or rotate the password if necessary.
``================================================``
