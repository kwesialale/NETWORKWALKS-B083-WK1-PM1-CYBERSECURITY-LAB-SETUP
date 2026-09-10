# 🔐 Cybersecurity Lab Environment Setup

## 📌 Project Overview

This repository documents the setup of a personal Cybersecurity testing lab built with **VirtualBox** and **Kali Linux**, completed as part of Week 1 Project Module 1 (WK1-PM1) of my Cybersecurity training with **Networkwalks Academy**.

The goal was to build an isolated, internet-connected virtual lab on my own laptop where I can safely practice ethical hacking and penetration testing techniques, without touching my host network or any production system.

---

## 🎯 Objectives

- Install and configure VirtualBox as the virtualization platform
- Deploy Kali Linux as the primary attacking/hacker machine
- Build an isolated lab network using a custom **NAT Network** in the `10.0.0.0/24` subnet
- Assign Kali Linux a static IP of `10.0.0.2/24`
- Confirm Kali Linux has full internet access for updates and tool installation
- Take a clean snapshot of the VM once the base setup is verified working

---

## 🛡️ Purpose of the Lab

This lab exists purely for **learning and ethical practice**. Having a self-contained environment means I can:

- Run scanning, enumeration, and exploitation tools against **my own VMs only**
- Break things, misconfigure things, and fix them again without any real-world risk
- Practice networking fundamentals (IP addressing, gateways, DNS) alongside security tooling
- Build muscle memory for lab setup, since this same NAT Network will host future target VMs (Windows, Server, Android, vulnerable machines like Kioptrix/Metasploitable)

---

## 🏗️ Lab Architecture

The lab sits on top of my host machine (macOS, MacBook Pro) running VirtualBox, with all VMs connected through a single custom NAT Network rather than the default VirtualBox NAT adapter. This gives every VM on the network a routable address in the same subnet and lets them talk to each other, while VirtualBox handles NAT out to the internet.

```
                 Host Machine (macOS)
                          │
                    VirtualBox
                          │
              ┌───────────────────────┐
              │   NAT Network         │
              │   NatNetwork1         │
              │   10.0.0.0/24         │
              │   Gateway: 10.0.0.1   │
              └───────────┬───────────┘
                          │
                 ┌────────┴────────┐
                 │   Kali Linux    │
                 │  10.0.0.2 /24   │
                 │  (Attacker VM)  │
                 └─────────────────┘
```

Future target machines (Windows 10/11, Server, Android, intentionally vulnerable VMs) will be attached to the same `NatNetwork1`, each with its own static IP in the `10.0.0.0/24` range.

---

## ⚙️ Lab Configuration

| Setting | Value |
|---|---|
| Hypervisor | Oracle VirtualBox |
| Attack Machine | Kali Linux 2025.4 (VirtualBox amd64) |
| Network Type | Custom NAT Network (`NatNetwork1`) |
| Subnet | `10.0.0.0/24` |
| Kali Linux IP | `10.0.0.2/24` |
| Gateway | `10.0.0.1` |
| DNS | `8.8.8.8` |
| Network Adapter Mode | NAT Network, Promiscuous Mode: **Allow All** |
| Host OS | macOS Monterey 12.7.6 (MacBook Pro, 13-inch, 2015, Core i5, 8GB RAM) |

---

# 🪜 Lab Setup Procedure

## Step 1. Install VirtualBox

Downloaded and installed the latest recommended version of Oracle VirtualBox for macOS from the official VirtualBox site, and confirmed the VirtualBox Manager launched correctly with an empty machine list ready for import.

## Step 2. Create the NAT Network

![Static IP Configuration](Screenshot%202026-09-10%20at%209.45.57%20AM.png)

Instead of using the default per-VM NAT adapter, I created a dedicated **NAT Network** so multiple VMs could share the same subnet and see each other.

Navigated to **VirtualBox → File → Tools → Network → NAT Networks**, created a network named `NatNetwork1`, and configured it with:

- **IPv4 Prefix:** `10.0.0.0/24`
- **Enable DHCP:** checked
- **IPv6:** left disabled

