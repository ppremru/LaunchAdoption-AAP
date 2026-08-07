# Use Case: Inventory & Playbooks Guide

This guide explains how to structure your Ansible inventory for Windows Kerberos authentication over WinRM and provides ready-to-use example playbooks.

---

## Table of Contents

* [1. Inventory Connection Architecture](#1-inventory-connection-architecture)
* [2. Inventory Configuration (YAML Format)](#2-inventory-configuration-yaml-format)
* [3. Connection Variables Reference](#3-connection-variables-reference)
* [4. Target Windows Host Prerequisites](#4-target-windows-host-prerequisites)
* [5. Example Playbooks](#5-example-playbooks)
  * [Playbook 1: Basic Connectivity Test (Ping & System Info)](#playbook-1-basic-connectivity-test-ping--system-info)
  * [Playbook 2: Windows Service Management](#playbook-2-windows-service-management)
* [6. Air-Gapped and Disconnected Considerations](#6-air-gapped-and-disconnected-considerations)

---

## 1. Inventory Connection Architecture

When executing Windows automation in Red Hat Ansible Automation Platform (AAP), Kerberos authentication occurs at the transport layer between the Execution Environment (EE) container and target Windows endpoints.

To ensure Kerberos works seamlessly:

* **No Playbook Changes Required:** Playbooks contain standard Windows tasks without hardcoded usernames, passwords, or transport settings.
* **Variables Defined in Inventory:** All WinRM and Kerberos parameters are assigned at the inventory level (Group or Host variables).
* **Fully Qualified Domain Names (FQDNs):** Target host names must use FQDNs so the Kerberos client can request Service Principal Name (SPN) tickets from Active Directory.

---

## 2. Inventory Configuration (YAML Format)

Below is an example of an Ansible inventory defined in YAML. You can store this file in a Git repository connected to AAP as a Project, or define these variables directly inside the AAP Controller UI.

```yaml
---
all:
  children:
    windows_servers:
      vars:
        ansible_connection: winrm
        ansible_winrm_transport: kerberos
        ansible_port: 5986
        ansible_winrm_scheme: https
        ansible_winrm_server_cert_validation: ignore
      hosts:
        win01.yourdomain.com:
        win02.yourdomain.com:
        dc01.yourdomain.com:
```

---

## 3. Connection Variables Reference

| Variable Name | Value | Description |
| :--- | :--- | :--- |
| `ansible_connection` | `winrm` | Specifies Windows Remote Management as the connection plugin. |
| `ansible_winrm_transport` | `kerberos` | Forces Kerberos authentication transport instead of NTLM or Basic. |
| `ansible_port` | `5986` | Sets the standard WinRM HTTPS port. |
| `ansible_winrm_scheme` | `https` | Ensures transport encryption over TLS/SSL. |
| `ansible_winrm_server_cert_validation` | `ignore` | Ignores untrusted/self-signed certificates (set to `validate` if endpoints use enterprise CA certificates). |

---

## 4. Target Windows Host Prerequisites

For a target Windows server to accept Kerberos authentication over WinRM, ensure the following requirements are met:

1. **Domain Joined:** The host must be joined to the Active Directory domain specified in your `krb5.conf`.
2. **WinRM HTTPS Listener (Port 5986):** WinRM must be configured with an active HTTPS listener.
3. **Firewall Rule:** Inbound port `5986` must be allowed on local host firewalls and network security groups.

> **Quick Setup:** WinRM HTTPS remoting can be automated on target hosts using the official Ansible PowerShell bootstrap script:
> `powershell -ExecutionPolicy Bypass -File Configure-RemotingForAnsible.ps1 -EnableCredSSP`

---

## 5. Example Playbooks

### Playbook 1: Basic Connectivity Test (Ping & System Info)

Save this file as `win_ping_test.yml`. This playbook verifies Kerberos connectivity and gathers basic host environment details.

```yaml
---
- name: Verify Windows Kerberos Connectivity
  hosts: windows_servers
  gather_facts: true

  tasks:
    - name: Ping Windows host
      ansible.windows.win_ping:

    - name: Display Host OS and Domain Info
      ansible.builtin.debug:
        msg:
          - "Hostname: {{ ansible_facts['hostname'] }}"
          - "Domain: {{ ansible_facts['domain'] }}"
          - "OS Name: {{ ansible_facts['os_name'] }}"
          - "Architecture: {{ ansible_facts['architecture'] }}"
```

### Playbook 2: Windows Service Management

Save this file as `win_manage_services.yml`. This playbook demonstrates practical administration tasks, such as inspecting and managing the Print Spooler service state.

```yaml
---
- name: Manage Windows Services
  hosts: windows_servers
  gather_facts: false

  tasks:
    - name: Ensure Print Spooler service is running
      ansible.windows.win_service:
        name: Spooler
        state: started
        start_mode: auto
      register: spooler_status

    - name: Output Spooler Service Status
      ansible.builtin.debug:
        var: spooler_status.state
```

---

## 6. Air-Gapped and Disconnected Considerations

When running inventories and playbooks in an air-gapped environment:

* **DNS Name Resolution:** Execution Environment containers must resolve host FQDNs (e.g., `win01.yourdomain.com`) to internal IP addresses using local DNS servers. IP addresses cannot be used in the inventory.
* **Pre-Packaged Collections:** Ensure the required collections (`ansible.windows`, `microsoft.ad`, `ansible.builtin`) are pre-baked into your Execution Environment or mirrored in Private Automation Hub.
* **Time Synchronization:** Verify NTP time synchronization across target Windows hosts, Domain Controllers, and AAP execution nodes. Kerberos ticket validation fails if clock skew exceeds 5 minutes.
