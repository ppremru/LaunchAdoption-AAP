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

The sanity check builds directly on your foundation skill and introduces concepts that will be developed further in AAP.

[Sanity Check](ansible-101-enabled.md)
