# My AAP: MVP Configuration Plan


*** WIP ***
WORK IN PROGRESS!!!!

Use case:
* Experimenting with AAP and RBAC for a "small team".  
* Small set of users that will maintain the AAP platform.
* Another set of users that will write, test, and run the playbooks.
* The users have domain knowledge/deeds such as windows, rhel, network, security, storage ... 
* In the future, there might be a team of people that only run code (help-desk).

## Table of Contents

1.  [Users](#1-users)
2.  [Organizations](#2-organizations)
3.  [Organization Administrator Teams](#3-organization-administor-teams)
4.  [Domain Specific Teams](#4-domain-specific-teams)
5.  [Domain Team Permissions](#5-domain-team-permissions)
6.  [Execution Environments](#6-execution-environments)
7.  [Credentials Setup](#7-credentials-setup)
8.  [Domain Specific Resources](#8-domain-specific-resources)
9.  [Checklists](#9-checklists)

-----

## 1 Users

Action: One of the Platform Administrators creates the local emergency accounts and configures the connection to the corporate Identity Provider (IdP).

### Platform Administrator Users

Use the default `admin` account to create and verify the platform-admin 1/2/3 accounts.

> Create local user-type System Administrator accounts for emergency access or "break glass" scenarios. Generally, user accounts are provided through an external authentication for centralized management and auditing; however, local system administrator accounts serve as a critical backup function.

| Name | User Type | Organization |
| :--- | :--- | :--- |
| `platform-admin-1` | System Administrator | Default |
| `platform-admin-2` | System Administrator | Default |
| `platform-admin-3` | System Administrator | Default |

> In this case, the Organization field does not limit the user's authority. The System Administrator role grants platform-wide structural control, regardless of the Organization setting.

### External Authentication Users

One of the platform administrators should configure the connection to the corporate Identity Provider (IdP) for standard users (EG: SAML/LDAP user mapping). The user role typically mapped to standard users is Member.

> Setting up external authentication is out of scope for this document.

## 2 Organizations

Action: One of the Platform Administrators creates the organization to house all teams and resources.

| Organization Name | Description (Purpose/Scope) |
| :--- | :--- |
| `primary-services` | Primary IT Automation Services |
| `secondary-services` | Specialized Resources |

## 3 Organization Administrator Teams

Action: One of the Platform Administrators creates these teams inside the Organizations and delegates control to them, removing the dependency on Platform Admins.

| Organization | Team Name | Description |
| :--- | :--- | :--- |
| `primary-services` | `primary-admins` | Manages the primary-services organization. |
| `secondary-services` | `seconary-admins` | Manages the secondary-services organization. |

> Note: One of the platform administrators should map (delegate) an external SAML/LDAP group to this group so that administrators are managed centrally.

## 4 Domain Specific Teams

Action: The Organization Administrator (`primary-admins`) creates these teams to separate duties by technical domain.

| Organization | Team Name | Description |
| :--- | :--- | :--- |
| `primary-services` | `RHEL-Specialists` | Manages RHEL/Linux systems. |
| `primary-services` | `Windows-Specialists` | Manages Windows systems. |
| `primary-services` | `Networking-Specialists` | Manages network devices. |
| `primary-services` | `VMware-Specialists` | Manages VMware virtualization assets. |
| `primary-services` | `Security-Specialists` | Manages security compliance content. |

> Note: Map external SAML/LDAP groups to these teams to automate user provisioning.

## 5 Domain Team Permissions

Action: The Organization Administrator (`primary-admins`) assigns the following roles to each Domain Team at the Organization Level.

| Role Name | Scope | What it allows the Team to do |
| :--- | :--- | :--- |
| Inventory Admin | Organization | Create and edit Inventories. |
| Project Admin | Organization | Create and sync Projects (git). |
| Credential Admin | Organization | Create Credentials. |
| Job Template Admin | Organization | Create and modify Job Templates. |

> MVP Warning: Granting these roles at the Organization level allows teams to create resources, but also allows them to see resources created by other teams within the same Organization. Strict naming conventions are required to prevent confusion.

## 6 Execution Environments

Action: One of the Platform Administrators ensures images are available globally. The Organization Administrator assigns them to the `primary-services` Organization.

| EE Name | Description | Used By |
| :--- | :--- | :--- |
| `AAP-Default-EE` | Standard Ansible 2.9+ and base collections. | General Purpose |
| `EE-Supported-RHEL` | Optimized for RHEL (system roles). | `RHEL-Specialists` |
| `EE-Supported-Windows` | Contains pywinrm and Windows collections. | `Windows-Specialists` |
| `EE-Supported-Network` | Contains network collections (Cisco, Juniper, etc.). | `Networking-Specialists` |

## 7 Credentials Setup

Action: The Domain Specific Teams create these inside the `primary-services` Organization using their specific secrets.

| Credential Name | Credential Type | Owner Team | Secret Data Needed |
| :--- | :--- | :--- | :--- |
| `RHEL-Git-Access` | Source Control | `RHEL-Specialists` | SSH Private Key (Git) |
| `Windows-Git-Access` | Source Control | `Windows-Specialists` | Personal Access Token / SSH Key |
| `RHEL-SSH-Key` | Machine | `RHEL-Specialists` | SSH Private Key & Sudo Password |
| `Windows-WinRM-Key` | Machine | `Windows-Specialists` | AD Service Account (User/Pass) |
| `Network-Device-Auth`| Machine | `Networking-Specialists` | TACACS/Radius (User/Pass) |

> Security Note: It is best practice for Domain Teams to enter these credentials themselves so that Platform Admins and Org Admins do not have to know the specific secrets (Private Keys/Passwords).

## 8 Domain Specific Resources

Action: The Domain Specific Teams create and maintain their own resources using the credentials and environments defined above.

| Resource Type | Name | Owner / Creator | Purpose |
| :--- | :--- | :--- | :--- |
| Inventories | `RHEL-Prod-Hosts` | `RHEL-Specialists` | Host segregation for Linux. |
| | `Windows-Prod-Hosts` | `Windows-Specialists` | Host segregation for Windows. |
| | `Network-Prod-Hosts` | `Networking-Specialists` | Host segregation for Network. |
| Projects | `RHEL-Patch-Project` | `RHEL-Specialists` | Container for Linux playbooks. |
| | `Windows-Config-Project` | `Windows-Specialists` | Container for Windows playbooks. |
| Job Templates | `RHEL-Restart-Service` | `RHEL-Specialists` | Runnable content for siloed tasks. |
| | `Windows-Disable-User` | `Windows-Specialists` | Runnable content for siloed tasks. |

## 9 Checklists

### Day 1: Platform Administrator (Any of the 3)

  * Verify Local Admins: Confirm `platform-admin-1`, `2`, and `3` are created.
  * Connect IDP: Configure SAML/LDAP settings.
  * Create Organizations: Create `primary-services` and `secondary-services`.
  * Create Admin Team: Create `primary-admins` team inside `primary-services`.
  * Assign Permissions: Grant `primary-admins` the Organization Administrator role.
  * Map IDP Group: Link SAML/LDAP group to `primary-admins`.
  * Verify EEs: Ensure Red Hat supported Execution Environments are present and assigned to `primary-services`.

### Day 2: Organization Administrator

  * Create Domain Teams: Create `RHEL-Specialists`, `Windows-Specialists`, etc..
  * Map Groups: Link SAML/LDAP groups to the Domain Teams.
  * Grant Roles: For each Domain Team, grant Organization-Level roles: Inventory Admin, Project Admin, Credential Admin, Job Template Admin, and Execute.
  * Verify Access: Confirm a Domain Team member can log in and sees the "Create" button for inventories.

### Day 3: Domain Specialist (RHEL Team)

  * Create Git Credential: Add `RHEL-Git-Access` (Source Control type).
  * Create Machine Credential: Add `RHEL-SSH-Key` (Machine type).
  * Create Inventory: Create `RHEL-Prod-Hosts` and add a test IP.
  * Create Project: Create `RHEL-Patch-Project` and sync with Git.
  * Create Job Template: Create `RHEL-Restart-Service`, linking the Inventory, Project, and Credentials.
  * Launch: Run the template and verify success.

-----

> NOTE: This document represents the full MVP stack for ONE vertical (RHEL).
> To complete the platform setup, repeat the "Day 3 Checklist" steps for the other Domain Teams (Windows, Network, etc.) using their respective credentials and playbooks.

----

[RBAC Triangle in AAP](./rbac-triangle.png)

----

