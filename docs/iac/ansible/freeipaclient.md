# Installing FreeIPA Client with an Ansible Playbook

## Introduction
FreeIPA provides centralized identity management, and installing a FreeIPA client allows a system to authenticate against a FreeIPA server. In this guide, we’ll use an Ansible playbook to automate the installation and configuration of a FreeIPA client on a CentOS 7 system, connecting it to an existing FreeIPA server.

## Environment
- **Operating System**: CentOS 7.
- **System Requirements**:
  - RAM: 2 GB
  - CPU: 2 cores
  - Disk: 20 GB
- **Components**:
  - FreeIPA Server (already installed and configured).
  - FreeIPA Client (to be installed on this system).

## Steps

### Step 1 - Set the Hostname
Configure the client’s hostname to a fully qualified domain name (FQDN):
```bash
hostnamectl set-hostname <your-hostname>
exec bash
```
- Replace `<your-hostname>` with the desired FQDN (e.g., `client1.example.com`).
- `exec bash` refreshes the shell to apply the change.

### Step 2 - Update the Hosts File
Add the IP and FQDN of both the FreeIPA server and the client to `/etc/hosts` on the client machine:
```bash
echo "<server-ip> <server-fqdn>" | sudo tee -a /etc/hosts
echo "<client-ip> <client-fqdn>" | sudo tee -a /etc/hosts
```
- Example:
  ```bash
  echo "192.168.1.100 ipa-server.example.com" | sudo tee -a /etc/hosts
  echo "192.168.1.101 client1.example.com" | sudo tee -a /etc/hosts
  ```

On the FreeIPA server, also add the client’s IP and FQDN to its `/etc/hosts`:
```bash
echo "<client-ip> <client-fqdn>" | sudo tee -a /etc/hosts
```
- Example: `echo "192.168.1.101 client1.example.com" | sudo tee -a /etc/hosts`

This ensures proper name resolution between the server and client.

### Step 3 - Configure the Nameserver
Edit `/etc/resolv.conf` to use the FreeIPA server’s IP as the nameserver:
```bash
sudo vi /etc/resolv.conf
```
Update it to:
```
nameserver <server-ip>
```
- Example: `nameserver 192.168.1.100`
- Save and exit. This directs DNS queries to the FreeIPA server.

### Step 4 - Install EPEL and Ansible
Install the EPEL repository and Ansible on the client:
```bash
sudo yum install -y epel-release
sudo yum install -y ansible
```
- `epel-release`: Adds extra packages for CentOS.
- `ansible`: Installs the automation tool.

### Step 5 - Create the Ansible Playbook
Create a playbook file (e.g., `freeipa-client.yml`) for the FreeIPA client setup:
```bash
vi freeipa-client.yml
```

Add the following configuration, customizing the placeholders:
```yaml
---
# Playbook to install and configure FreeIPA Client
- name: Configure IPA Client with username/password
  hosts: localhost
  become: true
  tasks:
    - name: Install EPEL Release
      yum:
        name: epel-release
        state: latest

    - name: Install Firewalld
      yum:
        name: firewalld
        state: latest

    - name: Disable SELinux
      selinux:
        state: disabled

    - name: Install BIND Utilities
      yum:
        name: bind-utils
        state: latest

    - name: Install FreeIPA Client Package
      yum:
        name: freeipa-client
        state: latest

    - name: Configure FreeIPA Client
      shell: |
        ipa-client-install --server=<server-fqdn> --domain=<domain> \
          --realm=<REALM> --principal=admin --password=<admin-password> \
          --mkhomedir --force-ntpd --unattended
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
  - `<server-fqdn>`: FreeIPA server’s FQDN (e.g., `ipa-server.example.com`).
  - `<domain>`: DNS domain (e.g., `example.com`).
  - `<REALM>`: Kerberos realm (e.g., `EXAMPLE.COM`—uppercase).
  - `<admin-password>`: FreeIPA admin password (set during server setup).
- Example: `--server=ipa-server.example.com --domain=example.com --realm=EXAMPLE.COM --password=AdminPass123`

### Step 6 - Run the Playbook
Execute the playbook to install and configure the FreeIPA client:
```bash
ansible-playbook freeipa-client.yml
```

Example output:
```
PLAY [Configure IPA Client with username/password] *********************************************

TASK [Gathering Facts] *************************************************************************
ok: [localhost]

TASK [Install EPEL Release] ********************************************************************
ok: [localhost]

TASK [Install Firewalld] ***********************************************************************
ok: [localhost]

TASK [Disable SELinux] *************************************************************************
ok: [localhost]

TASK [Install BIND Utilities] ******************************************************************
ok: [localhost]

TASK [Install FreeIPA Client Package] **********************************************************
ok: [localhost]

TASK [Configure FreeIPA Client] ****************************************************************
changed: [localhost]

TASK [Permit traffic in firewall for FreeIPA services] *****************************************
changed: [localhost] => (item=http)
changed: [localhost] => (item=https)
changed: [localhost] => (item=dns)
changed: [localhost] => (item=ntp)
changed: [localhost] => (item=freeipa-ldap)
changed: [localhost] => (item=freeipa-ldaps)

TASK [Reload firewall configuration] ***********************************************************
changed: [localhost]

PLAY RECAP *************************************************************************************
localhost                  : ok=9    changed=3    unreachable=0    failed=0    skipped=0
```

### Step 7 - Verify the Installation
1. **Initialize Kerberos Authentication**  
   On both the FreeIPA server and client, run:
   ```bash
   kinit admin
   ```
   - Enter the admin password when prompted. This tests Kerberos connectivity.

2. **Check the FreeIPA UI**  
   Open the FreeIPA server’s web interface (e.g., `https://ipa-server.example.com/ipa/ui`) in a browser. Navigate to the **Hosts** menu. If the client’s FQDN (e.g., `client1.example.com`) appears, the FreeIPA client installation was successful.

## Notes
- Ensure the FreeIPA server is running and accessible before starting.
- SELinux is disabled for simplicity; adjust for production environments.
- Verify network connectivity between the client and server (e.g., `ping <server-fqdn>`).
