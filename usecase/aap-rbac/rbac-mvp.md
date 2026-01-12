# RBAC 101: The 3-Tier MVP Model

## Table of Contents

* [Goal](#goal)
* [About Your Team](#about-your-team)
* [Strategy](#strategy)
* [Implementation Checklists](#implementation-checklists)
* [The Result](#the-result)

## Goal

This guide implements a scalable **Separation of Duty** model by dividing responsibilities into three distinct roles: Platform Management, Content Development, and Execution.

## About Your Team

Team layout for an MVP - minimal viable product. The immediate goal is to crawl with RBAC in AAP to produce an MVP. The long term goal is to organize for the enterprise.

Given:

* We have multiple lines of business (automation project teams). Example of automation project teams:
  * RHEL System Administration
  * Windows System Administration
  * VMWare System Administration
  * Network Administration
* Each automation team has at least three distinct personas:
  * Platform Administrator
  * Automation Specialist
  * Help Desk

### Persona Roles (MVP Model)

| Role | Responsibility | Description |
| :--- | :--- | :--- |
| **Platform Admin** | Structure & Security Owner | Owns the **AAP structure**. They create the Organization, Teams, and foundational connectivity. They ensure the platform is secure and structured correctly. |
| **Automation Specialist** | Builder & Primary Operator | The primary builder. They write Ansible playbooks, configure Job Templates, and **execute** the jobs. They have full control over the content they create. |
| **Help Desk** | Consumer (Future/Optional) | The consumer role. Used as the organization scales to provide restricted access for running specific, pre-approved jobs (like password resets) without access to code or secrets. |

### The MVP Architecture

[MVP for RBAC Diagram](./rbac-mvp-diagram1.md)

## Strategy

| Concept | AAP Term | Primary Function | Applied To (The Assets) |
| :--- | :--- | :--- | :--- |
| Division | **Organization** | Defines a secure, top-level division for resources and users. | All resources created within its scope. |
| User Groups | **Team** | Groups users to simplify the assignment of permissions. | Job Templates, Credentials, Inventories. |
| Permissions | **Role** | A pre-defined set of rights (the "key") for a resource. | Organization Structure (Admin) or Automation Resources (Execute/Use). |

## Implementation Checklists

We will use the **RHEL System Administration Project Team** as the example.

### Phase 1: Platform Administration - Create Organization, Teams, and Credentials

The **Platform Administrator** establishes the foundational structure, security boundaries, and user groups for the organization.

* **Step 1:** Create Organization: Navigate to **Access** $\to$ **Organizations** and create the top-level domain: `RHEL-Operations`.
* **Step 2:** Create Teams: Navigate to **Access** $\to$ **Teams** and create the three functional user groups:
  * `RHEL-Admins` (Grant role: **Organization Admin** on the new Org)
  * `RHEL-Specialists` (The Builders & Operators)
  * `RHEL-HelpDesk` (Optional: Future consumers)
* **Step 3:** Add Credential: Navigate to **Resources** $\to$ **Credentials** and create the privileged machine credential (SSH Key) within the Organization scope.

### Phase 2: Automation Specialist - Resource Creation & Association

These actions are performed by the **Automation Specialist**, who is a member of the **`RHEL-Specialists`** team.

* **Step 1:** Create Project: Navigate to **Resources** $\to$ **Projects** and create the Project, linking it to the Git repository containing the playbooks.
* **Step 2:** Create Inventory: Navigate to **Resources** $\to$ **Inventories** and define the host list: `RHEL-Prod-Hosts`.
* **Step 3:** Create Job Template: Navigate to **Resources** $\to$ **Job Templates** and create the job `Restart-Httpd-Service`.
* **Step 4:** Associate Resources: On the `Restart-Httpd-Service` Job Template, **associate** the **Inventory** and **Credential** *(Crucial: This ensures the Help Desk user is not prompted for credentials and cannot see them).*

### Phase 3: Assigning Access - The Separation of Duty

The Platform Admin enforces the separation of roles by assigning permissions to the specific resources created in Phase 2.

* **Step 1: Assign Specialist Access (Build & Run):**
  * Navigate to the **Project** $\to$ Access $\to$ Add Team $\to$ Grant `RHEL-Specialists` the **Project Admin** role (allows sync/update).
  * Navigate to the **Inventory** $\to$ Access $\to$ Add Team $\to$ Grant `RHEL-Specialists` the **Inventory Admin** role (full host list management).
  * Navigate to the **Job Template** $\to$ Access $\to$ Add Team $\to$ Grant `RHEL-Specialists` the **Job Template Admin** role (allows Edit **and** Execute).
* **Step 2: Assign Help Desk Access (Execute Only):**
  * Navigate to Template `Restart-Httpd-Service` $\to$ Access.
  * Add Team `RHEL-HelpDesk` $\to$ Role: **Execute** (Run only).
  * Grant `RHEL-HelpDesk` the **Use** role on the Inventory `RHEL-Prod-Hosts` (allows using hosts in the job).
  * Grant `RHEL-HelpDesk` the **Use** role on the Machine Credential (allows the job to use the key).

## The Result

1. **The Automation Specialist** logs in and has full control to build, maintain, and run their automation.
2. **The Platform Admin** maintains control of the infrastructure and RBAC structure.
3. **(Future)** The **Help Desk** can be granted limited "Launch" access to specific jobs without seeing sensitive data.
