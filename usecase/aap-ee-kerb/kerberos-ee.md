# How to Set Up Kerberos Authentication for Windows in AAP

Welcome! If you are a Windows administrator new to Ansible Automation Platform (AAP), configuring your platform to communicate with Windows servers securely is a crucial first step. By default, Windows environments heavily rely on Active Directory and Kerberos for authentication.

This guide will walk you through creating a custom **Execution Environment (EE)**—a containerized environment holding all required tools—that natively authenticates to Windows endpoints using Kerberos.

## Prerequisites

* A working installation of Red Hat Ansible Automation Platform (AAP 2.x).
* Access to a machine with `ansible-builder` installed to build your container image.
* A container registry to store your custom image, such as Private Automation Hub or Quay.
* A service account created in Active Directory for Ansible to use.

## Step 1: Create the Kerberos Configuration File (`krb5.conf`)

Kerberos requires explicit domain architecture details provided in `krb5.conf`. Domain names in Kerberos configurations must **always be written in ALL CAPS**.

Create a file named `krb5.conf` in your project directory:

```ini
[libdefaults]
  default_realm = YOURDOMAIN.COM
  dns_lookup_realm = false
  dns_lookup_kdc = true
  rdns = false
  ticket_lifetime = 24h
  renew_lifetime = 7d
  forwardable = true

[realms]
  YOURDOMAIN.COM = {
    kdc = dc01.yourdomain.com
    admin_server = dc01.yourdomain.com
  }

[domain_realm]
  .yourdomain.com = YOURDOMAIN.COM
  yourdomain.com = YOURDOMAIN.COM
