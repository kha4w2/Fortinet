# FortiGate VM Lab – Basic Deployment and Internet Access Policy

<img width="1100" height="460" alt="topology_final" src="https://github.com/user-attachments/assets/c9071814-e5df-4e4d-ac44-44ee6d4d94d2" />


## Introduction
This lab documents the step‑by‑step process of deploying a FortiGate virtual appliance on VMware Workstation, configuring its network interfaces, activating the license, and enabling internet access for an internal Windows 10 client through a security policy. The hands‑on exercise demonstrates fundamental FortiGate administration, including CLI and GUI management, interface setup, and traffic inspection.

## Objective
- Deploy a FortiGate VM (v7.6.6) on VMware Workstation.
- Connect the VM to a WAN (bridged) and an isolated LAN segment.
- Activate a valid FortiCare license.
- Configure a Windows 10 client in the LAN segment.
- Create an outbound firewall policy to allow internet access.
- Verify connectivity and analyze traffic logs using FortiView.

## Tools Used
- **Hypervisor**: VMware Workstation (with OVF/OVA support)
- **FortiGate**: FortiGate‑VM for VMware ESXi, version 7.6.6
- **Client VM**: Windows 10
- **Management**: Web browser, FortiGate CLI
- **Account**: FortiCare account (for license registration)

## Steps

### Step 1: Lab Topology Overview
The lab topology is defined to establish the connectivity model. The FortiGate VM operates in NAT mode. Port1 connects to the WAN side using the 192.168.1.0/24 subnet and ultimately reaches the internet. Port2 connects to the internal LAN on the 10.0.0.0/24 subnet, where the Windows 10 client will reside. This logical overview guides all subsequent interface and policy configurations.

<p align="center">
  <img width="946" height="380" alt="topology (4)" src="https://github.com/user-attachments/assets/0215235c-faf2-43ac-b9cc-9b7620101715" />
</p>

**Figure 1:** High‑level topology of the lab environment.

---

### Step 2: Accessing the Fortinet Support Portal
To obtain the FortiGate VM image, the official Fortinet Support portal is accessed at `https://support.fortinet.com/support/#/dashboard`. From the main dashboard, the **Downloads** section is expanded and **VM Images** is selected. This is the central location for downloading virtual appliance packages.

<p align="center">
  <img width="975" height="400" alt="image" src="https://github.com/user-attachments/assets/c92842d3-f513-4da4-b98e-bc68ee04fd03" />
</p>

**Figure 2:** Navigating to the Download section of the Fortinet Support site.

---

### Step 3: Selecting Platform, Version, and Downloading the Image
The specific FortiGate VM image for VMware is located by selecting **FortiGate** as the product, **VMWare ESXi** as the platform, and version **7.6.6**. The file labeled **New deployment of FortiGate for VMware** (FGT_VM64-v7.6.6.M-build3652-FORTINET.out.ovf.zip) is downloaded. This package contains the OVF template required to deploy the virtual appliance.

<p align="center">
  <img width="975" height="454" alt="image" src="https://github.com/user-attachments/assets/cd08928a-eca4-4096-b400-69cd60fc243c" />
</p>

**Figure 3:** Choosing the correct platform, version, and download file.

---

### Step 4: Extracting the Archive and Opening the OVF File
The downloaded ZIP archive is extracted, revealing multiple files including several `.ovf` variants. The file `FortiGate-VM64.ovf` (or the appropriate hardware version) is opened directly in VMware Workstation. OVF (Open Virtualization Format) is a multi‑file format that separates configuration and disk files, making it flexible for inspection and customization, unlike a single OVA archive that bundles everything together.

<p align="center">
  <img width="975" height="442" alt="image" src="https://github.com/user-attachments/assets/34aab3d9-f576-45c0-839c-7417744efb78" />
</p>

**Figure 4:** Extracted files and available OVF templates.

---

### Step 5: Configuring the WAN Interface as Bridged
After naming the VM and choosing a storage path, the virtual machine settings are opened. The first network adapter (**Network Adapter 1**) is configured to use a **Bridged** connection. This adapter represents the **WAN** interface and will connect the FortiGate directly to the physical network, allowing it to obtain an IP address and reach the internet.

<p align="center">
  <img width="975" height="318" alt="image" src="https://github.com/user-attachments/assets/c9b255fc-1e02-40b3-89ba-4c9e3c4adcbf" />
