---
layout: post
title: Proxmox Provisioning VMs using Terraform 
categories: [Virtualization]
tags: [linux, proxmox, terraform, provisioning]
enable_toc: true
render_with_liquid: true
date: 2024-06-25 00:00:00 +0530
---


## What is Terraform?

Terraform, created by HashiCorp, is an Infrastructure as Code (IaC) tool that lets you manage and set up your infrastructure using code. In the past, setting up or configuring infrastructure meant doing it all manually. But now, with Terraform, you can easily deploy infrastructure through code instead of using a WebUI or SSH. Plus, it allows you to manage configurations across various public and private cloud environments.

## Prepare User and API

Before installing Terraform, we must ensure that the user and API keys are created. This information is essential for deploying our VMs. 

### Step 1 - Create a User in Proxmox

![Proxmox UI](/assets/img/proxmox/terraform-proxmox.png)

- Log in to **Proxmox**.
- Navigate to the **Datacenter**.
- Locate **API Keys** under **Permissions**.
- Click on **Add**.
- Provide a random **Token ID**.
- Click **Add** to generate the keys.
- *Save the key securely, such as in a Password Manager or a hidden file.*

### Step 2 - Cloudinit Template as bash OS

Now, we need to create a cloud-init template that will serve as the base OS configuration. This template will be used when we deploy our code to your instance, ensuring that the machine is set up and configured according to your specifications.

### Step 3 - Installation of Terraform 

Next, we need to install Terraform to set up the environment, allowing us to create configuration files for deploying infrastructure. Terraform can be installed on Linux, Windows, or macOS, depending on the operating system of your main device.

##### Install dependencies 

```shell

sudo apt update
sudo apt install  software-properties-common gnupg2 curl

```

##### Add hashicorp repository

```shell

curl https://apt.releases.hashicorp.com/gpg | gpg --dearmor > hashicorp.gpg
sudo install -o root -g root -m 644 hashicorp.gpg /etc/apt/trusted.gpg.d/

```

##### Install Terraform

```shell

sudo apt install terraform

```

##### Check the current version

```shell

terraform --version

```

## Create a Config file on Editor

Now that we have Terraform installed on our machine, it's time to create configuration files. These files will describe our infrastructure as code, enabling us to deploy virtual machines.

### Step 1 - Create a main.tf file 

Open your preferred text editor and create a file named `main.tf`. This file will serve as the primary configuration file where we will define our variables and states.

### Step 2 - Add Proxmox provider - Telmate/proxmox

We need to let Terraform know which provider to use. A provider is basically the connector that allows Terraform to interact with different services. Since we're working with Proxmox, we'll need to use the Proxmox provider. The most reliable provider I've found is Telmate, an open-source project.

```tf

# file: main.tf

terraform {
  required_providers {
    proxmox = {
      source = "telmate/proxmox"
      version = "2.9.11"
    }
  }
}

```

### Step 3 - Initialize Terraform to grab provider

We need to initialize Terraform before adding our state to ensure that the provider is downloaded and available for use. Upon initialization, we should receive a message confirming the successful setup, it should installed plugins and have created a lock file.

```shell

terraform init

```

```text

# output message

Initializing the backend...
Initializing provider plugins...
- Reusing previous version of telmate/proxmox from the dependency lock file
- Using previously-installed telmate/proxmox v2.9.11

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

```

### Step 4 - Now describe your deployment state in main.tf

Once Terraform has been initialized, we need to define our deployment state according to our specifications and requirements.

```tf

# file: main.tf

terraform {
  required_providers {
    proxmox = {
      source = "telmate/proxmox"
      version = "2.9.11"
    }
  }
}

# Configure the Proxmox provider
provider "proxmox" {
  # The URL of the Proxmox API
  pm_api_url      = "https://your-proxmox-server:8006/api2/json"
  # The user to authenticate as
  pm_user         = "root@pam"
  # The password for the user
  pm_password     = "your-password"
  # Whether to ignore TLS certificate errors
  pm_tls_insecure = true
}

# Define a new VM resource
resource "proxmox_vm_qemu" "example" {
  # The name of the VM
  name        = "terraform-vm"
  # The Proxmox node where the VM will be created
  target_node = "proxmox-node"
  # Clone an existing VM template to create the new VM
  clone {
    # The ID of the VM template to clone
    vm_id     = "100"
    # Whether to perform a full clone
    full_clone = true
  }

  # Specify the OS type, in this case, using cloud-init
  os_type = "cloud-init"
  # Configure the VM's disk
  disks {
    # The ID of the disk
    id           = 0
    # The size of the disk
    size         = "10G"
    # The type of the disk
    type         = "scsi"
    # The storage location for the disk
    storage      = "local-lvm"
    # The storage type
    storage_type = "lvm"
  }

  # Configure the VM's network interface
  network {
    # The ID of the network interface
    id      = 0
    # The model of the network interface
    model   = "virtio"
    # The bridge to connect the network interface to
    bridge  = "vmbr0"
  }

  # Configure the CPU settings
  cpu {
    # The number of sockets
    sockets = 1
    # The number of cores per socket
    cores   = 2
  }

  # Configure the memory settings
  memory {
    # The amount of dedicated memory in MB
    dedicated = 2048
  }
  # Add an SSH public key for access
  sshkeys = <<EOF
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC3...
EOF

  # Set the IP configuration for the VM
  ipconfig0 = "ip=192.168.1.100/24,gw=192.168.1.1"
  # Manage lifecycle settings, ignoring changes to network and disks
  lifecycle {
    ignore_changes = [
      network,
      disks,
    ]
  }
}


```


