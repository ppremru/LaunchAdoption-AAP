# Phase 3: Run — Enterprise SSO & Identity Federation

## Executive Summary

This guide details the implementation of **Phase 3 (Run)** for Active Directory integration in **Ansible Automation Platform 2 (AAP 2)**.

In Phase 3, authentication is offloaded entirely from direct LDAP database queries to an enterprise Identity Provider (IdP) such as **Microsoft Entra ID (Azure AD)**, **Okta**, or **PingFederate** using **SAML 2.0** or **OpenID Connect (OIDC)**. This architecture enforces Multi-Factor Authentication (MFA), Conditional Access policies, and Single Sign-On (SSO) compliance while ensuring AAP never sees, receives, or stores user passwords.

---

## Identity Provider (IdP) Application Registration Prerequisites

Before configuring SAML 2.0 inside AAP Controller, register AAP as an Enterprise Application / Service Provider (SP) within your Identity Provider:

| Parameter | Value / Format | Description |
| :--- | :--- | :--- |
| **Identifier (Entity ID)** | `https://aap.yourdomain.com/sso/metadata/saml/` | Unique SAML identifier for the AAP SP instance. |
| **Reply URL (ACS URL)** | `https://aap.yourdomain.com/sso/complete/saml/` | Assertion Consumer Service endpoint where the IdP sends SAML tokens. |
| **Sign-On URL** | `https://aap.yourdomain.com/sso/login/saml/` | Endpoint triggering the SP-initiated login flow. |
| **SAML Certificate** | Base64 X.509 Certificate (`.pem` / `.crt`) | Certificate used by AAP to validate SAML assertions signed by the IdP. |

---

## Service Provider (SP) Certificate Generation

AAP requires its own X.509 public certificate and private key pair to sign SAML authentication requests sent to the IdP.

Run the following OpenSSL command on an AAP node to generate a self-signed key pair:

```bash
openssl req -new -x509 -days 3650 -nodes \
  -out /etc/tower/saml.crt \
  -keyout /etc/tower/saml.key \
  -subj "/CN=[aap.yourdomain.com/O=Enterprise](https://aap.yourdomain.com/O=Enterprise) IT/C=US"
```

---

## AAP Controller SAML 2.0 Configuration

In the AAP Controller UI, navigate to **Settings $\rightarrow$ Authentication $\rightarrow$ SAML**, or inject the following JSON configuration payload.

### 1. Service Provider & Entity Credentials

```json
{
  "SOCIAL_AUTH_SAML_SP_ENTITY_ID": "[https://aap.yourdomain.com/sso/metadata/saml/](https://aap.yourdomain.com/sso/metadata/saml/)",
  "SOCIAL_AUTH_SAML_SP_PUBLIC_CERT": "-----BEGIN CERTIFICATE-----\n<CONTENTS_OF_SAML_CRT>\n-----END CERTIFICATE-----",
  "SOCIAL_AUTH_SAML_SP_PRIVATE_KEY": "-----BEGIN PRIVATE KEY-----\n<CONTENTS_OF_SAML_KEY>\n-----END PRIVATE KEY-----",
  "SOCIAL_AUTH_SAML_ORG_INFO": {
    "en-US": {
      "name": "enterprise_it",
      "displayname": "Enterprise IT",
      "url": "[https://yourdomain.com](https://yourdomain.com)"
    }
  },
  "SOCIAL_AUTH_SAML_TECHNICAL_CONTACT": {
    "givenName": "AAP Platform Admin",
    "emailAddress": "aap-admin@yourdomain.com"
  },
  "SOCIAL_AUTH_SAML_SUPPORT_CONTACT": {
    "givenName": "Identity Ops",
    "emailAddress": "identity@yourdomain.com"
  }
}
```

---

### 2. Enabled Identity Providers (`SOCIAL_AUTH_SAML_ENABLED_STRATEGIES`)

Define the IdP metadata endpoint, certificate fingerprint, and claim mappings:

```json
{
  "SOCIAL_AUTH_SAML_ENABLED_STRATEGIES": {
    "entra_id": {
      "entity_id": "[https://sts.windows.net/00000000-0000-0000-0000-000000000000/](https://sts.windows.net/00000000-0000-0000-0000-000000000000/)",
      "url": "[https://login.microsoftonline.com/00000000-0000-0000-0000-000000000000/saml2](https://login.microsoftonline.com/00000000-0000-0000-0000-000000000000/saml2)",
      "x509cert": "<CONTENTS_OF_IDP_SIGNING_CERTIFICATE_WITHOUT_NEWLINES>",
      "attr_user_permanent_id": "[http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier](http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier)",
      "attr_first_name": "[http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname](http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname)",
      "attr_last_name": "[http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname](http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname)",
      "attr_username": "[http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name](http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name)",
      "attr_email": "[http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress](http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress)"
    }
  }
}
```

---

### 3. SAML Organization & Team Attribute Mapping

Dynamically assign AAP Organizations and Teams based on SAML claims (such as Azure AD Security Group Object IDs or SAML group assertion arrays) sent in the SAML token:

* **SAML Organization Map (`SOCIAL_AUTH_SAML_ORGANIZATION_MAP`):**
  ```json
  {
    "Default": {
      "users": true,
      "admins": {
        "attr": "[http://schemas.microsoft.com/ws/2008/06/identity/claims/groups](http://schemas.microsoft.com/ws/2008/06/identity/claims/groups)",
        "value": ["11111111-2222-3333-4444-555555555555"],
        "remove": true
      }
    }
  }
  ```

* **SAML Team Map (`SOCIAL_AUTH_SAML_TEAM_MAP`):**
  ```json
  {
    "Windows Operations": {
      "organization": "Default",
      "users": {
        "attr": "[http://schemas.microsoft.com/ws/2008/06/identity/claims/groups](http://schemas.microsoft.com/ws/2008/06/identity/claims/groups)",
        "value": ["aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee"],
        "remove": true
      }
    },
    "Network Infrastructure": {
      "organization": "Default",
      "users": {
        "attr": "[http://schemas.microsoft.com/ws/2008/06/identity/claims/groups](http://schemas.microsoft.com/ws/2008/06/identity/claims/groups)",
        "value": ["ffffffff-4444-4444-4444-aaaaaaaaaaaa"],
        "remove": true
      }
    }
  }
  ```

* **SAML Superuser Mapping (`SOCIAL_AUTH_SAML_USER_FLAGS_BY_ATTR`):**
  ```json
  {
    "is_superuser_attr": "[http://schemas.microsoft.com/ws/2008/06/identity/claims/groups](http://schemas.microsoft.com/ws/2008/06/identity/claims/groups)",
    "is_superuser_value": ["99999999-9999-9999-9999-999999999999"]
  }
  ```

---

## Just-In-Time (JIT) Provisioning & User Lifecycle

SAML 2.0 federation uses **Just-In-Time (JIT) provisioning**:

1. **User Onboarding:** A user accesses AAP for the first time via the SSO URL.
2. **Assertion Evaluation:** AAP validates the IdP signature, extracts user claims (`email`, `first_name`, `last_name`, `groups`), and automatically creates a local user profile.
3. **Dynamic Role Binding:** AAP binds the user to Organizations, Teams, and System flags based on SAML token claims.
4. **Access Revocation:** If a user is deactivated in the central IdP, their SSO token is invalidated centrally, preventing login access to AAP immediately.

---

## Phase 3 Post-Implementation Verification Checklist

Perform the following operational checks to validate SAML 2.0 / SSO integration:

1. **SP-Initiated SSO Test:** Navigate to `https://aap.yourdomain.com`. Click **SAML Login** and verify redirection to the enterprise IdP login page.
2. **MFA Challenge Verification:** Confirm that the IdP prompts for Multi-Factor Authentication (e.g., authenticator push / FIDO2 key) prior to completing redirect back to AAP.
3. **JIT Profile Creation:** Verify that a new user logging in via SAML has their username, email, first name, and last name populated accurately in AAP without manual pre-creation.
4. **Group Assertion RBAC Test:** Confirm that users carrying specific group GUIDs in their SAML assertion are mapped directly to their designated AAP Teams and Organizations.
