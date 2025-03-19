# Encrypting QCOW2 Custom Images

## Introduction
Encrypting virtual machine (VM) disk images, such as those in the QCOW2 format, adds a layer of security by protecting their contents with a password. This is particularly useful for sensitive data in environments like Qemu/KVM. In this guide, we’ll walk through the process of creating an encrypted QCOW2 image and converting an existing unencrypted image to an encrypted one using the `qemu-img` tool on a Linux system.

## Environment
- **Qemu/KVM**: A virtualization platform for managing VMs.
- **Operating System**: Linux (commands are tested on Linux distributions like Ubuntu, CentOS, or Rocky Linux).
- **Tools**: `qemu-img` (part of `qemu-utils` or `qemu-kvm` packages).

## Steps

### Step 1 - Prepare the Image
Start by identifying or preparing the QCOW2 image you want to encrypt. For this guide, we’ll assume you have an existing unencrypted image (e.g., `my_100G_custom_image.qcow2`) or will create a new one. Ensure `qemu-img` is installed:
```bash
sudo apt install qemu-utils  # For Debian/Ubuntu
sudo yum install qemu-img    # For RHEL/CentOS/Rocky
```

### Step 2 - Check Image Information
Before proceeding, inspect the details of your existing QCOW2 image to confirm its format and properties. Run:
```bash
qemu-img info my_100G_custom_image.qcow2
```

Example output:
```
image: my_100G_custom_image.qcow2
file format: qcow2
virtual size: 100 GiB (107374182400 bytes)
disk size: 1.2 GiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    compression type: zlib
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
```
This confirms the image is in QCOW2 format and not yet encrypted.

### Step 3 - Create an Encrypted QCOW2 Image
Create a new encrypted QCOW2 image with a password. In this example, we’ll use the password `tes123` and set the size to 10GB (you can adjust the size as needed). Run:
```bash
qemu-img create --object secret,id=sec0,data=tes123 -f qcow2 -o encrypt.format=luks,encrypt.key-secret=sec0 my_10G_custom_image-encrypted.qcow2 10G
```

Here’s what the command does:
- `--object secret,id=sec0,data=tes123`: Defines a secret (password) with ID `sec0` and value `tes123`.
- `-f qcow2`: Specifies the QCOW2 format.
- `-o encrypt.format=luks,encrypt.key-secret=sec0`: Enables LUKS encryption using the defined secret.
- `my_10G_custom_image-encrypted.qcow2`: The output file name.
- `10G`: The virtual size of the image (10 gigabytes).

After running this, you’ll have a new, empty, encrypted QCOW2 image.

### Step 4 - Convert an Existing Image to Encrypted Format
If you have an unencrypted image (e.g., `my_100G_custom_image.qcow2`) and want to encrypt it, use the `qemu-img convert` command. This copies the contents of the unencrypted image into the encrypted one created in Step 3. Run:
```bash
qemu-img convert --object secret,id=sec0,data=tes123 --image-opts driver=qcow2,file.filename=my_100G_custom_image.qcow2 --target-image-opts driver=qcow2,encrypt.key-secret=sec0,file.filename=my_10G_custom_image-encrypted.qcow2 -n -p
```

Breaking it down:
- `--object secret,id=sec0,data=tes123`: Specifies the password (`tes123`) for encryption.
- `--image-opts driver=qcow2,file.filename=my_100G_custom_image.qcow2`: Defines the source image.
- `--target-image-opts driver=qcow2,encrypt.key-secret=sec0,file.filename=my_10G_custom_image-encrypted.qcow2`: Specifies the encrypted target image.
- `-n`: Skips pre-allocating the target (faster but may fragment the file).
- `-p`: Displays a progress bar during conversion.

Wait for the process to complete. The duration depends on the size of the source image.

### Step 5 - Verify Encryption
Check the new encrypted image to confirm it’s properly encrypted. Run:
```bash
qemu-img info my_10G_custom_image-encrypted.qcow2
```

Example output:
```
image: my_10G_custom_image-encrypted.qcow2
file format: qcow2
virtual size: 10 GiB (10737418240 bytes)
disk size: 1.5 GiB
cluster_size: 65536
Format specific information:
    compat: 1.1
    compression type: zlib
    lazy refcounts: false
    refcount bits: 16
    encrypt:
        format: luks
        cipher-alg: aes-256
        cipher-mode: xts
        ivgen-alg: plain64
    corrupt: false
```
Look for the `encrypt` section in the output—this confirms the image is encrypted with LUKS.

## Using the Encrypted Image
To use this image in Qemu/KVM, you’ll need to provide the password when launching the VM. For example, with `virt-install` or a Qemu command line, include the secret object:
```bash
-object secret,id=sec0,data=tes123
```
Add this to your VM configuration or startup command, along with the path to the encrypted image.

## References
- [IBM Cloud: Creating an Encrypted Custom Image](https://cloud.ibm.com/docs/vpc?topic=vpc-create-encrypted-custom-image)
