
# 🔐 Security Controls

## Overview

The Enterprise Cybersecurity Homelab uses network segmentation, firewall policy enforcement, dedicated administrative systems, security monitoring, and isolated testing environments to reduce unnecessary communication between systems.

The primary firewall is pfSense, which controls traffic between VLANs and the physical home network.

---

# Network Segmentation

The lab is divided into separate security zones:

| Network | Purpose | Primary System |
|---|---|---|
| VLAN 10 | Server | DC01 |
| VLAN 20 | Administration | Admin JumpBox |
| VLAN 30 | Honeypot | T-Pot |
| VLAN 40 | Monitoring | Wazuh Server |
| VLAN 50 | Attack / Testing | Kali Linux |
| 192.168.99.0/24 | Untagged / Base LAN | pfSense LAN |
| 10.10.10.0/24 | Isolated Pentest Network | Kali + Metasploitable 2 |

Each zone is treated according to its role and risk level.

---

# Firewall Design Principles

The firewall policy follows these principles:

- Default deny where practical
- Specific host-based rules instead of broad subnet access
- Approved DNS only
- Limited outbound web access
- Restricted inter-VLAN communication
- Dedicated management access from the Admin JumpBox
- Isolation of vulnerable and honeypot systems
- Explicit access rules for security monitoring

---

# VLAN 10 — Server Network

Primary system:

    DC01

Allowed traffic includes:

- Wazuh agent communication to the Wazuh server
- Approved DNS
- NTP
- HTTP / HTTPS for updates
- ICMP for troubleshooting

Blocked traffic includes:

- Unnecessary access to protected lab networks
- Unapproved DNS
- Unapproved outbound services

---

# VLAN 20 — Administration

Primary system:

    Admin JumpBox

The JumpBox is used as the primary administrative workstation.

Allowed traffic includes:

- pfSense WebGUI over HTTPS
- RDP to DC01
- T-Pot management interface
- Wazuh Dashboard
- Approved DNS
- HTTP / HTTPS Internet access
- ICMP for troubleshooting

Broad temporary access rules were removed after specific management rules were created and tested.

---

# VLAN 30 — Honeypot

Primary system:

    T-Pot

Because the honeypot interacts with potentially malicious activity, VLAN30 uses a restrictive firewall policy.

Allowed traffic includes:

- Wazuh communication
- Approved DNS
- NTP
- HTTP / HTTPS Internet access
- ICMP to pfSense for troubleshooting

Blocked traffic includes:

- Access to protected internal networks
- Access to pfSense WebGUI
- Unapproved DNS servers
- Unnecessary outbound traffic

The final rule is an explicit default deny.

---

# VLAN 40 — Monitoring

Primary system:

    Wazuh Server

Allowed traffic includes:

- Approved DNS
- NTP
- HTTP / HTTPS for updates
- ICMP for troubleshooting

Security telemetry from monitored endpoints is permitted using rules on the originating VLANs.

Broad VLAN40 allow-all access was replaced with specific host-based rules.

---

# VLAN 50 — Attack / Testing

Primary system:

    Kali Linux

Kali is intentionally isolated from trusted networks.

Allowed traffic includes:

- Approved DNS
- NTP
- HTTP / HTTPS Internet access
- ICMP for troubleshooting
- Specific authorized testing traffic when required

Blocked traffic includes:

- Home network access
- Server network access
- Administration network access
- Honeypot network access
- Monitoring network access
- Base LAN access
- Unapproved DNS
- Unnecessary outbound services

Temporary testing exceptions can be created above the protection rules when a controlled exercise requires access to another lab system.

---

# Protected Network Alias

A pfSense alias is used to simplify containment rules.

The protected network alias contains:

    192.168.1.0/24
    192.168.10.0/24
    192.168.20.0/24
    192.168.30.0/24
    192.168.40.0/24
    192.168.99.0/24

This allows a single firewall rule to block access to multiple trusted networks.

---

# Default DNS

Only approved DNS resolvers are permitted for selected systems.

Approved resolvers:

    8.8.8.8
    1.1.1.1

pfSense aliases are used to simplify these rules.

After the approved DNS rule, a broader DNS block prevents systems from bypassing the approved resolvers.

---

# Wazuh Communication

The Wazuh server is located at:

    192.168.40.40

Endpoints are permitted to communicate with Wazuh using specific firewall rules.

Example:

    DC01
       |
       | TCP 1514
       v
    Wazuh Server

Return traffic is automatically permitted through pfSense state tracking.

---

# Administrative Access

Administrative access is centralized through the Admin JumpBox.

Examples include:

    JumpBox -> pfSense WebGUI
    JumpBox -> DC01 RDP
    JumpBox -> Wazuh Dashboard
    JumpBox -> T-Pot GUI

This reduces the need to expose management services broadly across the lab.

---

# Honeypot Isolation

T-Pot is located on VLAN30.

The honeypot is allowed only the services required for operation and monitoring.

Traffic toward trusted networks is blocked unless explicitly required.

---

# Vulnerable Target Isolation

Metasploitable 2 is not routed through pfSense.

It is attached to the isolated Proxmox bridge:

    vmbr2
    10.10.10.0/24

Kali Linux has a second virtual NIC attached to this network.

Conceptually:

    Kali
      |
      | vmbr2
      |
    Metasploitable 2

There is no physical uplink on vmbr2.

This allows controlled vulnerability testing without directly exposing Metasploitable 2 to the home network or primary lab VLANs.

---

# Key Security Improvements

During the firewall review, several issues were identified and corrected:

- Removed broad temporary allow-all rules
- Corrected rules using pfSense interface addresses instead of VM addresses
- Corrected rules placed on the wrong source interface
- Replaced broad subnet permissions with single-host rules
- Restricted DNS to approved resolvers
- Created protected network aliases
- Improved honeypot containment
- Improved attack-network isolation
- Corrected overlapping LAN addressing
- Validated routing and firewall behavior

---

# Validation

Security controls are tested using:

- ICMP connectivity tests
- Service-specific connection tests
- DNS resolution tests
- HTTP / HTTPS tests
- pfSense firewall logs
- Routing-table inspection
- ARP / neighbor inspection
- Proxmox network configuration review

---

# Related Documentation

- [Enterprise Homelab Overview](README.md)
- [Architecture](Architecture.md)
- [Network Design](Network-Design.md)
- [Lessons Learned](Lessons-Learned.md)