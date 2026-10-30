# Kali Linux Virtual Lab Setup with Oracle VirtualBox

> Cybersecurity Internship, Week 1 Task
> **Author:** Timothy Ojediran

---

## Overview

This project documents the step-by-step process of building an isolated virtual lab for security practice. Kali Linux 2026.2 was installed as a virtual machine in Oracle VirtualBox and connected to a dedicated **NAT Network**, so that the VM can reach the internet and, in future, communicate with other lab machines on the same private network without being exposed directly to the host's LAN.

**Objectives**

- Install and run Kali Linux inside a virtualised environment
- Create a VirtualBox NAT Network for the lab
- Configure the Kali VM with a static IPv4 address on that network
- Document the configuration for repeatability

---

## Tools Used

| Tool | Purpose |
|------|---------|
| **Oracle VirtualBox** | Hypervisor used to run the virtual machine on the host (Windows 11) |
| **Kali Linux 2026.2** | Penetration-testing and security-auditing Linux distribution |
| **7-Zip** | Extracting the compressed Kali VirtualBox image (`.7z`) |

---

## Step-by-Step Setup Guide

### Step 1: Check the prerequisites

- A host machine with at least 4 GB RAM (8 GB recommended) and 30 GB or more of free disk space
- Hardware virtualisation (Intel VT-x / AMD-V) enabled in the BIOS/UEFI
- Internet access to download the software

### Step 2: Download and install Oracle VirtualBox

1. Go to the official VirtualBox website and download the installer for your host OS (Windows in my case).
2. Run the installer and accept the default options.
3. Allow the network driver installation when Windows prompts for it, then finish the setup.

### Step 3: Download and install 7-Zip

1. Download 7-Zip from the official website (`7-zip.org`).
2. Install it with the default options. It is needed because the Kali VirtualBox image is distributed as a compressed `.7z` archive.

### Step 4: Download the Kali Linux 2026.2 VirtualBox image

1. Open the official Kali download page (`kali.org/get-kali`) and choose **Virtual Machines**.
2. Download the **VirtualBox 64-bit** image for Kali Linux 2026.2.
   
### Step 5: Extract the image with 7-Zip

1. Right-click the downloaded `.7z` file and select **7-Zip > Extract to "<folder name>\"**.
2. Wait for the extraction to finish. The folder will contain the virtual disk (`.vdi`) and its VM configuration file.

### Step 6: Create the virtual machine

1. Open **Oracle VirtualBox** and click **New**.
2. Enter a name for the VM (mine: `Linux`) and select **Type: Linux** with a **64-bit** version (Debian 64-bit is the closest match for Kali).
3. Set **Base Memory** to `2048 MB` (more if your host can spare it).
4. When asked about the hard disk, choose **Use an Existing Virtual Hard Disk File**, then browse to the extracted `.vdi` file and select it.
5. Click **Finish**. The VM now appears in the Machines list in the **Powered Off** state.

### Step 7: Create the NAT Network in VirtualBox

1. In VirtualBox Manager, open **Network** (or **File > Tools > Network Manager**) and select the **NAT Networks** tab.
2. Click **Create** and set:
   - **Name:** `NatNetwork`
   - **IPv4 Prefix:** `10.0.0.0/24`
   - **Enable DHCP:** checked
   - **Enable IPv6:** unchecked
3. Click **Apply**.

### Step 8: Attach the VM to the NAT Network

1. Select the VM and click **Settings > Network**.
2. Under **Adapter 1**, tick **Enable Network Adapter**.
3. Set **Attached to:** `NAT Network` and **Name:** `NatNetwork`.
4. Under **Advanced**, keep the adapter type as **Intel PRO/1000 MT Desktop**.
5. Click **OK** to save.

### Step 9: Start the VM and log in

1. Select the VM and click **Start**.
2. Log in with the account for the image. Pre-built Kali images ship with a default `kali` / `kali` login, so **change the password immediately** after the first login (run `passwd` in the terminal).

### Step 10: Configure a static IP address in Kali

1. Inside Kali, open **Advanced Network Configuration** (or run `nm-connection-editor` in the terminal).
2. Select **Wired connection 1** and click the gear/edit icon.
3. Open the **IPv4 Settings** tab and set **Method** to `Manual`.
4. Click **Add** and enter:
   - **Address:** `10.0.0.2`
   - **Netmask:** `24`
   - **Gateway:** `10.0.0.1`