</p>

**Figure 5:** Setting Network Adapter 1 to Bridged mode for the WAN connection.

---

### Step 6: Creating the LAN Segment for the Internal Network
The second network adapter (**Network Adapter 2**) is configured. Instead of a standard network type, **LAN Segment** is selected. A new LAN segment named **LAN‑1** is created. This isolated segment emulates the internal network; any virtual machine connected to it will have its traffic routed through the FortiGate.

<p align="center">
  <img width="975" height="504" alt="image" src="https://github.com/user-attachments/assets/769e7e04-b83e-4850-badc-a1cc7519be9b" />
</p>

**Figure 6:** Adding Network Adapter 2 and creating the LAN-1 segment.

---

### Step 7: Powering On and Initial Console Login
The virtual machine is powered on. After the system boots, the console login prompt appears. The default credentials are entered: username **admin** with an empty password (press Enter). The system then forces an immediate password change; this new password must be remembered as it is used for all future logins.

<p align="center">
  <img width="975" height="382" alt="image" src="https://github.com/user-attachments/assets/f8e34ad8-9b60-4930-9089-14da4c5b7ece" />
</p>

**Figure 7:** Console login prompt after the VM boots.

---

### Step 8: Checking System Status (CLI)
After logging in, the terminal may be cleared using `Ctrl+Shift+L`. The command `get system status` is executed to display vital system information, including the firmware version, build number, and license status. This step is essential to verify the initial state of the VM before any configuration.

<p align="center">
  <img width="721" height="550" alt="image" src="https://github.com/user-attachments/assets/0a489da9-fb0e-42af-87bc-65004a3a4b21" />
</p>

**Figure 8:** Output of `get system status` showing firmware details.

---

### Step 9: Identifying Invalid License Status
Scrolling further through the `get system status` output reveals the field **License Status: Invalid**. This indicates that the VM is currently unlicensed and will require activation via a FortiCare account to unlock full functionality.

<p align="center">
  <img width="819" height="554" alt="image" src="https://github.com/user-attachments/assets/2d2146af-859b-4227-bcab-38999cc080d1" />
</p>

**Figure 9:** Continuation of system status showing the license is invalid.

---

### Step 10: Assigning IP Addresses to Interfaces
The network interfaces are configured from the CLI. Port1 is set to obtain its IP address automatically via DHCP and is assigned the **WAN** role, with management access (Ping, HTTP, HTTPS, SSH, Telnet) enabled. Port2 is assigned a static IP of `10.0.0.1/24`, the **LAN** role, and a descriptive alias. These settings prepare the interfaces to match the planned topology.

<p align="center">
  <img width="975" height="315" alt="image" src="https://github.com/user-attachments/assets/e54a5ea0-ff39-4c25-81b9-58b42ac64708" />
</p>

**Figure 10:** CLI commands configuring Port1 (DHCP, WAN) and Port2 (static, LAN).

---

### Step 11: Verifying Interface Configuration
The command `get system interface physical` is used to confirm the assigned IP addresses and link status. The output shows Port1 has obtained `192.168.1.98/24` via DHCP and is up, while Port2 holds the static `10.0.0.1/24` and is also up. Other unused ports remain down.

<p align="center">
  <img width="975" height="483" alt="image" src="https://github.com/user-attachments/assets/3fee2505-956b-4b23-8922-7f6f60b99cf5" />
</p>

**Figure 11:** Physical interface status confirming correct IP assignments.

---

### Step 12: Testing Host‑to‑FortiGate Connectivity
A ping is initiated from the physical host (Windows workstation) to the FortiGate’s WAN IP address (`192.168.1.98`). Successful replies with a TTL of 255 confirm that the bridged network adapter is functioning correctly and the FortiGate is reachable on the physical network.

<p align="center">
  <img width="975" height="441" alt="image" src="https://github.com/user-attachments/assets/355618cf-fc6a-4546-8eca-ac7bddd87868" />
</p>

**Figure 12:** Successful ping from the host to the FortiGate WAN IP.

---

### Step 13: Accessing the FortiGate Web GUI
The FortiGate’s graphical management interface is accessed by opening a web browser and navigating to `https://192.168.1.98`. The admin credentials set during the initial console login are used to authenticate. The GUI provides a comprehensive platform for managing all aspects of the appliance.

