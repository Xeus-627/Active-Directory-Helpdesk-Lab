**Role:** Lead Engineer
**Project Focus:** IT Helpdesk, Identity Management, Endpoint Configuration
**Status:** Completed

# 📖 Executive Summary

This project demonstrates the end-to-end deployment of a simulated enterprise network using VMware. Built from bare-metal virtual machines, this lab showcases practical skills in Windows Server administration, secure internal routing, Active Directory identity management, and automated endpoint provisioning using Group Policy Objects (GPO).

# 🛠️ Tech Stack & Environment

- **Hypervisor:** VMware Workstation
- **Server Setup:** Windows Server (DC-01) - Static IP: `192.168.10.10`
- **Client Setup:** Windows 11 Enterprise (IT-DESK-01) - Static IP: `192.168.10.20`
- **Core Technologies:** Active Directory (AD DS), Routing and Remote Access (RRAS), NAT, GPO, PowerShell.

# 🏗️ Network Architecture

The network uses a dual-NIC architecture on the server to act as a secure edge router, completely isolating the internal client network from the external internet.
<img width="1059" height="646" alt="2026-05-24 12_48_48-NVIDIA GeForce Overlay" src="https://github.com/user-attachments/assets/a1d59c91-dfc6-430e-a18a-3a111b8a8a54" />

# 🚀 Step-by-Step Implementation

## Phase 1: Virtual Machine Provisioning & OS Installation

Built the foundational infrastructure from scratch using VMware Workstation to simulate a physical hardware deployment.

- Provisioned two bare-metal virtual machines, allocating CPU, RAM, and storage resources.
- Mounted ISO files and completed clean installations of **Windows Server** and **Windows 11 Enterprise**.
- Created an isolated virtual local area network (LAN Segment) within the hypervisor to ensure lab traffic was completely segmented from the local home network.

## Phase 2: Dual-NIC Network Configuration & Edge Routing (NAT)

Engineered the server to act as a bridge between the isolated corporate network and the outside world.

- **Dual-NIC Setup:** Configured the Windows Server with two physical Network Interface Cards (vNICs):
    - *External Adapter:* Connected to VMware NAT to receive an external DHCP address for internet access.
    - *Internal Adapter:* Connected to the isolated LAN segment and assigned a static IP (`192.168.10.10`).
- **Why RRAS was required:** By default, Windows Server does not route traffic between two different network adapters. To solve this, the **Routing and Remote Access (RRAS)** role was installed to turn the server into a software-defined router.
- **NAT Configuration:** Configured IPv4 Network Address Translation (NAT) within RRAS so the internal Windows 11 client could securely access the internet by masking its traffic behind the server's external IP.

<img width="625" height="441" alt="Routing" src="https://github.com/user-attachments/assets/babed29d-0d2d-4051-9db8-8e910eb79572" />

## Phase 3: Directory Services & Identity Management

Provisioned the central management server to handle domain authentication and organizational structure.

- Installed **Active Directory Domain Services (AD DS)** and promoted the server to a Domain Controller for the `lab.local` forest.
- Built a foundational Organizational Unit (OU) structure to logically separate users, computers, and administrative accounts.
- Provisioned standard enterprise user credentials (e.g., the `hadmin` account) to simulate employee onboarding.

<img width="1553" height="846" alt="Hadmin" src="https://github.com/user-attachments/assets/bdcedeba-4c43-4428-90e3-9ba52585c22b" />

## Phase 4: Client Domain Integration

Configured the standalone Windows 11 endpoint to communicate exclusively with the Domain Controller and joined it to the corporate network.

- Assigned a static IP (`192.168.10.20`) and set the Default Gateway and DNS strictly to `DC-01` (`192.168.10.10`).
- Verified internet routing and DNS resolution via Command Prompt ping tests.
- Executed the domain join and authenticated remotely using the provisioned `hadmin` credentials.

<img width="1248" height="745" alt="Cmd ong" src="https://github.com/user-attachments/assets/90ac65d6-da34-4c86-8a50-633eb3008440" />

## Phase 5: Group Policy Automation (GPO)

Enforced corporate security baselines and automated resource mapping to ensure standard configuration for the endpoint.

- **Security Control:** Created the `SEC-Block-Control-Panel` policy to restrict standard users from modifying PC settings.
- **Resource Mapping:** Created the `MAP-Enterprise-Drive` policy to automatically mount a shared server directory (`\\DC-01\Enterprise-Share`) to the `S:` drive upon login.
- **Execution:** Utilized PowerShell (`Invoke-GPUpdate`) to remotely force the client to pull down the new rules.

<img width="1349" height="719" alt="GPO" src="https://github.com/user-attachments/assets/2c135e1c-f57b-4512-b8ad-ca611efacf0c" />

## 🧠 Troubleshooting & Problem Solving

- **ICMP Ping Failures:** Diagnosed an issue where the Server could not ping the Client. Identified that the root cause was the default Windows 11 Defender Firewall dropping ICMP Echo Requests, successfully validating that the virtual network cables were configured correctly.
- **WMI/RPC Remote Policy Blocks:** Encountered an RPC server error when attempting to push a remote `gpupdate` from the server. Identified that remote Windows Management Instrumentation (WMI) requests were blocked by the endpoint firewall. Temporarily bypassed the firewall for lab efficiency, noting that a production environment requires specific port allowances (TCP 135) to allow domain controllers to manage client endpoints.
