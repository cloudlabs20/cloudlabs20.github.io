---
layout: post
title: Cloud-init templates in Proxmox Cluster
categories: [Virtualization]
tags: [proxmox, terraform]
enable_toc: true
render_with_liquid: true
date: 2024-06-15 00:00:00 +0530
---


## What is Cloud-init Drive?

In Proxmox, the "cloud-init drive" is a virtual disk that you attach to a virtual machine or container. This virtual disk tells the machine how to set itself up when it initially starts running. It's convenient because you can include details such as the machine's name, internet connectivity settings, and even the software it should install right from the outset. This facilitates the process of setting up new machines, particularly when managing a large number of them, by streamlining and expediting the configuration process.

## Proxmox VM GUI

![cloudinit-vm](/assets/img/proxmox/cloudinit-vm.png)

#### Creating a Virtual Machine

1. General: Begin by creating a VM with a high "VM ID" number and a unique VM name.
2. OS: Choose "Do not use any media" as we will attach the cloud-init drive later on. Maintain the "type" and "version" as default.
3. System: Leave all settings as default, but ensure to check the box for "Qemu agent".
4. Disks: Remove any existing attached disks since we'll be attaching the cloud-init drive later.
5. CPU: Configure with limited resources since this will serve as a template.
6. Memory: Again, limit the configuration as this will be used as a template.
7. Network: Keep all settings as default.
8. Confirm: Verify that all settings are aligned correctly, but **do not** check the box for "start with created".

#### Adding the Cloud-init Drive

![cloudinit-drive](/assets/img/proxmox/cloudinit-drive.png)

1. After creating the VM, navigate to the "hardware" section to add the cloud-init drive.
2. Select the desired storage location where you intend to store your VM data.
3. Proceed to configure settings in Cloud-init such as user, password, SSH, etc.

#### Command Line Process in Terminal

1. Begin by accessing the terminal and SSH into Proxmox.
2. Next, acquire the cloud image from the website. In this example, we're using Ubuntu Server.
3. Follow the provided code block containing all the necessary commands to complete the process.

```bash

1. wget https://cloud-images.ubuntu.com/minimal/releases/jammy/release/ubuntu-22.04-minimal-cloudimg-amd64.img

2. qm set [--node number--] --serial0 socket --vga serial0

3. mv [Downloaded iso] --new iso [change the iso to].qcow2

4. qemu-img resize new-iso.qcow2 20G

5. qm importdisk [--node number--] ubuntu-22.04.qcow2 --storage [storage]

```


#### Create SSH Key on the Host Machine
To access the VM from your host machine without entering the password each time, you need to create an SSH key and save the public key in the cloud-init tab.

- Generate an SSH key by using the command and following the prompts.

```shell
ssh-keygen
```

- Next, copy the key from the file path where it is stored.

```shell
cd ~/.ssh
cat [your-key].pub  # copy the key on your clipboard
```

- Return to Proxmox, navigate to the cloud-init tab under template, and paste the key into the SSH option.

![cloudinit-ssh](/assets/img/proxmox/cloudinit-ssh.png)


#### Final Steps to Finish the Process

1. In the hardware tab, select the drive, click "edit", and then add to activate the drive.
2. Proceed to the Options tab where you need to adjust the "Boot Order".
3. After ensuring that everything is set up correctly, convert the VM into a template.
4. As a final step, clone from the template to verify that everything is functioning correctly.

## Next steps to speed up the process.

Creating a cloud-init template offers the advantage of speeding up the process through automation. You can leverage this by developing an Ansible playbook that outlines the tasks for VM creation in Proxmox. Typically, this playbook encompasses tasks for VM creation, attaching the cloud-init drive, and applying any additional configurations you deem necessary.

## Conclusion 

Certainly, cloud-init isn't a requirement for creating a virtual machine in proxmox, but it can significantly expedite the process and eliminate redundancy. Once you've followed the outlined process, you can proceed to create VMs either by cloning the template or by utilizing an automation route through an Ansible playbook. This approach streamlines the VM creation process and enhances efficiency.
