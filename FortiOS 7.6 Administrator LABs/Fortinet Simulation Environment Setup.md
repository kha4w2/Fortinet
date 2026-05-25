# Fortinet Simulation Environment Setup
## FortiOS 7.6 — VMware Workstation Lab
 
 
<img width="1244" height="864" alt="Gemini_Generated_Image_k54sf9k54sf9k54s" src="https://github.com/user-attachments/assets/05d25501-c8f3-4340-a6f3-9d37ec2bcd25" />

 
## Table of Contents
1. [Introduction](#introduction)
2. [IP Addressing Table](#ip-addressing-table)
3. [Tools & Requirements](#tools--requirements)
4. [Implementation](#implementation)
---
 
## Introduction
 
This repository documents the setup of a Fortinet simulation environment built on VMware Workstation. The lab replicates a real-world enterprise network topology consisting of a Headquarters (HQ) site and a Branch (BR1) site, connected through a simulated WAN link.
 
The environment includes two FortiGate firewalls, two Windows 10 admin PCs, and is designed to support hands-on practice with FortiOS 7.6 features including administrator account management, interface configuration, and network segmentation.
 
All devices are virtualized using VMware Workstation LAN Segments to create isolated, conflict-free network segments that accurately simulate a production Fortinet deployment.
 
---
 
## IP Addressing Table
 
| Device | Interface / Port | LAN Segment | IP Address |
|---|---|---|---|
| HQ-NGFW-1 | port1 (WAN) | Bridged (Internet) | 192.168.1.98/24 (DHCP) |
| HQ-NGFW-1 | port2 (LAN) | LAN-1 | 10.0.11.254/24 |
| HQ-NGFW-1 | port3 (BR1-Link) | LAN-2 | 100.65.0.101/24 |
| HQ-PC-1 | NIC | LAN-1 | 10.0.11.1/24 |
| BR1-FGT | port1 (WAN) | Bridged (Internet) | 192.168.1.8/24 (DHCP) |
| BR1-FGT | port2 (LAN) | LAN-3 | 100.65.1.254/24 |
| BR1-FGT | port3 (HQ-Link) | LAN-2 | 100.65.0.102/24 |
| BR1-PC-1 | NIC | LAN-3 | 100.65.1.1/24 |
 
> **Default Gateway:** Each PC uses its local FortiGate port2 as the default gateway.  
> **DNS:** 8.8.8.8 on all endpoints.  
> **LAN-2** is the shared inter-site segment connecting HQ-NGFW-1 port3 ↔ BR1-FGT port3.
 
---
 
## Tools & Requirements
 
| Component | Details |
|---|---|
| Hypervisor | VMware Workstation Pro |
| FortiOS Image | FortiGate-VM64-v7.6.6 (OVF format) |
| Endpoint OS | Windows 10 (x2) |
| FortiCare Account | Required for VM eval license activation |
| Networking | VMware LAN Segments (LAN-1, LAN-2, LAN-3) |
| Management Access | PuTTY (SSH), Web Browser (HTTPS GUI) |
 
---
 
## Implementation
 
### Device 1 — HQ-NGFW-1 (Headquarters FortiGate)
 
#### 1.1 Network Adapter Assignment
 
Before booting the VM, configure the network adapters in VMware Workstation:
 
| VMware Adapter | LAN Segment | FortiGate Port | Purpose |
|---|---|---|---|
| Network Adapter 1 | Bridged (Automatic) | port1 | WAN / Internet |
| Network Adapter 2 | LAN-1 | port2 | HQ LAN |
| Network Adapter 3 | LAN-2 | port3 | Link to BR1-FGT |
 
<img width="975" height="997" alt="image" src="https://github.com/user-attachments/assets/815eeed0-32ed-4a1e-96e1-310d65489c16" />
*Figure 1: HQ-NGFW-1 VM Settings — Network Adapter 3 assigned to LAN-2 for the HQ-to-Branch inter-site link.*
 
---
 
#### 1.2 Interface Configuration (CLI)
 
```bash
# Configure port2 — HQ LAN
config system interface
    edit port2
        set mode static
        set ip 10.0.11.254 255.255.255.0
        set allowaccess ping https ssh
        set role lan
        set alias LAN
    next
end
 
# Configure port3 — Branch Link
config system interface
    edit port3
        set mode static
        set ip 100.65.0.101 255.255.255.0
        set allowaccess ping http https ssh telnet
        set role lan
        set alias Link-to-BR1
    next
end
```
 
<img width="909" height="615" alt="image" src="https://github.com/user-attachments/assets/268e6f05-42f0-4809-9519-16a65c800541" />
*Figure 2: HQ-NGFW-1 CLI — Configuring port2 (10.0.11.254/24) and port3 (100.65.0.101/24) with static IPs.*
 
---
 
#### 1.3 Verification
 
```bash
show system interface port2
show system interface port3
```
 
<img width="975" height="247" alt="image" src="https://github.com/user-attachments/assets/3f5b1200-1ec5-4cdd-a75a-b53762f4a5a2" />
*Figure 3: HQ-NGFW-1 CLI — Output of `show system interface` confirming correct IP, alias, role, and VDOM assignment for port2 and port3.*
 
<img width="975" height="217" alt="image" src="https://github.com/user-attachments/assets/0f8f0c2e-bd22-424f-b12e-703825506ef2" />
*Figure 4: HQ-NGFW-1 GUI — Network Interfaces page showing all three physical interfaces with correct IPs and administrative access settings.*
 
---
 
### Device 2 — HQ-PC-1 (Headquarters Admin PC)
 
#### 2.1 Network Adapter Assignment
 
| VMware Adapter | LAN Segment | Purpose |
|---|---|---|
| Network Adapter | LAN-1 | HQ LAN — connects to HQ-NGFW-1 port2 |
 
#### 2.2 Static IP Configuration (Windows)
 
| Field | Value |
|---|---|
| IP Address | 10.0.11.1 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 10.0.11.254 |
| DNS | 8.8.8.8 |
 
<img width="975" height="308" alt="image" src="https://github.com/user-attachments/assets/543f486d-060e-4156-9104-7951a67f5436" />
*Figure 5: HQ-PC-1 VM Settings — Single network adapter assigned to LAN-1, connecting the admin PC to the HQ FortiGate LAN.*
 
<img width="975" height="857" alt="image" src="https://github.com/user-attachments/assets/3781ac09-1f06-4e24-9739-82d3f004397c" />
*Figure 6: HQ-PC-1 TCP/IPv4 Properties — Static IP 10.0.11.1/24 with gateway 10.0.11.254 pointing to HQ-NGFW-1 port2.*
 
---
 
### Device 3 — BR1-FGT (Branch FortiGate)
 
#### 3.1 Network Adapter Assignment
 
| VMware Adapter | LAN Segment | FortiGate Port | Purpose |
|---|---|---|---|
| Network Adapter 1 | Bridged (Automatic) | port1 | WAN / Internet |
| Network Adapter 2 | LAN-3 | port2 | Branch LAN |
| Network Adapter 3 | LAN-2 | port3 | Link to HQ-NGFW-1 |
 
> ⚠️ **Critical:** LAN-2 must be assigned to port3 on **both** HQ-NGFW-1 and BR1-FGT. This is the shared segment that connects the two firewalls.
 
<img width="975" height="257" alt="image" src="https://github.com/user-attachments/assets/f215ff3d-fb71-4658-af4f-834b6a9101f1" />
*Figure 7: BR1-FGT VM Settings — Network Adapter 2 on LAN-3 (Branch LAN) and Network Adapter 3 on LAN-2 (inter-site link to HQ).*
 
---
 
#### 3.2 Interface Configuration (CLI)
 
```bash
# Configure port2 — Branch LAN
config system interface
    edit port2
        set mode static
        set ip 100.65.1.254 255.255.255.0
        set allowaccess http https ping ssh telnet
        set role lan
        set alias BR1-LAN
    next
end
 
# Configure port3 — HQ Link
config system interface
    edit port3
        set mode static
        set ip 100.65.0.102 255.255.255.0
        set allowaccess http https ssh ping
        set role wan
        set alias BR1-HQ-LINK
    next
end
```
 
<img width="705" height="256" alt="image" src="https://github.com/user-attachments/assets/587fcab7-2277-429e-913a-1b43a6a391bf" />
*Figure 8: BR1-FGT CLI — Configuring port2 as BR1-LAN (100.65.1.254/24) with LAN role for the branch local network.*
 
<img width="709" height="315" alt="image" src="https://github.com/user-attachments/assets/97026b1c-590f-4e34-97c3-8a8e289de55f" />
*Figure 9: BR1-FGT CLI — Configuring port3 as BR1-HQ-LINK (100.65.0.102/24) connecting to HQ-NGFW-1 port3 over LAN-2.*
 
---
 
#### 3.3 Verification
 
```bash
get system interface physical
```
 
<img width="778" height="578" alt="image" src="https://github.com/user-attachments/assets/46c83873-03f2-4023-bf6c-4e0397e01251" />
*Figure 10: BR1-FGT CLI — Output of `get system interface physical` confirming all three ports are up with correct static IPs.*
 
---
 
#### 3.4 Admin Password & Hostname
 
```bash
# Set admin password
config system admin
    edit admin
        set password Fortinet1!
    next
end
 
# Set hostname
config system global
    set hostname BR1-FGT
end
```
 
<img width="713" height="456" alt="image" src="https://github.com/user-attachments/assets/12b08215-98e2-4dba-bfa9-ad095eb55d2b" />
*Figure 11: BR1-FGT CLI — Setting the admin password and hostname, confirming successful login with new credentials.*
 
---
 
#### 3.5 Eval License Activation
 
```bash
# Set FortiCare account credentials
execute vm-license-options account-id <your-forticare-email>
execute vm-license-options account-password <your-password>
 
# Activate eval license
execute vm-license
```
 
> The VM will reboot automatically after license activation. This is expected behavior.
 
<img width="975" height="225" alt="image" src="https://github.com/user-attachments/assets/5cf71f6b-9449-4041-8a8b-5a274a57cce8" />
*Figure 12: BR1-FGT PuTTY Session — Activating the FortiGate VM evaluation license via CLI using FortiCare account credentials.*
 
---
 
#### 3.6 GUI Verification
 
After reboot, access the BR1-FGT GUI:
 
```
https://192.168.1.8
```
 
<img width="975" height="373" alt="image" src="https://github.com/user-attachments/assets/9dea4611-8fae-4d32-a61f-878620c2a00b" />
*Figure 13: BR1-FGT GUI — Network Interfaces page confirming BR1-HQ-LINK (port3) and BR1-LAN (port2) are correctly configured. Evaluation VM license banner visible in the top-right corner.*
 
---
 
### Device 4 — BR1-PC-1 (Branch Admin PC)
 
#### 4.1 Network Adapter Assignment
 
| VMware Adapter | LAN Segment | Purpose |
|---|---|---|
| Network Adapter | LAN-3 | Branch LAN — connects to BR1-FGT port2 |
 
<img width="975" height="618" alt="image" src="https://github.com/user-attachments/assets/1b2a2919-4d63-42fc-886f-cf7d7c9caab5" />
*Figure 14: BR1-PC-1 VM Settings — Single network adapter assigned to LAN-3, placing the branch PC on the same segment as BR1-FGT port2.*
 
---
 
#### 4.2 Static IP Configuration (Windows)
 
| Field | Value |
|---|---|
| IP Address | 100.65.1.1 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | 100.65.1.254 |
| DNS | 8.8.8.8 |
 
<img width="503" height="576" alt="image" src="https://github.com/user-attachments/assets/1c7031e9-9065-416e-b750-a8dd997a9dba" />
*Figure 15: BR1-PC-1 TCP/IPv4 Properties — Static IP 100.65.1.1/24 with gateway 100.65.1.254 pointing to BR1-FGT port2.*
 
---
 
## Notes
 
- The eval license on both FortiGate VMs limits each device to a maximum of **3 interfaces, 3 firewall policies, and 3 static routes**. This is sufficient for the labs in this repository.
- **LAN Segments** in VMware Workstation operate as isolated Layer 2 switches with no DHCP — all IPs are statically assigned.
- All passwords in this lab follow the Fortinet training standard: `Fortinet1!`
- Port1 on both FortiGates uses **Bridged (Automatic)** to share the host machine's physical NIC for internet/WAN access and eval license activation.
 