<p align="center">
  <img width="975" height="425" alt="image" src="https://github.com/user-attachments/assets/62e915c7-11f2-4a03-9de3-8f3717dd08dd" />
</p>

**Figure 13:** Browser login page for the FortiGate GUI.

---

### Step 14: License Activation Prompt
Upon the first GUI login, a license warning is displayed: “VM is not licensed or license is invalid for current VM configuration.” To proceed, the option to activate a FortiCare evaluation license is chosen. This requires entering the credentials of a FortiCare account.

<p align="center">
  <img width="975" height="578" alt="image" src="https://github.com/user-attachments/assets/bf462d9f-175c-4d5f-8fce-3ca42c3b8e11" />
</p>

**Figure 14:** FortiGate VM License page prompting for activation.

---

### Step 15: Entering FortiCare Credentials and System Reboot
The email and password of a FortiCare account are entered and submitted. The FortiGate contacts Fortinet’s licensing servers, validates the account, and then initiates a system reboot to apply the new license.

<p align="center">
  <img width="975" height="473" alt="image" src="https://github.com/user-attachments/assets/ecbfafc1-c79e-4072-85c5-5dd807a6e0fd" />
</p>

**Figure 15:** System reboot screen after FortiCare credentials are accepted.

---

### Step 16: Completing the Post‑Reboot Setup Wizard
After the reboot, the FortiGate Setup wizard appears. The required steps are followed, which include confirming the configuration migration options, patch upgrade settings, dashboard preferences, and enforcing the password change if not already done.

<p align="center">
  <img width="975" height="478" alt="image" src="https://github.com/user-attachments/assets/fe3afe8c-4572-4888-bc00-da72091cb56b" />
</p>

**Figure 16:** FortiGate Setup wizard after the reboot.

---

### Step 17: Confirming Successful License Activation
The main dashboard is displayed, now showing a fully operational system. The System Information widget confirms the license is valid, the VM is allocated 1 CPU and 2 GiB of RAM, and the operation mode is NAT. The system is ready for further configuration.

<p align="center">
  <img width="975" height="433" alt="image" src="https://github.com/user-attachments/assets/b16a6360-b85f-43b1-827c-9b14c4444a2f" />
</p>

**Figure 17:** Dashboard confirming valid license and operational status.

---

### Step 18: Reviewing Configured Interfaces in the GUI
Navigating to **Network > Interfaces** displays all interfaces. The WAN interface (port1) shows its DHCP‑assigned IP `192.168.1.98`, and the LAN interface (port2) shows the static IP `10.0.0.1/24`. Both interfaces are up and correctly reflect the CLI configuration, validating that the network setup is applied properly.

<p align="center">
  <img width="975" height="482" alt="image" src="https://github.com/user-attachments/assets/0044a388-a378-4f85-bdb3-6ccb62a39cf3" />
</p>

**Figure 18:** Interfaces overview page in the GUI.

---

### Step 19: Enabling Administrative Access on the LAN Interface
To allow management and connectivity testing from the internal network, the LAN interface (port2) is edited. Under **Administrative Access**, the options for **PING**, **SSH**, and **TELNET** are enabled. This ensures that the Windows 10 client in the LAN segment can communicate with the FortiGate for troubleshooting.

<p align="center">
  <img width="975" height="491" alt="image" src="https://github.com/user-attachments/assets/daab607c-5bfa-4f0c-b61b-dc4de924c30e" />
</p>

**Figure 19:** Enabling PING, SSH, and TELNET on the LAN interface.

---

### Step 20: Creating the Windows 10 Client VM and Attaching it to the LAN Segment
A virtual machine running Windows 10 is created. In its network settings, the network adapter is configured to use the **LAN‑1** segment that was previously created. This places the Windows 10 client inside the isolated internal network, directly behind the FortiGate.

<p align="center">
  <img width="975" height="754" alt="image" src="https://github.com/user-attachments/assets/d2305984-922a-44e4-b385-ae94c937b2de" />
</p>

**Figure 20:** Windows 10 VM settings showing the network adapter connected to LAN-1.

---

### Step 21: Assigning a Static IP Address to the Windows 10 Client
Inside the Windows 10 VM, the Ethernet adapter’s IPv4 properties are configured manually. A static IP of `10.0.0.10` is assigned with a subnet mask of `255.0.0.0`. The default gateway is set to `10.0.0.1` (the FortiGate LAN interface), and DNS servers (e.g., 8.8.8.8) are specified. This ensures the client can resolve domain names and route traffic through the firewall.