5. Set **DNS servers** to `8.8.8.8`.
6. Click **Save**, then disconnect and reconnect the wired connection so the new settings apply.


## Virtual Machine Specifications

| Setting | Value |
|---------|-------|
| VM Name | Linux (NATNetwork Configuration Inside Kali) |
| OS Type (in VirtualBox) | Ubuntu (64-bit) |
| Base Memory | 2048 MB |
| Storage | `Linux.vdi`, 25 GB (SATA) |
| Graphics Controller | VMSVGA (16 MB video memory) |
| Acceleration | Nested Paging, KVM Paravirtualization |
| Boot Order | Floppy, Optical, Hard Disk |
| Network Adapter 1 | Intel PRO/1000 MT Desktop, attached to **NAT Network** (`NatNetwork`) |

<img width="960" height="504" alt="screenshot-vm-properties" src="https://github.com/user-attachments/assets/24d97e0f-c167-4d0c-b4d4-db7c4b023dd8" />



## Network Configuration

### 1. VirtualBox NAT Network (host side)

Created under **File > Tools > Network > NAT Networks**:

| Setting | Value |
|---------|-------|
| Name | `NatNetwork` |
| IPv4 Prefix | `10.0.0.0/24` |
| DHCP Server | Enabled |
| IPv6 | Disabled |

<img width="960" height="540" alt="Screenshot-network-settings" src="https://github.com/user-attachments/assets/159d47da-4785-41ab-be9b-ce8feca3774b" />


### 2. VM network adapter

In the VM's **Settings > Network > Adapter 1**:

- **Attached to:** NAT Network
- **Name:** `NatNetwork`
- **Adapter type:** Intel PRO/1000 MT Desktop

### 3. Static IP inside Kali (guest side)

Configured through **Advanced Network Configuration > Wired connection 1 > IPv4 Settings**:

| Setting | Value |
|---------|-------|
| Method | Manual |
| Address | `10.0.0.2` |
| Netmask | `24` (255.255.255.0) |
| Gateway | `10.0.0.1` |
| DNS Server | `8.8.8.8` |

<img width="1920" height="1008" alt="Screenshot-kali-network-settings" src="https://github.com/user-attachments/assets/8c400427-cd0d-4571-9524-99cc07d5b704" />


### 4. Verification

Run the following in the Kali terminal to confirm the configuration:

```bash
ip a                  # confirm 10.0.0.2/24 is assigned
ip route              # confirm default route via 10.0.0.1
ping -c 4 10.0.0.1    # reach the NAT gateway
ping -c 4 8.8.8.8     # confirm internet connectivity
ping -c 4 google.com  # confirm DNS resolution
```

---

## Task Completed

- [x] Downloaded Kali Linux 2026.2 and extracted the archive with 7-Zip
- [x] Installed Oracle VirtualBox on the host machine
- [x] Created and configured the virtual machine (2 GB RAM, 25 GB disk, VMSVGA)
- [x] Created a NAT Network (`NatNetwork`, `10.0.0.0/24`) with DHCP enabled
- [x] Attached the VM's network adapter to the NAT Network
- [x] Booted Kali Linux successfully and logged in

  <img width="1920" height="1008" alt="Screenshot-kali-linux" src="https://github.com/user-attachments/assets/c93a62b9-d97b-4d20-a4d6-dcd2effb992c" />


- [x] Assigned a static IPv4 address (`10.0.0.2/24`) with gateway and DNS
- [x] Verified network connectivity from inside the VM
- [x] Updated the system and saved a clean baseline snapshot

---

## Lessons Learned

1. **Virtualisation keeps the lab safe.** Running Kali in a VM isolates security tools from the host operating system, so testing can be done without risking the main machine.
2. **NAT Networks suit multi-VM labs.** Unlike plain NAT, a NAT Network lets several VMs on the same network talk to each other while still sharing outbound internet access through the host.
3. **Network settings must agree on both sides.** The VirtualBox NAT Network prefix (`10.0.0.0/24`) and the static IP, netmask and gateway inside Kali must match, otherwise the VM has no connectivity.
4. **Static vs DHCP addressing.** A manual address (`10.0.0.2`) gives predictable IPs for lab documentation and future scans, but it must be chosen outside the DHCP-assigned range to avoid conflicts.
5. **Resources matter.** Allocating enough RAM and disk space affects VM performance, and nested paging / paravirtualisation settings help the VM run smoothly.
6. **Document as you go.** Screenshots and notes taken during setup make the process easy to repeat or troubleshoot later.

