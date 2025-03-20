# Installing FreeIPA Server with Ansible Playbook

## Introduction
FreeIPA is an open-source identity management system that provides centralized authentication, authorization, and account information. In this guide, we’ll use an Ansible playbook to automate the installation and configuration of a FreeIPA server on a CentOS 7 system. This approach simplifies the setup process and ensures consistency.

## Environment
- **Operating System**: CentOS 7.
- **System Requirements**: 
  - RAM: 3 GB
  - CPU: 2 cores
  - Disk: 20 GB
- **Tool**: FreeIPA (Identity, Policy, and Audit).

## Steps

### Step 1 - Set the Hostname
Configure the server’s hostname to a fully qualified domain name (FQDN) that FreeIPA will use:
```bash
hostnamectl set-hostname <your-hostname>
```
- Replace `<your-hostname>` with your desired FQDN (e.g., `ipa-server.example.com`).

Refresh the shell to apply the change:
```bash
exec bash
```

### Step 2 - Update the Hosts File
Add the server’s IP address and FQDN to `/etc/hosts` for local resolution:
```bash
echo "<your-ip> <your-fqdn>" | sudo tee -a /etc/hosts
```
- Replace `<your-ip>` with the server’s IP (e.g., `192.168.1.100`) and `<your-fqdn>` with the FQDN (e.g., `ipa-server.example.com`).
- Example: `echo "192.168.1.100 ipa-server.example.com" | sudo tee -a /etc/hosts`

### Step 3 - Verify Nameserver Configuration
Check `/etc/resolv.conf` to ensure the nameserver matches your network’s gateway or DNS server:
```bash
cat /etc/resolv.conf
```
- Example output:
  ```
  nameserver 192.168.1.1
  ```
- If incorrect, edit it manually (`sudo vi /etc/resolv.conf`) or ensure DHCP provides the correct value.

### Step 4 - Install EPEL and Ansible
Install the EPEL repository and Ansible:
```bash
sudo yum install -y epel-release
sudo yum install -y ansible
```
- `epel-release`: Adds extra packages for Enterprise Linux.
- `ansible`: Installs the automation tool.

### Step 5 - Create the Ansible Playbook
Create a file named `playbook.yml` for the FreeIPA installation:
```bash
vi playbook.yml
```

Add the following configuration, customizing the placeholders:
```yaml
---
# Ansible Playbook to install FreeIPA Server and its requirements on localhost
- hosts: localhost
  become: yes
  become_user: root
  become_method: sudo
  tasks:
    - name: Install EPEL Release
      yum:
        name: epel-release
        state: latest

    - name: Update all packages
      yum:
        name: '*'
        state: latest

    - name: Disable SELinux
      selinux:
        state: disabled

    - name: Install FreeIPA Server packages
      yum:
        name:
          - ipa-server
          - ipa-server-dns
        state: latest

    - name: Configure FreeIPA Server with DNS
      shell: |
        ipa-server-install --realm=<YOUR-REALM> --domain=<your-domain> \
          --ds-password=<directory-manager-password> --admin-password=<admin-password> \
          --mkhomedir --ssh-trust-dns --setup-dns --unattended --auto-forwarders
      args:
        executable: /bin/bash

    - name: Permit traffic in firewall for FreeIPA services
      firewalld:
        service: "{{ item }}"
        permanent: yes
        state: enabled
      with_items:
        - http
        - https
        - dns
        - ntp
        - freeipa-ldap
        - freeipa-ldaps

    - name: Reload firewall configuration
      shell: firewall-cmd --reload
```

- Replace:
  - `<YOUR-REALM>`: The Kerberos realm (e.g., `EXAMPLE.COM`—typically uppercase).
  - `<your-domain>`: The DNS domain (e.g., `example.com`).
  - `<directory-manager-password>`: Password for the Directory Manager.
  - `<admin-password>`: Password for the FreeIPA admin UI.
- Example: `--realm=EXAMPLE.COM --domain=example.com --ds-password=DirMgrPass123 --admin-password=AdminPass123`

### Step 6 - Run the Playbook
Execute the playbook to automate the installation and configuration:
```bash
ansible-playbook playbook.yml
```

### Step 7 - Verify the Installation
If successful, the output will look like this:
```
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [localhost] *********************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [localhost]

TASK [Install EPEL Release] **********************************************************************************************************************************
ok: [localhost]

TASK [Update all packages] ************************************************************************************************************************************
ok: [localhost]

TASK [Disable SELinux] ***************************************************************************************************************************************
ok: [localhost]

TASK [Install FreeIPA Server packages] ************************************************************************************************************************
ok: [localhost]

TASK [Configure FreeIPA Server with DNS] **********************************************************************************************************************
changed: [localhost]

TASK [Permit traffic in firewall for FreeIPA services] ********************************************************************************************************
changed: [localhost] => (item=http)
changed: [localhost] => (item=https)
changed: [localhost] => (item=dns)
changed: [localhost] => (item=ntp)
changed: [localhost] => (item=freeipa-ldap)
changed: [localhost] => (item=freeipa-ldaps)

TASK [Reload firewall configuration] **************************************************************************************************************************
changed: [localhost]

PLAY RECAP ***************************************************************************************************************************************************
localhost                  : ok=8    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
- `changed=3`: Indicates tasks that modified the system (e.g., FreeIPA setup, firewall rules).

### Step 8 - Test FreeIPA Availability
Check if the FreeIPA server is running by querying its FQDN:
```bash
curl <your-fqdn>
```
- Replace `<your-fqdn>` with the server’s FQDN (e.g., `ipa-server.example.com`).

### Step 9 - Confirm Redirection
You should see a redirect response:
```
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>301 Moved Permanently</title>
</head><body>
<h1>Moved Permanently</h1>
<p>The document has moved <a href="https://<your-fqdn>/ipa/ui">here</a>.</p>
</body></html>
```
- Copy the `https://<your-fqdn>/ipa/ui` link (e.g., `https://ipa-server.example.com/ipa/ui`).

### Step 10 - Access the FreeIPA UI
Test the UI endpoint:
```bash
curl https://<your-fqdn>/ipa/ui
```
- Example: `curl https://ipa-server.example.com/ipa/ui`

### Step 11 - Verify UI Accessibility
The response will include HTML content. Look for:
```
<noscript>This application requires JavaScript enabled.</noscript>
```
- This indicates the FreeIPA web interface is accessible. Open `https://<your-fqdn>/ipa/ui` in a browser to log in with the admin credentials.

## Notes
- Ensure the system has at least 3 GB RAM, as FreeIPA is resource-intensive.
- SELinux is disabled in this setup for simplicity; adjust for production use.
- Use strong passwords for security.