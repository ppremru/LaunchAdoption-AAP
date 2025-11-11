# Enablement Recommendations: Red Hat Enterprise Linux (RHEL)

## Why RHEL for an Automation Specialist?

As an automation specialist, you will interact with RHEL in two primary ways. Understanding the basics of RHEL is foundational because it is:

1. **The Control Plane:** Ansible Automation Platform runs on Red Hat Enterprise Linux. Your central AAP cluster, control nodes, and automation mesh nodes will all be RHEL-based.
2. **The Target:** A primary goal of automation is to manage other RHEL servers (and other operating systems) at scale. These are your "managed nodes."

This document provides the foundational skills to confidently operate in both of these roles—managing the platform itself and building reliable automation to manage other RHEL endpoints.

## Goals

Enablement goals include developing your RHEL skillset via RHLS or independent learning. Do you have a crawl-walk understanding of topics such as these:

- **Essential Linux commands** (navigating, viewing files, managing permissions).
- **Package management** using `dnf` (installing, removing, and updating software).
- **Service management** using `systemd` (starting, stopping, and enabling services).
- **Subscription management** (registering a system to get updates).
- **Understand RHEL Universal Base Image (UBI)** for building and running containerized automation.
- **Traverse the Web Console** for a visual overview of a system.

## Key References

- [Red Hat Enterprise Linux 9 Documentation](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9)
- [How to Register and Subscribe a RHEL System](https://access.redhat.com/solutions/253273)
- [Red Hat Enterprise Linux Blog](https://www.redhat.com/en/blog/channel/red-hat-enterprise-linux)
- [Red Hat Enterprise Linux Universal Base Image](https://catalog.redhat.com/software/base-images)
- [Red Hat Enterprise Linux Subscription Guide](https://www.redhat.com/en/resources/red-hat-enterprise-linux-subscription-guide#section-1)

## RHLS Core Recommendations

- [Free skills assessment to see where you should start training](https://skills.ole.redhat.com/en) :star:
- [Red Hat Enterprise Linux Technical Overview (RH024 - no cost)](https://www.redhat.com/en/services/training/rh024-red-hat-linux-technical-overview)
- [Red Hat System Administration I (RH124)](https://www.redhat.com/en/services/training/rh124-red-hat-system-administration-i)
- [Red Hat System Administration II (RH134)](https://www.redhat.com/en/services/training/rh134-red-hat-system-administration-ii)
- [Red Hat Certified System Administrator (RHCSA) exam (EX200)](https://www.redhat.com/en/services/training/ex200-red-hat-certified-system-administrator-rhcsa-exam)
- [*Stretch Goals* - Red Hat Enterprise Linux skills path](https://www.redhat.com/en/resources/enterprise-linux-skills-path-brief)

## Independent Learning Path Recommendations

### Labs

- [Interactive labs for Red Hat Enterprise Linux](https://www.redhat.com/en/interactive-labs/enterprise-linux) :star:
- [Start Building with Red Hat Enterprise Linux](https://developers.redhat.com/products/rhel/getting-started#iamnewtoredhatenterpriselinux)
- [Red Hat Enterprise Linux Technical Overview](https://www.redhat.com/en/services/training/rh024-red-hat-linux-technical-overview)  

### Reading Material and References

- [20 Essential Linux commands for every user](https://www.redhat.com/en/blog/20-essential-linux-commands-every-user?blaid=7662702)
- [Advanced Linux Commands Cheat Sheet](https://developers.redhat.com/cheat-sheets/advanced-linux-commands)

### Videos

- [Red Hat Enterprise Linux](https://www.youtube.com/@RedHatEnterpriseLinux)  
- [Why Web Console Might Change How You Manage RHEL](https://www.youtube.com/watch?v=YVrYHpC53bM)

### RHEL for Containers (UBI)

A core concept for modern automation is the **Execution Environment (EE)**, which is a container image that bundles Ansible and all its dependencies. These EEs are built on top of the RHEL Universal Base Image (UBI).

- [Red Hat Enterprise Linux Universal Base Image](https://catalog.redhat.com/software/base-images): The official, lightweight, and secure RHEL image for building your own containers.
- [Getting started with UBI](https://developers.redhat.com/products/rhel/ubi): Learn how to pull and use UBI images.
