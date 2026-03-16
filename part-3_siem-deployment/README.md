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

```
        Cyber Range
     (Attack Targets)
            │
            │
            ▼
        OPNsense
    (Firewall / Router)
            │
            │
            ▼
    Lab_Security VLAN 50
            │
            │
            ▼
        Wazuh SIEM
      192.168.50.169
```

---

# Step 5 – Install Wazuh SIEM

Wazuh was installed using the **Wazuh Installation Assistant**, which automatically deploys:

- Wazuh Manager  
- Wazuh Indexer (OpenSearch)  
- Wazuh Dashboard  

### Installation

```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

The installation script configures the entire SIEM stack required for log ingestion and monitoring.

---

# Step 6 – Access the Wazuh Dashboard

Once installation completed, the Wazuh dashboard became accessible through a web browser.

```
https://192.168.50.169
```

### Login Credentials

| Username | Password |
|---------|----------|
| admin | W4zO0foo13eva |

The dashboard provides:

- Security alerts
- Host monitoring
- Event search
- Compliance monitoring
- Security dashboards

---

# Step 7 – Verify Wazuh Services

After installation, the listening services were verified to confirm the SIEM stack was functioning.

```bash
sudo ss -tulnp | grep -E "1514|1515|55000|443"
```

### Important Ports

| Port | Service |
|-----|------|
| 1514 | Wazuh agent communication |
| 1515 | Wazuh agent enrollment |
| 55000 | Wazuh API |
| 443 | Wazuh dashboard |

This ensures the Wazuh manager, API, and dashboard are running.

---

# Step 8 – Verify OpenSearch Heap Allocation

To confirm the Wazuh indexer had sufficient memory allocation, the heap settings were inspected.

```bash
sudo cat /etc/wazuh-indexer/jvm.options | grep -E "Xms|Xmx"
```

The heap size was increased to **3GB** to support log indexing performance in the SIEM environment.

---

# Step 9 – Configure Static IP Address

A static IP address was configured so the SIEM server remains consistently reachable from other lab systems.

```bash
sudo nano /etc/netplan/01-network-manager-all.yaml
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

Apply the configuration:

```bash
sudo netplan apply
```

---

# Step 10 – Take Proxmox Snapshot

After verifying the SIEM installation was successful, a **Proxmox snapshot** was created.

This snapshot provides a recovery point if future configuration changes cause issues.

---

# Step 11 – Troubleshoot Network Connectivity

During testing, a connectivity issue appeared:

- OPNsense could ping the Wazuh server
- Other VMs in the cyber range could not

### Root Cause

Firewall policies were preventing traffic from reaching the SIEM.

### Resolution

Firewall rules were added in **OPNsense** allowing the cyber range VLAN to communicate with the SIEM network.

This restored connectivity between the cyber range and the Wazuh server.

---

# Step 12 – Enable Syslog Listener

To allow external systems to send logs to the SIEM, the Wazuh syslog listener was enabled.

```bash
sudo nano /var/ossec/etc/ossec.conf
```

After updating the configuration, the Wazuh manager was restarted.

```bash
sudo systemctl restart wazuh-manager
```

This allows log ingestion from monitored systems across the lab.

---

# Verification

After completing deployment:

- The Wazuh dashboard loaded successfully
- SIEM ports were listening correctly
- Firewall rules allowed connectivity
- Logs were successfully ingested into the SIEM

This confirmed the SOC monitoring platform was operational.

---

# Lessons Learned

This phase introduced key SOC infrastructure concepts:

### Security Network Segmentation
Security monitoring infrastructure is commonly placed in its own network segment to protect detection systems.

### SIEM Resource Allocation
Proper memory allocation improves log indexing performance.

### Firewall Rule Design
Monitoring infrastructure must remain reachable from monitored systems while still being protected.

### Infrastructure Snapshots
Snapshots provide safe rollback points when experimenting with security tooling.

---

# Screenshots to Capture

### Screenshot: VLAN 50 Creation  
**Filename:** `vlan50-lab-security.png`  
**Description:** Creation of VLAN 50 (Lab_Security) on the UCG-Ultra router.

---

### Screenshot: Proxmox Ubuntu VM Creation  
**Filename:** `proxmox-ubuntu-vm.png`  
**Description:** Ubuntu virtual machine created in Proxmox to host the Wazuh SIEM.

---

### Screenshot: Wazuh Installation Terminal  
**Filename:** `wazuh-installation.png`  
**Description:** Terminal showing Wazuh installation assistant running.

---

### Screenshot: Wazuh Dashboard Login  
**Filename:** `wazuh-dashboard-login.png`  
**Description:** Login page for the Wazuh dashboard.

---

### Screenshot: Wazuh Dashboard Overview  
**Filename:** `wazuh-dashboard-overview.png`  
**Description:** Main Wazuh dashboard displaying monitoring panels.

---

### Screenshot: Port Verification  
**Filename:** `wazuh-port-verification.png`  
**Description:** Terminal output showing required Wazuh ports listening.

---

### Screenshot: Static IP Configuration  
**Filename:** `wazuh-static-ip.png`  
**Description:** Netplan configuration file showing static IP settings.

---

### Screenshot: Archive Logs in Discover  
**Filename:** `wazuh-archive-logs.png`  
**Description:** Wazuh Discover view showing ingested logs from the cyber range.

---

# Next Phase

The next phase will expand the SOC-in-a-Box environment by integrating:

- Cyber range systems
- Attack simulation tools
- Log ingestion pipelines

This enables **realistic security monitoring and threat detection exercises** within the lab.
