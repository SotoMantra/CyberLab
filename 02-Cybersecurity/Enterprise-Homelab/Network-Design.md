# 🌐 Enterprise Homelab Network Design

## Overview

The Enterprise Cybersecurity Homelab uses virtual network segmentation to separate systems according to their purpose and security requirements.

Proxmox provides the virtual networking infrastructure, while pfSense operates as the primary firewall and router for the lab environment.

The physical Proxmox host connects to the home router through Ethernet. A Proxmox network bridge provides pfSense with connectivity toward the physical home network, while additional virtual networking provides connectivity to the internal lab environment.

---

# Network Topology

```
Internet
   │
   ▼
Home Router
   │
   │ Ethernet
   ▼
Proxmox Host
   │
   ▼
pfSense VM
   │
   ├── VLAN 10 — Server
   ├── VLAN 20 — Administration
   ├── VLAN 30 — Honeypot
   ├── VLAN 40 — Monitoring
   └── VLAN 50 — Attack / Testing
```

In addition to the pfSense-managed networks, an isolated virtual network connects Kali Linux to Metasploitable 2.

```
Kali Linux
    │
    │ Secondary virtual NIC
    ▼
Isolated Virtual Network
    │
    ▼
Metasploitable 2
```

---

## Proxmox Virtual Bridges

The Proxmox host uses three Linux bridges to separate physical connectivity, internal lab networking, and the isolated penetration-testing environment.

| Bridge | Address | Physical Uplink | VLAN Aware | Purpose |
|---|---|---|---|---|
| `vmbr0` | `192.168.1.250/24` | `enp0s31f6` | No | Home network connectivity and Proxmox management |
| `vmbr1` | None | None | Yes | Internal pfSense LAN and VLAN trunk |
| `vmbr2` | `10.10.10.1/24` | None | No | Isolated penetration-testing network |

### vmbr0 — Physical / Management Network

`vmbr0` is connected to the Dell OptiPlex physical Ethernet interface, `enp0s31f6`.

The bridge connects the Proxmox host and the pfSense WAN interface to the physical home network.

The Proxmox management interface uses:

    192.168.1.250/24

with the home router providing the upstream gateway.

### vmbr1 — Internal Lab Network

`vmbr1` has no physical bridge port and is configured as VLAN-aware.

It provides the internal virtual network used by pfSense and the segmented lab VLANs.

The following VLANs are carried through this environment:

- VLAN 10 — Servers
- VLAN 20 — Administration
- VLAN 30 — Honeypot
- VLAN 40 — Monitoring
- VLAN 50 — Attack / Testing

Because `vmbr1` has no physical uplink, the internal lab networks reach external networks through pfSense rather than connecting directly to the physical LAN.

### vmbr2 — Isolated Penetration-Testing Network

`vmbr2` provides a separate virtual network for controlled vulnerability testing.

Network:

    10.10.10.0/24

Proxmox bridge address:

    10.10.10.1/24

The bridge has no physical uplink and no upstream gateway.

Kali Linux and Metasploitable 2 connect to this network for controlled security testing.

This design prevents the intentionally vulnerable Metasploitable 2 system from being directly attached to the physical home network or the primary lab VLANs.

---

# Network Segments

## VLAN 10 — Server Network

**Purpose:** Server and directory services

**Primary System:**

- DC01 — Windows 11
    

**Services / Roles:**

- Active Directory
    
- DNS
    
- Windows administration
    
- Directory services
    

**Subnet:** To be documented  
**Gateway:** To be documented

---

## VLAN 20 — Administration

**Purpose:** Dedicated administrative access

**Primary System:**

- Admin JumpBox — Windows 10
    

The administration VLAN provides a dedicated location from which management activity can be performed.

Separating administrative systems from other workloads reduces unnecessary exposure of management activity to testing, honeypot, and server networks.

**Subnet:** To be documented  
**Gateway:** To be documented

---

## VLAN 30 — Honeypot

**Purpose:** Honeypot and threat-observation environment

**Primary System:**

- T-Pot — Linux
    

The honeypot network is separated from other lab systems because it is intended to interact with potentially malicious or untrusted activity.

Firewall rules should restrict unnecessary access from the honeypot network toward trusted systems.

**Subnet:** To be documented  
**Gateway:** To be documented

---

## VLAN 40 — Monitoring

**Purpose:** Centralized security monitoring

**Primary System:**

- Wazuh Server — Linux
    

The monitoring network hosts the security-monitoring infrastructure used to receive and analyze telemetry from supported systems.

Endpoints requiring monitoring can be permitted to communicate with the required Wazuh services while unrelated traffic remains restricted.

**Subnet:** To be documented  
**Gateway:** To be documented

---

## VLAN 50 — Attack / Testing

**Purpose:** Controlled security testing

**Primary System:**

- Kali Linux
    

Kali Linux is separated from administrative and server systems through a dedicated attack/testing network.

Access to other lab networks is controlled through pfSense firewall policies.

Kali also has a second virtual network connection to the isolated Metasploitable 2 environment.

**Subnet:** To be documented  
**Gateway:** To be documented

---

# Isolated Vulnerable Target Network

Metasploitable 2 does not reside on one of the primary lab VLANs.

Instead, an isolated virtual network provides direct connectivity between:

- Kali Linux
    
- Metasploitable 2
    

Conceptually:

