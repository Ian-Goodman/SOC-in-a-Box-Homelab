# Part 1: Infrastructure Setup

## Overview
Part 1 establishes the foundational infrastructure for the SOC-in-the-Box Homelab by implementing proper network segmentation, switching behavior, and firewall placement.

This phase focuses on building a stable lab environment that mirrors real-world enterprise infrastructure, including clear WAN/LAN separation, predictable VLAN behavior, firewall routing boundaries, and virtualization configurations that persist across reboots.

Documenting the troubleshooting process was also an important component of this phase, as it highlights the practical challenges involved in building a reliable security lab.

---

## Objective
The objective of this phase is to establish the core network infrastructure required to support the SOC-in-the-Box environment.

This includes:

- Implementing VLAN-based segmentation across the router and managed switch
- Deploying OPNsense as the perimeter firewall
- Validating proper trunk vs access port behavior
- Ensuring correct DHCP boundaries across VLANs
- Confirming configuration persistence after reboots
- Preparing the environment for future segmentation and SIEM deployment

---

## Environment / Scope
The infrastructure deployed in this phase includes the following components:

**Router**
- Ubiquiti UniFi Cloud Gateway Ultra (UCG-Ultra)
- Responsible for upstream network routing and initial VLAN definitions

**Switch**
- Managed switch enforcing VLAN trunk and access port roles

**Firewall**
- OPNsense deployed as a virtual machine in VirtualBox on the main homelab workstation

**SOC Services Host**
- Proxmox server reserved for Wazuh and additional SOC services (to be implemented in Part 3)

---

## Tools Used
The following tools and platforms were used during this phase:

- **OPNsense** – open-source firewall and routing platform
- **VirtualBox** – hypervisor used to deploy the firewall VM
- **Ubiquiti UCG-Ultra** – network router providing VLAN definitions
- **Managed Layer 2 Switch** – VLAN trunk and access configuration
- **Proxmox VE** – virtualization platform reserved for SOC services
- **Linux networking utilities** – used for validation and testing

---

## Architecture / Design Notes
This phase establishes the base architecture that the remainder of the SOC-in-the-Box project will rely on.

Key design principles implemented:

- Dedicated **WAN ingress VLAN** for firewall connectivity
- **Internal lab network isolated behind OPNsense**
- Clear **VLAN trunking model between router and switch**
- **Access ports for endpoints**
- Firewall positioned between ingress network and internal lab network

The architecture ensures that:

- External ingress traffic enters through VLAN 10
- Internal lab systems reside behind the firewall on VLAN 20
- Future networks (Cyber Range, AD Lab, SOC Services) will be segmented and protected by firewall policies.

---

<details>
<summary><strong>Implementation Steps</strong></summary>

### 1. Define VLAN Networks
The following VLANs were created on the router and documented for reproducibility.

| VLAN | Name | Subnet |
|-----|------|-------|
| 1 | Home | 192.168.1.0/24 |
| 10 | Lab Ingress (Firewall WAN) | 192.168.10.0/24 |
| 20 | Internal Lab (Firewall LAN) | 192.168.20.0/24 |
| 30 | Cyber Range / Targets | 192.168.30.0/24 |
| 40 | Windows / AD Lab | 192.168.40.0/24 |
| 50 | Security / SOC Services | 192.168.50.0/24 |
| 99 | Proxmox Management | 192.168.99.0/24 |

---

### 2. Configure DHCP Boundaries

| VLAN | DHCP Source |
|-----|-------------|
| VLAN 1 | UCG-Ultra |
| VLAN 10 | UCG-Ultra |
| VLAN 20 | OPNsense |
| VLAN 30 | OPNsense |
| VLAN 40 | OPNsense |
| VLAN 50 | UCG-Ultra |
| VLAN 99 | UCG-Ultra |

---

### 3. Configure Managed Switch Ports

#### Router Uplink (Trunk Port)

Port: 1

| VLAN | Mode |
|-----|------|
| VLAN 1 | Untagged |
| VLAN 10 | Tagged |
| VLAN 20 | Tagged |
| VLAN 30 | Tagged |
| VLAN 40 | Tagged |
| VLAN 50 | Tagged |
| VLAN 99 | Tagged |

PVID: 1

---

#### Lab Ingress Port (Firewall WAN)

Port: 7

| VLAN | Mode |
|-----|------|
| VLAN 10 | Untagged |

