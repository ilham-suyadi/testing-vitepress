# Setting Up a Local Yum Repository

## Introduction
A local Yum repository allows you to host and manage software packages on your own server, making it easier to install and update software without relying on an internet connection. In this guide, we’ll walk through the process of setting up a local repository on a RHEL-based system (e.g., Red Hat Enterprise Linux or Rocky Linux) using an operating system DVD ISO file and the Apache web server (`httpd`). By the end, you'll have a functional local repository accessible via a web browser.

## Requirements
Before starting, ensure you have the following:
- A DVD ISO file of the operating system (e.g., `rhel-8.6-x86_64-dvd.iso`).
- A server or virtual machine running a RHEL-based OS.
- The `tar` utility (usually pre-installed).
- The `httpd` package (Apache web server).
- The `createrepo` tool for generating repository metadata.

## Steps

### Step 1 - Install the Apache Web Server
If the Apache web server (`httpd`) isn’t already installed, install it using the `yum` package manager. Run the following command:

```bash
sudo yum install -y httpd
```

The `-y` flag automatically confirms the installation, saving you from manually approving it.

### Step 2 - Enable and Start the HTTPD Service
Once installed, enable the `httpd` service to start automatically on boot and launch it immediately. Use this command:

```bash
sudo systemctl enable --now httpd
```

To verify that the service is running, check its status:

```bash
sudo systemctl status httpd
```

If successful, you’ll see output indicating that the service is `active (running)`. This confirms your web server is operational.

### Step 3 - Create a Directory for the Repository
Next, create a directory to store your local repository files. This directory will be served by Apache, so we’ll place it under `/var/www/html`. For this example, we’ll name it `local_repo`, but you can choose any name:

```bash
sudo mkdir -p /var/www/html/local_repo
```

The `-p` flag ensures the parent directories are created if they don’t already exist.

### Step 4 - Mount the ISO File
Mount the DVD ISO file to a temporary directory (e.g., `/mnt`) to access its contents. Replace `rhel-8.6-x86_64-dvd.iso` with the name of your ISO file:

```bash
sudo mount -o loop rhel-8.6-x86_64-dvd.iso /mnt
```

The `-o loop` option allows you to mount the ISO as a loop device without needing a physical DVD drive.

### Step 5 - Extract ISO Contents
Navigate to the `/mnt` directory and extract the ISO’s contents into the repository directory (`/var/www/html/local_repo`). The `tar` command below copies everything efficiently:

```bash
cd /mnt
sudo tar cvf - . | (cd /var/www/html/local_repo/; tar xvf -)
```

Here’s what this command does:
- `tar cvf - .` archives the contents of `/mnt`.
- The pipe `|` sends the archive to the target directory.
- `(cd /var/www/html/local_repo/; tar xvf -)` extracts it there.

This process may take a few minutes, depending on the ISO size.

### Step 6 - Install the `createrepo` Tool
To turn the extracted files into a usable Yum repository, you’ll need the `createrepo` package. Install it along with `yum-utils` for additional repository management tools:

```bash
sudo yum install createrepo yum-utils
```

### Step 7 - Generate the Repository Metadata
Run the `createrepo` command to create the necessary metadata files for your local repository:

```bash
sudo createrepo /var/www/html/local_repo/
```

This generates a `repodata` directory inside `/var/www/html/local_repo/`, which Yum uses to recognize the repository.

### Step 8 - Test the Repository
Your local repository is now ready! Open a web browser and enter the following URL, replacing `<ip-address>` with your server’s IP address:

```
http://<ip-address>/local_repo
```

For example, if your server’s IP is `192.168.1.100`, you’d visit `http://192.168.1.100/local_repo`. You should see a directory listing of the repository files.

> **Note**: Adjust the URL based on your server’s IP address and the repository directory name if you used something other than `local_repo`.

### Optional - Configure Yum to Use the Local Repository
To use this repository with Yum, create a `.repo` file in `/etc/yum.repos.d/`. For example:

```bash
sudo tee /etc/yum.repos.d/local_repo.repo <<EOF
[local_repo]
name=Local Repository
baseurl=http://<ip-address>/local_repo
enabled=1
gpgcheck=0
EOF
```

Replace `<ip-address>` with your server’s IP. Then, test it with:

```bash
sudo yum repolist
```

## References
- [Red Hat Sysadmin Guide: Apache Yum/DNF Repo](https://www.redhat.com/sysadmin/apache-yum-dnf-repo)
- [LinuxTechi: Create Local Yum/DNF Repository on RHEL](https://www.linuxtechi.com/create-local-yum-dnf-repository-rhel/#google_vignette)
