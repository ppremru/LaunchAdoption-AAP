# 05-controller-as-code-automation.md - Configuration-as-Code for Identity & Access

## Executive Summary

This guide details automating Active Directory (AD) and Single Sign-On (SSO) configurations in **Ansible Automation Platform 2 (AAP 2)** using **Configuration-as-Code (CaC)** principles.

By leveraging the Red Hat Ansible Certified Content Collection `infra.controller_configuration`, all authentication settings—such as LDAP bind credentials, user search rules, group mappings, and SAML strategies—can be version-controlled in Git, encrypted via `ansible-vault`, and deployed through automated CI/CD pipelines.

---

## Ansible Collection Requirements

To manage AAP settings as code, install the required collection on your management or CI/CD execution environment:

```yaml
# requirements.yml
collections:
  - name: infra.controller_configuration
    version: ">=2.5.0"
```

Install via CLI:
```bash
ansible-galaxy collection install -r requirements.yml
```

---

## Vault Secret Encryption Matrix

Sensitive attributes (such as the LDAP Bind Password, SAML Private Keys, and IdP certificates) must never be stored in plain text. Store them in an encrypted Ansible Vault file (`vars/vault.yml`).

### Encrypting Secrets via Ansible Vault

```bash
ansible-vault create vars/vault.yml
```

### Vault Variable Definitions (`vars/vault.yml`)

```yaml
---
vault_ldap_bind_password: "SuperSecretServiceAccountPassword123!"

vault_saml_private_key: |
  -----BEGIN PRIVATE KEY-----
  MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQC...
  -----END PRIVATE KEY-----

vault_saml_sp_cert: |
  -----BEGIN CERTIFICATE-----
  MIIDXTCCAkWgAwIBAgIJAL9Y81...
  -----END CERTIFICATE-----
```

---

## Variable Structure for Identity Settings

Define your authentication settings inside a structured variable file (e.g., `vars/controller_auth_settings.yml`). The `infra.controller_configuration.settings` role consumes these key-value pairs directly.

```yaml
---
controller_settings:
  # ===========================================================================
  # Phase 1: Crawl - LDAPS Connection & User Search Settings
  # ===========================================================================
  LDAP_SERVER_URI: "ldaps://dc01.yourdomain.com:636 ldaps://dc02.yourdomain.com:636"
  LDAP_BIND_DN: "CN=svc_aap_bind,OU=ServiceAccounts,DC=yourdomain,DC=com"
  LDAP_BIND_PASSWORD: "{{ vault_ldap_bind_password }}"
  LDAP_START_TLS: false
  LDAP_REQUIRE_GROUP_TYPE: "ActiveDirectoryGroupType"
  LDAP_OPT_REFERRALS: 0

  LDAP_USER_SEARCH:
    - "OU=Users,DC=yourdomain,DC=com"
    - "SCOPE_SUBTREE"
    - "(|(sAMAccountName=%(user)s)(userPrincipalName=%(user)s))"

  LDAP_USER_ATTR_MAP:
    first_name: "givenName"
    last_name: "sn"
    email: "mail"

  # ===========================================================================
  # Phase 2: Walk - Group Search & RBAC Mapping
  # ===========================================================================
  LDAP_GROUP_SEARCH:
    - "OU=Groups,DC=yourdomain,DC=com"
    - "SCOPE_SUBTREE"
    - "(objectClass=group)"

  LDAP_GROUP_TYPE_PARAMS:
    name_attr: "cn"

  LDAP_ORGANIZATION_MAP:
    Default:
      users: "CN=SG_AAP_Default_Users,OU=Groups,DC=yourdomain,DC=com"
      admins: "CN=SG_AAP_Default_Admins,OU=Groups,DC=yourdomain,DC=com"
      remove_users: true
      remove_admins: true

  LDAP_TEAM_MAP:
    Windows Engineering:
      organization: "Default"
      users: "CN=SG_Windows_Admins,OU=Groups,DC=yourdomain,DC=com"
      remove: true
    Linux Operations:
      organization: "Default"
      users: "CN=SG_Linux_Ops,OU=Groups,DC=yourdomain,DC=com"
      remove: true

  LDAP_USER_FLAGS_BY_GROUP:
    is_superuser:
      - "CN=SG_AAP_System_Admins,OU=Groups,DC=yourdomain,DC=com"
    is_system_auditor:
      - "CN=SG_AAP_Security_Auditors,OU=Groups,DC=yourdomain,DC=com"
```

---

## Deployment Playbook

Create a playbook (`deploy_identity_config.yml`) to apply the authentication settings to AAP Controller using the `infra.controller_configuration` collection.

```yaml
---
- name: Apply AAP Active Directory & Authentication Settings
  hosts: localhost
  connection: local
  gather_facts: false

  vars_files:
    - vars/vault.yml
    - vars/controller_auth_settings.yml

  tasks:
    - name: Load Controller Authentication Settings Role
      ansible.builtin.include_role:
        name: infra.controller_configuration.settings
      vars:
        controller_hostname: "{{ lookup('env', 'CONTROLLER_HOST') }}"
        controller_username: "{{ lookup('env', 'CONTROLLER_USERNAME') }}"
        controller_password: "{{ lookup('env', 'CONTROLLER_PASSWORD') }}"
        controller_validate_certs: false
```

---

## Execution & CI/CD Pipeline Integration

### 1. Manual CLI Execution

Execute the playbook locally using an Ansible Vault password prompt or vault password file:

```bash
export CONTROLLER_HOST="[https://aap.yourdomain.com](https://aap.yourdomain.com)"
export CONTROLLER_USERNAME="admin"
export CONTROLLER_PASSWORD="YourAdminPassword"

ansible-playbook -i localhost, deploy_identity_config.yml --ask-vault-pass
```

---

### 2. GitLab CI / GitHub Actions Pipeline Example

Integrate this configuration playbook into a GitOps workflow to ensure any changes committed to the identity repository automatically apply to AAP Controller.

```yaml
# .gitlab-ci.yml
stages:
  - validate
  - deploy

validate_syntax:
  stage: validate
  script:
    - ansible-playbook --syntax-check deploy_identity_config.yml

deploy_identity_settings:
  stage: deploy
  only:
    - main
  script:
    - echo "$VAULT_PASSWORD" > .vault_pass
    - ansible-playbook -i localhost deploy_identity_config.yml --vault-password-file .vault_pass
    - rm -f .vault_pass
```

---

## Phase 5 Post-Implementation Verification Checklist

1. **Idempotency Check:** Re-run the playbook immediately after an initial deployment. Verify that `changed=0` is returned for all tasks.
2. **GitOps Drift Detection:** Ensure manual UI changes to LDAP settings are automatically overwritten during the next pipeline execution.
3. **Vault Security Audit:** Verify that plain-text passwords or private keys are not committed to Git repositories (`git status` and `git diff` clean).
