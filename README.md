# NETWORKWALKS-B084J-WK1-CYBERSECURITY-LAB_setup
Cybersecurity lab setup and practical work for week 1, including basic networking concept
# Cybersecurity Lab Environment Setup

Building an isolated virtual lab for ethical hacking and security practice using VirtualBox and Kali Linux.

![Skill](https://img.shields.io/badge/Skill-Cybersecurity-red) ![VirtualBox](https://img.shields.io/badge/VirtualBox-7.2.22-blue) ![Kali](https://img.shields.io/badge/Kali%20Linux-2026.3-black) ![Network](https://img.shields.io/badge/Network-10.0.0.0%2F24-green) ![Networkwalks](https://img.shields.io/badge/Networkwalks-WK1--PM1-orange)

---

## Project Overview

This project sets up a virtual cybersecurity testing lab on my own laptop using **VirtualBox** and **Kali Linux**. The lab runs on a private NAT Network so more machines can be added later and used as targets for authorized practice.

**Author:** [SAIRA SHAMIM]
**Program:** Networkwalks Cybersecurity Internship, Batch [B084J]
**Task:** WK1-PM1, Lab Setup (VirtualBox and Kali Linux)

> All testing is done only inside my own lab environment on devices I own.

---

## Lab Requirements

| Requirement | Status |
|---|---|
| VirtualBox as the base (latest version) | Done |
| Kali Linux as the attacker machine | Done |
| Network in subnet 10.0.0.0/24 | Done |
| NAT Network with 10.0.0.0/24 | Done |
| Clipboard and drag-and-drop enabled | Done |
| Shared folder (host Downloads) enabled | Done |
| Kali IP address 10.0.0.2/24 | Done |
| Kali has full internet access | Done |

---

## Environment

| Item | Details |
|---|---|
| Host OS | Windows 11 |
| Host RAM | 16 GB |
| Hypervisor | Oracle VirtualBox 7.2.22 |
| Guest OS | Kali Linux 2026.3 (installed from the official installer ISO) |
| VM resources | 4 GB RAM, 2 CPUs, 50 GB virtual disk |
| Desktop environment | Xfce |

## Network Design

| Item | Value |
|---|---|
| Network type | VirtualBox NAT Network (`NatNetwork`) |
| Subnet | 10.0.0.0/24 |
| DHCP | Enabled |
| Kali IP (manual) | 10.0.0.2/24 |
| Gateway | 10.0.0.1 |
| DNS | 8.8.8.8 |

---

## Step-by-Step Setup

### Step 1: Install VirtualBox
Downloaded the latest VirtualBox for Windows from virtualbox.org and installed it with default options.

### Step 2: Create the NAT Network
Opened **File > Tools > Network > NAT Networks**. The default network used the prefix `10.0.2.0/24`, so I changed it to **10.0.0.0/24** and kept DHCP enabled.

![NAT Network]("D:\My Softwares\week 1 internship\virtual_network.png")

### Step 3: Create the Kali VM
Downloaded the Kali Linux installer ISO from kali.org and created a new VM:
- Type: Linux, Debian (64-bit)
- Memory: 4096 MB, CPUs: 2
- Disk: 50 GB (dynamically allocated)
- Unattended installation left **disabled**, so I could run the installer myself

### Step 4: Configure VM settings
- **Network:** Adapter 1 attached to **NAT Network** (`NatNetwork`), Promiscuous Mode set to **Allow All**
- **General > Advanced:** Shared Clipboard and Drag-and-Drop set to **Bidirectional**

![VM Network settings]("D:\My Softwares\week 1 internship\kali_NATnetwork.png")
![Clipboard and drag-and-drop]("D:\My Softwares\week 1 internship\kali_drag&drop.png")

### Step 5: Install Kali Linux
Booted the VM from the ISO and ran the graphical installer:
- Hostname: `kali`, domain name left empty
- Partitioning: Guided, use entire disk, all files in one partition
- Software selection: default (Xfce, top10, default tools)
- Installed the GRUB boot loader on `/dev/sda`

After installation completed, the ISO was ejected and the VM booted into the Kali desktop.

### Step 6: Install VirtualBox Guest Additions
Because Kali was installed from the ISO, Guest Additions were not preinstalled. I installed them from inside Kali:

```bash
sudo apt update
sudo apt install -y virtualbox-guest-x11
sudo reboot
```

After the reboot, screen auto-resize and clipboard sharing worked.

### Step 7: Configure the shared folder

![Shared folder settings]("D:\My Softwares\week 1 internship\kali_working.png")

Added the Windows **Downloads** folder under VM **Settings > Shared Folders** with **Auto-mount** enabled and read-only turned off. Then added my user to the `vboxsf` group and rebooted:

```bash
sudo usermod -aG vboxsf $USER
sudo reboot
```


### Step 8: Set the static IP address
In **Edit Connections > Wired connection 1 > IPv4 Settings**, I set the method to **Manual** with:
- Address: 10.0.0.2, Netmask: 24, Gateway: 10.0.0.1
- DNS: 8.8.8.8

![Wired Connections settings]("D:\My Softwares\week 1 internship\kali_wired_connection_working.png")

Then restarted the connection:

```bash
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

### Step 9: Take a snapshot
Took a VM snapshot named **Clean Kali 10.0.0.2** so I can restore this working state at any time.

![Snapshot]("D:\My Softwares\week 1 internship\kali_snapshot.png")

---

## Verification

**IP address (`ip a`):** `eth0` has 10.0.0.2/24 and is UP.

![ip a]("D:\My Softwares\week 1 internship\ping a.png")

**Connectivity:** Gateway, public IP, and DNS name all respond with 0% packet loss.

```bash
ping -c 3 10.0.0.1
ping -c 3 8.8.8.8
ping -c 3 google.com
```

![Ping tests]("D:\My Softwares\week 1 internship\ping(internet-working).png")

**Shared folder:** The Windows Downloads folder is visible inside Kali.

```bash
ls /media/sf_Downloads
```

![Shared folder](screenshots/07-shared-folder.png)

---

## Troubleshooting

| Problem | Solution |
|---|---|
| The NAT Network used 10.0.2.0/24 instead of the required 10.0.0.0/24 | Edited the IPv4 prefix of `NatNetwork` to 10.0.0.0/24 and clicked Apply. |
| The Kali installer pre-filled the hostname and domain fields with values taken from DHCP | Replaced the hostname with `kali` and cleared the domain field. |
| Installer seemed stuck on "Starting up the partitioner" | Waited and refreshed the screen, and the installer continued. |
| Clipboard, drag-and-drop, and shared folder did not work after an ISO install | Installed `virtualbox-guest-x11` and rebooted. |
| Shared folder could not be accessed | Added my user to the `vboxsf` group and rebooted. |
| Possible internet problems after setting a static IP (common on VirtualBox 7 with new Kali versions) | Ran `nmcli` with `ipv4.dad-timeout 0`, restarted the connection, and, if needed, switched DNS to 10.0.0.1. |

---

## What I Learned

- How a NAT Network isolates lab machines while still allowing internet access
- The difference between a pre-built Kali VM and a manual ISO install, including why Guest Additions must be installed manually
- How to set static IPs with NetworkManager and verify them with `ip a` and `ping`
- How snapshots protect a clean, working lab state

## Next Steps
- Add vulnerable target machines (Windows, Android, CTF VMs) on the same NAT Network
- Practice scanning and reconnaissance only against machines I own

---

## Disclaimer

This lab is for education only. All activity takes place in my own virtual environment. Never test systems without written permission from the owner.

---
*Networkwalks Cybersecurity Internship | Instructor: Waqas Karim CCIE*
