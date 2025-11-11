# Enablement Recommendations: Ansible 101

## Goals and Why It Matters

The goal is to develop the core skills for writing and running Ansible automation. These are the "building blocks" of everything you will do, and they are the code that Ansible Automation Platform (AAP) will run and manage.

Do you have a crawl-walk level of these skills:

* **Understand Inventory:** Define *what* servers and devices you want to automate. This is the first step, as AAP needs a target list (an inventory) before it can run any job.
* **Write Playbooks, Roles, & Collections:** Use these components to write the *how*—the actual automation tasks. This is the code that AAP will pull from Git to execute.
* **Develop Troubleshooting Skills:** Learn how to read and debug Ansible's output. This is a critical developer skill for quickly fixing failed automation jobs in or out of AAP.
* **Lint Your Code:** Use tools like `ansible-lint` to check your code for errors and best practices *before* you commit it. This ensures your automation is clean, maintainable, and less likely to fail in production.

> Note:  
> 1: The focus is on building core ansible skills using ansible-core, not AAP
> 2: These skills can be learned either with or without the Ansible Automation Platform (AAP)
> 3: [AAP topics](../aap/aap.md) are introduced in a different section along with use cases

## Key References

* [Ansible Community Documentation](https://docs.ansible.com/)
* [Good Practices for Ansible](https://redhat-cop.github.io/automation-good-practices/)
* [Ansible Forum Community Q&A Forum](https://forum.ansible.com/)

## RHLS Core Recommendations

* [Take a free skills assessment to see where you should start training](https://skills.ole.redhat.com/en) :star:
* [Red Hat Enterprise Linux Automation with Ansible (RH294)](https://www.redhat.com/en/services/training/rh294-red-hat-linux-automation-with-ansible)
* [Red Hat Certified Engineer (RHCE) exam (EX294)](https://www.redhat.com/en/services/training/ex294-red-hat-certified-engineer-rhce-exam-red-hat-enterprise-linux-9)

## Independent Learning Path Recommendations

### Labs

* [Getting started with Red Hat Ansible Automation Platform](https://cloud.redhat.com/learn/getting-started-red-hat-ansible-automation-platform) 🥇
* [Ansible Basics: Automation Technical Overview](https://www.redhat.com/en/services/training/do007-ansible-essentials-simplicity-automation-technical-overview) (no charge)
* [Red Hat Ansible Automation Platform learning hub](https://www.redhat.com/en/technologies/management/ansible/learn) 🥇

### Reading Material and References

* [Ansible vs. Red Hat Ansible Automation Platform](https://www.redhat.com/en/technologies/management/ansible/ansible-vs-red-hat-ansible-automation-platform)  :star:
* [Learning Ansible Basics](https://www.redhat.com/en/topics/automation/learning-ansible-tutorial) :star:
* [What is an Ansible Playbook?](https://www.redhat.com/en/topics/automation/what-is-an-ansible-playbook)
* [What is an Ansible Role—and how is it used?](https://www.redhat.com/en/topics/automation/what-is-an-ansible-role)  
* [What is an Ansible Module—and how does it work?](https://www.redhat.com/en/topics/automation/what-is-an-ansible-module#creating-and-sharing-ansible-modules)  
* [What is Ansible automation hub and why should you use it?](https://www.redhat.com/en/blog/what-ansible-automation-hub-and-why-should-you-use-it)  
* [Ansible Content Collections](https://www.redhat.com/en/technologies/management/ansible/content-collections)  
* [Find mistakes in your playbooks with Ansible Lint](https://www.redhat.com/en/blog/ansible-lint)  
* [6 troubleshooting skills for Ansible playbooks](https://www.redhat.com/en/blog/troubleshoot-ansible-playbooks)  

### Ebooks

* [Tales from the field: A system administrator's guide](https://www.redhat.com/rhdc/managed-files/co-system-administrators-guide-to-IT-automation-ebook-1933814OM-202503-en.pdf)

### Videos

* [Ansible Video Channel on YouTube](https://www.youtube.com/playlist?list=PLdu06OJoEf2ZWrbPxrQwktHsN1wYzYtHx)
* [Alex Dworjan's Ansible Video YouTube Channel](https://www.youtube.com/watch?v=goclfp6a2IQ&list=PL2_OBreMn7FqZkvMYt6ATmgC0KAGGJNAN)  

---

## Sanity Check

### Your First "Hello World" (Ad-Hoc Command)

Before you write a full playbook, your first step is to use the `ansible` command-line tool to run a single task. This is called an **ad-hoc command**.

This exercise builds directly on your `foundation.md` and `rhel.md` skills. You will use the `inventory.yml` file you already built to run a basic `ping` module against a host.

1. **Install Ansible:** On your RHEL machine, install `ansible-core`:

    ```bash
    sudo dnf install ansible-core
    ```

2. **Define Inventory:** Use the `inventory.yml` file you created in the `foundation.md` exercise.

    ```yaml
    ---
    all:
      hosts:
        webserver-01:
          ansible_host: 192.168.1.10
    ```

3. **Run the Command:** From your terminal, run the following command:

    ```bash
    # ansible [inventory] -m [module] [host_pattern]
    ansible -i inventory.yml -m ping all
    ```

4. **Expected Output:** If successful, the `ping` module will return a "pong," confirming you have a working inventory and a valid connection.

    ```json
    webserver-01 | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python3"
        },
        "changed": false,
        "ping": "pong"
    }
    ```

> This simple success is the foundation for all other Ansible skills.

### Key Troubleshooting Skills

> **Note on `ansible-core`:**
> `ansible-core` is the name of the software package you install. The *commands* it provides for running automation are `ansible` (for ad-hoc commands) and `ansible-playbook` (for running playbooks). The table below lists the correct commands and flags for troubleshooting.

When a playbook fails, your first instinct should be to use the verbosity flags. This is the most important troubleshooting skill you can learn.

| Command | What it does |
| :--- | :--- |
| `ansible-playbook my_playbook.yml -v` | **Verbose.** Shows you the basic output for each task. |
| `ansible-playbook my_playbook.yml -vv` | **Very Verbose.** Good for seeing module inputs. |
| `ansible-playbook my_playbook.yml -vvv` | **Extra Verbose.** Shows all details, including connection information. |
| `ansible-playbook my_playbook.yml --check` | **Check Mode.** Runs the playbook without making any changes. |
| `ansible-playbook my_playbook.yml --syntax-check` | **Syntax Check.** Verifies your playbook's YAML syntax is valid. |
| `ansible-lint my_playbook.yml` | **Linter.** Checks your playbook for best practices and potential errors. |

---
