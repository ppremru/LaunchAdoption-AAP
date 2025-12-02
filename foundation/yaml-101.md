# YAML Syntax 101: The Standard Language of Automation

**Goal:** Master the industry-standard language for configuration and automation.

## Why Do I Care About YAML?

YAML (YAML Ain't Markup Language) isn't just for Ansible - modern tools such as Kubernetes, GitHub and Helm ... use YAML to define what they do. By learning this format, you can define configurations that work across teams and platforms.

## A Few Things to Know About YAML Syntax

To read and write YAML, start with mastering these five core concepts:

1. **Indentation:** The golden rule of structure.
2. **Key-Value Pairs:** How to define settings.
3. **Lists:** How to define collections of items.
4. **Comments:** How to leave notes for humans.
5. **Booleans:** How to toggle settings on and off.

### 1. Indentation (The Golden Rule)

YAML relies entirely on indentation to show structure (nesting). This is what makes it readable but also fragile.

* Use Spaces, Not Tabs: Always use 2 spaces for each level of indentation.
* Align Your Data: Items at the same level must be perfectly aligned vertically.

<!-- end list -->

```yaml
parent_item:
  child_item: value  # Indented 2 spaces
  another_child: value
```

### 2. Key-Value Pairs (Dictionaries)

This is the most common data structure. It connects a name (key) to a value.

* Syntax: `key: value` (There must be a space after the colon).

<!-- end list -->

```yaml
name: "Web Server"
state: present
port: 80
```

### 3. Lists (Arrays)

Lists are used when you have multiple items under one key (e.g., a list of packages to install or ports to open).

* Syntax: Use a dash `-` followed by a space.

<!-- end list -->

```yaml
packages_to_install:
  - httpd
  - firewalld
  - vim
```

### 4. Comments

Comments are for humans. The automation engine ignores them. Use them to explain *why* a config exists.

* Syntax: Start the line (or part of the line) with a hash `#`.

<!-- end list -->

```yaml
# Ensure this server is always reachable
ansible_host: 192.168.1.10
```

### 5. Booleans (True/False)

Used for toggles like "admin access" or "service enabled."

* Valid values: `true`, `false` (Best practice is lowercase).

## Putting It Together (Ansible Context)

Here is how those rules look in an Ansible Playbook from your future Ansible training.

```yaml
---                               # Start of file marker
- name: Gather hostname           # List item (The Play)
  hosts: all                      # Key-Value pair
  tasks:                          # List of tasks starts here
    - name: Run setup             # List item (The Task)
      ansible.builtin.setup:      # Module call
        filter: ansible_hostname  # Parameter (Key-Value)
```

## Troubleshooting Tip

If you get a syntax error, 90% of the time it is indentation.

* Did you use a tab instead of a space?
* Did you forget the space after the colon?
* Are your list dashes aligned?

> **Tools:** Use the Red Hat YAML extension in VSCode to catch these errors automatically while you type.