```
VLAN 50
   │
   ▼
Kali Linux
   │
   │ Secondary NIC
   ▼
Isolated Network
   │
   ▼
Metasploitable 2
```

The isolated network is intended to prevent Metasploitable 2 from directly communicating with:

- The home network
    
- The Internet
    
- Server systems
    
- Administrative systems
    
- Honeypot infrastructure
    
- Monitoring infrastructure
    

Connectivity testing should be used to verify these isolation assumptions.

---

# pfSense

pfSense provides the primary Layer 3 routing and firewall functionality for the segmented lab environment.

## Responsibilities

- Routing between permitted networks
    
- Firewall enforcement
    
- Inter-VLAN traffic control
    
- NAT
    
- Network isolation
    
- Controlled access to required services
    

The firewall policy is intended to follow a default-restrictive approach in which communication is allowed according to system requirements rather than providing unrestricted access between every VLAN.

# pfSense

pfSense provides the primary Layer 3 routing and firewall functionality for the segmented lab environment.

## Responsibilities

- Routing between permitted networks
- Firewall enforcement
- Inter-VLAN traffic control
- NAT
- Network isolation
- Controlled access to required services

The firewall policy is intended to follow a default-restrictive approach in which communication is allowed according to system requirements rather than providing unrestricted access between every VLAN.

## VLAN Parent Interface

The lab VLANs are configured in pfSense using `em0 (LAN)` as their parent interface.

The following VLANs share the `em0` parent interface:

| pfSense Interface | VLAN | Gateway | Purpose |
|---|---:|---|---|
| `em0.10 (OPT1)` | 10 | `192.168.10.1/24` | Server |
| `em0.20 (OPT2)` | 20 | `192.168.20.1/24` | Administration |
| `em0.30 (OPT3)` | 30 | `192.168.30.1/24` | Honeypot |
| `em0.40 (OPT4)` | 40 | `192.168.40.1/24` | Monitoring |
| `em0.50 (OPT5)` | 50 | `192.168.50.1/24` | Attack / Testing |
VLAN tagging allows these logically separate networks to share the same virtual parent interface while pfSense provides Layer 3 routing and firewall enforcement between them.

---

# Traffic Flow

## Administrative Traffic

The Admin JumpBox provides a dedicated system for administrative access to lab resources.

```
Admin JumpBox
     │
   VLAN 20
     │
   pfSense
     │
     ▼
Authorized Lab Systems
```

Access is controlled according to the firewall rules configured in pfSense.

---

## Monitoring Traffic

Systems monitored by Wazuh send required security telemetry toward the monitoring environment.

```
Monitored Endpoint
      │
      ▼
   pfSense
      │
      ▼
   VLAN 40
      │
      ▼
 Wazuh Server
```

Only required monitoring communication should be permitted.

---

## Security Testing Traffic

Kali Linux can generate controlled testing activity against authorized lab targets.

```
Kali
  │
VLAN 50
  │
pfSense
  │
  ▼
Authorized Lab Target
```

Firewall rules determine which lab networks and services Kali is permitted to reach.

For intentionally vulnerable testing, Kali can instead communicate directly with Metasploitable 2 through the isolated virtual network.

---

# Current Network Inventory

| Network          | Subnet            | Gateway        | Primary System   | Purpose                   |
| ---------------- | ----------------- | -------------- | ---------------- | ------------------------- |
| VLAN 10          | `192.168.10.0/24` | `192.168.10.1` | DC01             | Server / Active Directory |
| VLAN 20          | `192.168.20.0/24` | `192.168.20.1` | Admin JumpBox    | Administration            |
| VLAN 30          | `192.168.30.0/24` | `192.168.30.1` | T-Pot            | Honeypot                  |
| VLAN 40          | `192.168.40.0/24` | `192.168.40.1` | Wazuh Server     | Security monitoring       |
| VLAN 50          | `192.168.50.0/24` | `192.168.50.1` | Kali Linux       | Attack / testing          |
| Isolated Network | 10.10.10.0/24     | None           | Metasploitable 2 | Vulnerable testing target |

---

# Validation Plan

The network design should be verified rather than assumed to be functioning as intended.

Planned validation includes:

- Verify each VM receives or uses the intended network configuration
    
- Verify VLAN membership
    
- Verify default gateways
    
- Verify DNS resolution where required
    
- Verify permitted inter-VLAN communication
    
- Verify blocked inter-VLAN communication
    
- Verify Wazuh endpoint connectivity
    
- Verify JumpBox administrative access
    
- Verify Kali access to authorized testing targets
    
- Verify Kali can reach Metasploitable 2
    
- Verify Metasploitable 2 cannot reach trusted networks
    
- Verify Metasploitable 2 cannot reach the physical home network
    
- Verify Metasploitable 2 cannot reach the Internet
    

---

# Future Documentation

The following information will be added after verification in pfSense and Proxmox:

- VLAN subnets
    
- Gateway addresses
    
- pfSense interface assignments
    
- Proxmox bridge configuration
    
- Firewall rule matrix
    
- DHCP configuration
    
- DNS configuration
    
- Network validation results
    

---

# Related Documentation

- [Enterprise Homelab Overview](https://chatgpt.com/c/README.md)
    
- [Architecture](https://chatgpt.com/c/Architecture.md)
    
- [Security Controls](https://chatgpt.com/c/Security-Controls.md)
    
- [Lessons Learned](https://chatgpt.com/c/Lessons-Learned.md)