<p align="center">
  <img width="975" height="470" alt="image" src="https://github.com/user-attachments/assets/9dde8bb5-6f10-4438-9c0d-2f9342b5d539" />
</p>

**Figure 21:** Static IP configuration on the Windows 10 Ethernet adapter.

---

### Step 22: Testing Internal Network Connectivity
From the Windows 10 command prompt, a ping is sent to the FortiGate’s LAN IP (`10.0.0.1`). All four replies are received with a latency of less than 1 ms, confirming that the internal network segment is functioning correctly and the FortiGate is reachable from the client.

<p align="center">
  <img width="975" height="504" alt="image" src="https://github.com/user-attachments/assets/829e514c-f351-4651-9e47-e7b1fd647ad2" />
</p>

**Figure 22:** Successful ping from Windows 10 to 10.0.0.1.

---

### Step 23: Identifying the Implicit Deny Policy
An attempt to ping the public IP `8.8.8.8` from the Windows 10 client results in timeouts. Inspection of the firewall policies on the FortiGate reveals only a single entry: the default **Implicit Deny** rule. This rule blocks all traffic that is not explicitly permitted, explaining the lack of internet access.

<p align="center">
  <img width="975" height="234" alt="image" src="https://github.com/user-attachments/assets/4e9aaee7-ac85-420d-8104-fdcf634ec150" />
</p>

**Figure 23:** Implicit deny policy blocking internet access, resulting in ping timeouts.

---

### Step 24: Creating an Outbound Internet Access Policy
To permit internet access, a new **IPv4 Policy** is created. The incoming interface is set to **LAN (port2)** and the outgoing interface to **WAN (port1)**. The source and destination are set to `all`, and the action is **ACCEPT**. Crucially, **NAT** is enabled to translate the internal client’s IP address to the WAN interface’s IP. No security profiles are applied, keeping the rule simple for basic connectivity testing.

<p align="center">
  <img width="975" height="673" alt="image" src="https://github.com/user-attachments/assets/2a81e4e7-759f-438d-9f88-b1ac92df65d0" />
</p>
<p align="center">
  <img width="975" height="40" alt="image" src="https://github.com/user-attachments/assets/9fa0495e-29f3-4c54-bdc7-0487cdd131d9" />
</p>

**Figure 24:** New IPv4 policy allowing LAN-to-WAN traffic with NAT enabled.

---

### Step 25: Verifying Internet Access from the Windows 10 Client
From the Windows 10 command prompt, pings are sent to `8.8.8.8` and `google.com`. Both targets reply successfully, with average round‑trip times of 45–47 ms. This proves that the security policy is active and NAT is translating internal addresses correctly, enabling full internet access.

<p align="center">
  <img width="975" height="683" alt="image" src="https://github.com/user-attachments/assets/5e6562e0-6657-4e3b-81fe-68ac7981a38c" />
</p>

**Figure 25:** Successful pings to 8.8.8.8 and google.com after policy creation.

---

### Step 26: Monitoring Policy Traffic and Active Sessions
Returning to the FortiGate’s policy list, the newly created rule now shows traffic statistics: **148.2 MB** of data transferred, **93 active sessions**, and a hit count visible in the graph. This real‑time view confirms that traffic from the Windows 10 client is being processed and forwarded by the policy.

<p align="center">
  <img width="975" height="358" alt="image" src="https://github.com/user-attachments/assets/06d69027-4d73-42a7-96a9-c25212fab3d1" />
</p>

**Figure 26:** Policy statistics showing bytes, packets, and active sessions.

---

### Step 27: Analyzing Traffic Details with FortiView
The **FortiView Sources** dashboard is opened to gain deeper visibility. It displays the source IP `10.0.0.10` (the Windows 10 client), its detected device name, bandwidth usage, and session count. This logging capability validates that all traffic is being inspected and recorded, completing the lab verification.

<p align="center">
  <img width="975" height="363" alt="image" src="https://github.com/user-attachments/assets/e53eb5b8-6b62-4e49-9ec5-0a40d1a7277b" />
</p>

**Figure 27:** FortiView dashboard showing traffic and device details for the Windows 10 client.

---

> **Lab Completed** – The FortiGate VM is fully deployed, licensed, and successfully routing internet traffic for an internal Windows client with full logging and visibility.
