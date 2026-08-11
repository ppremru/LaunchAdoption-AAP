# Entra ID SAML Application Registration & Manifest Template

## Executive Summary

This document provides a pre-configured **Microsoft Entra ID (formerly Azure AD)** Enterprise Application manifest and configuration guide for **Ansible Automation Platform 2 (AAP 2)** SAML 2.0 Single Sign-On (SSO).

Identity administrators can import this JSON manifest directly into the Microsoft Entra Admin Center or deploy it via Microsoft Graph PowerShell to automate the creation of the Service Provider (SP) trust, claim assertions, group attribute mappings, and signing certificate parameters.

---

## Technical Attribute Assertion Mapping

The following table maps Microsoft Entra ID user claims to AAP 2 SAML attributes required for user profile population and Role-Based Access Control (RBAC):

| Entra ID Claim Identifier | SAML Claim Value / Source | AAP Setting Equivalent |
| :--- | :--- | :--- |
| `nameidentifier` | `user.userprincipalname` | `attr_user_permanent_id` |
| `givenname` | `user.givenname` | `attr_first_name` |
| `surname` | `user.surname` | `attr_last_name` |
| `emailaddress` | `user.mail` | `attr_email` |
| `groups` | `user.groups` (SecurityGroup Object IDs) | `SOCIAL_AUTH_SAML_TEAM_MAP` |

---

## Entra ID Application Manifest Template

In the Microsoft Entra Admin Center, navigate to **App registrations $\rightarrow$ [Your App] $\rightarrow$ Manifest** and apply the following payload:

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "appId": "11111111-1111-1111-1111-111111111111",
  "acceptMappedClaims": true,
  "accessTokenAcceptedVersion": 2,
  "addIns": [],
  "allowPublicClient": false,
  "appRoles": [
    {
      "allowedMemberTypes": ["User"],
      "description": "Full System Administrator access to AAP 2 Controller.",
      "displayName": "AAP System Administrator",
      "id": "22222222-2222-2222-2222-222222222222",
      "isEnabled": true,
      "origin": "Application",
      "value": "AAP_System_Admin"
    },
    {
      "allowedMemberTypes": ["User"],
      "description": "Standard operator access mapped to AAP Teams.",
      "displayName": "AAP Platform Operator",
      "id": "33333333-3333-3333-3333-333333333333",
      "isEnabled": true,
      "origin": "Application",
      "value": "AAP_Operator"
    }
  ],
  "identifierUris": [
    "[https://aap.yourdomain.com/sso/metadata/saml/](https://aap.yourdomain.com/sso/metadata/saml/)"
  ],
  "informationalUrls": {
    "termsOfService": null,
    "support": null,
    "privacy": null,
    "marketing": null
  },
  "keyCredentials": [],
  "knownClientApplications": [],
  "logoutUrl": "[https://aap.yourdomain.com/sso/complete/saml/](https://aap.yourdomain.com/sso/complete/saml/)",
  "oauth2AllowImplicitFlow": false,
  "oauth2AllowUrlPathMatching": false,
  "oauth2Permissions": [],
  "oauth2RequirePostResponse": false,
  "optionalClaims": {
    "idToken": [],
    "accessToken": [],
    "saml2Token": [
      {
        "name": "email",
        "source": null,
        "essential": false,
        "additionalProperties": []
      },
      {
        "name": "groups",
        "source": null,
        "essential": false,
        "additionalProperties": [
          "emit_as_roles"
        ]
      }
    ]
  },
  "orgRestrictions": [],
  "parentalControlSettings": {
    "countriesBlockedForMinors": [],
    "legalAgeGroupRule": "Allow"
  },
  "passwordCredentials": [],
  "replyUrlsWithType": [
    {
      "url": "[https://aap.yourdomain.com/sso/complete/saml/](https://aap.yourdomain.com/sso/complete/saml/)",
      "type": "Web"
    }
  ],
  "requiredResourceAccess": [
    {
      "resourceAppId": "00000003-0000-0000-c000-000000000000",
      "resourceAccess": [
        {
          "id": "e1fe6dd8-5000-4d87-9e86-22c169965870",
          "type": "Scope"
        },
        {
          "id": "37f7f235-527b-4136-acb5-45ba9d168a7b",
          "type": "Scopes"
        }
      ]
    }
  ],
  "samlMetadataUrl": null,
  "signInUrl": "[https://aap.yourdomain.com/sso/login/saml/](https://aap.yourdomain.com/sso/login/saml/)",
  "signInAudience": "AzureADMyOrg",
  "tags": [
    "WindowsAzureActiveDirectoryCustomSingleSignOnApplication"
  ],
  "tokenEncryptionKeyId": null
}
```

---

## Automated Deployment via Microsoft Graph PowerShell

To register the application programmatically without using the Azure portal UI, execute the following Microsoft Graph PowerShell script:

```powershell
# Connect to Microsoft Entra ID
Connect-MgGraph -Scopes "Application.ReadWrite.All", "Directory.ReadWrite.All"

# Define Application Parameters
$appName = "Ansible Automation Platform 2 (AAP)"
$entityId = "[https://aap.yourdomain.com/sso/metadata/saml/](https://aap.yourdomain.com/sso/metadata/saml/)"
$replyUrl = "[https://aap.yourdomain.com/sso/complete/saml/](https://aap.yourdomain.com/sso/complete/saml/)"

# Create Enterprise Application Registration
$app = New-MgApplication -DisplayName$appName `
    -IdentifierUris $entityId `
    -SignInAudience "AzureADMyOrg"

# Configure Web Redirect Reply URLs
Update-MgApplication -ApplicationId $app.Id `
    -Web @{
        RedirectUris = @($replyUrl)
    }

# Create Associated Service Principal
$sp = New-MgServicePrincipal -AppId $app.AppId

# Configure Group Claims Assertion (Security Groups Only)
$groupConfig = @{
    groupMembershipClaims = "SecurityGroup"
}
Update-MgApplication -ApplicationId $app.Id -BodyParameter $groupConfig

Write-Host "AAP Enterprise App Registered Successfully. App ID: $($app.AppId)"
```

---

## Post-Registration Verification

1. **Verify Certificate Export:** Navigate to **Single sign-on $\rightarrow$ SAML Certificates** in Entra ID and download the **Certificate (Base64)**. Paste this certificate string into AAP's `SOCIAL_AUTH_SAML_ENABLED_STRATEGIES` setting under `x509cert`.
2. **Assign Test Users:** Under **Users and groups**, assign test security groups to the application.
3. **Validate Group GUID Claims:** Ensure group assertions emit standard Azure Object GUIDs (e.g., `a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d`) to match against `SOCIAL_AUTH_SAML_TEAM_MAP` inside AAP.
