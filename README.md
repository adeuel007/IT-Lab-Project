# IT Home Lab: Active Directory Environment

A hands-on enterprise network simulation built with Windows Server 2022 (Evaluation) and Windows 11 in Oracle VirtualBox. The lab stands up a domain controller and a domain-joined client to practice deploying, managing, and troubleshooting a typical Active Directory environment.

**Status:** In Progress *(Last Update: 10/23/2026)*

**Skills demonstrated:** Active Directory deployment, DNS configuration and troubleshooting, OU and group design, multi-homed network setup, VM stability diagnosis, snapshot-based change management.

## Lab Environment

| Component | Details |
|---|---|
| Hypervisor | Oracle VirtualBox 7.2.20 |
| Server | Windows Server 2022 Evaluation (VM) |
| Client | Windows 11 (VM) |
| Network | Dual network: internet-facing and isolated domain network |
| Host CPU | AMD Ryzen 7 5700X (8 cores) |
| Host RAM | 32 GB (2x16 GB Corsair Vengeance) |
| LAB-DC01 (Server VM) | 8 GB RAM, 4 cores |
| LAB-WIN11 (Client VM) | 6 GB RAM, 4 cores |

                   ┌───────────────────────────────┐
                   │       PHYSICAL HOST           │
                   │       Windows 11 Home         │
                   │       32 GB RAM               │
                   │       VirtualBox 7.2          │
                   └──────────────┬────────────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
              NAT NETWORK                HOST-ONLY #2
             Internet Access              Isolated Lab
              10.0.2.0/24                192.168.10.0/24
                    │                          │
          ┌─────────┴────────┐         ┌───────┴────────┐
          │                  │         │               │
    ┌─────▼──────┐     ┌─────▼──────┐  │               │
    │ VirtualBox │     │ LAB-DC01   │  │               │
    │ NAT        │     │ Adapter 1  │  │               │
    │ Gateway    │     │ 10.0.2.15  │  │               │
    │ 10.0.2.2   │     └────────────┘  │               │
    └────────────┘                     │               │
                                       │               │
                              ┌────────▼──────┐ ┌──────▼────────┐
                              │   LAB-DC01    │ │  LAB-WIN11    │
                              │ Windows Server│ │ Windows 11 Pro│
                              │     2022      │ │   Client VM   │
                              │               │ │               │
                              │ 192.168.10.10 │ │ 192.168.10.20 │
                              │ Static IP     │ │ Static IP     │
                              │               │ │               │
                              │ AD DS + DNS   │ │ Domain Join   │
                              │ lab.local     │ │  Successful!  │
                              └───────────────┘ └───────────────┘

## Walkthrough

Each step is documented in its own file. Read them in order.

1. [Planning](01-planning.md): domain controller VM settings and configuration
2. [Windows Server 2022 Install](02-WindowsServer22-Install.md): GUI install, admin accounts, Guest Additions
3. [Network Configuration](03-Network-Configuration.md): dual network architecture
4. [Active Directory Deployment](04-Active-Directory-Deployment.md): AD DS roles and domain controller promotion
5. [DNS Configuration](05-DNS-Configuration.md): configuration, validation, and troubleshooting
6. [Stability Investigation](06-Stability-Investigation.md): VM freezes, service misconfigurations, resource adjustments
7. [OU and Group Design](07-OU-And-Group-Design.md): Organizational Units and groups
8. [Client Endpoint](08-Client-Endpoint.md): Windows 11 client configuration

Also in this repo: [Lessons Learned Log](Lessons-Learned-Log.md) | [Users Created](Users_Created.md)

## Key Lessons Learned

- Multi-homed domain controllers need careful DNS registration.
- Not every Event Viewer error is an active problem. Context matters.
- `dcdiag` is a starting point for an investigation, not the final answer.
- VM instability can cascade into misleading service warnings.
- A clean snapshot strategy saves an enormous amount of time while experimenting.
- Separating internet-facing and domain networks closely mirrors enterprise design.

See the full [Lessons Learned Log](Lessons-Learned-Log.md) for details.
