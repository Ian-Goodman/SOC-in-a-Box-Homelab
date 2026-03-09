# SOC-in-the-Box Homelab

## Overview

**SOC-in-the-Box** is a segmented cybersecurity homelab designed to simulate real-world **Security Operations Center (SOC)** infrastructure and workflows.

The project focuses on building a realistic **Blue Team defensive environment** that emphasizes:

- proper **network architecture**
- **firewall placement and segmentation**
- **traffic visibility**
- **centralized logging and monitoring**
- **security telemetry generation**

Rather than deploying tools immediately, the lab is intentionally built **from the network layer upward**, documenting the infrastructure, troubleshooting, and architectural decisions required to create a stable SOC environment.

Each phase of the project reflects a real step in building a security monitoring environment used in enterprise networks.

---

# Project Goals

The primary goals of this homelab are to:

- Design and implement **enterprise-style network segmentation**
- Deploy a **virtualized firewall architecture**
- Route all lab traffic through a monitored security boundary
- Generate realistic **security telemetry**
- Deploy a **centralized SIEM platform**
- Simulate attacker activity and detection workflows
- Document troubleshooting and engineering decisions throughout the build

---

# Lab Architecture

The SOC-in-the-Box environment is distributed across **two physical systems**.

## System 1 — Proxmox Server (SOC Infrastructure)

Hosts centralized security tooling and monitoring services.

Planned services include:

- **Wazuh SIEM**
  - Manager
  - Indexer
  - Dashboard
- Additional SOC tools as the environment expands

This system acts as the **central visibility and monitoring layer** of the lab.

---

## System 2 — Main Homelab Workstation (Operational Environment)

Runs the firewall and the operational lab environment.

Components include:

- **OPNsense Firewall**
  - Virtualized in VirtualBox
  - Enforces segmentation and firewall policies

- **Kali Linux**
  - Attack simulation / operator system

- **Vulnerable Linux Targets**
  - Exploitation practice and telemetry generation

- **Windows Environment (Planned)**
  - Active Directory
  - Windows endpoints

This system generates the **network traffic and events monitored by the SOC infrastructure**.

---

# Network Architecture

The lab uses a **zone-based firewall model**.

Each network segment is routed through **OPNsense**, which enforces policy and generates logs.

| Zone | Purpose |
|-----|--------|
| WAN | Upstream network |
| LAN | Attacker / operator systems |
| CYBER_RANGE | Vulnerable target systems |
| AD_LAB | Windows domain environment |
| SOC_SERVICES | Security monitoring infrastructure |

This design mirrors enterprise security architecture where **all traffic between zones passes through a firewall**.

---

# Technologies Used

| Category | Technology |
|--------|-------------|
| Virtualization | VirtualBox, Proxmox VE |
| Firewall | OPNsense |
| Networking | VLAN segmentation, managed switching |
| SIEM | Wazuh |
| Attack Platform | Kali Linux |
| Vulnerable Targets | Chronos, Metasploitable |
| Monitoring | Firewall logs, endpoint telemetry |

---

# Project Roadmap

The lab is built in documented phases.

Each phase expands the infrastructure and security visibility.

---

## Part 1 — Infrastructure Setup

📁 [Part 1 Documentation](part-1_infrastructure-setup/)

Focus:

- VLAN architecture design
- Managed switch configuration
- OPNsense deployment in VirtualBox
- WAN / LAN separation
- Firewall installation stability and persistence

Outcome:

A stable firewall architecture with correct network segmentation and deterministic routing.

---

## Part 2 — Network Segmentation & Firewall Policy

📁 [Part 2 Documentation](part-2_network-segmentation/)

Focus:

- Implementing **zone-based firewall segmentation**
- Assigning interfaces for each security zone
- Configuring DHCP per network segment
- Enforcing firewall policies between zones
- Implementing controlled outbound filtering
- Generating deny logs for telemetry

Outcome:

The lab transitions from a flat network into a **true segmented security environment**.

---

## Part 3 — SIEM Deployment & Telemetry Collection *(In Progress)*

Focus:

- Deploy Wazuh on the Proxmox server
- Forward firewall logs to SIEM
- Deploy endpoint telemetry agents
- Validate log ingestion pipelines
- Build baseline dashboards

Outcome:

Centralized **security monitoring and log visibility** across the lab.

---

## Part 4 — Windows Environment & Endpoint Telemetry *(Planned)*

Focus:

- Deploy Windows Active Directory
- Domain-join client systems
- Configure endpoint telemetry
- Expand detection surface

---

## Part 5 — Attack Simulation & Detection Engineering *(Planned)*

Focus:

- Simulate attacker activity
- Monitor alerts and detection rules
- Map detections to **MITRE ATT&CK**
- Document SOC triage workflows

---

# Future Enhancements

Planned improvements include:

- Full Wazuh SIEM deployment
- Windows Active Directory lab
- Additional segmented networks
- Detection engineering exercises
- MITRE ATT&CK detection mapping
- Incident response workflow simulation
- Ticketing and alert triage integration

---

# Skills Demonstrated

This project demonstrates practical experience in:

- Network segmentation architecture
- Firewall deployment and policy design
- Virtualized network infrastructure
- Security telemetry generation
- Infrastructure troubleshooting
- Security monitoring architecture
- SOC engineering fundamentals

---

# Connect

🔗 LinkedIn  
https://www.linkedin.com/in/iangoodman13/
