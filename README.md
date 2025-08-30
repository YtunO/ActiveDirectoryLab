# Active Directory Home Lab (VirtualBox + Windows Server + Windows 10/11)

I built this home lab as a graduate student in a Master’s program in Cybersecurity & Information Assurance.  
This project was inspired by **Josh Madakor’s YouTube tutorial**:  
📺 Watch the original video here: [YouTube Link](https://youtu.be/MHsI8hJmggI?si=JndBJOHAcuPhE79c)  

---

## Objective
This project was built to give me practical experience setting up Active Directory and Windows networking in a home lab. My goal was to understand how services like DNS, DHCP, and NAT work together, learn how to join a client to a domain, and get comfortable managing users and groups. Writing everything down also helped me practice explaining technical steps in a clear way for my portfolio.

---

## Skills Learned
- Configuring **Active Directory Domain Services (AD DS)**
- Setting up **DNS, DHCP, and NAT routing** in Windows Server
- Automating **bulk user creation at scale** (1,000 users with PowerShell ISE)
- Understanding **Windows networking** (IP addressing, internal vs external networks)
- Creating and managing **domain users, groups, and organizational units (OUs)**
- Practicing **identity and access management (IAM)** in a safe lab environment
- Using **snapshots and resets** for testing and troubleshooting
- Documenting and presenting a technical project for a portfolio

---

## Tools Used
- **Oracle VirtualBox** (virtualization platform)
- **Windows Server 2019/2022** (Domain Controller)
- **Windows 10/11** (domain client machine)
- **Active Directory Users and Computers (ADUC)**
- **DHCP, DNS, and Routing and Remote Access Services (RRAS)**
- **PowerShell ISE** (for bulk user creation)

---

## Table of Contents
1. [What You’ll Build](#what-youll-build)  
2. [Before You Start](#before-you-start)  
3. [Network Plan](#network-plan)  
4. [Create the Domain Controller VM](#create-the-domain-controller-vm)  
5. [Configure the DC: Name, NICs, Static IP](#configure-the-dc-name-nics-static-ip)  
6. [Install AD DS and Create the Forest](#install-ad-ds-and-create-the-forest)  
7. [Set Up NAT with Routing and Remote Access](#set-up-nat-with-routing-and-remote-access)  
8. [Install and Configure DHCP](#install-and-configure-dhcp)  
9. [Bulk Add 1,000 Users with PowerShell ISE](#bulk-add-1000-users-with-powershell-ise)  
10. [Create the Client VM](#create-the-client-vm)  
11. [Join the Client to the Domain](#join-the-client-to-the-domain)  
12. [Verification Checklist](#verification-checklist)  
13. [Snapshots & Reset Tips](#snapshots--reset-tips)  
14. [Notes & Safety](#notes--safety)  
15. [Credits](#credits)  

---

## What You’ll Build
- **One Windows Server Domain Controller** providing:
  - Active Directory Domain Services (AD DS)
  - DNS
  - DHCP
  - NAT routing (so the client can reach the internet through the DC)  
- **One Windows 10/11 Client** joined to the domain  
- An **internal, isolated lab network**  

*Add your overview screenshot here.*

---

## 2. Before You Start
- Oracle VirtualBox installed  
- Windows Server 2019/2022 ISO  
- Windows 10 or 11 ISO  
- At least 12 GB RAM available on your host (suggestion: DC 4–6 GB, Client 4 GB)  
- Sufficient disk space (about 40–60 GB total)  

---

## 3. Network Plan
- Lab subnet: `172.16.0.0/24`  
- Domain Controller:  
  - NIC 1 → NAT (internet access for the DC)  
  - NIC 2 → Internal Network (static IP `172.16.0.1`, DNS = itself)  
- Client:  
  - NIC → Internal Network (gets IP from DHCP on the DC)  

*Add a small topology image here.*

---

## 4. Create the Domain Controller VM
1. Create a new VM for Windows Server in VirtualBox.  
2. Assign memory (4–6 GB), 2 CPUs, and a 40–60 GB disk.  
3. Add two network adapters:  
   - Adapter 1 → NAT  
   - Adapter 2 → Internal Network (name your internal network, e.g., `LABNET`).  
4. Attach the Windows Server ISO and complete the installation.  

*Add screenshots of VirtualBox settings and OS install.*

---

## 5. Configure the DC: Name, NICs, Static IP
1. Rename the computer to `DC` and restart.  
2. Rename the NICs for clarity:  
   - NAT adapter → `INTERNET`  
   - Internal adapter → `INTERNAL`  
3. On the `INTERNAL` adapter, set:  
   - IP address: `172.16.0.1`  
   - Subnet mask: `255.255.255.0`  
   - DNS server: `172.16.0.1`  

*Add screenshots of adapter names and IPv4 settings.*

---

## 6. Install AD DS and Create the Forest
1. In Server Manager, add the role: **Active Directory Domain Services**.  
2. Promote the server to a domain controller.  
3. Create a new forest (example: `mydomain.com`).  
4. Accept defaults and restart when prompted.  

*Add screenshots of the AD DS wizard and domain setup.*

---

## 7. Set Up NAT with Routing and Remote Access
1. In Server Manager, add the role: **Remote Access** (with Routing).  
2. Open the Routing and Remote Access console.  
3. Run the wizard → choose **NAT**.  
4. Select the `INTERNET` adapter as the public interface → start the service.  

*Add screenshots of RRAS setup and status.*

---

## 8. Install and Configure DHCP
1. In Server Manager, add the role: **DHCP Server**.  
2. In the DHCP console, create a new IPv4 scope:  
   - Start: `172.16.0.100`  
   - End: `172.16.0.200`  
   - Subnet mask: `255.255.255.0`  
   - Router (gateway): `172.16.0.1`  
   - DNS server: `172.16.0.1`  
3. Authorize and activate the DHCP server.  

*Add screenshots of DHCP scope configuration.*

---

## 9. Bulk Add 1,000 Users with PowerShell ISE
**Goal:** Test scalability and automation by creating **1,000 domain users** in Active Directory.  

**Steps:**  
1. Prepare a text file with 1,000 names (one per line, e.g., “First Last”).  
2. On the Domain Controller, open **PowerShell ISE as Administrator**.  
3. Load the bulk-user script and point it to:  
   - The target OU (e.g., `_USERS`)  
   - Your domain (e.g., `mydomain.com`)  
   - The names list file  
4. Run the script and watch as users are created in bulk.  
5. Verify in **Active Directory Users and Computers (ADUC):**  
   - Open the OU → confirm ~1,000 users exist  
   - Optional: sort by creation date or check properties  

**Screenshots to add here:**  
- ISE showing script execution  
- ADUC with ~1,000 user objects listed  

**What I learned:**  
- Automating account creation at scale saves hours of manual work.  
- Bulk actions require good **OU structure and naming standards**.  
- Snapshots are critical — easy rollback if something breaks.  

---

## 10. Create the Client VM
1. Create a new VM for Windows 10/11 in VirtualBox.  
2. Assign memory (4 GB), 2 CPUs, and a 40 GB disk.  
3. Add one adapter: **Internal Network** (same as the DC’s `LABNET`).  
4. Attach the Windows ISO, install the OS, and complete setup.  

*Add screenshots of VirtualBox client settings and install.*

---

## 11. Join the Client to the Domain
1. Confirm the client received an IP in `172.16.0.x` from DHCP.  
2. Rename the computer to `CLIENT1` (optional).  
3. Join the domain (`mydomain.com`).  
4. Restart and sign in with a domain user account.  

*Add screenshots of domain join and sign-in.*

---

## 12. Verification Checklist
**Domain Controller**  
- INTERNAL adapter = `172.16.0.1`, DNS = `172.16.0.1`  
- NAT enabled on INTERNET NIC (RRAS running)  
- DHCP scope active (`172.16.0.100–200`) and authorized  
- AD DS and DNS healthy  
- **_USERS OU contains ~1,000 user accounts**  

**Client**  
- Receives DHCP address in `172.16.0.x`  
- Can ping DC and resolve domain name  
- Can log in with a domain account  

---

## 13. Snapshots & Reset Tips
- Take a snapshot after each milestone:  
  - Base OS installed  
  - DC promoted  
  - RRAS/DHCP configured  
  - **Before and after bulk user creation**  
  - Client joined  
- Roll back if networking or user creation fails.  

---

## 14. Notes & Safety
- This is a **lab only** — keep it isolated from your real/home network.  
- Use **test credentials** (never real data).  
- Don’t commit ISO or VDI files to GitHub.  

---

## Credits
- Inspired by **Josh Madakor’s YouTube video**:
- [How to Setup a Basic Home Lab Running Active Directory (Oracle VirtualBox) | Add Users w/PowerShell](https://youtu.be/MHsI8hJmggI?si=JndBJOHAcuPhE79c)
- Documented as part of my learning process as a graduate student in Cybersecurity & Information Assurance  
