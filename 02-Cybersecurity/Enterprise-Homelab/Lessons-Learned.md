# 🧠 Lessons Learned

## Overview

Building this homelab exposed several networking and firewall-design mistakes that were not obvious from the initial configuration.

The most valuable lessons came from troubleshooting real connectivity problems, validating assumptions, and correcting the design without rebuilding the environment from scratch.

---

# 1. Avoid Overlapping Subnets

## Problem

pfSense WAN received:

    192.168.1.96/24

from the physical home network.

At the same time, the pfSense LAN interface was configured as:

    192.168.1.1/24

This placed both WAN and LAN in:

    192.168.1.0/24

even though they existed on separate virtual Layer 2 networks.

## Why It Was a Problem

Even when two interfaces are connected to separate Layer 2 segments, overlapping Layer 3 subnets create unnecessary routing ambiguity and make troubleshooting more difficult.

The physical home network also used:

    192.168.1.1

as the home router address.

This made the design especially confusing because the same address existed conceptually on both the home network and the isolated pfSense LAN side.

## Investigation

The issue was identified by reviewing:

- pfSense WAN and LAN interface addresses
- Proxmox Linux bridge configuration
- Routing tables
- DHCP configuration
- VM network assignments
- ARP / neighbor information

The Proxmox network design showed:

    vmbr0 = physical/home network
    vmbr1 = isolated VLAN-aware internal lab bridge
    vmbr2 = isolated penetration-testing network

No lab VM was using the untagged pfSense LAN network.

## Solution

The pfSense LAN network was changed from:

    192.168.1.1/24

to:

    192.168.99.1/24

The physical home network remained unchanged.

## Result

The final design clearly separates:

    Home Network      192.168.1.0/24
    Base Lab LAN      192.168.99.0/24
    VLAN 10           192.168.10.0/24
    VLAN 20           192.168.20.0/24
    VLAN 30           192.168.30.0/24
    VLAN 40           192.168.40.0/24
    VLAN 50           192.168.50.0/24

## Lesson

Network addressing should be planned before deployment.

Unique non-overlapping subnets make routing, firewall rules, documentation, and troubleshooting significantly easier.

---

# 2. Understand pfSense "Address" vs "Subnet"

## Problem

Several firewall rules were originally created using options such as:

    VLAN20ADMIN address
    VLAN40MONITORING address

under the assumption that those values represented the VM on that VLAN.

They do not.

## What They Actually Mean

For VLAN40:

    VLAN40MONITORING address
    = pfSense interface address
    = 192.168.40.1

while:

    VLAN40MONITORING subnet
    = entire VLAN40 network
    = 192.168.40.0/24

and:

    Wazuh Server
    = 192.168.40.40

The same concept applies to every pfSense interface.

## Impact

Firewall rules appeared correct visually but did not match traffic from the intended VM.

For example, rules intended for Wazuh were actually matching traffic originating from:

    192.168.40.1

instead of:

    192.168.40.40

As a result, disabling the broad allow-all rule caused Wazuh to lose network connectivity.

## Solution

Rules intended for a specific VM were changed to:

    Source: Single host or alias

using the VM's actual IP address.

## Lesson

Always determine whether a rule should apply to:

- pfSense itself
- one specific host
- an entire subnet

before selecting the source or destination.

---

# 3. Firewall Rules Belong on the Source Interface

## Problem

Some inter-VLAN rules were created on the destination VLAN instead of the source VLAN.

Example:

    JumpBox -> T-Pot

was originally configured under VLAN30.

However, the traffic originates from VLAN20.

## Correct Model

pfSense evaluates traffic when it enters the firewall.

Therefore:

    JumpBox
       |
       | VLAN20
       v
    pfSense
       |
       v
    T-Pot VLAN30

The rule belongs on:

    Firewall -> Rules -> VLAN20

## Additional Example

A rule allowing:

    VLAN40 -> VLAN20 ICMP

should be placed under VLAN40, not VLAN20.

## Lesson

For normal routed traffic in pfSense:

> Put the rule on the interface where the traffic originates.

This mental model makes inter-VLAN firewall design much easier to understand.

---

# 4. Rule Order Matters

## Problem

Some restrictive rules were placed below broader allow rules.

Because pfSense evaluates rules from top to bottom and stops at the first match, lower rules never had a chance to apply.

Example:

    PASS VLAN40 -> ANY
    BLOCK VLAN40 -> ANY

The block rule was ineffective because the allow rule already matched all IPv4 traffic.

## Another Example

A rule such as:

    PASS T-Pot -> ANY TCP 443

placed above:

    BLOCK T-Pot -> Internal_Network

would allow T-Pot to reach internal HTTPS services before the block rule was evaluated.

## Solution

Rules were reordered using the following pattern:

    Specific allows
    Specific blocks
    Limited outbound access
    Default deny

## Lesson

Firewall rules are not just a list of permissions.

They are an ordered decision process.

---

# 5. Broad Temporary Rules Are Useful but Dangerous

## Problem

During initial setup, VLAN20 and VLAN40 used broad temporary rules such as:

    Allow All Temporarily
    Wazuh network -> ANY

These were useful for testing connectivity but remained broader than necessary.

## Risk

A broad rule can hide configuration mistakes because everything appears to work even when more specific rules are incorrect.

