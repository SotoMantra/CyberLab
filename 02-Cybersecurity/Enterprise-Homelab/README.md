# 🛡️ Enterprise Cybersecurity Homelab

A segmented virtual cybersecurity environment designed to simulate a small enterprise network and provide hands-on experience with networking, systems administration, identity management, security monitoring, attack simulation, and incident investigation.

The environment is hosted on Proxmox and uses pfSense for routing, firewalling, and network segmentation.

---

## 🎯 Project Objectives

The primary goals of this lab are to:

- Build and administer a virtual enterprise-style network
- Practice network segmentation using VLANs
- Configure firewall policies between security zones
- Deploy and manage Windows Active Directory
- Centralize security monitoring with Wazuh
- Deploy honeypots using T-Pot
- Generate controlled security activity using Kali Linux
- Practice log analysis and security investigations
- Develop Linux and Windows administration skills
- Document real troubleshooting scenarios and solutions

---

# 🏗️ High-Level Architecture

The environment is hosted on a Proxmox virtualization server.

pfSense provides routing and firewall controls between multiple isolated network segments.

    Internet
        │
        ▼
    ┌─────────┐
    │ pfSense │
    └────┬────┘
         │
         ├── VLAN 10 — Servers
         │
         ├── VLAN 20 — Administration
         │
         ├── VLAN 30 — Honeypot
         │
         ├── VLAN 40 — Monitoring
         │
         ├── VLAN 50 — Attack / Testing
         │
         └── VLAN 99 — Management

Each network segment serves a specific role and can be controlled using firewall policies.

A detailed architecture diagram will be maintained in:

[Architecture Documentation](Architecture.md)

---

# 🌐 Network Segmentation

| VLAN | Role | Example Systems |
|---|---|---|
| VLAN 10 | Servers | Domain Controller, Windows systems |
| VLAN 20 | Administration | Administrative Jumpbox |
| VLAN 30 | Honeypot | T-Pot |
| VLAN 40 | Monitoring | Wazuh |
| VLAN 50 | Attack / Testing | Kali Linux |
| VLAN 99 | Management | Infrastructure management |

Segmentation allows security policies to control which systems and services are permitted to communicate between network zones.

More information:

[Network Design](Network-Design.md)

---

# 🖥️ Core Infrastructure

## Proxmox

Proxmox provides the virtualization layer used to host the lab's virtual machines and services.

The environment allows systems to be isolated, modified, tested, and restored without requiring separate physical hardware for every component.

## pfSense

pfSense provides:

- Inter-VLAN routing
- Firewall policies
- Network isolation
- NAT
- Controlled service access
- Network traffic management

---

# 🪟 Windows / Active Directory

The lab includes a Windows Active Directory environment using the domain:

    homelab.local

The environment is used to practice:

- Active Directory Domain Services
- User and group administration
- Organizational Units
- Group Policy
- DNS
- Administrative access
- Windows endpoint management

Systems include a domain controller, Windows endpoints, and an administrative jumpbox.

---

# 🔎 Security Monitoring

## Wazuh

Wazuh provides centralized security monitoring for systems within the lab.

The environment is used to practice:

- Endpoint monitoring
- Log collection
- Security event analysis
- Agent deployment
- Alert investigation
- Security configuration
- Troubleshooting agent connectivity

Windows and Linux systems can send security telemetry to the Wazuh monitoring environment.

---

# 🍯 Honeypot Environment

## T-Pot

T-Pot is deployed within a dedicated honeypot VLAN to provide an isolated environment for observing potentially malicious activity.

Honeypot services include technologies such as:

- Cowrie
- Dionaea

The honeypot network is separated from production-style lab systems to reduce the risk of unintended access.

---

# ⚔️ Attack / Testing Environment

Kali Linux is deployed within a dedicated attack/testing VLAN.

The system is used for controlled security testing and for generating activity that can be observed by monitoring systems.

Examples include:

- Authentication testing
- Network reconnaissance
- Port scanning
- Security event generation
- Detection testing

Testing is performed only against systems within the lab environment.

---

# 🔐 Security Design

The environment follows several basic security principles:

### Network Segmentation

Systems with different security roles are separated using VLANs.

### Least Privilege

Firewall policies are designed to permit required communication rather than unrestricted access between networks.

### Dedicated Administration

Administrative activity can be performed through a dedicated Jumpbox rather than directly exposing management interfaces.

### Centralized Monitoring

Security telemetry is collected by Wazuh for analysis.

### Isolation

Honeypot and attack/testing systems are separated from other infrastructure.

Detailed controls are documented in:

[Security Controls](Security-Controls.md)

---

# 🧰 Technologies

| Category | Technologies |
|---|---|
| Virtualization | Proxmox |
| Firewall / Routing | pfSense |
| Identity | Windows Active Directory |
| Windows | Windows Server, Windows endpoints |
| Linux | Kali Linux, Linux servers |
| SIEM / Monitoring | Wazuh |
| Honeypots | T-Pot, Cowrie, Dionaea |
| Networking | VLANs, DNS, DHCP, NAT |
| Remote Connectivity | Tailscale / WireGuard |
| Security Testing | Kali Linux |

---

# 🔧 Troubleshooting & Learning

An important goal of this project is documenting problems rather than only showing the finished environment.

Troubleshooting areas include:

- Firewall rules
- VLAN communication
- DNS resolution
- Wazuh agent connectivity
- Linux services
- Docker containers
- Honeypot connectivity
- NAT and port forwarding
- VPN routing

These scenarios provide hands-on experience with identifying symptoms, investigating logs and configurations, determining root causes, implementing fixes, and verifying results.

---

# 🧠 Skills Demonstrated

This project provides hands-on experience with:

- Network segmentation
- TCP/IP networking
- Firewall administration
- Windows Server
- Active Directory
- Linux administration
- SIEM
- Security monitoring
- Log analysis
- Virtualization
- VPN technologies
- Honeypots
- Security testing
- Troubleshooting
- Technical documentation

---

# 🚀 Future Improvements

Planned improvements include:

- Expand Wazuh detection capabilities
- Deploy Sysmon to Windows endpoints
- Develop custom detection rules
- Perform documented attack-and-detect exercises
- Improve architecture diagrams
- Expand security logging
- Add vulnerability management
- Explore automated response workflows

---

# 📚 Documentation

- [Architecture](Architecture.md)
- [Network Design](Network-Design.md)
- [Security Controls](Security-Controls.md)
- [Lessons Learned](Lessons-Learned.md)

---

## 📌 Project Status

**Status:** Active / Ongoing

This lab continues to evolve as new technologies, security controls, monitoring capabilities, and testing scenarios are added.
