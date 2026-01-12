# 102 -- The Acme Evolution Path: Automation & Zero Trust

## Introduction: Logic Before the Platform

Acme utilizes the **Ansible Automation Platform (AAP)** to implement and enforce **Zero Trust (ZT)** security policies. This evolution path is designed to move newcomers from basic local automation to enterprise-scale security orchestration.

We start on the **Command Line (CLI)** to build a fundamental understanding of Ansible logic. This ensures that when you move into AAP, you can effectively troubleshoot playbooks, interpret execution logs, and design resilient ZT workflows.

---

## The Four Stages of Technical Adoption

### Stage 1: The Foundation (Static Automation)

* **Technical Focus:** YAML syntax, local inventory files, and basic fact gathering.
* **ZT Objective:** **Pillar 2 (Device Identity).** Use Ansible to query system details and verify that an asset matches its expected identity profile.
* **Goal:** Run your first "Discovery" playbook locally against a known target.

### Stage 2: The Toolbox (Certified Content Collections)

* **Technical Focus:** Moving from "Built-in" modules to **Red Hat Certified Collections**.
* **ZT Objective:** **Pillar 7 (Visibility).** Use vendor-supported toolsets (for Windows, Cisco, Cloud) to gather deep telemetry and verify security settings without writing custom code.
* **Goal:** Install and use a Certified Collection to query a specialized platform.

### Stage 3: The Source of Truth (Dynamic Inventory)

* **Technical Focus:** Configuring **Inventory Plugins** for VMware, AWS, or Azure.
* **ZT Objective:** **Continuous Monitoring.** ZT requires real-time knowledge of network assets. Dynamic inventories ensure Ansible always targets the "live" environment rather than an outdated text file.
* **Goal:** Connect Ansible to a cloud provider to discover hosts automatically.

### Stage 4: The Enterprise Hub (Ansible Automation Platform)

* **Technical Focus:** Git-based workflows (IaC), Role-Based Access Control (RBAC), and centralized logging.
* **ZT Objective:** **Pillar 1 (User Identity) & Pillar 6 (Orchestration).** Ensuring only authorized admins trigger remediations and that every change is logged and auditable.
* **Goal:** Sync a Git repository to AAP and execute a Job Template with specific user permissions.

---

## Zero Trust & Technical Resources

### The Frameworks

At Acme, our automation goals are guided by these federal standards:

* **DoD Zero Trust Strategy:** [DoD CIO Library](https://dodcio.defense.gov/library/)
> Focuses on the 7 Pillars: User, Device, App/Workload, Data, Network, Automation, and Visibility.


* **CISA Zero Trust Maturity Model:** [CISA ZTMM](https://www.cisa.gov/resources-tools/resources/zero-trust-maturity-model)

### Training & Documentation

* **Interactive Labs:** [Red Hat Ansible Labs](https://www.redhat.com/en/interactive-labs/ansible)
* **Best Practices:** [Ansible Community Best Practices](https://docs.ansible.com/ansible/latest/tips_tricks/ansible_tips_tricks.html)
* **Zero Trust Strategy:** [Automating Zero Trust with Red Hat](https://www.google.com/search?q=https://www.redhat.com/en/resources/automated-zero-trust-whitepaper)

---

### Summary of the Path

| Phase | Tooling | ZT Alignment |
| --- | --- | --- |
| **Stage 1** | CLI & Local Files | Device Identity |
| **Stage 2** | Certified Collections | Visibility & Telemetry |
| **Stage 3** | Inventory Plugins | Continuous Monitoring |
| **Stage 4** | AAP & Git | Orchestration & RBAC |

---