PVID: 10

---

#### Management Network Port

Port: 4

| VLAN | Mode |
|-----|------|
| VLAN 99 | Untagged |

PVID: 99

---

### 4. Deploy OPNsense Firewall VM

| Setting | Value |
|-------|------|
| Disk Controller | SATA (AHCI) |
| Disk Size | 32 GB |
| Installation Method | Installer mode |

---

### 5. Configure OPNsense Network Interfaces

| Interface | Adapter | Network |
|---------|--------|--------|
| WAN | Bridged Adapter | Workstation second NIC (VLAN 10 access port) |
| LAN | Internal Network | `OPNsense_lan` |

---

### 6. Configure LAN Addressing

| Setting | Value |
|-------|------|
| LAN IP | 192.168.20.1 |
| Subnet | /24 |
| DHCP Enabled | Yes |
| DHCP Range | 192.168.20.100 – 192.168.20.200 |

</details>

---

<details>
<summary><strong>Validation / Testing</strong></summary>

### VLAN Validation
- Workstation second NIC receives an IP from VLAN 10
- Firewall WAN receives a **192.168.10.x** address
- Default gateway correctly resolves to **192.168.10.1**

### Firewall Validation
- LAN interface remains **192.168.20.1**
- DHCP on LAN distributes **192.168.20.x addresses**

### Persistence Validation
After rebooting the firewall VM:

- Interface assignments remain correct
- LAN configuration remains intact
- DHCP service remains operational

### Test VM Validation
A test VM connected to the internal network confirmed:

- DHCP assignment within the **192.168.20.x range**
- Successful ping to **192.168.20.1**
- Traffic routed through OPNsense as expected

</details>

---

<details>
<summary><strong>Screenshots / Evidence</strong></summary>

### Switch VLAN Configuration

![Switch VLAN configuration](assets/SwitchVLANsettings.png)

---

### VirtualBox Firewall Adapters

![VirtualBox Adapter 1](assets/vbox_opnsense_adapter1.png)

![VirtualBox Adapter 2](assets/vbox_opnsense_adapter2.png)

---

### VirtualBox Storage Configuration

![VirtualBox SATA configuration](assets/vbox_opnsense_storage_sata.png)

</details>

---

<details>
<summary><strong>Problems Encountered</strong></summary>

### VLAN DHCP Misbehavior

**Symptoms**
- Devices repeatedly received **192.168.1.x addresses**
- Firewall WAN incorrectly pulled management network addresses

**Root Cause**

The uplink port on the managed switch was incorrectly tagged, causing VLAN leakage.

---

### OPNsense Configuration Loss

**Symptoms**

- Settings disappeared after reboot
- Filesystem mounted read-only
- Disk usage showed 100%

**Root Cause**

The firewall VM was initially configured with an IDE controller and installed in Live Mode rather than Installer Mode.

</details>

---

<details>
<summary><strong>Fixes / Lessons Learned</strong></summary>

### Switch VLAN Fix

Corrected trunk port configuration:

- VLAN 1 set to **untagged**
- Lab VLANs set to **tagged**
- PVID set to **1**

Corrected access port:

- VLAN 10 set to **untagged**
- PVID set to **10**

---

### Firewall Persistence Fix

Resolved by:

- Rebuilding the VM using a **SATA controller**
- Installing OPNsense using the **installer account**
- Removing the installation ISO after installation
- Prioritizing hard disk boot order

</details>

---

## Skills Demonstrated

This phase demonstrates several practical infrastructure and security engineering skills:

- VLAN segmentation and switch configuration
- Firewall deployment and interface configuration
- Virtualization networking in VirtualBox
- DHCP boundary design
- Network troubleshooting and root cause analysis
- Infrastructure documentation and reproducibility

These skills are directly applicable to SOC engineering and network security roles.

---

## Next Phase

### Part 2: Network Segmentation & Firewall Policy

The next phase expands the lab by implementing segmentation between internal networks and applying firewall policy controls.

Planned objectives include:

- Activating VLANs 30, 40, and 50
- Implementing firewall rules for inter-VLAN communication
- Assigning VMs to appropriate networks
- Validating routing, DNS, and reachability per segment

Following this phase, **Part 3 will introduce SIEM deployment**, including Wazuh installation on the Proxmox server and validation of log ingestion from firewall and endpoints.
