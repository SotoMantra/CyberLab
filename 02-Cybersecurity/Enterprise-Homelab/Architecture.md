# 🏗️ Enterprise Homelab Architecture

## Overview

The Enterprise Cybersecurity Homelab is hosted on a Dell OptiPlex running Proxmox VE.

The physical Proxmox host connects to the home router through Ethernet. Proxmox provides the virtualization layer for the lab, while a pfSense virtual machine acts as the primary router and firewall between the physical home network and the isolated virtual lab networks.

This design allows multiple security zones to operate on a single physical server while maintaining logical separation between systems.

---

# Physical Architecture

```
Internet
   │
   ▼
Home Router
   │
   │ Ethernet
   ▼
Dell OptiPlex
   │
   ▼
Proxmox VE
   │
   ├── pfSense VM
   ├── DC01
   ├── Admin JumpBox
   ├── T-Pot
   ├── Wazuh Server
   ├── Kali Linux
   └── Metasploitable 2
```

The Dell OptiPlex provides the compute, memory, networking, and storage resources for the virtual environment.

---

# Virtual Network Architecture

pfSense acts as the gateway between the physical home network and the segmented cybersecurity environment.

```
Internet
   │
   ▼
Home Router
   │
   │ Physical Ethernet
   ▼
┌──────────────────────────────┐
│       Dell OptiPlex         │
│                             │
│        Proxmox VE           │
│                             │
│     ┌──────────────┐        │
│     │   pfSense    │        │
│     │ Firewall /   │        │
│     │ Router VM    │        │
│     └──────┬───────┘        │
│            │                │
│       Virtual Networks      │
│            │                │
└────────────┼────────────────┘
             │
   ┌─────────┼─────────┬─────────┬─────────┐
   │         │         │         │         │
   ▼         ▼         ▼         ▼         ▼
VLAN 10   VLAN 20   VLAN 30   VLAN 40   VLAN 50
Servers    Admin    Honeypot   Monitor   Attack
   │         │         │         │         │
   ▼         ▼         ▼         ▼         ▼
 DC01     JumpBox     T-Pot     Wazuh      Kali
                                              │
                                       Secondary NIC - vmbr2
                                              │
                                              ▼
                                   Isolated Virtual
                                       Network - 10.10.10.0/24
                                              │
                                              ▼
                                     Metasploitable 2
```

---

# Network Zones

|Network|Zone|Primary System|Purpose|
|---|---|---|---|
|VLAN 10|Server|DC01|Server and directory services|
|VLAN 20|Administration|Admin JumpBox|Administrative access|
|VLAN 30|Honeypot|T-Pot|Honeypot and threat observation|
|VLAN 40|Monitoring|Wazuh Server|Centralized security monitoring|
|VLAN 50|Attack / Testing|Kali Linux|Controlled security testing|
|Isolated Virtual Network|Vulnerable Target|Metasploitable 2|Isolated vulnerable target|

---

# Virtual Machines

## pfSense

**Role:** Firewall / Router

pfSense is deployed as a virtual machine within Proxmox.

One side of pfSense connects through a Proxmox network bridge toward the physical home network. The other side provides routing and firewall services for the segmented virtual lab environment.

Responsibilities include:

- Routing
    
- Network segmentation
    
- Firewall policy enforcement
    
- Inter-VLAN traffic control
    
- NAT
    
- Lab network isolation
    

---

## DC01

**Operating System:** Microsoft Windows 11  
**Network:** VLAN 10 — Server Network

DC01 provides Active Directory and Windows-related services for the lab environment.

The system is used to practice:

- Active Directory Domain Services
    
- DNS
    
- User administration
    
- Group administration
    
- Organizational Units
    
- Group Policy
    
- Windows administration
    

---

## Admin JumpBox

**Operating System:** Microsoft Windows 10  
**Network:** VLAN 20 — Administration

The Admin JumpBox provides a dedicated administrative workstation for managing systems within the lab.

Placing the JumpBox on a dedicated administration network separates management activity from normal server, monitoring, honeypot, and security-testing workloads.

---

## T-Pot

**Operating System:** Linux  
**Network:** VLAN 30 — Honeypot

T-Pot provides the honeypot environment.

The honeypot resides within a dedicated network segment to separate potentially malicious or untrusted activity from the rest of the lab infrastructure.

The environment can be used to study:

- Honeypot activity
    
- Connection attempts
    
- Attack patterns
    
- Security telemetry
    
- Network behavior
    

---

## Wazuh Server

**Operating System:** Linux  
**Network:** VLAN 40 — Monitoring

Wazuh provides centralized security monitoring and log analysis for systems within the environment.

The dedicated monitoring network separates security-monitoring infrastructure from the endpoints and systems being monitored.

The environment is used to practice:

- Endpoint monitoring
    
