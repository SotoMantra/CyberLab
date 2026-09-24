# Cybersecurity & Technology Homelab

A hands-on technical lab for developing practical experience in cybersecurity, systems administration, networking, Linux, virtualization, containers, Python, and local AI infrastructure.

This repository documents the architecture, implementation, troubleshooting, experiments, and lessons learned while building and maintaining my lab environment.

---

## 🧪 Lab Environment

My homelab is designed to simulate real-world infrastructure while providing an environment for security testing, monitoring, troubleshooting, and experimentation.

### Core Technologies

- Proxmox
- pfSense
- Windows Server
- Active Directory
- Linux
- Wazuh
- T-Pot
- Kali Linux
- Docker
- Tailscale
- WireGuard
- Python
- Git

---

## 🏗️ Current Architecture

The cybersecurity environment uses segmented networks for different systems and security functions.

| Network | Purpose |
|---|---|
| VLAN 10 | Servers |
| VLAN 20 | Administration |
| VLAN 30 | Honeypot |
| VLAN 40 | Security Monitoring |
| VLAN 50 | Attack / Testing |
| VLAN 99 | Management |

Architecture diagrams and detailed network documentation will be added as the environment develops.

---

## 🛡️ Cybersecurity Projects

### Enterprise Cybersecurity Homelab

Virtualized enterprise-style environment built with Proxmox and pfSense featuring network segmentation, Windows Server, Active Directory, Windows and Linux endpoints, and dedicated security systems.

### Wazuh Security Monitoring

Centralized security monitoring environment used to collect and analyze events from Windows and Linux endpoints.

### T-Pot Honeypot

Isolated honeypot environment used to observe attack activity and practice security monitoring and analysis.

### Active Directory Lab

Windows domain environment used to practice identity management, organizational units, security groups, Group Policy, DNS, permissions, and endpoint administration.

---

## 🖥️ Infrastructure

The lab includes hands-on work with:

- Virtual machines and containers
- Network segmentation
- Firewall policies
- DNS and DHCP
- VPN connectivity
- Remote administration
- Linux server administration
- Docker deployments

---

## 🐧 Linux

Linux is used extensively throughout the environment for administration, security tooling, containers, troubleshooting, and daily workstation use.

Current environments include:

- CachyOS / Arch Linux
- Ubuntu
- Kali Linux
- Linux servers and containers

---

## 🤖 Local AI

Separate local AI infrastructure is used to experiment with:

- Local large language models
- Ollama
- AI agents
- Docker
- GPU acceleration
- Retrieval and knowledge systems
- Automation

---

## 🐍 Programming & Automation

Python and shell scripting projects are used to develop automation and software development skills.

Projects include:

- Password Generator
- Quiz Generator
- Goal Tracker
- Daily Digest
- Future security automation tools

---

## 🔎 Troubleshooting

A major focus of this repository is documenting not only successful implementations but also:

- Errors encountered
- Diagnostic processes
- Commands and tools used
- Root causes
- Solutions
- Verification methods
- Lessons learned

This provides a record of practical troubleshooting experience rather than only completed installations.

---

## 🎯 Current Focus

Current areas of development include:

- Security monitoring
- Detection engineering
- Linux administration
- Networking
- Active Directory
- Docker
- Python
- Cloud security
- Local AI infrastructure

---

## 📈 Repository Status

This repository is actively maintained as the homelab evolves.

Documentation, architecture diagrams, security experiments, troubleshooting records, and projects will continue to be added.