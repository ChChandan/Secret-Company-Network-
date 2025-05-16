# **Secret Company Network – Network Design Documentation**  

## **Network Overview**  
This Packet Tracer simulation models a **small-to-medium-sized enterprise network** for a company with three primary premises:  
- **Main Office Building (MOB)** – The central hub hosting core departments.  
- **Sales Office Building (SOB)** – A remote branch handling sales operations.  
- **DMZ Zone** – A secured segment hosting public-facing services.  

The network integrates **advanced routing, switching, security, and management protocols** to ensure high availability, security, and scalability. 
- All the  devices enable password is **$ecretcomp@ny**.

---
![image](https://github.com/user-attachments/assets/bebfdf31-9c08-41df-9f2a-0e10a10f0315)


## **Network Devices**  
The topology consists of the following key devices:  
- **Routers:**  
  - **ISP Router (Cisco 4331)** – Simulates the internet service provider connection.  
  - **MOBR1 (Cisco 4331)** – Main Office Border Router handling inter-VLAN routing, VPN, and OSPF.  
  - **SOBR1 (Cisco 4331)** – Sales Office Border Router managing remote connectivity.  
- **Switches:**  
  - **MOB1 & MOB2 (Cisco 2960)** – Layer 2 switches for Main Office VLAN segmentation.  
  - **SOB1 (Cisco 2960)** – Layer 2 switch for Sales Office VLANs.  
  - **DMZ Switch (Cisco 2960)** – Isolates public-facing services.  
  - **MS1 (Cisco 3560-24PS Multilayer Switch)** – Provides high-performance inter-VLAN routing.  
- **Servers & Endpoints:**  
  - **Networking Server** – Centralized DHCP and NTP server.  
  - **User PCs** – Department-specific workstations across VLANs.  

---

## **Networking Technologies Implemented**  
The network employs the following key technologies:  
- **VLAN Segmentation** – Logical separation of departments for security and traffic management.  
- **OSPF (Open Shortest Path First)** – Dynamic routing for efficient path selection.  
- **SSH (Secure Shell)** – Encrypted remote device management.  
- **DHCP (Dynamic Host Configuration Protocol)** – Automated IP assignment with reserved ranges.  
- **NTP (Network Time Protocol)** – Synchronized timekeeping for logs and security.  
- **NAT (Network Address Translation)** – Enables private-to-public IP translation for internet access.  
- **Firewall & ACLs (Access Control Lists)** – Restricts unauthorized access between segments.  
- **Inter-VLAN Routing** – Facilitates communication between VLANs via Router-on-a-Stick and MLS.  
- **DMZ (Demilitarized Zone)** – Isolates public services (e.g., web servers) from internal networks.  
- **VPN (Virtual Private Network)** – Secure remote access for IT administration.  

---

## **Detailed Network Architecture**  

### **1. Main Office Building (MOB)**  
The MOB network is segmented into multiple **department-based VLANs**:  

| VLAN ID | Network Address      | Default Gateway   | VLAN Name    |  
|---------|----------------------|-------------------|--------------|  
| 10      | 192.168.10.0/24      | 192.168.10.1      | Management   |  
| 20      | 192.168.20.0/24      | 192.168.20.1      | Finance      |  
| 30      | 192.168.30.0/24      | 192.168.30.1      | HR           |  
| 40      | 192.168.40.0/24      | 192.168.40.1      | Marketing    |  

#### **Key Features:**  
- **DHCP Reservations:** The first five IPs (`.1`–`.5`) in each subnet are reserved for future network services.  
- **Inter-VLAN Routing:** Achieved via **Router-on-a-Stick (ROAS)** on **MOBR1** and **hardware-accelerated routing** on **MS1 (Multilayer Switch)**.  
- **OSPF Routing:** Ensures dynamic path selection between MOB, SOB, and DMZ networks.  

---

### **2. Sales Office Building (SOB)**  
The SOB network is simplified with a single **IT VLAN**:  

| VLAN ID | Network Address      | Default Gateway   | VLAN Name    |  
|---------|----------------------|-------------------|--------------|  
| 10      | 172.168.1.0/24       | 172.168.1.1       | IT           |  

#### **Key Features:**  
- **Remote SSH Access:** IT personnel can securely manage MOB devices using **SSHv2** for encrypted sessions.  
- **VPN Connectivity:**  
  - **ISAKMP Policy:** Uses **AES-256 encryption**, **Diffie-Hellman Group 5**, and a pre-shared key (`cisco12345`).  
  - **Crypto Map:** Tied to **ACL 101** with a **900-second security-association lifetime**.  

---

### **3. Multilayer Switch (MS1 – Cisco 3560-24PS)**  
The **Cisco 3560 Multilayer Switch** was selected over a traditional router for:  
- **Performance:** Hardware-based Layer 3 switching reduces latency.  
- **Cost Efficiency:** Combines switching and routing in a single device.  
- **Simplified Management:** Handles **VLANs, QoS, and ACLs** without additional hardware.  
- **Scalability:** Supports future growth with minimal reconfiguration.  

---

### **4. Networking Server (Centralized DHCP & NTP)**  
This server provides two critical functions:  

#### **DHCP Server**  
- Dynamically assigns IPs to all VLANs with **department-specific scopes**.  
- **Reserved IP Ranges:** Ensures critical services (e.g., printers, servers) avoid conflicts.  

#### **NTP Server**  
- Maintains **time synchronization** across all devices.  
- Critical for **log accuracy, security audits, and encrypted communications**.  

---
### **4.Cisco ASA Firewall Configuration Documentation**


This Cisco ASA 5500-series firewall (running ASA version 9.6(1)) serves as the primary security appliance for the organization's network infrastructure. The device is configured to enforce security policies between three key network segments while providing essential network services.

### Interface Configuration
The firewall interfaces are configured with a hierarchical security model:

| Interface | Nameif | Security Level | IP Address | Status | Purpose |
|-----------|--------|---------------|------------|--------|---------|
| Gig1/1 | outside | 0 | 209.165.225.1/30 | Active | Internet connection |
| Gig1/2 | DMZ | 50 | 172.168.10.1/24 | Active | DMZ services |
| Gig1/3 | inside | 100 | 20.1.1.1/30 | Active | Internal network |
| Gig1/4-1/8 | - | - | - | Disabled | Reserved for future use |

### Security Policies
### Zone-Based Security Model
The firewall implements a three-tiered security model:
- **Outside (0)**: Untrusted Internet zone
- **DMZ (50)**: Semi-trusted zone for public services
- **Inside (100)**: Trusted internal network zone

### Access Control Lists
- **traffic_out ACL**: Permits ICMP traffic from any source to any destination on the outside interface
- **ACL 101**: Explicitly blocks TCP traffic from DMZ (172.168.10.0/24) to internal network (10.1.1.0/24)

### Network Address Translation
- **DMZ Host**: 172.168.10.2 dynamically NATed to outside interface
- **Internal Network**: 10.1.1.0/24 dynamically NATed to outside interface

### Dynamic Routing
- OSPF Process 101 configured with internal network 20.1.1.0/30 in Area 0
- Logs adjacency changes for monitoring purposes

### Remote Access
- SSH access restricted to 172.168.1.0/24 on inside interface
- Local authentication enabled for console access
- Administrative credentials stored using encrypted password hashing

### Time Synchronization
- NTP authentication enabled
- Synchronizing with time server at 192.168.100.100

### Traffic Inspection
Default inspection policies enabled for:
- DNS (with maximum message length restriction)
- FTP
- ICMP
- TFTP

### Operational Parameters
- Telnet session timeout: 5 minutes
- SSH session timeout: 5 minutes

