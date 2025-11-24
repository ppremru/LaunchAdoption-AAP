# MVP Use Case

## Requirements

Establish an initial RBAC model (MVP) for a small team before scaling to the enterprise.

* **Business Units**:  IT Services for two departments (Transportation and Education)
* **Platform Administrators:** A small team managing the AAP platform
* **Automation Specialists:** A set of users, grouped by domain expertise, who will write, test, and run playbooks
* **Domain Teams:** Groups of that users have specific domain expertise (Windows, RHEL, Network, Security, or Storage).
* **Help Desk:** Prepare for a future team that will only execute pre-approved automation

## Table of Contents

1. [Users](#1-users)
2. [Organizations](#2-organizations)
3. [Organization Administrator Teams](#3-organization-administration-teams)
4. [Domain Specific Teams](#4-domain-specific-teams)
5. [Domain Team Permissions](#5-domain-team-permissions)
6. [Execution Environments](#6-execution-environments)
7. [Credentials Setup](#7-credentials-setup)
8. [Domain Specific Resources](#8-domain-specific-resources)

-----

## 1 Users

This section establishes the user management model, we will:

* **Leverage the Default Admin:** To get started, we will log in with the `admin` user, which was defined in the installation process.  
* **Create Platform Administrators:** We will create three local users dedicated to system administrators, which provides us a path forward to no longer use the default `admin`.
* **Configure External Authentication:** We will introduce the concept that an external Identity Provider (IdP) connection will manage authentication for normal users and teams.

### Create Platform Administrator Users

**Action:** As the `admin`, create three new system administrators.

| Name | User Type | Organization |
| :--- | :--- | :--- |
| `platform-admin-1` | System Administrator | Default |
| `platform-admin-2` | System Administrator | Default |
| `platform-admin-3` | System Administrator | Default |

> *Note: While the UI requires an Organization selection (e.g., Default), the System Administrator User Type grants full platform-wide authority that supersedes this setting.*

### Configure External Authentication

**Action:** As a platform administrator, initiate configuration to the corporate Identity Provider (IdP) for normal users.

| Mapping | Setting | Description |
| :--- | :--- | :--- |
| User Type | Normal User | Platform-wide type to allow a user to log into AAP|
| Organization | Default | Default is the only Organization Defined|

> *Note: Configuring external authentication is out of scope for this document.  Keep in mind that we are at the initial steps of configuration, there will be more updates to the SAML/LDAP mapping as the MVP configuration evolves.  Refer to [APPENDIX - More on Configure External Authentication](#more-on-configure-external-authentication).*

-----

## 2 Organizations

This section establishes the organizational model for our two IT service teams, transportation and education.  Within AAP, **Organizations** align with business units - providing logical boundaries for teams and automation assets.   See [APPENDIX - More on  Organizations](#more-on-organizations) below for further detail.

### Create Organizations

**Action:** As a platform administrator, create two organizations.

| Organization Name | Description (Purpose/Scope) |
| :--- | :--- |
| `IT-services-transportation` | IT Automation Services for the transportation Division |
| `IT-services-education` | Specialized Resources for the education Division|

> *Note:  For the purpose of demonstration we created two organizations, however, for much of this MVP, we are focusing only on `IT-services-transportation`.*

-----

## 3 Organization Administration Teams

This section defines an administration team delegated to manage the organization.  What does this mean?  Members of this team will be granted Organization Administration privileges, but will not have full System Administrative privileges - they will be able to add other teams and resources specific to their organization.

The delegation ensures the Platform Administrators can focus on platform health while the Organization Administrators manage automation artifacts for their specific business unit.

### Create Admin Teams

**Action:** As a platform administrator, create a dedicated admin team for each Organization and grant authority. This is a multi-step process: first create the Team, then via the Organization Access Tab, grant Organization Administration role to the team.
Users will be added via the external authentication.

| Organization | Team Name | Description |  Access Tab - Role |
| :--- | :--- | :--- | :--- |
| `IT-services-transportation` | `admins-transportation` | Manages the IT-services-transportation organization | Organization Admin |
| `IT-services-education` | `admins-education` | Manages the IT-services-education organization | Organization Admin |

> *Note: It is best practice to map an external SAML/LDAP group (e.g., CN=APP_AAP_transportation_Admins) to this team so that administrative access is managed centrally by your Identity Provider.*

## 4 Domain Specific Teams

This section creates Domain Specific Teams which will be used to group users based on their technical domain. This enforces separation of duties, ensuring that specialists will manage only the resources relevant to their expertise (e.g., RHEL vs. Windows).

### Create the Domain Teams

**Action:** As a member of the `admins-transportation` team, create the domain-specific teams to establish functional silos. You are only establishing the team; you will grant permissions to these teams in the [Domain Team Permissions Section](#5-domain-team-permissions) and add users via external authentication.

| Organization | Team Name | Description |
| :--- | :--- | :--- |
| `IT-services-transportation` | `RHEL-Specialists` | Manages RHEL/Linux systems|
| `IT-services-transportation` | `Windows-Specialists` | Manages Windows systems|
| `IT-services-transportation` | `Networking-Specialists` | Manages network devices|
| `IT-services-transportation` | `VMware-Specialists` | Manages VMware virtualization assets|
| `IT-services-transportation` | `Security-Specialists` | Manages security compliance content|

> *Note: Map external SAML/LDAP groups (e.g., AD-Windows-Admins) to these teams to automate user provisioning.*

## 5 Domain Team Permissions

This section defines how to **delegate creation rights** for the domain teams. By default, only Organization Admins can create resources. You must explicitly grant the Domain Teams permission to create their own resources (Projects, Inventories, Job Templates).

### Delegate Resource Creation Rights

**Action:** As a member of the `admins-transportation` team, via the Organization Access Tab, for each Domain Team (e.g., RHEL-Specialists), assign the specific creation roles.  

| Role | Description |
| :--- | :--- |
| **Inventory Admin** | Has all permissions to all inventories in the organization. |
| **Project Admin** | Has all permissions to all projects in the organization. |
| **Credential Admin** | Has all permissions to all credentials in the organization. |
| **Job Template Admin** | Has all permissions to all job templates in the organization. |

> *Critical MVP Note:  Granting these roles at the Organization level allows teams to create new resources, but it also grants administrative access to all resources of that type within the Organization.*
>
> * *Risk: A `RHEL-Specialist` could technically edit a `Windows-Specialist` inventory.*
> * *Mitigation: Use strict Naming Conventions (e.g., always prefixing with `RHEL-` or `Win-`) to prevent accidental crossover.*
>

## 6 Execution Environments

Automation runs inside **Execution Environments (EEs)**, which are container images that hold the necessary tools (like Ansible, Python, and System Roles). Teams cannot run automation unless the correct EE is assigned to their Organization.

**Action:** Log in as a Platform Administrator to create these Execution Environments globally. Then, navigate to the `IT-services-transportation` Organization's **Execution Environments** tab to add them.

| Name | Description |
| :--- | :--- |
| `AAP-Default-EE` | Standard Ansible 2.9+ and base collections. |
| `EE-Supported-RHEL` | Optimized for RHEL (system roles). Intended for RHEL-Specialists. |
| `EE-Supported-Windows` | Contains pywinrm and Windows collections. Intended for Windows-Specialists. |
| `EE-Supported-Network` | Contains network collections. Intended for Networking-Specialists. |

## 7 Credentials Setup

Credentials are encrypted secrets (SSH keys, tokens, passwords) required to access external systems. This section delegates credential creation to the Domain Teams, ensuring that sensitive secrets remain with the subject matter experts.

### Create Credentials

**Action:** Log in as a member of the specific Domain Team (e.g., `RHEL-Specialists`). Navigate to **Resources** -> **Credentials** and create these credentials inside the `IT-services-transporation` Organization.

| Name | Credential Type | Team | Secret Data Needed |
| :--- | :--- | :--- | :--- |
| `RHEL-Git-Access` | Source Control | `RHEL-Specialists` | SSH Private Key (Git) |
| `Windows-Git-Access` | Source Control | `Windows-Specialists` | Personal Access Token / SSH Key |
| `RHEL-SSH-Key` | Machine | `RHEL-Specialists` | SSH Private Key & Sudo Password |
| `Windows-WinRM-Key` | Machine | `Windows-Specialists` | AD Service Account (User/Pass) |
| `Network-Device-Auth`| Machine | `Networking-Specialists` | TACACS/Radius (User/Pass) |

> *Note: It is best practice for Domain Teams to enter these credentials themselves so that Platform Admins and Org Admins do not have to know the specific secrets.*

## 8 Domain Specific Resources

This section details the creation of the primary automation assets. Each Domain Team uses their specific permissions to build the isolated resources required for their technical silo.

### Create Domain Specific Resources

**Action:** Log in as a member of the specific Domain Team. Navigate to the specific Resource type (Inventories, Projects, Templates) to create the following assets.

| Resource Type | Name | Team | Purpose |
| :--- | :--- | :--- | :--- |
| **Inventory** | `RHEL-Prod-Hosts` | `RHEL-Specialists` | Host segregation for Linux |
| **Inventory** | `Windows-Prod-Hosts` | `Windows-Specialists` | Host segregation for Windows |
| **Inventory** | `Network-Prod-Hosts` | `Networking-Specialists` | Host segregation for Network |
| **Project** | `RHEL-Patch-Project` | `RHEL-Specialists` | Container for Linux playbooks |
| **Project** | `Windows-Config-Project` | `Windows-Specialists` | Container for Windows playbooks |
| **Job Template** | `RHEL-Restart-Service` | `RHEL-Specialists` | Runnable content for siloed tasks |
| **Job Template** | `Windows-Disable-User` | `Windows-Specialists` | Runnable content for siloed tasks |

## 9 Checklists

### Day 1: Platform Administrator (Any of the 3)

* **Verify Local Admins:** Confirm `platform-admin-1`, `2`, and `3` are created.
* **Connect IDP:** Configure SAML/LDAP settings.
* **Create Organizations:** Create `IT-services-transportation` and `IT-services-education`.
* **Create Admin Team:** Create `admins-transportation` team inside `IT-services-transportation`.
* **Assign Permissions:** Grant `admins-transportation` the Organization Administrator role.
* **Map IDP Group:** Link SAML/LDAP group to `admins-transportation`.
* **Verify EEs:** Ensure Red Hat supported Execution Environments are present and assigned to `IT-services-transportation`.

### Day 2: Organization Administrator

* **Create Domain Teams:** Create `RHEL-Specialists`, `Windows-Specialists`, etc.
* **Map Groups:** Link SAML/LDAP groups to the Domain Teams.
* **Grant Roles:** For each Domain Team, grant Organization-Level roles: Inventory Admin, Project Admin, Credential Admin, Job Template Admin.
* **Verify Access:** Confirm a Domain Team member can log in and sees the "Create" button for inventories.

### Day 3: Domain Specialist (RHEL Team)

* **Create Git Credential:** Add `RHEL-Git-Access` (Source Control type).
* **Create Machine Credential:** Add `RHEL-SSH-Key` (Machine type).
* **Create Inventory:** Create `RHEL-Prod-Hosts` and add a test IP.
* **Create Project:** Create `RHEL-Patch-Project` and sync with Git.
* **Create Job Template:** Create `RHEL-Restart-Service`, associating the Inventory, Project, and Credentials.
* **Launch:** Run the template and verify success.

-----

## APPENDIX

### More on Organizations

**The simplest, most important best practice:** Start with a single Organization and use Teams for 99% of your access control.

> **The Metaphor:** Think of an Organization as a building and Teams as the locked doors inside it. It is much easier to manage one building with good locks than it is to manage ten separate buildings.

#### 1. The Default: One Organization, Many Teams (Highly Recommended)

This is the most flexible and scalable model for most enterprises.

**Why it is better:**

* **Resource Sharing:** Everyone lives in the same "building." This means a Job Template for the "App Team" can easily *Use* a Project from the "DevOps Team" and a Credential from the "Security Team." This promotes reusability.
* **Simplicity:** You manage all your resources (Inventories, Projects, Credentials) in one place.
* **Clear Delegation:** You use Teams (like `app-operators`, `network-admins`, `auditors`) to control who can do what. This is the correct way to handle separation of duties for groups that are part of the same company.

#### 2. The Exception: When to Create a Second Organization

You should only create a new Organization when you have a strict requirement for **total isolation**. This is for *segregation*, not just *delegation*.

**Ask these questions. If the answer is "yes," you may need a new Organization:**

* **Is there a strict compliance or data silo requirement?**
  * *Example:* "The Finance-SOX team has resources that are part of a legal audit. The regular IT-Admins must be incapable of even seeing that these resources exist. An accidental permission grant cannot be allowed."
* **Am I managing completely separate, unrelated business units?**
  * *Example:* "We are a corporation that owns a Transportation Division and an Education Division. They have zero overlap in users, servers, or playbooks. They are effectively two different companies."
* **Am I a multi-tenant provider?**

-----

### More on Configure External Authentication

#### 1. Initial Setup (Global Identity)

At the very beginning, before you have created your custom Organizations, the only thing you can definitively define for a user is their **User Type**.

* **User Type:** You define them as a **Normal User**. This is a platform-wide setting that simply means "this person can log in, but they are not a Superuser".
* **Default Org:** If they log in right now, before you create anything else, they will typically just land in the **Default** organization because it is the only one that exists.

#### 2. Organization Creation (The Destination)

You then move to **Section 2** and manually create your specific Organizations (e.g., `IT-services-transportation`). Now the "destination" exists.

#### 3. Assigning the Role (Membership)

**This is when the "Member" role becomes relevant.**

Once the Organization exists, you (or your SAML mapping) can assign the user the **Member** role for that specific Organization.

* **The Logic:** You cannot be a "Member" of a club that hasn't been built yet.
* **The Result:** The user logs in, AAP sees they are a **Normal User**, checks the map, sees they belong to `IT-services-transportation`, and grants them the **Member** role for that specific Organization.
