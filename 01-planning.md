# Active Directory Lab

## Project Overview

Build a realistic enterprise Active Directory lab using Oracle VirtualBox and Windows Server 2022. The environment will eventually support Windows client management, Group Policy, security monitoring, and hands-on cybersecurity exercises.

## Current Project Status

**Core infrastructure established**

The lab currently has a functioning Active Directory domain controller, a Windows 11 Pro client, configured networking, and a successful domain join.

| Component                                | Status                   |
| ---------------------------------------- | ------------------------ |
| VirtualBox environment                   | Complete                 |
| Windows Server 2022 installation         | Complete                 |
| Active Directory Domain Services (AD DS) | Configured               |
| DNS                                      | Configured and tested    |
| Domain controller health checks          | Passed                   |
| Lab network configuration                | Complete                 |
| Windows 11 Pro client                    | Installed and configured |
| Domain join                              | Successful               |
| Domain account authentication            | Verified                 |
| Recovery snapshots                       | Created                  |
| Group Policy configuration               | Pending                  |
| Endpoint security baseline               | Pending                  |
| Security monitoring and log analysis     | Pending                  |

---

## 1. Host Machine

| Component        | Configuration            |
| ---------------- | ------------------------ |
| **Hypervisor**   | Oracle VirtualBox 7.2.20 |
| **Host RAM**     | 32 GB                    |
| **Logical CPUs** | 16                       |
| **Host Network** | Ethernet                 |

---

## 2. Virtual Machines

### Domain Controller: `LAB-DC01`

| Component               | Configuration                                      |
| ----------------------- | -------------------------------------------------- |
| **Operating System**    | Windows Server 2022 Evaluation, Desktop Experience |
| **RAM**                 | 8 GB                                               |
| **CPUs**                | 4                                                  |
| **Virtual Disk**        | 80 GB VDI                                          |
| **Disk Controller**     | SATA                                               |
| **Graphics Controller** | VBoxSVGA                                           |
| **Video Memory**        | 128 MB                                             |
| **3D Acceleration**     | Disabled                                           |
| **Domain**              | `lab.local`                                        |
| **Installed Roles**     | Active Directory Domain Services, DNS              |

#### Network Configuration

| Adapter   | Connection Type | IPv4 Address       | Purpose                |
| --------- | --------------- | ------------------ | ---------------------- |
| Adapter 1 | NAT             | `10.0.2.15`        | Internet connectivity  |
| Adapter 2 | Host-Only       | `192.168.10.10/24` | Domain and lab network |

The host-only adapter provides the dedicated network path for communication between lab machines.

### Windows Client: `LAB-WIN11`

| Component             | Configuration              |
| --------------------- | -------------------------- |
| **Operating System**  | Windows 11 Pro             |
| **RAM**               | 6 GB                       |
| **CPUs**              | 4                          |
| **Virtual Disk**      | 80 GB                      |
| **Domain Membership** | `lab.local`                |
| **Status**            | Successfully domain-joined |

#### Network Configuration

| Adapter   | Connection Type | IPv4 Address       | DNS Server             | Purpose                |
| --------- | --------------- | ------------------ | ---------------------- | ---------------------- |
| Adapter 1 | NAT             | DHCP-assigned      | NAT-side configuration | Internet connectivity  |
| Adapter 2 | Host-Only       | `192.168.10.20/24` | `192.168.10.10`        | Domain and lab network |

The client uses `LAB-DC01` as its DNS server on the lab network. This allows Windows to locate Active Directory domain services through DNS.

---

## 3. Active Directory Configuration

| Setting               | Value           |
| --------------------- | --------------- |
| **Domain Name**       | `lab.local`     |
| **Domain Controller** | `LAB-DC01`      |
| **DNS Server**        | `192.168.10.10` |
| **Domain Client**     | `LAB-WIN11`     |

Active Directory Domain Services and DNS have been installed and configured on `LAB-DC01`.

The domain controller has passed the following validation checks:

```powershell
dcdiag /q
dcdiag /test:dns /q
nltest /dsgetdc:lab.local
nslookup lab.local
```

These checks confirmed that the domain controller's core diagnostics, DNS diagnostics, domain controller discovery, and domain name resolution were functioning as expected at the time of testing.

---

## 4. Windows Client Domain Join

The Windows 11 Pro endpoint was prepared and successfully joined to `lab.local`.

### Preparation

- Downloaded a fresh Windows 11 ISO from Microsoft.
- Verified the ISO hash using PowerShell's `Get-FileHash` and compared it against Microsoft's published hash.
- Confirmed the Windows 11 Pro edition during installation.
- Configured NAT and host-only network adapters.
- Completed initial Windows setup and installed updates.
- Created a clean baseline snapshot.

### Connectivity Validation

Before joining the domain, the following were verified:

- The client could communicate with `LAB-DC01`.
- DNS queries directed to `192.168.10.10` resolved correctly.
- The `lab.local` domain could be located.
- Domain controller discovery succeeded.

### Authentication Validation

After joining the domain, both accounts successfully authenticated to the client:

- `LAB\ynot.admin`
- `LAB\tony.employee`

**Result:** The domain join and subsequent domain-account logins were successful.

A recovery snapshot named `Domain Joined - Baseline` was created after validation.

---

## 5. Network Topology

```text
                      Internet
                         |
                   VirtualBox NAT
                         |
                +------------------+
                |     LAB-DC01     |
                | Windows Server   |
                |      2022        |
                |                  |
                | NAT: 10.0.2.15   |
                | LAN: 192.168.10.10
                +---------+--------+
                          |
                  Host-Only Network
                  192.168.10.0/24
                          |
                +---------+--------+
                |     LAB-WIN11    |
                |   Windows 11 Pro |
                |                  |
                | LAN: 192.168.10.20
                | DNS: 192.168.10.10
                +------------------+
```

The intended design separates Internet access from domain traffic. The host-only network provides connectivity between the domain controller and the client without relying on the NAT network for Active Directory communication.

---

## 6. Stability and Recovery

During development, `LAB-DC01` experienced recurring freezes and unresponsive shutdowns. Reverting to a previous snapshot restored a stable state, after which the domain health checks passed and the client domain join succeeded.

Windows Update was suspected of contributing to the instability, but the exact root cause has not been conclusively established.

Recovery snapshots have been used to preserve known-good configurations and reduce the risk of losing progress during troubleshooting.

---

## 7. Next Phase: Endpoint Management and Security

With the domain controller and first domain-joined workstation operational, the next stage is to build out enterprise administration and defensive security capabilities.

### Planned Tasks

- [ ] Verify the `LAB-WIN11` computer object and its intended Active Directory OU.
- [ ] Configure and test Group Policy Objects (GPOs).
- [ ] Establish a Windows 11 security baseline.
- [ ] Configure Windows Defender Firewall policies.
- [ ] Configure Windows auditing and event logging.
- [ ] Test standard-user versus administrative permissions.
- [ ] Generate controlled security events.
- [ ] Practice investigating Windows logs from a defensive security perspective.
- [ ] Explore centralized log collection and SIEM workflows.

---

## Project Milestone

The lab has progressed beyond basic VM preparation. It now contains a working Active Directory domain and a Windows 11 endpoint that can authenticate domain users.

This establishes the foundation for the next objective: **managing, securing, monitoring, and investigating a domain-connected Windows workstation in a controlled enterprise-style environment.**
