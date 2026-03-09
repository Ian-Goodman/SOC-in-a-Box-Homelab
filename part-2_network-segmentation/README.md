# Part 2: Network Segmentation & Firewall Policy

## Overview

Part 2 focuses on implementing true zone-based segmentation inside the SOC-in-the-Box Homelab using OPNsense as a multi-interface firewall.

This phase establishes deterministic routing, stable DHCP per segment, enforced inter-zone access control, and controlled egress filtering. It validates that lateral movement paths can be blocked and logged — preparing the environment for SIEM ingestion in Part 3.

This is where the homelab transitions from a flat test network into an enterprise-style zone-based architecture.

---

## Objective

The objective of this phase is to implement enterprise-style network segmentation across the lab environment using a zone-based firewall model.

This includes:

- Implementing zone-based segmentation across lab networks
- Assigning persistent firewall interfaces for each network segment
- Enabling DHCP services for each security zone
- Connecting VMs to appropriate segmented internal networks
- Enforcing inter-zone firewall policies
- Implementing controlled outbound filtering
- Generating deny logs for SIEM telemetry
- Validating configuration persistence across system reboots

---

## Environment / Scope

The segmentation architecture deployed in this phase includes the following components.

**Firewall**

- OPNsense deployed in VirtualBox
- Responsible for routing and policy enforcement between security zones

**Virtualization Platform**

- VirtualBox used to host firewall and lab VMs

**Network Segments**

- LAN (attacker/operator network)
- CYBER_RANGE (vulnerable systems)
- AD_LAB (Windows domain environment)

**SOC Services Host**

- Proxmox server reserved for Wazuh and SOC services (implemented in Part 3)

---

## Tools Used

The following tools were used during this phase:

- **OPNsense** – multi-interface firewall platform
- **VirtualBox** – hypervisor used to simulate segmented networks
- **Kali Linux** – attacker workstation
- **Chronos** – vulnerable Linux host
- **Metasploitable** – intentionally vulnerable Linux host
- **Linux networking utilities** – validation and troubleshooting
- **OPNsense firewall logging** – deny event telemetry generation

---

## Architecture / Design Notes

Instead of relying on VLAN tagging inside the hypervisor, segmentation is implemented using one dedicated firewall interface per zone.

This approach mirrors real enterprise firewall architecture.

### Security Zones

| Zone | Purpose |
|-----|--------|
| WAN | Upstream internet connectivity |
| LAN | Attacker / operator network |
| CYBER_RANGE | Vulnerable targets |
| AD_LAB | Windows domain environment |

### Network Addressing

| Segment | Purpose | Subnet | Gateway |
|--------|--------|-------|-------|
| WAN | Upstream network | 192.168.10.0/24 | DHCP |
| LAN | Attacker network | 192.168.20.0/24 | 192.168.20.1 |
| CYBER_RANGE | Vulnerable targets | 192.168.30.0/24 | 192.168.30.1 |
| AD_LAB | Windows domain | 192.168.40.0/24 | 192.168.40.1 |

### VirtualBox Network Mapping

Each firewall interface connects to a dedicated VirtualBox Internal Network.

| VirtualBox Network | Firewall Zone |
|-------------------|--------------|
| LAB_LAN | LAN |
| CYBER_RANGE | CYBER_RANGE |
| AD_LAB | AD_LAB |

This model provides deterministic Layer 3 boundaries and simplifies troubleshooting compared to hypervisor VLAN tagging.

---

<details>
<summary><strong>Implementation Steps</strong></summary>

### 1. Interface Assignment & Persistence Fix

Initial segmentation attempts caused DHCP failure on Kali (169.254.x.x address).

Root cause analysis determined that VirtualBox adapter order had drifted, causing interface mappings inside OPNsense to change.

Resolution steps:

- Powered off the OPNsense VM
- Reordered VirtualBox adapters to enforce stable mapping
- Reassigned firewall interfaces in the OPNsense console
- Rebooted firewall and validated configuration persistence

Final adapter layout:

| Adapter | Interface | Function |
|-------|---------|---------|
| Adapter 1 | WAN | Bridged upstream |
| Adapter 2 | LAN | LAB_LAN network |
| Adapter 3 | CYBER_RANGE | CYBER_RANGE network |
| Adapter 4 | AD_LAB | AD_LAB network |

This configuration ensures interface stability across reboots.

### 2. DHCP Per Zone

DHCP services were enabled per network segment to ensure deterministic addressing within each security zone.

| Zone | DHCP Range | Purpose |
|-----|-----------|--------|
| LAN | 192.168.20.x | Attacker / operator systems |
| CYBER_RANGE | 192.168.30.x | Vulnerable targets |
| AD_LAB | 192.168.40.x | Windows domain systems |

Validated DHCP leases:

- Kali → 192.168.20.x
- Chronos → 192.168.30.x
- Metasploitable → 192.168.30.x

</details>

---

## Firewall Policy (Key Deliverable)

