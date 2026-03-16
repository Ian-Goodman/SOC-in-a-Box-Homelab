# Part 3 – SIEM Deployment (Wazuh)

---

## Overview

In this phase of the **SOC-in-a-Box Homelab**, a Security Information and Event Management (SIEM) platform was deployed to provide centralized log collection, monitoring, and security analysis.

For this implementation, **Wazuh** was installed on a dedicated Ubuntu virtual machine hosted on **Proxmox**. The SIEM server was placed in its own network segment (**VLAN 50 – Lab_Security**) to simulate how security monitoring infrastructure is typically deployed in enterprise environments.

This SIEM layer allows the lab to:

- Centralize logs from cyber range systems
- Detect suspicious activity using security rules
- Monitor authentication events and system behavior
- Visualize security data through dashboards
- Simulate real SOC monitoring workflows

This step transforms the homelab into a **functional Security Operations Center simulation environment**.

---

## Architecture Purpose

In production environments, security monitoring infrastructure is often isolated in a **dedicated security network segment**. This ensures detection systems remain protected while still receiving telemetry from across the network.

To replicate this architecture, a new VLAN was created specifically for security infrastructure.

### Security Monitoring Network

| VLAN | Name | Purpose |
|-----|------|------|
| 50 | Lab_Security | Security monitoring infrastructure |

The Wazuh SIEM resides inside this VLAN and receives logs from systems across the cyber range.

---

# Network Architecture
