<h1>Active Directory Home Lab</h1>

<h2>Description</h2>
In this lab..
<br />


<h2>Languages and Utilities Used</h2>

- <b>PowerShell</b> 
- <b>Diskpart</b>

<h2>Environments Used </h2>

- <b>Windows 10</b> (21H2)

<h2>Program walk-through:</h2>

# Active Directory Home Lab (VirtualBox + Windows Server + Windows 10/11)

I built this project while completing my **Master’s in Cybersecurity & Information Assurance**.  
It’s a small, realistic Active Directory lab that runs on a single computer using **Oracle VirtualBox**.  
The lab includes a Windows Server **Domain Controller** (AD DS, DNS, DHCP, NAT) and a Windows **Client**, plus a PowerShell script to bulk-create users.

---

## Table of Contents
1. [What You’ll Build](#what-youll-build)  
2. [Before You Start](#before-you-start)  
3. [Plan the Network](#plan-the-network)  
4. [Create the Domain Controller VM](#create-the-domain-controller-vm)  
5. [Configure the DC: Name, NICs, Static IP](#configure-the-dc-name-nics-static-ip)  
6. [Install AD DS and Create the Forest](#install-ad-ds-and-create-the-forest)  
7. [Set Up NAT with Routing and Remote Access](#set-up-nat-with-routing-and-remote-access)  
8. [Install and Configure DHCP](#install-and-configure-dhcp)  
9. [Create the Client VM](#create-the-client-vm)  
10. [Join the Client to the Domain](#join-the-client-to-the-domain)  
11. [Bulk-Create Users with PowerShell (Optional)](#bulk-create-users-with-powershell-optional)  
12. [Verification Checklist](#verification-checklist)  
13. [Snapshots & Reset Tips](#snapshots--reset-tips)  
14. [Repo Contents](#repo-contents)  
15. [Notes & Safety](#notes--safety)  
16. [Credits](#credits)

---

## What You’ll Build
- **1× Domain Controller (DC)** on Windows Server (AD DS, DNS, DHCP, NAT via RRAS)  
- **1× Client** on Windows 10/11 joined to the domain  
- **Internal network** for the lab, and **internet access** for the client through the DC  
- **PowerShell script** to create test users in bulk

---

## Before You Start
- **VirtualBox** installed on your host machine  
- **Windows Server 2019/2022** ISO (for the DC)  
- **Windows 10 or 11** ISO (for the client)  
- At least **12 GB RAM** free on the host  
  - Suggested: DC = 4–6 GB RAM, Client = 4 GB RAM  
- Enough disk space (plan ~40–60 GB total for both VMs)

> I do not include ISO/VDI files in this repo.

---

## Plan the Network
| Component | Network       | Addressing & Notes                              |
|-----------|---------------|--------------------------------------------------|
| DC – NIC 1| **NAT**       | VirtualBox NAT (internet for the DC)            |
| DC – NIC 2| **Internal**  | `172.16.0.1/24` (static). DNS = `172.16.0.1`     |
| Client    | **Internal**  | DHCP from DC (`172.16.0.100–172.16.0.200`)       |

I use **172.16.0.0/24** for the lab. Feel free to adjust, but keep it consistent.

---

## Create the Domain Controller VM
1. **New VM → Windows Server (2019/2022)**  
2. Suggested settings:  
   - **Memory:** 4096–6144 MB  
   - **CPU:** 2 vCPUs  
   - **Disk:** 40–60 GB (dynamically allocated is fine)  
3. **Network** (VirtualBox → VM → Settings → Network):  
   - **Adapter 1:** **NAT** (leave defaults)  
   - **Adapter 2:** **Internal Network** (create or choose a name like `LABNET`)  
4. **Storage:** Attach the Server ISO and install Windows normally.

---

## Configure the DC: Name, NICs, Static IP
1. **Rename the computer** to `DC` → restart.  
2. **Rename NICs** in *Network Connections* (optional but clearer):  
   - NAT adapter → `INTERNET`  
   - Internal adapter → `INTERNAL`  
3. **Set static IP** on the `INTERNAL` NIC:  
   - IP: `172.16.0.1`  
   - Mask: `255.255.255.0`  
   - DNS: `172.16.0.1` (the DC will run DNS)

---

## Install AD DS and Create the Forest
1. Open **Server Manager → Add roles and features** → **Active Directory Domain Services** → Install.  
2. In Server Manager, click the yellow flag → **Promote this server to a domain controller**.  
3. **Add a new forest**, domain name: `mydomain.com` (use your own if you prefer).  
4. Accept defaults, install, and **reboot** when prompted.  
5. After reboot, **sign in with the domain** Administrator account.

---

## Set Up NAT with Routing and Remote Access
Goal: let the **Client** on the Internal network reach the internet **through the DC**.

1. **Server Manager → Add roles and features** → **Remote Access**.  
2. On the Role Services step, select **Routing** → Install.  
3. Open **Routing and Remote Access** (RRAS) → right-click the server → **Configure and Enable**.  
4. Choose **NAT** → select the **public interface** (your `INTERNET`/NAT NIC).  
5. Start the service and confirm the green status light.

---

## Install and Configure DHCP
1. **Server Manager → Add roles and features** → **DHCP Server** → Install.  
2. Open **DHCP** (mmc). Under **IPv4**, create a new scope:  
   - **Start IP:** `172.16.0.100`  
   - **End IP:** `172.16.0.200`  
   - **Subnet Mask:** `255.255.255.0`  
   - **Router (Gateway):** `172.16.0.1`  
   - **DNS Server:** `172.16.0.1`  
3. **Authorize** the DHCP server (if prompted) and **activate** the scope.

---

## Create the Client VM
1. **New VM → Windows 10/11**  
2. Suggested settings: **4 GB RAM**, **2 vCPUs**, **40 GB disk**.  
3. **Network:**  
   - **Adapter 1:** **Internal Network** (same name as the DC’s internal network, e.g., `LABNET`)  
   - No NAT on the client (the DC handles internet).  
4. **Install Windows**, complete OOBE, and sign in locally.

---

## Join the Client to the Domain
1. On the client, run `ipconfig`. You should have a **172.16.0.x** address from DHCP.  
2. Test connectivity:
   - `ping 172.16.0.1` (DC)  
   - `ping mydomain.com` (DNS resolution)  
   - `ping www.microsoft.com` (internet via DC)  
3. **Rename** the client to `CLIENT1` (optional), then **join the domain** `mydomain.com`.  
4. Reboot and **sign in with a domain account**.

---

## Bulk-Create Users with PowerShell (Optional)
This repo includes a script under `scripts/create-ad-users.ps1` and a sample `scripts/names-sample.txt`.

**Run on the DC** in an elevated PowerShell window (as a domain admin):

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
cd C:\Path\to\repo\scripts
.\create-ad-users.ps1 -DomainDN "DC=mydomain,DC=com" -OuName "_USERS" -NamesFile ".\names-sample.txt"