This happened with VLAN40.

Once the broad Wazuh rule was disabled, the incorrect source configuration became visible.

## Solution

Broad rules were replaced with specific rules for:

- DNS
- NTP
- HTTP
- HTTPS
- ICMP
- Wazuh communication
- administrative access

Temporary rules were disabled only after the replacement rules were tested.

## Lesson

Temporary allow-all rules are useful during troubleshooting, but they should be:

- clearly labeled
- temporary
- disabled before finalizing the design

---

# 6. Use Single-Host Rules When Possible

## Problem

Several early rules used whole VLAN subnets as sources.

Example:

    VLAN40 subnet

even though only the Wazuh server needed access.

## Improvement

Rules were narrowed to the actual host:

    192.168.40.40

This means future systems added to VLAN40 do not automatically inherit the same permissions.

The same approach was used for:

- Admin JumpBox
- T-Pot
- Kali Linux
- DC01
- Wazuh

## Lesson

A `/24` subnet rule may authorize up to 254 usable hosts.

A `/32` or single-host rule authorizes exactly one IPv4 address.

Least privilege applies to network rules too.

---

# 7. CIDR Prefixes Represent Binary Network Bits

While creating aliases and network rules, CIDR notation became more important.

IPv4 addresses contain 32 bits.

For example:

    /24

means:

    24 network bits
    8 host bits

which gives:

    2^8 = 256 total addresses

A single IPv4 host can be represented as:

    /32

Examples:

    8.8.8.8/32
    1.1.1.1/32

This means only those exact hosts are included.

## Lesson

CIDR notation is not arbitrary syntax.

It directly represents how many binary bits identify the network.

---

# 8. Use DNS Aliases Instead of Repeating Rules

## Problem

Separate rules were originally created for:

    8.8.8.8
    1.1.1.1

## Improvement

A pfSense alias can group approved DNS resolvers.

Example:

    Default_DNS

containing:

    8.8.8.8
    1.1.1.1

Firewall rules can then reference the alias rather than duplicating configuration.

## Lesson

Aliases improve:

- readability
- maintainability
- consistency
- future updates

---

# 9. Restrict DNS to Prevent Bypass

## Design

Systems are allowed to query approved DNS servers first.

A broader rule then blocks DNS traffic to all other destinations.

Example:

    PASS  Host -> Default_DNS TCP/UDP 53
    BLOCK Host -> ANY          TCP/UDP 53

## Benefit

This prevents a system from bypassing the intended DNS policy by manually configuring another resolver.

## Lesson

A security policy is stronger when the approved path is explicitly allowed and alternate paths are explicitly blocked.

---

# 10. NTP Is Important for Security Monitoring

UDP port 123 is used by NTP.

Accurate time is especially important for:

- Wazuh alerts
- Windows event logs
- Linux logs
- firewall logs
- authentication
- certificates
- incident timelines

During troubleshooting, the Wazuh server showed a time difference caused by timezone configuration rather than failed NTP synchronization.

## Lesson

Clock synchronization and timezone configuration are separate concepts.

A system can be correctly synchronized while displaying local time in the wrong timezone.

---

# 11. Virtual Bridges Are Virtual Switches

The Proxmox network became much easier to understand after separating the concepts of NICs and bridges.

## Physical NIC

    enp0s31f6

This is the Dell OptiPlex physical Ethernet interface.

## Proxmox Bridges

    vmbr0
    vmbr1
    vmbr2

These behave like virtual Ethernet switches.

## pfSense Virtual NICs

    vtnet0
    em0

These are virtual network interfaces attached to Proxmox bridges.

## Relationship

    Physical NIC
        |
      vmbr0
        |
    pfSense WAN

and:

    pfSense em0
        |
      vmbr1
        |
      VLANs

## Lesson

A Linux bridge is not a NIC.

It connects interfaces together similarly to a physical network switch.

---

# 12. vmbr2 Provides True Test-Network Isolation

Metasploitable 2 is connected only to:

    vmbr2
    10.10.10.0/24

The bridge has:

- no physical uplink
- no upstream gateway

Kali Linux has a second virtual NIC on the same bridge.

This allows:

    Kali <-> Metasploitable 2

without directly connecting Metasploitable to the home network or routed VLAN environment.

## Lesson

Network isolation should be implemented architecturally, not only with firewall rules.

Removing the physical/routed path entirely provides a stronger containment boundary.

---

# 13. Validate Security Controls Instead of Assuming They Work

Throughout the lab, assumptions were tested using:

    ip addr
    ip route
    ip neigh
    ping
    curl
    ss
    resolvectl
    timedatectl

as well as:

- pfSense firewall logs
- Proxmox network configuration
- interface assignments
- DHCP leases
- service-specific tests

## Lesson

A configuration being present does not prove the control works.

Security controls should be validated using both positive and negative testing.

Examples:

    Expected allowed traffic -> verify success
    Expected blocked traffic -> verify failure

---

# Key Takeaway

The most important lesson from this phase of the lab was that building the network was only the first step.

The more valuable work was:

    Build
      |
      v
    Test
      |
      v
    Identify mistakes
      |
      v
    Understand root cause
      |
      v
    Correct the design
      |
      v
    Validate again
      |
      v
    Document the result

That process provided more practical networking and cybersecurity experience than simply following an installation guide.