- Log collection
    
- Security event analysis
    
- Agent management
    
- Alert investigation
    
- Security monitoring
    

---

## Kali Linux

**Operating System:** Kali Linux  
**Network:** VLAN 50 — Attack / Testing  
**Additional Network:** Isolated Virtual Attack Network

Kali Linux provides the security-testing workstation for the environment.

The VM has access to the attack/testing VLAN as well as the isolated virtual network containing Metasploitable 2.

This allows Kali to communicate with the intentionally vulnerable target without placing Metasploitable 2 directly onto the primary lab networks.

Kali is used for controlled testing within the homelab environment.

---

## Metasploitable 2

**Operating System:** Linux  
**Network:** Isolated Virtual Attack Network

Metasploitable 2 provides an intentionally vulnerable target for controlled security testing.

The system is not connected to the primary lab VLANs. Instead, it resides on a dedicated isolated virtual network.

Kali Linux is connected to the same isolated network, allowing it to communicate directly with Metasploitable 2.

The design is intended to prevent the vulnerable target from communicating directly with trusted lab systems or the physical home network.

Conceptually:

```
VLAN 50
   │
   ▼
┌────────┐
│  Kali  │
└───┬────┘
    │
    │ Secondary virtual NIC
    ▼
┌────────────────────────┐
│ Isolated Virtual       │
│ Attack Network         │
│                        │
│   ┌────────────────┐   │
│   │ Metasploitable │   │
│   │       2        │   │
│   └────────────────┘   │
└────────────────────────┘
```

Metasploitable 2 is intentionally separated from:

- The physical home network
    
- The Internet
    
- VLAN 10 — Servers
    
- VLAN 20 — Administration
    
- VLAN 30 — Honeypot
    
- VLAN 40 — Monitoring
    

The isolation should be validated through connectivity testing as part of the lab documentation.

---

# Security Architecture

## Network Segmentation

Systems are separated into network zones according to their function and security requirements.

Server, administration, honeypot, monitoring, and attack/testing workloads operate on separate VLANs.

---

## Firewall Enforcement

pfSense controls traffic between the segmented networks.

Firewall policies can be used to permit required services while preventing unnecessary communication between security zones.

---

## Administrative Separation

Administrative activity is performed through a dedicated Windows JumpBox located on VLAN 20.

This provides a separate management environment rather than using security-testing or general-purpose systems for administrative access.

---

## Security Monitoring

Wazuh operates from VLAN 40 and provides centralized monitoring for supported endpoints within the lab.

This allows security events from multiple systems to be collected and analyzed from a dedicated monitoring environment.

---

## Honeypot Isolation

T-Pot resides on VLAN 30.

Keeping the honeypot environment separate reduces unnecessary communication between potentially hostile activity and other lab systems.

---

## Attack-System Separation

Kali Linux resides on VLAN 50, separating security-testing activity from administrative and server networks.

Kali also has controlled access to the isolated virtual network used for Metasploitable 2.

---

## Vulnerable Target Isolation

Metasploitable 2 is intentionally vulnerable.

Rather than placing it on one of the primary lab VLANs, it resides on an isolated virtual network accessible by Kali Linux.

This creates a contained environment for vulnerability testing and attack simulation.

---

# Architecture Design Principles

The lab follows several core design principles:

### Segmentation

Systems with different functions and risk levels are separated into different networks.

### Least-Required Connectivity

Systems should only communicate with networks and services required for their role.

### Administrative Separation

Management activity is separated from attack/testing activity.

### Monitoring

Security telemetry is centralized through Wazuh.

### Isolation

Intentionally vulnerable and honeypot systems are separated from trusted systems.

### Controlled Testing

Security testing is performed against systems owned and operated within the lab environment.

---

# Skills Demonstrated

This architecture provides hands-on experience with:

- Proxmox virtualization
    
- Virtual networking
    
- pfSense
    
- Network segmentation
    
- VLANs
    
- Firewall policies
    
- Routing
    
- NAT
    
- Windows administration
    
- Active Directory
    
- Linux administration
    
- SIEM
    
- Security monitoring
    
- Honeypots
    
- Vulnerable systems
    
- Security testing
    
- Network isolation
    
- Troubleshooting
    
- Security architecture
    
- Technical documentation
    

---

# Related Documentation

- [Enterprise Homelab Overview](https://chatgpt.com/c/README.md)
    
- [Network Design](https://chatgpt.com/c/Network-Design.md)
    
- [Security Controls](https://chatgpt.com/c/Security-Controls.md)
    
- [Lessons Learned](https://chatgpt.com/c/Lessons-Learned.md)
    

---

## Project Status

**Status:** Active / Ongoing

The architecture will continue to evolve as additional security controls, monitoring capabilities, attack simulations, and infrastructure services are introduced.