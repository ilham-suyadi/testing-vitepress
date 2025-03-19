# Converting and Resizing Images in Libvirt/KVM and VirtualBox

## Introduction
Virtual machine (VM) disk images come in various formats depending on the virtualization platform, such as `.qcow2` for KVM, `.vdi` for VirtualBox, or `.img` (raw). This guide explains how to convert between these formats and resize them using tools like `qemu-img` on Linux or VirtualBox utilities on Windows. Whether you’re working with Libvirt/KVM or VirtualBox, these steps will help you manage your VM images effectively.

## Environment
- **Libvirt/KVM**: A virtualization solution for Linux.
- **Operating System**: Linux (for KVM/Libvirt) or Windows (for VirtualBox).
- **Image Formats**: `.qcow2` (KVM), `.vdi` (VirtualBox), `.img` (raw), etc.
- **VirtualBox**: A cross-platform virtualization tool.

## Supported `qemu-img` Format Strings
The `qemu-img` tool supports multiple image formats. Use the corresponding argument when converting:

| **Image Format**      | **Argument to `qemu-img`** |
|-----------------------|----------------------------|
| QCOW2 (KVM, Xen)      | `qcow2`                   |
| QED (KVM)             | `qed`                     |
| Raw                   | `raw`                     |
| VDI (VirtualBox)      | `vdi`                     |
| VHD (Hyper-V)         | `vpc`                     |
| VMDK (VMware)         | `vmdk`                    |

## Converting Images on Linux (Libvirt/KVM)

This section assumes you’re using a Linux system. While the commands are similar for Libvirt on Windows, the environment setup may differ slightly.

### Steps
1. **Prepare Your Environment**  
   Ensure `qemu-kvm` or `libvirt` is installed on your system. You’ll also need a sample VM image (e.g., `.qcow2`, `.vdi`) ready for conversion. Install `qemu-utils` if it’s not already present:
   ```bash
   sudo apt install qemu-utils  # For Debian/Ubuntu
   sudo yum install qemu-img    # For RHEL/CentOS/Rocky
   ```

2. **Navigate to the Image Directory**  
   Move to the directory containing the image you want to convert. Replace `/path/to/image` with your actual path:
   ```bash
   cd /path/to/image
   ```

3. **Convert the Image**  
   Use the `qemu-img convert` command to change the image format. The syntax is:
   ```bash
   sudo qemu-img convert -f <source-format> -O <destination-format> <source-image> <output-image>
   ```
   - `-f`: Source format (e.g., `qcow2`).
   - `-O`: Output format (e.g., `vdi`).
   - `<source-image>`: Original image file.
   - `<output-image>`: New image file after conversion.

   **Example**: Convert a `.qcow2` image to `.vdi` (VirtualBox format):
   ```bash
   sudo qemu-img convert -f qcow2 -O vdi CentOS-7-x86_64-GenericCloud.qcow2 centos7.vdi
   ```

   Wait for the process to complete. Afterward, verify the new file exists:
   ```bash
   ls -lrth
   ```

   You should see both the original and converted images in the directory.

## Resizing Image Size on Linux

Resizing a VM image adjusts its storage capacity. This is useful when you need more space or (less commonly) want to shrink an image.

### Steps
1. **Navigate to the Image Directory**  
   Open a terminal on your Linux VM or host and change to the directory containing the image:
   ```bash
   cd /path/to/image
   ```

2. **Resize the Image**  
   Use the `qemu-img resize` command. The default size increase is additive, but shrinking requires the `--shrink` flag. Syntax:
   ```bash
   qemu-img resize <image-name> <new-size>
   ```

   **Example**: Resize an image named `centos7` to 50GB:
   ```bash
   qemu-img resize centos7 50G
   ```

   > **Note**: Resizing typically only increases capacity. To reduce the size (e.g., smaller than the current capacity), use `--shrink`. Shrinking requires the image’s filesystem to be resized first to avoid data loss.

   **Example**: Shrink an image to 20GB:
   ```bash
   qemu-img resize centos7 --shrink 20G
   ```
   Output:
   ```
   Image resized.
   ```

   Be cautious with shrinking, as it can corrupt data if the filesystem isn’t adjusted beforehand.

## Converting VDI to Raw Image on Windows with VirtualBox

For Windows users running VirtualBox, you can convert a `.vdi` image to a raw `.img` format using the `VBoxManage` tool.

### Steps
1. **Open Command Prompt and Navigate to VirtualBox Directory**  
   Open `cmd` (Command Prompt) and switch to the VirtualBox installation directory:
   ```cmd
   cd "C:\Program Files\Oracle\VirtualBox"
   ```

2. **Run the Conversion Command**  
   Use `VBoxManage clonehd` to convert the `.vdi` file to raw format. Syntax:
   ```cmd
   VBoxManage clonehd "<source-path>.vdi" "<output-path>.img" --format raw
   ```

   **Example**: Convert a VirtualBox image located in the user’s VMs folder:
   ```cmd
   VBoxManage clonehd "C:\Users\247\VirtualBox VMs\CentOS\CentOS.vdi" "C:\Users\247\VirtualBox VMs\CentOS\centos7_convert.img" --format raw
   ```

   Adjust the file paths to match your environment (e.g., replace `247` with your username and ensure the paths point to your `.vdi` file).

3. **Wait for Completion**  
   The process may take a few minutes depending on the image size. Once finished, check the output directory for the new `.img` file.

## References
- [OpenStack Image Guide: Convert Images](https://docs.openstack.org/image-guide/convert-images.html)
- [AskUbuntu: Convert QCOW2 to Physical Machine](https://askubuntu.com/questions/195139/how-to-convert-qcow2-virtual-disk-to-physical-machine-and-reversely)
- [Medium: Convert VDI to Raw in Windows](https://gioacchinolonardo.medium.com/convert-vdi-virtualbox-to-raw-in-windows-c96bded29640)
