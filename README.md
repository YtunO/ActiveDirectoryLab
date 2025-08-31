# Active Directory Home Lab (VirtualBox + Windows Server + Windows 10/11)

## Objective
This project was built to give me practical experience setting up Active Directory and Windows networking in a home lab. My goal was to understand how services like DNS, DHCP, and NAT work together, learn how to join a client to a domain, and get comfortable creating and managing users and groups in Active Directory. 


Project was inspired by **Josh Madakor’s YouTube tutorial**:  
📺 Watch the original video here: [YouTube Link](https://youtu.be/MHsI8hJmggI?si=JndBJOHAcuPhE79c)  


---

## Skills Learned
- Configuring **Active Directory Domain Services (AD DS)**
- Setting up **DNS, DHCP, and NAT routing** in Windows Server
- Automating **bulk user creation at scale** (1,000+ users with PowerShell ISE)
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

<img width="1169" height="688" alt="Topology" src="https://github.com/user-attachments/assets/ced4b73b-6f9e-48bf-bdff-cd6cdb4ce76f" />

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

---

## 4. Create the Domain Controller VM
1. Create a new VM for Windows Server in VirtualBox.  
2. Assign memory (4–6 GB), 2 CPUs, and a 40–60 GB disk.  
3. Add two network adapters:  
   - Adapter 1 → NAT  
   - Adapter 2 → Internal Network (name your internal network, e.g., `LABNET`).  
4. Attach the Windows Server ISO and complete the installation.  

<img width="1089" height="576" alt="1" src="https://github.com/user-attachments/assets/f5b68ce1-ec9b-4dca-a686-b6fd2e3c80ee" />
<img width="1080" height="567" alt="2" src="https://github.com/user-attachments/assets/a50d5c3d-3850-48e1-ab00-c3c064987117" />
<img width="1018" height="794" alt="3" src="https://github.com/user-attachments/assets/1db46506-b640-4d29-8aa1-36090e353d0c" />

---

## 5. Configure the DC: Name, NICs, Static IP
1. Rename the computer to `DC` and restart.  
2. Rename the NICs for clarity:  
   - NAT adapter → `INTERNET`  
   - Internal adapter → `INTERNAL`  
3. On the `INTERNAL` adapter, set:  
   - IP address: `172.16.0.1`  
   - Subnet mask: `255.255.255.0`  
   - DNS server: `172.0.0.1`

<img width="1024" height="768" alt="5" src="https://github.com/user-attachments/assets/074b23c3-0ff8-4be9-842a-3ba12bb85d1e" />
<img width="1024" height="768" alt="7" src="https://github.com/user-attachments/assets/e9561818-1331-427e-a8ec-d8a4586790e8" />
<img width="1024" height="768" alt="8" src="https://github.com/user-attachments/assets/a4a371c5-757d-43c5-abbf-7613ab70f0f4" />

---

## 6. Install AD DS and Create the Forest
1. In Server Manager, add the role: **Active Directory Domain Services**.  
2. Promote the server to a domain controller.  
3. Create a new forest (example: `mydomain.com`).  
4. Accept defaults and restart when prompted.  

<img width="1024" height="768" alt="13" src="https://github.com/user-attachments/assets/2f951d25-d69c-433a-810b-69e199eb453b" />
<img width="1024" height="768" alt="15" src="https://github.com/user-attachments/assets/14411952-b41b-4e5f-9796-842e13136dd2" />

---

## 7. Set Up NAT with Routing and Remote Access
1. In Server Manager, add the role: **Remote Access** (with Routing).  
2. Open the Routing and Remote Access console.  
3. Run the wizard → choose **NAT**.  
4. Select the `INTERNET` adapter as the public interface → start the service.  

<img width="1024" height="768" alt="25" src="https://github.com/user-attachments/assets/0ac89ef2-a82f-49fa-8d7b-65d0c374e906" />
<img width="1024" height="768" alt="29" src="https://github.com/user-attachments/assets/7621e15b-4671-4926-9e6b-efc6eadb2548" />
<img width="1024" height="768" alt="31" src="https://github.com/user-attachments/assets/8b1982b1-77e6-4af6-8688-dbed09e25d14" />
<img width="1024" height="768" alt="33" src="https://github.com/user-attachments/assets/0171e4f9-d5cc-4004-8686-0f119e371c02" />

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

<img width="1024" height="768" alt="37" src="https://github.com/user-attachments/assets/45e73b95-c0cc-40e1-8a20-8285fa4441c8" />
<img width="1024" height="768" alt="40" src="https://github.com/user-attachments/assets/e6bccb10-d9b0-4919-b9e6-d4d1b7405afb" />
<img width="1024" height="768" alt="43" src="https://github.com/user-attachments/assets/06094202-c591-4d42-aeb9-b1648af9e6ee" />

---

## 9. Bulk Add 1,000 Users with PowerShell ISE
**Goal:** Test scalability and automation by creating **1,000 domain users** in Active Directory.  

**Steps:**  
1. Prepare a text file with 1,000+ names (one per line, e.g., “First Last”).  
2. On the Domain Controller, open **PowerShell ISE as Administrator**.  
3. Load the bulk-user script and point it to:  
   - The target OU (e.g., `_USERS`)  
   - Your domain (e.g., `mydomain.com`)  
   - The names list file  
4. Run the script and watch as users are created in bulk.  
5. Verify in **Active Directory Users and Computers (ADUC):**  
   - Open the OU → confirm ~1,000+ users exist  

<img width="1024" height="768" alt="55" src="https://github.com/user-attachments/assets/0586d22a-1560-4882-b3a0-54e7b3193cc9" />
<img width="1024" height="768" alt="56" src="https://github.com/user-attachments/assets/4b274220-3ed3-47d9-83b7-5950f5a9a86d" />
<img width="1024" height="768" alt="57" src="https://github.com/user-attachments/assets/aeb4d51a-3aa0-44d6-bd79-949bba4efab8" />

---

## 10. Create the Client VM
1. Create a new VM for Windows 10/11 in VirtualBox.  
2. Assign memory (4 GB), 2 CPUs, and a 40 GB disk.  
3. Add one adapter: **Internal Network** (same as the DC’s `LABNET`).  
4. Attach the Windows ISO, install the OS, and complete setup.  

<img width="954" height="573" alt="60" src="https://github.com/user-attachments/assets/12be7d2e-9e7a-4af7-be85-e1d0ab43b878" />
<img width="1024" height="768" alt="61" src="https://github.com/user-attachments/assets/bd0663e6-2c08-4ee7-a481-063901a516e7" />


---

## 11. Join the Client to the Domain
1. Confirm the client received an IP in `172.16.0.x` from DHCP.  
2. Rename the computer to `CLIENT0` (optional).  
3. Join the domain (`mydomain.com`).  
4. Restart and sign in with a domain user account.  

<img width="1024" height="768" alt="64" src="https://github.com/user-attachments/assets/c22670d5-b265-491a-99fc-6fed4f143730" />
<img width="1024" height="768" alt="65" src="https://github.com/user-attachments/assets/aee117c6-b93b-4ffc-8a46-3648edee11b2" />
<img width="1024" height="768" alt="66" src="https://github.com/user-attachments/assets/8da9d741-6d33-4707-afe9-90d398d1a2a7" />
<img width="1822" height="842" alt="70" src="https://github.com/user-attachments/assets/b0eecab4-eff7-4220-b248-9b56a64af75c" />
<img width="1024" height="768" alt="69" src="https://github.com/user-attachments/assets/2f412d0c-9898-4eca-ac9e-a5f341ed5e03" />
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
  [How to Setup a Basic Home Lab Running Active Directory (Oracle VirtualBox) | Add Users w/PowerShell](https://youtu.be/MHsI8hJmggI?si=JndBJOHAcuPhE79c)
- Documented as part of my learning process as a graduate student in Cybersecurity & Information Assurance  