The primary deliverable of Part 2 is enforcing policy boundaries between security zones.

### CYBER_RANGE Security Policy

The CYBER_RANGE represents exploitable hosts.  
Firewall policy is designed for containment and controlled egress.

Rule order (top → bottom):

1. Allow DHCP to firewall  
2. Allow ICMP to firewall  
3. Block CYBER_RANGE → LAN (log enabled)  
4. Block CYBER_RANGE → AD_LAB (log enabled)  
5. Allow DNS outbound (53)  
6. Allow HTTP outbound (80)  
7. Allow HTTPS outbound (443)  
8. Block all remaining traffic (log enabled)

This rule set enforces strong segmentation while still allowing realistic outbound behavior.

---

## Controlled Egress Model

Rather than allowing unrestricted outbound traffic, the CYBER_RANGE zone is restricted to:

- DNS (53)
- HTTP (80)
- HTTPS (443)

All other outbound ports are blocked and logged.

This model simulates realistic attacker infrastructure behavior:

- Malware beaconing over HTTPS
- Payload downloads
- DNS resolution activity
- Blocked command-and-control traffic

These deny events generate high-quality telemetry for future SIEM ingestion.

---

## LAN Policy (Attacker Zone)

The LAN segment represents an attacker foothold.

Policy is intentionally permissive outbound to simulate an operator network.

Kali systems must be able to:

- Reach CYBER_RANGE systems
- Reach AD_LAB systems
- Reach the internet

IPv6 rules were reviewed to ensure firewall bypass conditions were not introduced.

---

<details>
<summary><strong>Validation / Testing</strong></summary>

The following validation steps confirmed segmentation behavior.

### DHCP Validation

- Kali receives 192.168.20.x lease
- CYBER_RANGE hosts receive 192.168.30.x leases
- AD_LAB network ready for Windows deployment

### Segmentation Validation

Confirmed:

- CYBER_RANGE cannot reach LAN
- CYBER_RANGE cannot reach AD_LAB
- Firewall blocks appear in logs

### Outbound Filtering Validation

Tested outbound connectivity from CYBER_RANGE hosts:

| Port | Result |
|----|------|
| 53 | Allowed |
| 80 | Allowed |
| 443 | Allowed |
| All others | Blocked |

### Persistence Validation

After firewall reboot:

- Interface assignments remain stable
- DHCP services remain active
- Firewall rule order preserved

</details>

---

<details>
<summary><strong>Screenshots / Evidence</strong></summary>

### VirtualBox Adapter Configuration

![VirtualBox OPNsense adapters](assets/vbox_opnsense_adapters.png)

---

### OPNsense Interface Assignments

![OPNsense interface assignments](assets/opnsense_interface_assignments.png)

---

### DHCP Leases

![OPNsense DHCP leases](assets/opnsense_dhcp_leases.png)

---

### CYBER_RANGE Firewall Rules

![CYBER_RANGE firewall rules](assets/opnsense_cyber_range_rules.png)

</details>

---

<details>
<summary><strong>Problems Encountered</strong></summary>

### DHCP Failure After Segmentation

Symptoms:

- Kali received a 169.254.x.x address
- DHCP services appeared unavailable

Root Cause:

VirtualBox adapter order drift caused OPNsense interface reassignment.

---

### Firewall Rules Not Triggering

Symptoms:

- Block rules appeared ineffective

Root Cause:

Firewall rule order placed allow rules above block rules.

---

### CYBER_RANGE Firewall Reachability Issue

Symptoms:

- Systems could not reach firewall gateway

Root Cause:

Traffic to "This Firewall" was not explicitly allowed.

</details>

---

<details>
<summary><strong>Fixes / Lessons Learned</strong></summary>

### Interface Stability Fix

Resolved by:

- Reordering VirtualBox adapters
- Reassigning interfaces inside OPNsense
- Validating persistence across reboot

---

### Firewall Rule Ordering

Resolved by placing block rules above allow rules to respect first-match firewall logic.

---

### Gateway Reachability

Added explicit firewall rule allowing CYBER_RANGE traffic to the firewall gateway.

</details>

---

## Skills Demonstrated

This phase demonstrates several advanced security engineering skills:

- Enterprise-style zone-based segmentation
- Firewall policy development and enforcement
- Controlled egress filtering design
- Firewall rule ordering and logic validation
- Network troubleshooting in virtualized environments
- Generation of high-quality security telemetry
- Infrastructure validation and documentation

These skills closely mirror real SOC engineering and network security operations.

---

## Next Phase

### Part 3: SOC Telemetry & SIEM Deployment

Part 3 will focus on deploying the SOC monitoring stack.

Planned objectives include:

- Forwarding OPNsense firewall logs to a SIEM platform (Wazuh)
- Deploying endpoint monitoring agents
- Simulating exploitation from Kali → CYBER_RANGE
- Capturing lateral movement attempts
- Building a detection and investigation workflow

This phase transitions the homelab from infrastructure into **active SOC monitoring and threat detection**.
