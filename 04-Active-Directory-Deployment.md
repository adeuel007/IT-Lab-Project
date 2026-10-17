
# Project Title

A brief description of what this project does and who it's for

# Phase 4: Active Directory Installation

## Objective

Install and configure Active Directory Domain Services (AD DS) and DNS on `LAB-DC01`, then promote the server to a Domain Controller and establish the `lab.local` domain.

---

## 1. Installed Server Roles

Installed the following Windows Server roles:

| Role | Purpose |
|---|---|
| **Active Directory Domain Services (AD DS)** | Provides centralized identity management, domain authentication, and directory services. |
| **DNS Server** | Resolves hostnames and provides the DNS records required for Active Directory domain discovery and operation. |

These roles establish the core identity and name-resolution infrastructure for the lab.

---

## 2. Domain Controller Promotion

- Promoted `LAB-DC01` to a Domain Controller.
- Created a new Active Directory forest.
- Established the domain name `lab.local`.
- Configured DNS as part of the domain infrastructure.
- Completed the promotion process and restarted the server.

### Domain Configuration

| Setting | Value |
|---|---|
| **Server Name** | `LAB-DC01` |
| **Domain Name** | `lab.local` |
| **Forest** | `lab.local` |
| **Directory Service** | Active Directory Domain Services |
| **DNS Service** | Installed and configured |

---

## 3. Promotion Warnings

During the promotion process, two warnings were observed.

### DNS Delegation Warning

The promotion wizard reported a DNS delegation warning.

**Assessment:** Expected in this lab environment.

The `lab.local` domain was created as a new forest, and no parent DNS zone was configured to delegate authority to it. A parent-zone delegation is not required for the isolated lab domain to function.

### Static IP Address Warning

The promotion wizard reported a warning concerning the server's network addressing, specifically the NAT adapter.

**Assessment:** Understood in the context of the lab's dual-network design.

The domain controller uses two network adapters:

- **NAT:** Provides Internet connectivity.
- **Host-Only:** Provides private domain-network connectivity using the static address `192.168.10.10/24`.

The lab interface has a static IPv4 address and no default gateway. The NAT interface provides external connectivity.

Because Domain Controllers depend heavily on stable network configuration and correct DNS registration, the interfaces must be configured carefully to avoid registering unintended addresses in DNS.

---

## 4. Validation and Health Checks

After promotion, the domain controller was tested using Windows diagnostic tools.

```powershell
dcdiag /q
dcdiag /test:dns /q
nltest /dsgetdc:lab.local
nslookup lab.local
```

### Results

- `dcdiag /q` completed without reporting errors.
- `dcdiag /test:dns /q` completed without reporting errors.
- `nltest /dsgetdc:lab.local` successfully located the domain controller.
- `nslookup lab.local` successfully resolved the domain name.

These checks provided evidence that the core domain controller, DNS, and domain discovery functions were operating correctly at the time of testing.

---

## 5. Phase 4 Status

**Status:** Complete

The server was successfully promoted to a Domain Controller, and the `lab.local` Active Directory domain was established.

The DNS delegation and network configuration warnings were evaluated in the context of the lab architecture rather than treated as automatic indicators of failure.

Subsequent validation confirmed that the domain controller and DNS services were functioning correctly.

---

## Lessons Learned

- **Domain Controller promotion is only the beginning.** Successful installation must be followed by health checks and connectivity validation.
- **Warnings require context.** A DNS delegation warning in a newly created forest does not automatically indicate a broken DNS configuration.
- **Multi-homed Domain Controllers require careful DNS configuration.** Multiple network adapters can introduce unintended DNS registrations if interface settings are not managed properly.
- **Validate services after major changes.** Tools such as `dcdiag`, `nltest`, and `nslookup` help verify different aspects of Active Directory health and name resolution.

---

## Milestone

**The `lab.local` Active Directory domain is established, with `LAB-DC01` operating as its Domain Controller.**

This created the foundation for domain account management, organizational units, security groups, Group Policy, and the successful addition of the `LAB-WIN11` domain client.