### Step 5 - Run Terraform Plan to review

After defining our desired deployment state according to our specifications, we need to run **terraform plan**. This command in Terraform enables you to preview the changes that will be made by your Terraform configuration before actually applying them.

```shell

terraform plan 

```

```text

# output message

Terraform will perform the following actions:

  # proxmox_vm_qemu.example will be created
  + resource "proxmox_vm_qemu" "example" {
      + agent         = (known after apply)
      + balloon       = (known after apply)
      + bios          = (known after apply)
      + boot          = (known after apply)
      + bootdisk      = (known after apply)
      + clone         = {
          + full_clone = true
          + vm_id      = "100"
        }
      + cores         = 2
      + cpu           = {
          + cores   = 2
          + sockets = 1
        }
      + disks         = [
          + {
              + id           = 0
              + size         = "10G"
              + storage      = "local-lvm"
              + storage_type = "lvm"
              + type         = "scsi"
            },
        ]
      + id            = (known after apply)
      + ipconfig0     = "ip=192.168.1.100/24,gw=192.168.1.1"
      + memory        = {
          + dedicated = 2048
        }
      + name          = "terraform-vm"
      + network       = [
          + {
              + bridge = "vmbr0"
              + id     = 0
              + model  = "virtio"
            },
        ]
      + os_type       = "cloud-init"
      + sshkeys       = <<-EOF
            ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC3...
        EOF
      + target_node   = "proxmox-node"
    }

Plan: 1 to add, 0 to change, 0 to destroy.

```

### Step 6 - Run Terraform Apply to deploy

Once we have reviewed all the changes that Terraform will implement through the configuration, we need to execute **terraform apply**. This command in Terraform applies the necessary changes to achieve the desired state as defined in your .tf files. It is the stage where Terraform actually modifies your infrastructure according to the execution plan generated by **terraform plan**.

```shell

terraform apply

```

```text

# output message

Terraform will perform the following actions:

  # proxmox_vm_qemu.example will be created
  + resource "proxmox_vm_qemu" "example" {
      + agent         = (known after apply)
      + balloon       = (known after apply)
      + bios          = (known after apply)
      + boot          = (known after apply)
      + bootdisk      = (known after apply)
      + clone         = {
          + full_clone = true
          + vm_id      = "100"
        }
      + cores         = 2
      + cpu           = {
          + cores   = 2
          + sockets = 1
        }
      + disks         = [
          + {
              + id           = 0
              + size         = "10G"
              + storage      = "local-lvm"
              + storage_type = "lvm"
              + type         = "scsi"
            },
        ]
      + id            = (known after apply)
      + ipconfig0     = "ip=192.168.1.100/24,gw=192.168.1.1"
      + memory        = {
          + dedicated = 2048
        }
      + name          = "terraform-vm"
      + network       = [
          + {
              + bridge = "vmbr0"
              + id     = 0
              + model  = "virtio"
            },
        ]
      + os_type       = "cloud-init"
      + sshkeys       = <<-EOF
            ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC3...
        EOF
      + target_node   = "proxmox-node"
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

proxmox_vm_qemu.example: Creating...
proxmox_vm_qemu.example: Still creating... [10s elapsed]
proxmox_vm_qemu.example: Still creating... [20s elapsed]
proxmox_vm_qemu.example: Still creating... [30s elapsed]
proxmox_vm_qemu.example: Creation complete after 40s [id=terraform-vm]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.


```

### Step 7 - Verify installation via SSH access

After running terraform apply to deploy our configuration, we need to return to the Proxmox interface to verify that the VM has been successfully deployed. Once confirmed, check the Cloud-init configuration in the Proxmox UI to ensure that all settings specified in the .tf file have been correctly applied before starting the VM.

Next, verify that you can SSH into the VM using the provided SSH key from the deployment. Once you have successfully accessed the VM via SSH, you should be all set.

### Step 8 - Remove deployed VM using Terraform Destroy

By any means, If you ever decide that you no longer need the deployed VM and want to remove it from Proxmox, you can simply accomplish this by running **terraform destroy** command. This command will take care of tearing down the infrastructure managed by your Terraform setup. Essentially, it reverse everything that **terraform apply** did, wiping out all the resources defined in your .tf files.

```shell

terraform destroy

```

```text

# output message

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # aws_instance.example will be destroyed
  - resource "aws_instance" "example" {
      - ami           = "ami-0c55b159cbfafe1f0" -> null
      - instance_type = "t2.micro" -> null
      - tags          = {
          - "Name" = "example-instance"
        } -> null
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you want to perform these actions in workspace "default"?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_instance.example: Destroying... [id=i-0abcd1234efgh5678]
aws_instance.example: Still destroying... [id=i-0abcd1234efgh5678, 10s elapsed]
aws_instance.example: Destruction complete after 15s

Destroy complete! Resources: 1 destroyed.
```


## Summary

Terraform is a game changer for managing infrastructure. It lets you handle everything as code, so you can forget about the hassle of manual setups and configurations through WebUI or SSH. Whether you're working with public or private clouds, Terraform makes it super easy to manage your infrastructure. By following the steps outlined in the blog post, you'll have a solid starting point to use Terraform for deploying your infrastructure, whether it's on a local on-premise server or a cloud platform. 
 
