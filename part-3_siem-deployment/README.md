# Part 3: SIEM Deployment & SOC Telemetry

## Overview

Part 3 focuses on deploying centralized security monitoring infrastructure within the SOC-in-a-Box Homelab.

This phase introduces a **Security Information and Event Management (SIEM)** platform using **Wazuh**, deployed on a dedicated Ubuntu virtual machine hosted on **Proxmox**. The SIEM server resides inside a dedicated security monitoring segment (**VLAN 50 – Lab_Security**) to simulate enterprise environments where monitoring systems operate within protected security infrastructure networks.

This monitoring layer allows the lab to:

- Centralize logs from cyber range systems
- Detect suspicious activity using security rules
- Monitor authentication events and system behavior
- Visualize security telemetry through dashboards
- Simulate real SOC monitoring workflows

This phase transitions the homelab from infrastructure architecture into a **functional SOC monitoring environment**.

---

## Objective

The objective of this phase is to deploy enterprise-style SOC monitoring infrastructure capable of receiving telemetry from across the lab environment.

This includes:

- Deploying a SIEM platform using Wazuh
- Creating a dedicated security monitoring network segment
- Hosting the SIEM inside Proxmox infrastructure
- Configuring static addressing for the SIEM server
- Verifying Wazuh services and listening ports
- Enabling log ingestion capabilities
- Validating connectivity between monitored systems and the SIEM
- Establishing snapshot recovery points for SOC infrastructure

---

## Environment / Scope

The SOC monitoring architecture deployed in this phase includes the following components.

**SIEM Platform**

- Wazuh Manager
- Wazuh Indexer (OpenSearch)
- Wazuh Dashboard

**Virtualization Platform**

- Proxmox used to host the SIEM virtual machine

**Operating System**

- Ubuntu Server deployed as the SIEM host

**Security Monitoring Network**

- VLAN 50 (Lab_Security) dedicated to SOC infrastructure

**Firewall / Network Control**

- OPNsense routing traffic between lab segments and security infrastructure

---

## Tools Used

The following tools were used during this phase:

- **Wazuh** – Security Information and Event Management platform
- **Ubuntu Server** – SIEM host operating system
- **Proxmox** – virtualization platform hosting SOC infrastructure
- **OPNsense** – firewall and routing between segmented networks
- **Linux networking utilities** – service and port verification
- **Netplan** – Linux network configuration management
- **Wazuh Dashboard** – visualization and monitoring interface

---

## Architecture / Design Notes

Enterprise SOC architectures commonly isolate monitoring infrastructure inside a **dedicated security network segment**.

This design ensures:

- Monitoring systems remain protected from compromise
- Detection infrastructure remains stable during attacks
- Telemetry can be centrally aggregated
- Security analysts can investigate events without interfering with production infrastructure

### Security Monitoring Network

| VLAN | Name | Purpose |
|-----|------|------|
| 50 | Lab_Security | Security monitoring infrastructure |

### SIEM Host

| Host | Role | IP Address |
|-----|------|-----------|
| Wazuh Server | SIEM Manager / Indexer / Dashboard | 192.168.50.169 |

---

<details>
<summary><strong>Implementation Steps</strong></summary>

### 1. Create Security Monitoring VLAN

A dedicated network segment was created to host SOC monitoring infrastructure.

| VLAN | Name | Purpose |
|-----|------|------|
| 50 | Lab_Security | Security monitoring systems |

This VLAN isolates monitoring infrastructure while still allowing telemetry ingestion.

---

### 2. Deploy Ubuntu SIEM Host

An Ubuntu Server virtual machine was created inside Proxmox to host the Wazuh stack.

The VM was connected to **VLAN 50** and configured with the following static address:

```
192.168.50.169
```

---

### 3. Install Wazuh SIEM

Wazuh was installed using the **Wazuh Installation Assistant**, which automatically deploys:

- Wazuh Manager  
- Wazuh Indexer (OpenSearch)  
- Wazuh Dashboard  

Installation command:

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

---

### 4. Verify Wazuh Services

After installation, listening services were verified.

```bash
sudo ss -tulnp | grep -E "1514|1515|55000|443"
```

Expected services:

| Port | Service |
|-----|------|
| 1514 | Wazuh agent communication |
| 1515 | Wazuh agent enrollment |
| 55000 | Wazuh API |
| 443 | Wazuh dashboard |

---

### 5. Configure Static IP Address

The SIEM server was configured with a static IP to ensure consistent connectivity.

Configuration file:

```
/etc/netplan/01-network-manager-all.yaml
```

Example configuration:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens18:
      addresses:
        - 192.168.50.169/24
      gateway4: 192.168.50.1
      nameservers:
        addresses:
          - 8.8.8.8
