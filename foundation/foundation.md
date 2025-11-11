# Enablement Recommendations:  Foundational Topics

This document focuses on some *prerequisites* for automation. The subsequent documents will teach you *how* to build automation. This high-level context explains how these pieces fit together.

The tools in this document (Git, YAML, VSCode) are the foundational building blocks you will use to write the code that AAP will eventually run.

## Goals

The goal of this foundational document is to build the core toolset and mindset required for an Automation Specialist. Before writing your first Ansible Playbook or using AAP, you must be comfortable with the tools and concepts that the platform is built on.

```mermaid
graph LR
    A[Introduce Concepts] --> B[Master Syntax];
    B --> C[Configure Your IDE];
    C --> D[Use Version Control];
    D --> E[5. Practice - Tie it all together];
```

This document provides a path to:

- **Introduce Concepts:** Grasp the high-level ideas of **IaC** (Infrastructure as Code) and **DevOps**, which are the "why" behind modern automation.
- **Master Syntax:** Write **YAML** syntax and use a **linter** to verify it, as YAML is the language for all Ansible content (playbooks, variable files, etc.).
- **Configure Your IDE:** Set up **VSCode** as a professional automation editor, integrating it with YAML validation and Git.
- **Use Version Control:** Experiment with basic **git** commands and workflows, as AAP uses Git as the "source of truth" for all automation code.

## Independent Learning Path Recommendations

Build your toolset - learn some yaml, vscode, and git.  Then integrate the tools.

### Introduce Concepts

- [What is Infrastructure as Code (**IaC**)?](https://www.redhat.com/en/topics/automation/what-is-infrastructure-as-code-iac)
- [What is **DevOps**?](https://www.redhat.com/en/topics/devops/what-is-devops)

### VsCode Essentials

- [Visual Studio Code](https://code.visualstudio.com/docs) download, tutorials and setup!
- [YAML Language Support by Red Hat](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml)
  - **Note:** This extension provides the **YAML linting** (real-time error checking) mentioned in our goals.
- [GitLens — Git supercharged](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens)
  - **Note:** This powerful extension helps you visualize code authorship (Git blame) and repository history directly within your editor.

### Git Essentials

- [A Beginner's Guide to Git](https://developers.redhat.com/articles/2023/08/02/beginners-guide-git-version-control#)
- [Learn **Git** (References to book, videos...)](https://git-scm.com/learn)
- [Git best practices: Workflows for GitOps deployments](https://developers.redhat.com/articles/2022/07/20/git-workflows-best-practices-gitops-deployments#)

## Bring it all together!

- [GitLens — Git supercharged](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens)
- [Intro to Git in VS Code](https://code.visualstudio.com/docs/sourcecontrol/intro-to-git)

### Key Git Commands to Know

While the guides provide full details, focus on understanding this core set of commands first. This is the basic workflow you will use every day.

| Command | What it does | Example / Note |
| :--- | :--- | :--- |
| `git clone [url]` | Creates a local copy of a remote repository on your computer. | `git clone https://github.com/user/repo.git` |
| `git branch [name]` | Creates a new branch for you to work on. | `git branch feature/new-playbook` |
| `git switch [name]` | Switches your active working directory to the specified branch. | `git switch feature/new-playbook` (Replaces older `checkout` command) |
| `git status` | Shows you the current state of your repository (changed files, etc.). | You will use this constantly. |
| `git add [file]` | Stages a file, telling Git you want to include it in the next commit. | `git add .` (Stages all changed files) |
| `git commit -m "..."` | Saves a "snapshot" of your staged changes to the repository's history. | `git commit -m "Add new inventory file"` |
| `git pull` | Fetches the latest changes from the remote repository and merges them. | Keeps your local branch up-to-date. |
| `git push` | Uploads your local commits to the remote repository. | Shares your work with the team and AAP. |

### Hands-On Practice: Tying It All Together

This simple exercise will confirm you have successfully set up your environment by combining all the foundational topics.

- **Prerequisite:** [Intro to Git in VS Code](https://code.visualstudio.com/docs/sourcecontrol/intro-to-git)

Your goal is to:

1.  Create a new folder (e.g., `my-automation-project`).
2.  Open this folder in **VSCode**.
3.  Create a new file named `inventory.yml`.
4.  Write a simple **YAML** structure in this file. Use the Red Hat YAML extension to check for errors in real-time.
    > **Example `inventory.yml`:**
    > ```yaml
    > ---
    > all:
    >   hosts:
    >     webserver-01:
    >       ansible_host: 192.168.1.10
    >       location: "new-york"
    >     webserver-02:
    >       ansible_host: 192.168.1.11
    >       location: "boston"
    >   vars:
    >     region: "east"
    > ```
5.  Use the **Git** integration in VSCode (or the command line) to:
    * Initialize a new Git repository (`git init`).
    * Add the `inventory.yml` file to staging.
    * Make your first commit (e.g., `git commit -m "Initial inventory setup"`).