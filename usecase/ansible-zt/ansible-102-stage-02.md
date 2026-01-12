# Stage 2: The Toolbox (Certified Content Collections)

## Introduction

As an Acme administrator, you shouldn't have to "reinvent the wheel" for common tasks like managing Windows Registries or Cisco Switches. **Collections** allow you to package and distribute playbooks, roles, and modules.

This stage aligns with **Pillar 7 (Visibility)** and **Pillar 6 (Automation/Orchestration)** by standardizing the toolsets used to gather telemetry and enforce security policies.

### The Three Key Parts of Stage 2

1. **Ansible Galaxy:** The community hub for discovering automation content.
2. **Certified Content:** High-assurance collections maintained by Red Hat and partners (Microsoft, Cisco, etc.).
3. **Namespace & Collection Name:** Understanding the "Fully Qualified Collection Name" (FQCN) to ensure you are calling the right tool.

---

## Part 1: Understanding the "Certified" Difference

At Acme, we prioritize **Certified Content** over community-made scripts whenever possible.

* **Community (Galaxy):** Open-source, created by individuals. Great for inspiration, but use with caution in high-security environments.
* **Certified (Automation Hub):** Tested, hardened, and supported by Red Hat and the vendor. These are required for Acme's production **Zero Trust** workflows to ensure supply-chain security.

---

## Part 2: Namespaces and FQCN

To use a collection, you must refer to it by its **Fully Qualified Collection Name (FQCN)**. This prevents conflicts between different modules with the same name.

**The Anatomy of a Tool:**
`microsoft.ad.user`

* **Namespace:** `microsoft` (The owner)
* **Collection:** `ad` (The toolbox)
* **Module:** `user` (The specific tool)

---

## Part 3: Using Collections in a Playbook

In this example, we will use the `microsoft.windows` and `community.general` collections. These provide much deeper "Visibility" into the system than the basic modules used in Stage 1.

### `advanced_discovery.yml`

```yaml
---
- name: Advanced Security Discovery
  hosts: windows
  gather_facts: yes

  # COLLECTIONS: This keyword tells Ansible which toolboxes to open.
  collections:
    - microsoft.windows
    - community.general

  tasks:
    - name: Query Windows Security Updates
      # We are using a specialized module from the microsoft.windows collection
      microsoft.windows.win_updates:
        state: searched
      register: update_results

    - name: Report Update Compliance (Pillar 7)
      debug:
        msg: "Found {{ update_results.found_update_count }} missing security updates."

    - name: Check for Unauthorized Local Users
      microsoft.windows.win_user:
        name: "Guest"
        state: query
      register: guest_status

    - name: Visibility Report
      debug:
        msg: "Security Alert: Guest account is {{ guest_status.state }}"
      when: guest_status.state == "present"

```

---

## Part 4: Managing Collections via Requirements

As you scale at Acme, you won't install collections manually. You will use a `requirements.yml` file. This file allows **AAP** to automatically download the correct tools when your job runs.

### `collections/requirements.yml`

```yaml
---
collections:
  - name: microsoft.windows
    version: 2.1.0
  - name: cisco.ios
  - name: redhat.rhel_system_roles

```

---

## Stage 2 Review & Resources

By the end of Stage 2, you should understand that you are an **assembler** of trusted tools. You use Certified Collections to ensure that Acme's automation is stable, secure, and vendor-supported.

* **Search for Tools:** [Ansible Galaxy](https://galaxy.ansible.com)
* **Acme’s Official Hub:** [Red Hat Automation Hub](https://www.google.com/search?q=https://console.redhat.com/ansible/automation-hub)
* **Documentation:** [Using Collections](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html)

---
