# Phase 3: Network Configuration and Troubleshooting

## Objective

Design and configure a dual-network architecture for the Active Directory lab, separating Internet connectivity from private domain communications.

The design provides Internet access for updates and external resources while maintaining a dedicated network for communication between lab machines.

---

## 1. Network Architecture

Implemented a dual-network configuration using two VirtualBox network adapters.

### Adapter 1: Internet Connectivity

**Purpose:** Provide Internet access to the virtual machine.

**Configuration:**
- **Connection Type:** NAT

**Provides:**
- Windows Updates
- General Internet connectivity
- External DNS lookups

NAT allows the virtual machine to access external networks through the host's network connection without requiring a separate physical network interface.

### Adapter 2: Private Active Directory Network

**Purpose:** Provide isolated communication between the domain controller and future lab machines.

**Configuration:**
- **Connection Type:** Host-Only Adapter
- **IPv4 Address:** `192.168.10.10`
- **Subnet Mask:** `255.255.255.0` (`/24`)
- **Default Gateway:** None

The host-only network is intentionally isolated from the external network. It serves as the dedicated communication path for Active Directory, DNS, and future lab services.

All future lab machines requiring domain connectivity will use this network.

---

## 2. Network Addressing

### Domain Controller: `LAB-DC01`

| Setting | Configuration |
|---|---|
| Internet Adapter | NAT |
| NAT IPv4 Address | `10.0.2.15` |
| Lab Adapter | Host-Only |
| Lab IPv4 Address | `192.168.10.10/24` |
| Lab Default Gateway | None |
| Lab DNS Service | `192.168.10.10` |

The domain controller uses its private lab interface for Active Directory network communication and DNS services.

**Design consideration:** The lab interface intentionally has no default gateway. The NAT adapter provides the route to external networks.

---

## 3. Network Troubleshooting

### Initial Problems

During the initial network configuration, several connectivity issues were encountered:

- Automatic Private IP Addressing (APIPA) addresses in the `169.254.x.x` range.
- Failed ping attempts between the host and virtual machines.
- DNS registration and name-resolution issues.
- Inconsistent network connectivity during initial configuration.

An APIPA address indicated that an adapter had not obtained a usable IPv4 address through its expected configuration method.

### Troubleshooting Performed

The following steps were taken to diagnose and resolve the issues:

1. Verified the VirtualBox Host-Only Adapter configuration.
2. Created a dedicated host-only network for the lab.
3. Assigned static IPv4 addressing to the domain controller's lab interface.
4. Inspected interface configuration and addressing.
5. Tested network connectivity and neighbor discovery.
6. Investigated DNS configuration and name resolution.

### Diagnostic Commands

The following Windows networking tools were used:

| Command | Purpose |
|---|---|
| `ipconfig` | Inspect IP addressing, subnet masks, gateways, and DNS configuration |
| `ping` | Test IP connectivity to another host |
| `arp -a` | Inspect the local ARP cache and discovered IPv4 neighbors |
| `Test-NetConnection` | Test network reachability and, when specified, TCP connectivity to a target port |

Example commands:

```powershell
ipconfig /all
ping 192.168.10.10
arp -a
Test-NetConnection 192.168.10.10
```

These tools helped distinguish between addressing problems, connectivity failures, and DNS-related issues.

---

## 4. Final Results

The network troubleshooting process eventually established the required connectivity.

| Test | Result |
|---|---|
| Host-only network configuration | Successful |
| Static IPv4 addressing | Configured |
| Host-only communication | Verified |
| Internet connectivity | Verified |
| External DNS resolution | Verified |
| Active Directory DNS resolution | Verified |

The network now supports communication between `LAB-DC01` and the Windows 11 client, `LAB-WIN11`.

The client was subsequently configured to use `192.168.10.10` as its DNS server on the lab network and successfully joined to the `lab.local` domain.

---

## 5. Final Network Topology

```text
                  External Network
                         |
                  Host Ethernet
                         |
                 VirtualBox NAT
                         |
                +------------------+
                |     LAB-DC01     |
                | Windows Server   |
                |      2022        |
                |                  |
                | NAT: 10.0.2.15   |
                |                  |
                | LAB: 192.168.10.10
                +---------+--------+
                          |
                  Host-Only Network
                  192.168.10.0/24
                          |
                +---------+--------+
                |     LAB-WIN11    |
                |   Windows 11 Pro |
                |                  |
                | LAB: 192.168.10.20
                | DNS: 192.168.10.10
                +------------------+
```

---

## 6. Lessons Learned

- **Separate network roles deliberately.** NAT and host-only networking serve different purposes and should be configured according to the intended traffic flow.
- **APIPA addresses are diagnostic clues.** An address in the `169.254.x.x` range can indicate that automatic address assignment failed or that the adapter lacks a usable static configuration.
- **DNS is foundational to Active Directory.** A machine can have IP connectivity and still fail to locate domain services when DNS is misconfigured.
- **Use multiple diagnostic tools.** `ipconfig`, `ping`, `arp -a`, and `Test-NetConnection` provide different pieces of evidence. No single command proves that every layer of connectivity is working.
- **A dedicated lab subnet improves control.** The `192.168.10.0/24` network provides a predictable addressing scheme for future domain clients and servers.
- **A missing default gateway can be intentional.** The lab adapter does not need its own gateway when the NAT adapter handles external connectivity.

---

## Phase 3 Status

**Status:** Complete

The dual-network architecture is operational. The domain controller has Internet access through NAT and private lab connectivity through the host-only network.

This network foundation supports the established `lab.local` domain and the successfully joined Windows 11 endpoint. It will also support future servers, Group Policy testing, security monitoring, and defensive cybersecurity exercises.
