# Launch Adoption - AAP

Welcome! This repository serves as the entry point for our **Community of Practice (CoP) for Ansible Automation Platform (AAP) developer enablement**.

## Our Mission: Increase "Speed to Capability Delivery"

> How does this CoP framework impact our mission?

This framework aims to achieve "speed to capability delivery" by developing a **standardized and repeatable path to competency**.

Instead of developers wasting time guessing, searching, or making mistakes, this CoP provides a clear roadmap from day one.

* **Foundational Tools** (Git, YAML, VSCode) create a **common toolset**, eliminating friction and ensuring everyone speaks the same "language".
* **Ansible Building Blocks** provide a **shared methodology** for writing automation. This makes code reusable, reliable, and faster to develop.
* **Ansible Automation Platform (AAP)** provides the **platform for speed**. It allows the team to deploy their automation as a secure, self-service "capability" that the entire organization can use, rather than keeping it locked on a developer's laptop.

By getting everyone proficient in this *full stack* (Tools \-\> Ansible \-\> AAP), we dramatically reduce the time from a new idea to a delivered, automated service.

> *The CoP maintains a versioned set of living documents that provide recommendations - it will take a commitment of time and effort to cultivate this repository specifically for a team. Notice the entry point is automation developer, not installation, infrastructure management, nor day-to-day administration of the AAP environment itself.*

## About our CoP for AAP Enablement

Our **domain**, **community**, and **practices**:

| About Our CoP | Description |
| :--- | :--- |
| [Why a CoP Focused on Enablement?](./about/aboutCoP.md) | Defines the CoP, its benefits, and links to automation maturity. |
| [Practice: Enablement Framework](./about/aboutPractice.md) | Explains our "Crawl, Walk, Run" model and session roadmap. |
| [Domain: Team Mission](./about/aboutDomain.md) | *To be defined in the CoP Kickoff Session.* |
| [Community: Team Profiles](./about/aboutCommunity.md) | *To be defined in the CoP Kickoff Session.* |

> Tip: [Formatting Files for this Repo](./about/aboutFormat.md)

## Our Enablement Framework

Our Enablement Framework aligns **Onboarding Topics** with **Product Adoption Sessions**.

### Onboarding Topics

The **Onboarding Topics** are grouped into sections in order to support a crawl, walk, run approach to AAP enablement.

| Topic | Focus Area |
| :--- | :--- |
| [Red Hat: Core Sites & Programs](./about/aboutRHResources.md) | :star: Key Red Hat resources (Customer Portal, Developer). |
| [Red Hat Enterprise Linux (RHEL)](./rhel/rhel.md) | Foundational RHEL skills (the "Control Plane" & "Target"). |
| [Foundational Tools](./foundation/foundation.md) | Core tools and concepts (Git, YAML, VSCode, IaC). |
| [Ansible Building Blocks](./ansible/ansible.md) | Core Ansible skills (Playbooks, Roles, Inventory). |
| [Containerization Essentials](./container/container.md) | Container skills (Podman, Buildah) for Execution Environments. |
| [Ansible Automation Platform (AAP)](./aap/aap.md) | The platform (Controller, Workflows, Hub) that runs it all. |

### Product Adoption Sessions

The **Product Adoption Sessions** provide the team with a roadmap to increase AAP Enablement. It is expected that the sessions are facilitated by a Red Hat Adoption Specialist Architect. The community (team) should agree upon a set of collaborative sessions to align the community and domain -- and keep a log of events.

Refer to our [Practice Framework](./about/aboutPractice.md) for a table of recommended MILESTONES to drive sessions.  Our goals include:

```mermaid
graph TD
    subgraph U[Persona - Automation Specialis]
        A(You)
    end

    subgraph S[Foundational Tools]
       direction TB
       Git
       VsCode
       YAML
       Lint
    end

    subgraph Build[Ansible Building Blocks]
       direction TB
       Playbooks
       Inventory
       Variables
       Collections
    end

    subgraph AAP[Ansible Automation Platform]
        direction TB
        Containers
        Resources
        UI
    end

    U --- |Use|S
    U --- |Develop|Build
    U --- |Execute|AAP
```

## Use Cases

* [Use Case with RBAC](./usecase/raci.md)

---
> :link: *[This repo in web format](https://ppremru.github.io/LaunchAdoption-AAP/)*
> :star: *Favorites*
---
