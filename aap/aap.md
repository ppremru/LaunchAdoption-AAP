# Enablement Recommendations: Ansible Automation Platform

## Goals and Why It Matters

The goal is to understand how Ansible Automation Platform (AAP) *runs, scales, and governs* the automation you learned to write in the previous sections.

This document focuses on the core platform components a developer interacts with. It does **not** cover installation or day-2 operations of the platform itself.

* **Understand the Controller:** Learn how the **Automation Controller** (formerly Tower) acts as the web UI and API for all automation. This is where you will launch jobs, manage credentials, and view reports.
* **Use Job Templates:** Understand that a **Job Template** is the "run button" in AAP. It combines your **Project** (from Git), **Inventory** (targets), and **Credentials** (secrets) into a single, repeatable, and auditable job.
* **Connect to Git (Projects):** See how AAP **Projects** link directly to your Git repositories. This is the "GitOps" connection: AAP pulls your playbooks from Git to run them.
* **Manage Secrets (Credentials):** Learn how AAP securely stores **Credentials** (like SSH keys or API tokens). This allows you to run automation without ever exposing your secrets in your playbooks.
* **Use EEs and Automation Hub:** Understand that AAP runs all jobs inside **Execution Environments (EEs)**. **Automation Hub** is the registry where AAP finds and stores your EEs and Ansible Collections.
* **Chain Jobs (Workflows):** Learn how **Workflows** let you link multiple Job Templates together to build powerful, end-to-end automation (e.g., "provision server" -> "configure app" -> "run tests").

## Key References

* [Red Hat Ansible Automation Platform](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.6)
* [Red Hat Developer - Automation Topics](https://developers.redhat.com/topics/automation)
* [Red Hat Blog - Ansible Channel](https://www.redhat.com/en/blog/channel/red-hat-ansible-automation)

## RHLS Core Recommendations

* [Take a free skills assessment to see where you should start training](https://skills.ole.redhat.com/en) :star:
* [Red Hat Ansible Automation Platform Skills Path](https://www.redhat.com/en/resources/ansible-automation-platform-skills-path-brief)
* [Developing Advanced Automation with Red Hat Ansible Automation Platform AU374](https://www.redhat.com/en/services/training/au374-developing-advanced-automation-red-hat-ansible-automation-platform)
* [Managing Enterprise Automation with Red Hat Ansible Automation Platform D467](https://www.redhat.com/en/services/training/do467-managing-enterprise-automation-red-hat-ansible-automation-platform)

## Independent Learning Path Recommendations

### Labs

* [Interactive labs for Red Hat Ansible Automation Platform](https://www.redhat.com/en/interactive-labs/ansible) :star:
* [Red Hat Ansible Automation Platform learning hub](https://www.redhat.com/en/technologies/management/ansible/learn) :star:

### Reading Material and References

* [Red Hat Ansible Automation Platform: A beginner’s guide](https://www.redhat.com/en/resources/ansible-automation-platform-beginners-guide-ebook)
* [Use Cases - End-to-end automation with a single platform](https://www.redhat.com/en/technologies/management/ansible/use-cases) :star: :star:
* [Red Hat Ansible Automation Platform learning hub](https://www.redhat.com/en/technologies/management/ansible/learn) :star:

### Videos

* [A video tour of Ansible Automation Platform 2](https://www.youtube.com/watch?v=7GJjhZoYEus&list=PLdu06OJoEf2bp-PNtxPP_2n7Avkax8TED&index=2)

---

## Sanity Check: Key AAP Components for a Developer

You've learned how to write automation (`ansible.md`) and build containers (`container.md`). Ansible Automation Platform (AAP) is the platform that brings all those skills together. Here are the core components you will interact with as a developer:

| Component | What It Is (The "Why") | Connects To... |
| :--- | :--- | :--- |
| **Automation Controller** | The web UI and API for running, scheduling, and auditing all automation. | *Everything* |
| **Project** | A pointer to a specific **Git Repository** in your version control system. | `foundation.md` (Git) |
| **Playbook** | The `.yml` file (inside your Project) that defines your automation tasks. | `ansible.md` (Ansible) |
| **Inventory** | The list of targets (servers, devices) you want to run automation against. | `ansible.md` (Ansible) |
| **Credential** | An encrypted object in the Controller that stores your secrets (SSH keys, API tokens). | `ansible.md` (Ansible) |
| **Execution Env (EE)** | The **Container Image** that your automation runs inside. | `container.md` (Containers) |
| **Automation Hub** | The **Container Registry** where EEs and Ansible Collections are stored and managed. | `container.md` (Containers) |
| **Job Template** | The "Run Button." It combines a **Project**, **Playbook**, **Inventory**, **Credential**, and **EE**. | *Everything* |
| **Workflow** | A visual tool to chain multiple **Job Templates** together in a specific order. | *Everything* |