This network sat alongside the default `NatNetwork` (`10.0.2.0/24`) that VirtualBox creates automatically, so I made sure my Kali VM was pointed at `NatNetwork1` specifically and not the default one.

## Step 3. Import Kali Linux

Downloaded the pre-built Kali Linux VirtualBox image (`kali-linux-2025.4-virtualbox-amd64`) from the official Kali site and imported it as an appliance into VirtualBox.

While reviewing the VM settings, I noticed VirtualBox flagged **"Invalid settings detected"** on the base machine — the Base Memory was set close to my host's physical RAM ceiling (6089 MB on an 8GB host), and the VM had more virtual CPUs assigned than my host's 2 physical CPUs could comfortably support. I brought Base Memory down to a safer **2048 MB** and adjusted the CPU count to bring the VM back within recommended limits before continuing.

![VirtualBox settings showing System configuration](Screenshot%202026-09-10%20at%209.54.51%20AM.png)

Under **System → Network**, I set:
- **Adapter 1 → Attached to:** NAT Network
- **Name:** NatNetwork1
- **Promiscuous Mode:** Allow All
- **Virtual Cable Connected:** checked

## Step 4. Configure the Kali Linux Network

Booted the Kali VM and opened a terminal to check the network state:

![Kali Linux desktop running in VirtualBox](Screenshot%202026-09-09%20at%204.04.57%20PM.jpg)

```bash
ip a
```

At this point `eth0` had picked up a **DHCP address of `10.0.0.3/24`** from the NAT Network — working, but not the static address the lab required (`10.0.0.2/24`).

![Terminal troubleshooting network connection with ifconfig and nmcli](Screenshot%202026-09-09%20at%204.20.09%20PM.jpg)

I opened the **Wired connection 1** settings via the NetworkManager applet, switched **IPv4 Method** to **Manual**, and set:

![kali linux network configuration settings](Screenshot%202026-09-10%20at%209.48.46%20AM.png)

- **Address:** `10.0.0.2`
- **Netmask:** `24`
- **Gateway:** `10.0.0.1`
- **DNS servers:** `8.8.8.8`

Saving this and cycling the connection took a few attempts (documented in the Problems section below) before it took effect properly. Once it did, `ip a` confirmed:

```
eth0: inet 10.0.0.2/24 brd 10.0.0.255 scope global noprefixroute eth0
```

I then confirmed full internet access with:

```bash
ping google.com
```
![Successful ping to google.com confirming internet access](Screenshot%202026-09-09%20at%204.22.48%20PM.jpg)

which returned successful replies with 0% packet loss, confirming Kali could route out through the NAT Network to the internet.

## Step 5. Create a Clean VM Snapshot

Once the static IP was confirmed working and internet access verified, I took a **VirtualBox Snapshot** of the Kali VM in this known-good state (`Snapshot 1`). This gives me a clean restore point to fall back to before running any risky tools or exploits, without having to redo the network setup from scratch.

---

# 🔎 Lab Verification

To confirm the lab was fully functional, I ran the following checks from inside Kali Linux:

| Check | Command | Result |
|---|---|---|
| Interface IP | `ip a` | `eth0` shows `10.0.0.2/24` ✅ |
| Legacy interface view | `ifconfig` | `eth0` UP, RX/TX packets flowing, 0 errors ✅ |
| Internet connectivity | `ping google.com` | 9/9 packets received, 0% loss ✅ |
| Browser connectivity | Firefox → google.com | Search results loaded normally ✅ |

This confirmed Kali Linux was correctly addressed on the `10.0.0.0/24` NAT Network, could resolve DNS, and had full outbound internet access — meeting every requirement of the task.

---

# 🐞 Problems Encountered & Solutions

## Problem 1. Internet Connectivity After Static IP Configuration

**Issue:** After switching `eth0` from DHCP to a manual static IP (`10.0.0.2/24`), the interface would sometimes come up without a usable connection, or the change wouldn't apply cleanly through `ifconfig eth0 down` / `ifconfig eth0 up` alone.

