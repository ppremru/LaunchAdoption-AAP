# Stage 3: The Source of Truth (Dynamic Inventory)

## Introduction

In Stage 1, we used a static `inventory.ini` file. In Stage 3, we replace that file with an **Inventory Plugin**. This plugin acts as a translator between Ansible and your infrastructure providers (like AWS, Azure, or VMware).

This aligns with **Pillar 7 (Visibility and Analytics)** and **Continuous Monitoring**. You cannot secure what you cannot see; Dynamic Inventory ensures that 100% of your active assets are accounted for in every automation run.

### The Three Key Parts of Stage 3

1. **Inventory Plugins:** The code that communicates with your cloud or virtualization API.
2. **Plugin Configuration (`.yml`):** The file that defines *where* to look and *how* to group discovered hosts.
3. **Keyed Groups:** Automatically organizing hosts based on their metadata (e.g., "all servers in the 'Production' VPC").

---

## Part 1: Static vs. Dynamic Thinking

At Acme, manual inventories are considered a security risk. If an admin spins up a new server and forgets to add it to the `.ini` file, that server remains unpatched and unmonitored.

* **Static:** "I think these 10 servers exist."
* **Dynamic:** "I am asking the Cloud Provider to tell me exactly what is running right now."

---

## Part 2: Configuring the Plugin

Instead of a list of IPs, we create a configuration file that ends in `aws_ec2.yml`, `azure_rm.yml`, or `vmware_vm_inventory.yml`. This tells Ansible to use the corresponding plugin from the **Collections** we learned about in Stage 2.

### Example: `acme_aws_inventory.yml`

```yaml
---
# This tells Ansible to use the AWS EC2 plugin
plugin: amazon.aws.aws_ec2

# Define the regions to scan for assets
regions:
  - us-east-1
  - us-west-2

# KEYED GROUPS: This is the "Magic" of Stage 3.
# It automatically creates Ansible groups based on the Tags in AWS.
keyed_groups:
  - key: tags.Environment
    prefix: env
  - key: tags.Role
    prefix: role
  - key: instance_type
    prefix: type

# Only include hosts that are actually running (Pillar 2: Device Identity)
filters:
  instance-state-name: : running

```

---

## Part 3: Testing the Real-Time Feed

Once the configuration is set, you don't run a playbook yet. First, you verify that Ansible can "see" the environment.

**Command:**
`ansible-inventory -i acme_aws_inventory.yml --graph`

**Output Example:**

```text
@all:
  |--@env_prod:
  |  |--ec2-3-80-1-10.compute-1.amazonaws.com
  |  |--ec2-3-80-1-11.compute-1.amazonaws.com
  |--@role_webserver:
  |  |--ec2-3-80-1-10.compute-1.amazonaws.com
  |--@type_t3_medium:
  |  |--ec2-3-80-1-10.compute-1.amazonaws.com

```

---

## How Stage 3 Supports Zero Trust

* **Eliminates Shadow IT:** If an unauthorized user spins up a VM, the Dynamic Inventory will find it on the next scan, allowing Acme’s security playbooks to immediately apply "deny-all" firewall rules.
* **Metadata-Based Security:** We can write playbooks that target `hosts: env_prod`. Ansible will only run against servers that the Cloud Provider has cryptographically verified as belonging to the Production environment.

---

## Stage 3 Review & Resources

Before moving to **Stage 4**, practice running the `ansible-inventory` command against a lab environment. Notice how you no longer need to type IP addresses.

* **Guide:** [How to use Inventory Plugins](https://www.google.com/search?q=https://docs.ansible.com/ansible/latest/plugins/inventory.html)
* **VMware Example:** [Community VMware Inventory Plugin](https://www.google.com/search?q=https://docs.ansible.com/ansible/latest/collections/community/vmware/vmware_vm_inventory_inventory.html)
* **AWS Example:** [Amazon AWS EC2 Inventory Plugin](https://www.google.com/search?q=https://docs.ansible.com/ansible/latest/collections/amazon/aws/aws_ec2_inventory.html)

---
