# 08 - Client Endpoint

## Objective

Stand up a Windows 11 VM client and join it to the `lab.local` domain.

---

## Method

### 1. Obtain Windows 11 Installation Media

- Downloaded a fresh Windows 11 ISO directly from Microsoft.
- Verified the integrity of the ISO using PowerShell and `Get-FileHash`.
- Compared the generated hash against the hash value provided by Microsoft.
- **Result:** Hashes matched. ISO integrity verified.

Example command:

```powershell
Get-FileHash "C:\Path\To\Windows11.iso"
```

---

### 2. Verify Windows 11 Edition

- Created the Windows 11 VM in VirtualBox.
- Confirmed that the installation media provided the **Windows 11 Pro** edition.
- Windows 11 Pro is required for joining the machine to the `lab.local` Active Directory domain.

---

### 3. Configure Network Adapters

Configured the VM with two network adapters:

| Adapter | Connection Type | Purpose |
|---|---|---|
| Adapter 1 | NAT | Internet access |
| Adapter 2 | Host-Only | Communication with the lab network / domain controller |

The network configuration is designed to separate Internet access from domain and lab traffic.

---

### 4. Configure Windows 11

Completed the initial Windows 11 setup process, including:

- Language configuration
- Regional preferences
- Initial Windows preferences
- Default local user account creation

---

### 5. Update Windows

- Installed available Windows updates.
- Confirmed the client was functioning normally following the update process.

---

### 6. Create Clean Snapshot

After completing the initial Windows 11 configuration:

- Created a clean VM snapshot.
- This snapshot served as a known-good baseline before making domain-related configuration changes.

---

### 7. Configure Domain Network Connectivity

Configured the Windows 11 client's lab network adapter for communication with the domain controller.

The client was configured with:

- Lab network IP address: `192.168.10.20`
- Subnet mask: `255.255.255.0`
- DNS server: `192.168.10.10` (`LAB-DC01`)

DNS configuration was particularly important because Active Directory relies heavily on DNS for locating domain services.

---

### 8. Verify DNS and Domain Controller Connectivity

Before attempting the domain join, verified that the client could communicate with the domain controller and resolve the `lab.local` domain.

Testing confirmed:

- `LAB-DC01` was reachable from the Windows 11 client.
- DNS queries directed to the domain controller resolved correctly.
- The `lab.local` domain could be located.
- The client was able to identify the domain controller as an authoritative DNS source.

These checks established that the primary prerequisites for joining the domain were functioning correctly.

---

### 9. Join Windows 11 to the Domain

The Windows 11 client was successfully joined to the:

```text
lab.local
```

The machine was restarted following the domain join to complete the process.

**Result:** ✅ Domain join successful.

---

### 10. Verify Domain Authentication

Following the domain join, authentication was tested using domain accounts.

Successfully authenticated with:

```text
LAB\ynot.admin
```

and:

```text
LAB\tony.employee
```

Both accounts were able to log into the Windows 11 endpoint successfully.

**Result:** ✅ Domain authentication successful.

This confirmed that the endpoint could successfully communicate with Active Directory and authenticate domain users.

---

### 11. Create Post-Domain-Join Snapshot

After successfully joining the endpoint to the domain and verifying authentication:

- Created a new VM snapshot.
- This snapshot represents the current known-good state of the Windows 11 domain client.

This provides a recovery point before beginning additional endpoint configuration, Group Policy testing, security hardening, and other lab exercises.

---

## Current Status

**Status:** 🟢 Complete

The Windows 11 client has been successfully:

- Installed
- Configured
- Updated
- Networked
- Connected to the lab DNS infrastructure
- Joined to the `lab.local` Active Directory domain
- Tested with multiple domain accounts
- Captured in a post-domain-join snapshot

### Current Lab Topology

```text
                    Internet
                       │
                       │
                    NAT
                       │
                ┌─────────────┐
                │  LAB-DC01   │
                │             │
                │ Windows     │
                │ Server 2022 │
                └──────┬──────┘
                       │
                192.168.10.0/24
                       │
                ┌──────┴──────┐
                │  LAB-WIN11  │
                │             │
                │ Windows 11  │
                │     Pro     │
                └─────────────┘
```

---

## Next Steps

With the first Windows client successfully established as a domain member, the next phase of the lab can focus on endpoint management and security.

Potential next steps include:

- Configure and test Group Policy.
- Establish a Windows workstation security baseline.
- Configure Windows Firewall policies.
- Configure auditing and Windows Event Logging.
- Test domain user/group permissions.
- Practice administrative vs. standard-user workflows.
- Generate controlled security events for investigation.
- Begin collecting and analyzing endpoint logs as part of SOC practice.

---

## Milestone

> **First domain-joined Windows client successfully established.**

This represents a major milestone for the lab. The environment now contains both an Active Directory domain controller and a functioning Windows 11 domain endpoint, providing a foundation for more realistic enterprise administration and cybersecurity exercises.
