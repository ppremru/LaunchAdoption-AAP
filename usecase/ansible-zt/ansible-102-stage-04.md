# Stage 4: The Enterprise Hub (Ansible Automation Platform)*

In a **Zero Trust** environment, the CLI is risky because it lacks centralized auditing and access control. Stage 4 solves this by providing a governed "Control Plane" for all your Stage 1, 2, and 3 work.

## Introduction

Up until now, you have been running automation from your own workstation. In Stage 4, you will move your code into **Git** and use **AAP** to execute it. This transition allows Acme to enforce security policies at scale, with full visibility into who ran what, when, and where.

This aligns with **Pillar 1 (User Identity)** and **Pillar 6 (Automation & Orchestration)**.

### The Three Key Parts of Stage 4

1. **Version Control (Git):** The "Single Source of Truth" for your code.
2. **Role-Based Access Control (RBAC):** Defining exactly which admins can run which playbooks.
3. **Job Templates:** Creating "Push-Button" automation that anyone (with permission) can use.

---

## Part 1: The Git Workflow (Infrastructure as Code)

At Acme, we no longer keep playbooks on our local hard drives. They live in a Git repository.

* **Review:** Changes are proposed via "Pull Requests" and peer-reviewed.
* **Sync:** AAP automatically pulls the latest code from Git.
* **Audit:** We have a permanent record of every line of code ever changed.

---

## Part 2: Governance with RBAC**

In Stage 1, if you had the SSH key, you could do anything. In Stage 4, AAP acts as the **Policy Enforcement Point**.

* **Least Privilege:** You can give a junior admin permission to run a "Security Scan" playbook without giving them the actual root passwords or SSH keys.
* **Credential Management:** AAP stores the sensitive passwords (for Windows or Linux) in an encrypted vault. The admin never sees the password; they only see the "Run" button.

---

## Part 3: Creating a Job Template

A Job Template is the "App-ification" of your playbook. It combines your **Playbook (from Git)**, your **Inventory (from Stage 3)**, and your **Credentials**.

**The Job Template View in AAP:**

* **Inventory:** "Acme-AWS-Dynamic-Inventory"
* **Project:** "Security-Audit-Playbooks"
* **Playbook:** `discovery_lab.yml`
* **Credentials:** "Domain-Admin-Vault"

---

## How Stage 4 Completes the Zero Trust Mission

* **Pillar 1 (User Identity):** AAP integrates with Acme's Active Directory/SSO. We know exactly which human triggered an automation task.
* **Pillar 6 (Orchestration):** AAP can coordinate complex workflows (e.g., "Scan the server, if it fails, quarantine it in the firewall, then notify the security team").
* **Centralized Logging:** Every task performed by Ansible is logged in AAP. This provides the "Audit Trail" required for federal compliance and incident response.

---

## Stage 4 Review & Resources

Stage 4 is the "Final State" for an Acme administrator. Your goal is to move all the discovery and enforcement logic you learned in the previous stages into this governed platform.

* **Interactive Lab:** [Writing Your First Controller Job Template](https://www.redhat.com/en/interactive-labs/ansible)
* **Video:** [Understanding Ansible Controller RBAC](https://www.ansible.com/resources/get-started)
* **Documentation:** [AAP Controller User Guide](https://docs.ansible.com/automation-controller/latest/html/userguide/index.html)

---

### Congratulations!

You have completed the **Acme Evolution Path**. You started with a single line of YAML and ended with an Enterprise-grade security orchestration platform.