**Troubleshooting steps taken:**
1. Toggled the interface manually with `sudo ifconfig eth0 down` and `sudo ifconfig eth0 up` — this brought the link up but didn't reliably reapply the static addressing.
2. Used `nmcli` instead of raw `ifconfig` to manage the connection properly at the NetworkManager level:
   ```bash
   sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
   sudo nmcli connection down "Wired connection 1"
   sudo nmcli connection up "Wired connection 1"
   ```
3. Re-verified with `ifconfig` and `ip a` until `eth0` showed the correct static address with active RX/TX traffic and no dropped packets.
4. Confirmed the fix by pinging `google.com` successfully.

**Root cause:** Duplicate Address Detection (DAD) delays on the wired connection profile were interfering with how quickly the static IP was accepted after toggling the interface — disabling the DAD timeout and cycling the connection through `nmcli` resolved it.

## Problem 2.  VirtualBox "Invalid Settings Detected" — VM Over-Provisioned

**Issue:** VirtualBox flagged **"Invalid settings detected"** on the Kali VM, warning that the assigned Base Memory (6089 MB) was over 70% of my host's total 8GB RAM, and that more virtual CPUs were assigned than my host's 2 physical CPUs — both of which risked degrading VM performance or preventing the VM from starting cleanly.

**Troubleshooting steps taken:**
1. Opened **VM Settings → System → Motherboard** and reduced Base Memory from 6089 MB down to **2048 MB**, leaving enough headroom for the host OS.
2. Checked **System → Processor** and brought the vCPU count back in line with the host's physical core count.
3. Reviewed **System → Acceleration** to confirm **Nested Paging** was enabled under Hardware Virtualization for better performance.
4. Re-opened Settings to confirm the "Invalid settings detected" warning had cleared before starting the VM again.

**Root cause:** The VM was originally over-provisioned relative to the host's actual hardware (MacBook Pro, 2015, dual-core i5, 8GB RAM), which VirtualBox correctly flagged before it caused instability.

---

# 💡 What I Learned

### 1. NAT vs NAT Network
A regular NAT adapter isolates each VM on its own private link to the host, so VMs can't see each other. A **NAT Network** creates a shared virtual switch that multiple VMs can join, while VirtualBox still handles outbound NAT to the internet — which is exactly what's needed for a multi-VM lab.

### 2. Virtual Machine Networking
Working through the DHCP-to-static transition on `eth0` gave me a much clearer picture of how Linux NetworkManager profiles (`nmcli`), the legacy `ifconfig` tool, and the newer `ip` command relate to each other, and why toggling an interface isn't always enough to force new settings to apply.

### 3. Static IP Configuration
Manually assigning an address, netmask, gateway, and DNS server through the GUI (and validating it from the CLI) reinforced core addressing concepts — and showed me firsthand how a small thing like DAD timeout can block a static IP from taking effect.

### 4. VM Snapshots
Taking a snapshot immediately after confirming a clean, working network configuration means I always have a safe rollback point — a habit I plan to repeat before every future lab exercise.

### 5. Documentation
Writing this process down as I went (including the dead ends) made it much easier to retrace my steps when something broke, and it's the same habit I'll need for reporting findings during actual penetration testing engagements.

---

# 🔐 Security & Ethical Use

This lab is strictly for **authorized, personal learning purposes**. Every machine involved is a VM I own and control on my own hardware. No tools or techniques from this lab are to be used against any system, network, or device without explicit, written authorization.

---

# 🔗 Tools & Resources

- [VirtualBox](https://virtualbox.org/wiki/Downloads) — hypervisor used to host the lab
- [Kali Linux](https://kali.org/get-kali) — attacker VM image
- [Networkwalks Academy](https://networkwalks.com) — course and lab task guidance

---

# 👤 Author

Alale Matthew
Cybersecurity Professional B083
https://www.linkedin.com/in/matthewalale/

## 📌 Project Information

**Task:** Cybersecurity Lab Setup
**Program:** Cybersecurity Training, Networkwalks Academy
**Environment:** VirtualBox on macOS (MacBook Pro, 2015, Intel Core i5, 8GB RAM)
