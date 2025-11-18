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

### Phase 1: Platform Admin (Structure & Access)

* **Create Organization:** Create `RHEL-Operations`.
* **Create Teams:** Create the three functional teams:
    * `RHEL-Admins` (Assign role: Organization Admin)
    * `RHEL-Specialists` (The Builders & Operators)
    * `RHEL-HelpDesk` (Optional: Future consumers)
* **Add Credential:** Add the privileged machine credential (SSH Key) to the Organization so Specialists can use it.

### Phase 2: Automation Specialist (Content & Logic)

* **Create Project:** Connect AAP to the Git repository containing the playbooks.
* **Create Inventory:** Define the `RHEL-Prod-Hosts` list.
* **Create Job Template:** Create the job `Restart-Httpd-Service`.
* **Attach Resources:** **Crucial:** Attach the *Inventory* and *Credential* to the Template now so the Help Desk user is not prompted later.

### Phase 3: Assigning Access

The Platform Admin or Team Lead delegates access to the specific resources created by the Automation Specialist.

* **Assign Specialist Access (Build & Run):**
    * Grant `RHEL-Specialists` -> **Project Admin** role on the Project (allows sync/update).
    * Grant `RHEL-Specialists` -> **Inventory Admin** role on `RHEL-Prod-Hosts` inventory.
    * Grant `RHEL-Specialists` -> **Job Template Admin** role on Job Templates (allows Edit **and** Execute).
* **Assign Help Desk Access (Future Scale):**
    * Navigate to Template `Restart-Httpd-Service` -> Access.
    * Add Team `RHEL-HelpDesk`.
    * Role: **Execute** (Run only).

## The Result

1. **The Automation Specialist** logs in and has full control to build, maintain, and run their automation.
2. **The Platform Admin** maintains control of the infrastructure and RBAC structure.
3. **(Future)** The **Help Desk** can be granted limited "Launch" access to specific jobs without seeing sensitive data.
