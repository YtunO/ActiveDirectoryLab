# Active Directory Home Lab (VirtualBox + Windows Server + Windows 10/11)

I built this as a graduate student in a Master’s program in Cybersecurity & Information Assurance.  
This project was inspired by **Josh Madakor’s YouTube tutorial**:  
*“How to Setup a Basic Home Lab Running Active Directory (Oracle VirtualBox) | Add Users w/PowerShell”*  

📺 Watch the original video here: [YouTube Link](https://youtu.be/MHsI8hJmggI?si=JndBJOHAcuPhE79c)  

My goal was to challenge myself, grow my technical skills, and better understand how **Active Directory and Windows networking** work together in practice.

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
9. [Create the Client VM](#create-the-client-vm)  
10. [Join the Client to the Domain](#join-the-client-to-the-domain)  
11. [Verification Checklist](#verification-checklist)  
12. [Snapshots & Reset Tips](#snapshots--reset-tips)  
13. [Notes & Safety](#notes--safety)  
14. [Credits](#credits)

---

## What You’ll Build
- One Windows Server Domain Controller providing:
  - Active Directory Domain Services (AD DS)
  - DNS
  - DHCP
  - NAT routing (so the client can reach the internet through the DC)
- One Windows 10/11 Client joined to the domain
- An internal, isolated lab network

*Add your overview screenshot here.*

---

## Before You Start
- Oracle VirtualBox installed
- Windows Server 2019/2022 ISO
- Windows 10 or Windows 11 ISO
- At least 12 GB RAM available on your host (suggestion: DC 4–6 GB, Client 4 GB)
- Sufficient disk space (about 40–60 GB total)

---

## Network Plan
- Lab subnet: 172.16.0.0/24
- Domain Controller:
  - NIC 1: NAT (internet access for the DC)
  - NIC 2: Internal Network (static IP 172.16.0.1, DNS points to itself)
- Client:
  - NIC: Internal Network (gets IP from DHCP on the DC)

*Add a small topology image here.*

---

## Create the Domain Controller VM
1. Create a new VM for Windows Server in VirtualBox.
2. Assign memory (4–6 GB), 2 CPUs, and a 40–60 GB disk.
3. Add two network adapters:
   - Adapter 1: NAT
   - Adapter 2: Internal Network (name your internal network, for example “LABNET”).
4. Attach the Windows Server ISO and complete the installation.

*Add screenshots of VirtualBox settings and OS install.*

---

## Configure the DC: Name, NICs, Static IP
1. Rename the computer to DC and restart.
2. In Network Connections, rename the adapters for clarity:
   - NAT adapter → INTERNET
   - Internal adapter → INTERNAL
3. On the INTERNAL adapter, set:
   - IP address: 172.16.0.1
   - Subnet mask: 255.255.255.0
   - DNS server: 172.16.0.1

*Add screenshots of adapter names and IPv4 settings.*

---

## Install AD DS and Create the Forest
1. In Server Manager, add the role: Active Directory Domain Services.
2. Promote the server to a domain controller.
3. Create a new forest (example domain: mydomain.com).
4. Accept defaults and restart when prompted.

*Add screenshots of the AD DS wizard and domain setup.*

---

## Set Up NAT with Routing and Remote Access
1. In Server Manager, add the role: Remote Access, including Routing.
2. Open the Routing and Remote Access console.
3. Run the configuration wizard, choose NAT.
4. Select the INTERNET adapter as the public interface and start the service.

*Add screenshots of RRAS setup and status.*

---

## Install and Configure DHCP
1. In Server Manager, add the role: DHCP Server.
2. In the DHCP console, create a new IPv4 scope:
   - Start: 172.16.0.100
   - End: 172.16.0.200
   - Subnet mask: 255.255.255.0
   - Router (gateway): 172.16.0.1
   - DNS server: 172.16.0.1
3. Authorize and activate the DHCP server.

*Add screenshots of DHCP scope configuration.*

---

## Create the Client VM
1. Create a new VM for Windows 10/11 in VirtualBox.
2. Assign memory (4 GB), 2 CPUs, and a 40 GB disk.
3. Add one network adapter:
   - Adapter 1: Internal Network (use the same internal network name as the DC).
4. Attach the Windows ISO, install the OS, and complete setup.

*Add screenshots of VirtualBox client settings and install.*

---

## Join the Client to the Domain
1. Confirm the client received an address in the 172.16.0.x range from DHCP.
2. Rename the computer to CLIENT1 (optional).
3. Join the domain (example: mydomain.com).
4. Restart and sign in with a domain user account.

*Add screenshots of domain join and sign-in.*

---

## Verification Checklist
- **Domain Controller**
  - INTERNAL adapter shows 172.16.0.1 and DNS set to 172.16.0.1
  - Routing and Remote Access is running with NAT on the INTERNET adapter
  - DHCP scope 172.16.0.100–172.16.0.200 is active and authorized
  - AD DS and DNS show healthy status
- **Client**
  - Receives a 172.16.0.x address from DHCP
  - Can reach the DC and resolve the domain name
  - Successfully joined to the domain and signs in with a domain account

---

## Snapshots & Reset Tips
- Take a VirtualBox snapshot after each milestone:
  - Base OS installed
  - DC promoted
  - RRAS and DHCP configured
  - Client joined to the domain
- If networking breaks, roll back to the last good snapshot and repeat the last step carefully.

---

## Notes & Safety
- This is a lab environment; keep it isolated from your home or production network.
- Use test credentials and rotate them if you share screenshots publicly.
- Do not upload large or licensed media files (like ISOs or virtual disks) to GitHub.

---

## Credits
- Inspired by **Josh Madakor’s YouTube video**:  
  [How to Setup a Basic Home Lab Running Active Directory (Oracle VirtualBox) | Add Users w/PowerShell]([https://www.youtube.com/your-video-link](https://youtu.be/MHsI8hJmggI?si=JndBJOHAcuPhE79c))  
- Documented as part of my work as a **graduate student in a Master’s program in Cybersecurity & Information Assurance**
