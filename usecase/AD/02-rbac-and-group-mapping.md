# Phase 2: Walk — Role-Based Access Control & Group Mapping

## Executive Summary

This guide details the implementation of **Phase 2 (Walk)** for Active Directory integration in **Ansible Automation Platform 2 (AAP 2)**.

While Phase 1 verified user identity (authentication), Phase 2 automates permissions (authorization). By mapping Active Directory Security Groups directly to AAP Organizations, Teams, and System Administrator roles, access governance is offloaded entirely to Active Directory. When an employee changes roles or leaves the organization, updating their AD group membership instantly updates or revokes their AAP permissions upon their next login.

---

## Group Search & Type Configuration

AAP uses the underlying `django-auth-ldap` library to query Active Directory groups. To properly evaluate Active Directory group objects—including nested group memberships—AAP must be instructed to use the `ActiveDirectoryGroupType`.

In the AAP Controller UI under **Settings $\rightarrow$ Authentication $\rightarrow$ LDAP**, define the group search parameters:

### 1. Group Search Filter (`LDAP_GROUP_SEARCH`)

Define the Base DN where AAP searches for Security Groups, the scope, and the filter matching group objects:

```json
{
  "LDAP_GROUP_SEARCH": [
    "OU=Groups,DC=yourdomain,DC=com",
    "SCOPE_SUBTREE",
    "(objectClass=group)"
  ]
}
```

### 2. Group Type Parameters (`LDAP_GROUP_TYPE_PARAMS`)

Set the attribute name used for AD group objects (typically `cn` in Active Directory):

```json
{
  "LDAP_GROUP_TYPE_PARAMS": {
    "name_attr": "cn"
  }
}
```

---

## Role-Based Access Control (RBAC) Mapping

AAP allows mapping AD Security Groups to three distinct levels of authority: Organizations, Teams, and System Superuser flags.

### 1. Organization Mapping (`LDAP_ORGANIZATION_MAP`)

Organization Mapping grants users membership or administrative rights within an AAP Organization. 

> **Critical Rule:** Setting `"remove_users": true` and `"remove_admins": true` ensures that if a user is removed from the designated AD Security Group, AAP automatically revokes their Organization access during their next login.

```json
{
  "Default": {
    "users": "CN=SG_AAP_Default_Users,OU=Groups,DC=yourdomain,DC=com",
    "admins": "CN=SG_AAP_Default_Admins,OU=Groups,DC=yourdomain,DC=com",
    "remove_users": true,
    "remove_admins": true
  },
  "Finance Automation": {
    "users": "CN=SG_AAP_Finance_Users,OU=Groups,DC=yourdomain,DC=com",
    "admins": "CN=SG_AAP_Finance_Admins,OU=Groups,DC=yourdomain,DC=com",
    "remove_users": true,
    "remove_admins": true
  }
}
```

---

### 2. Team Mapping (`LDAP_TEAM_MAP`)

Team Mapping assigns users to specific Teams within an AAP Organization. Job Template permissions, Inventory access, and Credentials are typically granted to Teams rather than individual users.

```json
{
  "Windows Engineering": {
    "organization": "Default",
    "users": "CN=SG_Windows_Admins,OU=Groups,DC=yourdomain,DC=com",
    "remove": true
  },
  "Linux Operations": {
    "organization": "Default",
    "users": "CN=SG_Linux_Ops,OU=Groups,DC=yourdomain,DC=com",
    "remove": true
  },
  "Network Engineering": {
    "organization": "Default",
    "users": "CN=SG_Network_Engineers,OU=Groups,DC=yourdomain,DC=com",
    "remove": true
  }
}
```

---

### 3. System-Wide Superuser & Auditor Mapping (`LDAP_USER_FLAGS_BY_GROUP`)

To grant global platform administrator or read-only system auditor rights based on AD membership, configure `LDAP_USER_FLAGS_BY_GROUP`:

```json
{
  "is_superuser": [
    "CN=SG_AAP_System_Admins,OU=Groups,DC=yourdomain,DC=com"
  ],
  "is_system_auditor": [
    "CN=SG_AAP_Security_Auditors,OU=Groups,DC=yourdomain,DC=com"
  ]
}
```

---

## Active Directory Nested Group Membership Handling

In complex Active Directory environments, Security Groups are often nested inside other Security Groups (e.g., `SG_Windows_Admins` is a member of `SG_AAP_Default_Users`).

AAP’s `ActiveDirectoryGroupType` natively resolves nested group structures during the login handshake using AD's internal `memberOf` evaluation. 

### How AAP Handles Group Nesting

1. When a user logs in, AAP queries AD for the user's Distinguished Name (DN).
2. `ActiveDirectoryGroupType` evaluates direct group memberships as well as recursively inherited group memberships.
3. AAP matches the resulting list of group DNs against `LDAP_ORGANIZATION_MAP`, `LDAP_TEAM_MAP`, and `LDAP_USER_FLAGS_BY_GROUP`.

> **Performance Tip:** If your Active Directory has deep group nesting trees (e.g., more than 5 levels deep), restrict `LDAP_GROUP_SEARCH` to a specific Organizational Unit (OU) containing only AAP-relevant groups to keep login response times under 1 second.

---

## Phase 2 Post-Implementation Verification Checklist

To verify that Phase 2 RBAC and automated access lifecycle enforcement are working correctly, perform these four checks:

1. **Team Assignment Test:** Log in with a user account present in `SG_Windows_Admins`. Navigate to **Access $\rightarrow$ Users $\rightarrow$ [User Name] $\rightarrow$ Teams** and verify the user is automatically added to the "Windows Engineering" team.
2. **Superuser Grant Test:** Log in with a user in `SG_AAP_System_Admins`. Verify the user has full system access and that the "System Administrator" flag is toggled on in their profile.
3. **Automated Revocation Test:** 
   * Remove a test user from `SG_Windows_Admins` in Active Directory.
   * Have the test user log out of AAP and log back in.
   * Verify the user has been automatically removed from the "Windows Engineering" team in AAP.
4. **Audit Trail Verification:** Inspect **Access $\rightarrow$ Activity Stream** to confirm AAP logs group-driven team additions and removals during user authentication events.
