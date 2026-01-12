# Stage 1: The Foundation (Static Automation)

## Introduction

This is the first stage of the Acme Evolution Path. Before scaling to the Ansible Automation Platform (AAP), we must master the core mechanics of "Infrastructure as Code." In this stage, you will perform a **Discovery Scan** to verify system identity and security posture.

This aligns with the **Zero Trust Maturity Model**, specifically **Pillar 2 (Device Identification)** and **Pillar 7 (Visibility)**.

### The Six Key Parts of Stage 1

1. **Project Structure:** Organizing files into a professional, scalable layout.
2. **Inventory:** Defining the manual "Source of Truth" for your targets.
3. **Group Variables:** Securely managing connection settings for different platforms.
4. **Facts:** The discovery engine that identifies system details automatically.
5. **Modules:** The specialized tools used to interact with Operating Systems.
6. **Conditionals (`when`):** Logic that allows one playbook to handle both Linux and Windows.

---

## Part 1: Project Structure

Ansible requires a specific directory structure to apply settings correctly. This organization is the first step in moving away from manual scripts toward managed code.

**Ensure your folder looks like this:**

```text
acme-automation/
├── inventory.ini           # (Part 2) The list of targets
├── discovery_lab.yml       # (Parts 4-6) The automation logic (Playbook)
└── group_vars/             # (Part 3) Folder for group-specific settings
    └── windows.yml         # Connection details for Windows

```

---

## Part 2: Inventory

The inventory file is your map. In Stage 1, we use a **Static Inventory** to manually define the IP addresses or hostnames of our assets.

### `inventory.ini`

```ini
[linux]
192.168.1.10

[windows]
192.168.1.20

[all_servers:children]
linux
windows

```

---

## Part 3: Group Variables

We use `group_vars` to separate **how** we connect from **what** we do. This ensures our playbooks remain generic and reusable.

### `group_vars/windows.yml`

```yaml
---
# Connection settings applied only to the [windows] group
ansible_user: "Administrator"
ansible_password: "SecurePassword123"
ansible_connection: "winrm"
ansible_winrm_server_cert_validation: ignore

```

---

## Parts 4, 5, & 6: Facts, Modules, and Conditionals

The **Playbook** combines these three parts to perform the work. This script is "Read-Only," meaning it gathers intelligence without altering the system.

### `discovery_lab.yml`

```yaml
---
# FOCUS: Pillar 2 (Device) and Pillar 7 (Visibility)
- name: Zero Trust Resource Discovery
  hosts: all
  become: no  
  gather_facts: yes # PART 4: Automatically gathers system intelligence.

  tasks:
    - name: Discovery Scan of Device and OS
      # PART 5: The 'debug' module displays findings to the console.
      debug:
        msg: 
          - "Hostname: {{ ansible_hostname }}"
          - "Family: {{ ansible_os_family }}" 
          - "Distribution: {{ ansible_distribution }} {{ ansible_distribution_version }}"

    # DATA COLLECTOR: Feeds the Zero Trust reporting engine.
    - name: Save Discovery Data to Local Report
      # PART 5: The 'lineinfile' module records findings for auditing.
      lineinfile:
        path: "./discovery_report.txt"
        line: "Host: {{ ansible_hostname }} | OS: {{ ansible_distribution }} | IP: {{ ansible_default_ipv4.address }}"
        create: yes
      delegate_to: localhost 

    # --- LINUX SECTION ---
    - name: Linux Only - Query Firewall Status
      ansible.builtin.service_facts:
      # PART 6: Logic to ensure Linux tools only run on Linux.
      when: ansible_os_family == "RedHat"

    # --- WINDOWS SECTION ---
    - name: Windows Only - Query Firewall Status
      # PART 5: Win_service is the tool for Windows Service Control.
      ansible.windows.win_service:
        name: MpsSvc
      register: win_fw_status
      # PART 6: Logic to ensure Windows tools only run on Windows.
      when: ansible_os_family == "Windows"

    - name: Report Windows Firewall
      debug:
        msg: "Windows Firewall is {{ win_fw_status.state }}"
      when: ansible_os_family == "Windows"

```

---

## Stage 1 Review & Resources

Before moving to **Stage 2**, ensure you can explain why we use `delegate_to: localhost` and how `ansible_facts` supports device identification.

* **Documentation:** [Variables and Facts](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html)
* **Best Practices:** [Ansible Tips and Tricks](https://docs.ansible.com/ansible/latest/tips_tricks/ansible_tips_tricks.html)

---
