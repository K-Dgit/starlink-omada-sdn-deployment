# Starlink + TP-Link Omada SDN: Residential Network Deployment

**Client:** 7-unit apartment complex, Lagos, Nigeria
**Role:** Subject Matter Expert and sole network engineer
**Scope:** Full end-to-end network design, deployment, and ongoing remote management
**Author:** KingDavid Oyedemi

---

## Overview

Complete network infrastructure deployment for a newly built 7-unit residential apartment complex. The project integrated Starlink satellite internet with a TP-Link Omada Software-Defined Networking (SDN) environment, using a Windows PC running the Omada Software Controller as the network management server.

The controller was configured to auto-start as a persistent Windows service using NSSM (Non-Sucking Service Manager), ensuring it behaves like a hardware controller even after reboots. Remote access to both the controller and the server PC were secured with multi-factor authentication and strong password policies. Starlink RBAC was used to provision Technician-level access for ongoing dish and service management without sharing admin credentials.

---

## Network Topology

```
Starlink Dish
      |
Starlink Router (WAN gateway)
      |
Network Switch
      |
      |-- Windows PC (Omada Software Controller + NSSM service)
      |
      |-- TP-Link EAP Access Points x7
            |
            |-- Resident SSID (private, WPA2/WPA3)
            |-- Guest SSID (isolated VLAN, captive portal)
```

---

## Components Used

| Component | Role |
|---|---|
| Starlink Dish + Router | WAN internet source |
| Network Switch | Core LAN switching, connects all devices |
| Windows PC | Hosts the Omada Software Controller |
| TP-Link Omada Software Controller | SDN management of all APs |
| NSSM (Non-Sucking Service Manager) | Persists controller as a Windows service |
| TP-Link EAP Access Points x7 | Wi-Fi coverage per unit and common areas |
| AnyDesk | Secure remote access to controller PC |
| Starlink App (RBAC) | Technician-level dish and service management |
| WiFiman | Wi-Fi coverage mapping and AP placement planning |

---

## Key Skills Demonstrated

- End-to-end residential network design and deployment
- TP-Link Omada Software Controller installation and SDN configuration
- Windows service persistence using NSSM for always-on controller operation
- VLAN design and guest network isolation
- Captive portal configuration
- Secure remote access setup with MFA and strong credential policy
- Starlink RBAC configuration for delegated technician access
- Wi-Fi site survey and AP placement using WiFiman
- Ongoing remote network administration

---

## Deployment Steps

### 1. Site Survey with WiFiman

Before any equipment was installed, WiFiman was used to:

- Map the physical layout of all 7 units and common areas
- Survey existing signal environment to identify interference sources
- Plan optimal AP placement for full coverage without excessive overlap
- Determine cable routing paths for PoE runs from switch to each AP location

### 2. Starlink Integration

Starlink was connected to the network switch via the Starlink ethernet adapter, with the Starlink Router acting as the WAN gateway providing DHCP to the LAN.

### 3. Omada Software Controller Installation

The Omada Software Controller was installed on the Windows PC connected to the switch. This PC acts as the SDN controller, managing all EAPs centrally.

The controller is accessible at:
```
http://[controller-ip]:8088
https://[controller-ip]:8043
```

### 4. NSSM Service Persistence

By default, the Omada Software Controller only runs while a user is logged in. To make it behave like a hardware controller, always running in the background and surviving reboots, NSSM was used to register it as a Windows service.

This is the officially documented TP-Link method for Omada Software Controller persistence on Windows.

```cmd
# Run Command Prompt as Administrator
# Navigate to NSSM location
cd C:\NSSM

# Install Omada Controller as a Windows service
nssm install "Omada Controller"
```

In the NSSM GUI that opens:
- **Application Path:** path to the Omada Controller executable (e.g. `C:\Users\[user]\Omada Controller\bin\controller.bat`)
- **Startup Directory:** Omada Controller installation directory
- **Service name:** Omada Controller

After installation:
```cmd
# Start the service
nssm start "Omada Controller"

# Verify it is running
nssm status "Omada Controller"
```

After a reboot, verify via Windows Task Manager that `java.exe` and `mongod.exe` are both running. These are the Omada Controller processes. The controller web interface should be accessible without anyone logging into the PC.

Key NSSM behaviours that make this reliable:
- Monitors the running process and restarts it automatically if it crashes
- Starts before any user logs in, on Windows boot
- Logs activity to the Windows Event Log for auditing

### 5. AP Adoption and SSID Configuration

All 7 EAPs were connected to the switch via PoE and adopted into the Omada Software Controller. From the controller, the following were configured site-wide:

**Resident Network**
- Private SSID with WPA3 security
- Dedicated VLAN, isolated from guest traffic
- Full LAN access within the resident network

**Guest Network**
- Separate SSID with captive portal on first connection
- Isolated VLAN, no access to resident devices or controller PC
- Per-client bandwidth limits for fair usage
- Session timeout for security

### 6. Secure Remote Access

Two layers of remote access were configured:

**Omada Controller (TP-Link Cloud)**
- Controller linked to TP-Link Cloud account
- 2FA enabled on the TP-Link Cloud account
- Extended password complexity enforced
- Allows managing all network settings, APs, SSIDs, and client connections from any browser worldwide

**Controller PC (AnyDesk)**
- AnyDesk installed on the Windows PC
- Extended password complexity set on the AnyDesk access credential
- Allows full remote desktop access to the PC for deeper troubleshooting, NSSM service management, and controller administration when cloud access is insufficient

### 7. Starlink RBAC: Technician Role

Rather than sharing the Starlink admin credentials for ongoing support, Starlink Role-Based Access Control (RBAC) was used to assign Technician-level access separately.

The Technician role in Starlink provides:
- Ability to reboot, stow, and configure the Starlink terminal and router remotely
- Manage service lines and update IP policy
- View device telemetry and account information
- Add or remove Starlink devices

This keeps the admin account secure while giving the support engineer the access needed to manage the dish and router remotely through the Starlink app without billing or account-level permissions.

---

## Challenges and How I Resolved Them

**Controller stopping on PC logout**
The Omada Software Controller is a Java application that closes when the Windows session ends. Without persistence, the controller going down meant all AP management was lost. Resolved by registering it as a Windows service using NSSM, the method officially documented by TP-Link. The controller now starts at boot and runs continuously regardless of user login state.

**AP coverage overlap in adjacent units**
With 7 APs in close proximity, co-channel interference was a risk. Resolved using WiFiman for pre-deployment site survey to plan AP placement, and the Omada controller's automatic channel selection post-deployment to minimize interference.

**Remote dish access without sharing admin credentials**
The client needed to grant ongoing support access to the Starlink dish without exposing the full Starlink admin account. Resolved using Starlink RBAC to assign Technician role access with appropriate permissions scoped to hardware management only.

---

## Results

- Full Wi-Fi coverage across all 7 units and common areas
- Resident and guest networks fully isolated from each other
- Omada Software Controller running persistently as a Windows service via NSSM, surviving reboots without intervention
- Full remote management of the network via TP-Link Cloud with 2FA
- Full remote access to controller PC via AnyDesk
- Starlink dish and router remotely manageable via Technician RBAC role
- No on-site visits required for routine network administration

---

KingDavid Oyedemi | [linkedin.com/in/king-david-tech](https://linkedin.com/in/king-david-tech) | kdoyedemi@gmail.com
