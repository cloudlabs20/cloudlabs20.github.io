---
layout: post
title: Proxmox VE Server Installation
categories: [Virtualization]
tags: [server, hardware, proxmox, hypervisor]
enable_toc: true
render_with_liquid: true
date: 2024-06-10 00:00:00 +0530
---


## What is Proxmox?

![proxmox](/assets/img/proxmox/proxmox-sample.png)

Proxmox Virtual Environment (Proxmox VE) is a free and open-source platform for managing server virtualization. It provides an easy-to-use web interface for handling virtual machines (VMs), containers, and storage resources. Proxmox VE supports KVM for full virtualization and LXC for containerization, offering a flexible solution for different virtualization requirements.

## Installation of Proxmox

Now that you understand what Proxmox is and have decided to use it as the primary OS for your server, installing Proxmox is a straightforward process. Here’s a step-by-step guide:

1. **Download Proxmox VE ISO**: Start by downloading the Proxmox VE ISO image from the official Proxmox [website](https://proxmox.com/en/proxmox-virtual-environment/overview).

2. **Create Bootable Media**: Use tools like [Rufus](https://rufus.ie/en/) or [Etcher](https://etcher.download/) to create a bootable USB drive from the downloaded ISO file.

3. **Boot from USB**: Insert the bootable USB drive into your server and boot from it. This may require adjusting the BIOS/UEFI settings to prioritize booting from the USB drive.

4. **Install Proxmox VE**:
	- Select the "Install Proxmox VE" option from the boot menu.
    - Follow the on-screen instructions to configure the installation, including selecting the target disk for installation, setting up the timezone, and creating a root password.

5. **Complete Installation**: Once the installation is complete, the system will prompt you to remove the installation media and reboot.

## Post Installation Steps

After installing Proxmox VE, there are several important steps to ensure your system is properly configured and updated.

### Disable the Enterprise Repository

By default, Proxmox VE is configured to use the enterprise repository, which requires a subscription. Although there are a few steps to disable this repository manually, a community-created helper script on GitHub can simplify this process. You can find this script at [helper-scripts.com](https://helper-scripts.com/).

This script offers several options for managing Proxmox VE repositories, including:

- Disabling the Enterprise Repo
- Enabling the No-Subscription Repo
- Adding or correcting Ceph Package sources
- Adding the test/beta Repo
- Disabling the subscription nag
- Updating Proxmox VE
- Rebooting the system

Using this script can streamline your post-installation cleanup and configuration tasks.

```bash

bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/post-pve-install.sh)"

```

After the Installation, please go through each option mentioned above and verified that everything has been disabled and removed. 


## Increase Storage in the Shell

![proxmox-shell](/assets/img/proxmox/proxmox-shell.png)

By default, Proxmox separates local storage into two disks: Local and LVM. Typically, the LVM disk shouldn't be used unless you have a specific reason.

To increase storage, you may need to delete the existing Logical Volume Manager (LVM) storage and create a new setup. Be cautious, as this process will delete all data on the LVM.

### Commands for Deleting LVM Storage

First, list the existing logical volumes:

```bash

lvdisplay

```

To delete a logical volume:

```bash

lvremove /dev/your-vg-name/your-lv-name

```

Replace `your-vg-name` and `your-lv-name` with the appropriate names from your setup.

### Increase the Local storage in the shell

After deleting the LVM storage, run the following commands to gain full access to your local storage. Once these commands are executed, you should see the actual total capacity of your local storage available for use.

```shell

lvremove /dev/pve/data


```

```shell

lvresize -l +100%FREE /dev/pve/root

```

```shell

resize2fs /dev/mapper/pve-root

```

These commands will:

1. Remove the existing LVM data volume.
2. Resize the root logical volume to use all available space.
3. Resize the file system on the root logical volume to match the new size.

### Initialize Disk and Set Up Directory

You can use additional storage disks in Proxmox for VM storage or backup storage. Additionally, you can upload multiple ISO files to boot your VMs from the GUI prompt when initializing the VM.

Commands you need to initialize a new disk:

```bash

fdisk /dev/sdX

```

Replace `/dev/sdX` with your target disk. Follow the fdisk prompts to create a new partition. Then, format the partition:

```bash

mkfs.ext4 /dev/sdX1

```

Mount the new partition to a directory:

```bash

mkdir /mnt/newstorage mount /dev/sdX1 /mnt/newstorage

```

To make the mount persistent across reboots, add it to `/etc/fstab`:

```bash

echo '/dev/sdX1 /mnt/newstorage ext4 defaults 0 0' >> /etc/fstab

```

## Best Practices for Proxmox

1. **Regular Backups**: Regularly back up your VMs and containers to prevent data loss.
2. **Monitor System Health**: Use monitoring tools to keep an eye on system performance and health.
3. **Security Updates**: Keep Proxmox VE and its components up to date with security patches.
4. **Network Configuration**: Properly configure networking to ensure high availability and performance.
5. **Resource Allocation**: Allocate resources judiciously to avoid overcommitting CPU and memory.

## Conclusion

Proxmox VE is a powerful virtualization platform that offers robust features for managing VMs and containers. By following the installation and post-installation steps, you can set up a reliable and efficient virtualization environment. Additionally, implementing best practices will ensure your Proxmox VE setup remains secure, efficient, and resilient. Whether you're running a small lab or a large data center, Proxmox VE provides the tools you need to succeed in virtualization.

In the next post, I'll continue with the process of implementing Cloud-init templates to ensure an easier process for creating VMs from existing cloud images.