```

Apply configuration:

```bash
sudo netplan apply
```

---

### 6. Verify Indexer Heap Allocation

To ensure proper log indexing performance, heap allocation was inspected.

```bash
sudo cat /etc/wazuh-indexer/jvm.options | grep -E "Xms|Xmx"
```

Heap allocation was increased to **3GB**.

---

### 7. Enable Syslog Log Ingestion

The Wazuh syslog listener was enabled to allow external systems to send logs to the SIEM.

Configuration file:

```
/var/ossec/etc/ossec.conf
```

After editing the configuration, the manager service was restarted.

```bash
sudo systemctl restart wazuh-manager
```

---

### 8. Create Proxmox Snapshot

Once installation and connectivity were verified, a **Proxmox snapshot** was created to preserve the SIEM baseline configuration.

</details>

---

## SIEM Deployment (Key Deliverable)

The primary deliverable of Part 3 is the deployment of a centralized SIEM capable of ingesting telemetry across the lab environment.

Core components deployed:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard

Dashboard access:

```
https://192.168.50.169
```

Credentials used for initial login:

| Username | Password |
|---------|----------|
| admin | --- |

The dashboard provides:

- Security alerts
- Host monitoring
- Event search
- Compliance monitoring
- Security telemetry dashboards

---

## Supporting Architecture

```
Cyber Range Systems
        │
        │
        ▼
     OPNsense
 (Firewall / Router)
        │
        │
        ▼
 Lab_Security VLAN
        │
        │
        ▼
     Wazuh SIEM
   192.168.50.169
```

This architecture mirrors real enterprise SOC monitoring pipelines where telemetry from multiple segments is forwarded to centralized detection infrastructure.

---

<details>
<summary><strong>Validation / Testing</strong></summary>

The following validation steps confirmed successful SIEM deployment.

### Service Verification

Confirmed listening ports for:

- Wazuh Manager
- Wazuh API
- Wazuh Dashboard
- Agent communication

### Dashboard Access

Verified browser access to:

```
https://192.168.50.169
```

### Network Connectivity

Confirmed:

- OPNsense firewall could reach the SIEM host
- Connectivity from cyber range networks restored after firewall rule updates

### Log Ingestion

Verified logs successfully appearing inside the Wazuh dashboard.

</details>

---

<details>
<summary><strong>Screenshots / Evidence</strong></summary>

### VLAN 50 Creation

![VLAN 50 Lab Security](assets/vlan50-lab-security.png)

---

### Proxmox Ubuntu VM Creation

![Proxmox Ubuntu VM](assets/proxmox-ubuntu-vm.png)

---

### Wazuh Installation

![Wazuh Installation](assets/wazuh-installation.png)

---

### Wazuh Dashboard Login

![Wazuh Dashboard Login](assets/wazuh-dashboard-login.png)

---

### Wazuh Dashboard Overview

![Wazuh Dashboard](assets/wazuh-dashboard-overview.png)

---

### Port Verification

![Wazuh Port Verification](assets/wazuh-port-verification.png)

---

### Static IP Configuration

![Wazuh Static IP](assets/wazuh-static-ip.png)

---

### SIEM Log Ingestion

![Wazuh Logs](assets/wazuh-archive-logs.png)

</details>

---

<details>
<summary><strong>Problems Encountered</strong></summary>

### SIEM Network Reachability Issue

Symptoms:

- OPNsense firewall could successfully ping the Wazuh server
- Cyber range systems were unable to reach the SIEM

Root Cause:

Firewall rules were preventing communication between network segments.

</details>

---

<details>
<summary><strong>Fixes / Lessons Learned</strong></summary>

### Firewall Policy Adjustment

Connectivity was restored by creating firewall rules allowing cyber range networks to communicate with the SIEM monitoring segment.

---

### SIEM Infrastructure Isolation

Deploying monitoring infrastructure inside a dedicated VLAN improves stability and protects detection systems during security testing.

---

### Infrastructure Snapshots

Creating Proxmox snapshots provides safe rollback points when experimenting with SOC tooling.

</details>

---

## Skills Demonstrated

This phase demonstrates several SOC engineering and security operations skills:

- SIEM platform deployment
- Security monitoring architecture design
- Enterprise network segmentation for monitoring infrastructure
- Linux server configuration
- Security telemetry pipeline creation
- Firewall troubleshooting and policy design
- Infrastructure snapshot management
- Security dashboard monitoring and analysis

These skills align closely with real **SOC analyst and detection engineering responsibilities**.

---

## Next Phase

### Part 4: Threat Simulation & Detection

The next phase will focus on generating realistic security events within the cyber range.

Planned objectives include:

- Deploying Wazuh agents on monitored systems
- Forwarding firewall logs into the SIEM
- Simulating exploitation from Kali → CYBER_RANGE
- Generating security alerts
- Performing investigation workflows inside the Wazuh dashboard

This phase will transition the environment from **passive monitoring to active threat detection and incident investigation**.
