# Lessons Learned

> **Living document:** This section will continue to grow as the lab develops and new problems are encountered.

---

## Multi-Homed Domain Controllers Require Careful DNS Registration Configuration

- I ended up with multiple network adapters pointing to an unintended DNS configuration.
- The original objective was **network segmentation**:
  - One adapter configured for **NAT** to provide Internet access.
  - A second adapter configured for the **isolated lab network** to handle inter-VM/domain communication.
- Multi-homed domain controllers require careful consideration of which interfaces register in DNS and how DNS traffic is handled.
- An incorrect DNS configuration can create confusing symptoms because Active Directory relies heavily on DNS for domain discovery and communication.

**Key takeaway:** A multi-homed domain controller isn't simply a server with two network cables. DNS registration and interface configuration have to be deliberately designed.

---

## Not Every Event Viewer Error Represents an Active Problem

- Event Viewer exists to log **everything that happens on a system**.
- Not every warning or error represents an active problem that needs immediate remediation.
- Context matters. An event needs to be evaluated against:
  - When it occurred.
  - What the system was doing at the time.
  - Whether the behavior was expected.
  - Whether related events indicate an actual failure.
- Some events are simply Windows documenting something that happened.

> Sometimes Event Viewer is just telling you that you restarted the machine, and it took that personally.

**Key takeaway:** Don't treat Event Viewer like a list of fires that all need to be extinguished. Correlate events with system behavior before deciding that something is actually broken.

---

## `dcdiag` Is a Starting Point for Investigation, Not the Final Answer

- `dcdiag` was one of the first major diagnostic tools I was introduced to because this is my first Active Directory domain controller.
- It has become one of the most useful system tools I am actively using.
- `dcdiag` is designed to diagnose the health of **Active Directory Domain Services (AD DS)** and troubleshoot problems involving domain controllers.
- It performs a comprehensive collection of tests covering areas such as:
  - DNS configuration.
  - Active Directory connectivity.
  - Replication.
  - Domain controller services.
  - Security-related configuration.
  - Other critical domain controller functions.
- Passing `dcdiag` does not automatically mean that every aspect of the environment is healthy. It provides diagnostic evidence that needs to be interpreted alongside Event Viewer, DNS testing, network testing, and observed system behavior.

### `dcdiag` Commands Used

```powershell
dcdiag /s:<ServerName>
```

Tests a specific domain controller.

```powershell
dcdiag /q
```

Filters the output to display only errors, making it useful for quickly identifying problems without reading the full diagnostic output.

```powershell
dcdiag /test:DNS
```

Runs the DNS-specific diagnostic tests.

**Key takeaway:** `dcdiag` is an excellent starting point for diagnosing AD problems, but it is one piece of the larger troubleshooting process.

---

## VM Stability Issues Can Cascade Into Misleading Service Warnings

- I initially provisioned **2 vCPUs** for Windows Server 2022 Evaluation Edition.
- As the lab became more complex, I began experiencing progressively worse performance and stability issues.
- I increased the domain controller to **4 vCPUs** as a performance test.
- After increasing the CPU allocation, I observed longer periods of stability and fewer apparent crashes/freezes.
- I will continue monitoring performance rather than assuming that adding more CPU permanently solved the problem.
- This also reinforced the importance of distinguishing between:
  - A service actually failing.
  - A service being stopped because Windows is shutting down.
  - A service becoming unresponsive because the underlying VM is experiencing resource or stability problems.

### Event ID 41

One event I have become very familiar with is **Event ID 41**.

Windows records this event when the system restarts without completing a clean shutdown.

In my case, some of these events were expected consequences of having to forcefully restart or power off the VM after it became unresponsive.

> Windows does not particularly enjoy being restarted when it wasn't expecting it.

**Key takeaway:** A service warning or Event ID 41 can be a symptom of a larger stability problem rather than the original cause.

---

## A Clean Snapshot Strategy Saves Enormous Amounts of Time When Experimenting

- Virtual machine snapshots have repeatedly allowed me to recover from configuration mistakes and unstable system states.
- A snapshot taken **before a major configuration change** provides a known recovery point.
- However, snapshots are not a substitute for proper backups, and a complicated snapshot/differencing-disk chain should be treated carefully.
- I have learned to create intentional checkpoints after reaching known-good states rather than relying on one "latest" snapshot.

**Key takeaway:** Before making a major change, create a known-good checkpoint. Future me will appreciate it.

---

## Separating Internet-Facing and Domain Networks Closely Mirrors Enterprise Designs

- The lab was deliberately designed with separate network paths.
- The domain controller has:
  - A **NAT interface** for Internet connectivity.
  - A **Host-Only interface** for isolated lab communication.
- The Windows client similarly uses the isolated lab network for domain communication while retaining NAT connectivity.
- This provides practical experience with network segmentation and demonstrates why infrastructure systems should not necessarily treat every network interface as equivalent.

**Key takeaway:** Network segmentation isn't just something to read about. Even a small virtual lab can demonstrate why separating trusted/internal traffic from Internet-facing connectivity matters.

---

# Troubleshooting Log

## 2026-08-07: Windows Update Freeze and Snapshot Rollback

### Incident

The domain controller experienced a freeze associated with Windows Update activity.

A forced restart did not resolve the issue, so I rolled the virtual machine back to a previous checkpoint.

### Post-Rollback Verification

After restoring the checkpoint, I verified the state of the domain controller and discovered that the custom **Organizational Units (OUs)** and **security groups** I had previously created were missing.

This led to an important discovery:

> The snapshot had been taken **before** the custom OUs and groups were created.

The AD configuration itself was not necessarily damaged. The snapshot simply represented an earlier point in the lab's development.

---

## Attempting to Locate the Missing OUs

I initially attempted:

```powershell
Get-ADOrganizationalUnit -Filter * | Select-Object Name, DistinguishedName
```

The command was not recognized.

### Checking for the Active Directory Module

I then ran:

```powershell
Get-Module -ListAvailable ActiveDirectory
```

This confirmed that the **ActiveDirectory PowerShell module existed** on the system.

I then attempted to load it:

```powershell
Import-Module ActiveDirectory
```

The expected cmdlets still were not immediately available.

### Locating the Available AD Cmdlets

I used:

```powershell
Get-Command -Module ActiveDirectory |
    Where-Object Name -like "*OrganizationalUnit*"
```

This identified the Organizational Unit-related cmdlets that were available from the module.

I then explicitly invoked the Active Directory command to avoid ambiguity around module loading:

```powershell
ActiveDirectory\Get-ADOrganizationalUnit -Filter * |
    Select-Object Name, DistinguishedName
```

### Result

No custom OUs were populated.

This confirmed that the custom OU structure was not present in the restored snapshot.

---

## Active Directory Health Verification

Rather than immediately assuming that Active Directory itself was damaged, I used `dcdiag` with multiple diagnostic options to verify the state of the domain controller.

The checks were used to evaluate:

- Active Directory health.
- DNS health.
- Replication.
- Windows Time configuration.

The results indicated that the underlying Active Directory infrastructure was healthy.

### Conclusion

The missing OUs and groups were ultimately determined to be a **snapshot timeline issue**, not evidence of an Active Directory failure.

The snapshot had simply been created before those objects were added.

### Recovery Plan

The custom OUs and groups were repopulated manually.

A new snapshot was then created after the configuration reached a known-good state.

**Lesson learned:** Always label snapshots according to the actual state they represent. A snapshot called "working" is much less useful than one called something like:

```text
Phase 2 - AD DS + DNS - Before OU Configuration
```

or:

```text
Phase 3 - OU + Groups Complete
```

---

# Overall Takeaway

The biggest lesson from building this lab so far has been that **troubleshooting is rarely about finding one command that says "broken" or "working."**

The process has required combining:

- System behavior.
- Event logs.
- Active Directory diagnostics.
- DNS testing.
- Network configuration.
- VM resource allocation.
- Snapshot history.
- Configuration state.
- And plain old timeline reconstruction.

The lab has already demonstrated an important distinction between **symptoms and causes**.

A warning doesn't necessarily mean failure.  
A failed command doesn't necessarily mean the underlying service is broken.  
A missing AD object doesn't necessarily mean AD is damaged.  
And an unexpected shutdown event doesn't necessarily tell me *why* the machine became unresponsive.

The goal going forward is to keep building the habit of asking:

> **"What evidence would distinguish these possible causes?"**

rather than simply asking:

> **"What can I change to make this error disappear?"**

8/20-- I'm quickly realizing the "Its Always DNS" meme has merit.

# Windows 11 Domain Client DNS Troubleshooting
**Issue:**
Windows 11 client could communicate with the domain controller by IP address, but `nslookup lab.local` initially returned:
```text
*** Unknown can't find lab.local: Non-existent domain
```
This prevented reliable domain-name resolution and needed to be corrected before attempting the domain join.
## 1. Initial Symptom
The client was configured with:
```text
Client IPv4: 192.168.10.20
Subnet Mask: 255.255.255.0
```
The initial `nslookup` for `lab.local` returned:
```text
Server:  Unknown
Address: 192.168.4.1

*** Unknown can't find lab.local: Non-existent domain
```
### Likely causes
Several possibilities were considered:
* Client configured with an incorrect DNS server.
* Domain controller's DNS service or `lab.local` zone was malfunctioning.
* Client and domain controller could not communicate across the lab network.
* Multiple network adapters were causing Windows to select the wrong DNS server.
* DNS configuration had not actually been applied to the intended network adapter.

Rather than immediately changing multiple settings, connectivity and DNS functionality were tested independently.

---

## 2. Tested Connectivity to the Domain Controller
The client successfully pinged:
```text
192.168.10.10
```
This established that:
* The client could reach `LAB-DC01`.
* The `192.168.10.0/24` lab network was functioning.
* Basic network connectivity was not the problem.

This allowed us to focus on DNS rather than troubleshooting the underlying network connection.

---

## 3. Tested the Domain Controller's DNS Directly
The following command was used:
```cmd
nslookup lab.local 192.168.10.10
```
This explicitly instructed the client to query **LAB-DC01's DNS server**, bypassing whatever DNS server Windows had selected automatically.
The lookup succeeded.
### Conclusion
This was an important isolation point:
**LAB-DC01's DNS service was functioning correctly and could resolve `lab.local`.**
Therefore, the problem was not primarily with the DNS server or the `lab.local` DNS zone.
The fault was on the **client's DNS selection/configuration**.

---

## 4. Inspected the Client's Network Adapters

`ipconfig /all` revealed that the Windows 11 client had **two network adapters**.
### Adapter 1: NAT
```text
IPv4 Address: 10.0.2.15
Default Gateway: 10.0.2.2
DNS Server: 192.168.4.1
```
### Adapter 2: Lab/Host-Only
```text
IPv4 Address: 192.168.10.20
Subnet Mask: 255.255.255.0
DNS Server: 192.168.10.10
```
This exposed the actual problem.
The client had a functional lab adapter configured to use the domain controller for DNS, but the NAT adapter was still advertising another DNS server:
```text
192.168.4.1
```
Windows could therefore select the NAT adapter's DNS configuration when performing normal DNS queries.

---

## 5. Corrected DNS Configuration
The client was configured so that both adapters use the domain controller as their DNS server:
```text
DNS: 192.168.10.10
```
The NAT adapter's **gateway was left unchanged**:
```text
Gateway: 10.0.2.2
```
This distinction is important.
* **Gateway** determines where traffic leaves the local network.
* **DNS server** determines where hostname-to-IP resolution is performed.
The client therefore retained Internet connectivity through the NAT adapter while using `LAB-DC01` for DNS.
The lab adapter retained:
```text
IP: 192.168.10.20
Gateway: blank
DNS: 192.168.10.10
```
A gateway was intentionally not added to the isolated lab adapter because the domain controller is providing DNS, not acting as the client's Internet gateway.

---

## 6. Flushed and Re-registered DNS
The following commands were run:
```cmd
ipconfig /flushdns
ipconfig /registerdns
```
### Why?
`ipconfig /flushdns` clears cached DNS information so that Windows does not continue relying on previous failed or incorrect DNS results.
`ipconfig /registerdns` causes the client to attempt registration of its DNS information with the configured DNS infrastructure. *Note: This needs to be run with elevated privileges* 

This ensured subsequent tests were using the corrected configuration rather than potentially cached information.

---

## 7. Re-tested DNS Resolution
After correcting the configuration, both adapters were able to resolve the domain.
One adapter reported:
```text
Server: Unknown
Address: 192.168.10.10
```
while the lab adapter reported:
```text
Name: lab.local
Address: 192.168.10.10
```
The `Unknown` result for the DNS server name was determined **not to be a DNS failure** because the DNS server's IP address was correctly identified as:
```text
192.168.10.10
```
The difference is likely related to reverse DNS/PTR resolution of the DNS server's own IP address. The important test was whether `lab.local` itself resolved successfully.

---

## 8. Verified Domain Controller Discovery
Finally:
```cmd
nltest /dsgetdc:lab.local
```
successfully returned information about:
```text
LAB-DC01
```
### Why this test matters
`nltest /dsgetdc` goes beyond simply asking DNS:
> "What IP address belongs to `lab.local`?"
It tests whether the client can locate a domain controller for the specified Active Directory domain.
Successful discovery of `LAB-DC01` confirmed that:
* DNS resolution is functioning.
* The client can communicate with the domain controller.
* Active Directory domain controller discovery is functioning.
* `lab.local` is discoverable from the client.
* The client is now in a suitable state to attempt the domain join.

---

# Root Cause

**The primary issue was incorrect DNS selection caused by the client's dual-network-adapter configuration.**
The NAT adapter was receiving:
```text
192.168.4.1
```
as its DNS server, while the lab adapter correctly pointed to:
```text
192.168.10.10
```
Because Windows had multiple active network interfaces, it could use the NAT adapter's DNS configuration for normal queries. That DNS server had no knowledge of the lab's `lab.local` Active Directory domain, resulting in:
```text
Non-existent domain
```
### Resolution
Both client adapters were configured to use the domain controller:
```text
192.168.10.10
```
while the NAT adapter retained its normal Internet gateway:
```text
10.0.2.2
```
Subsequent DNS and domain-controller discovery tests succeeded.
**Current status: DNS and domain-controller discovery are confirmed operational. The client is ready for the domain-join phase.** 🟢



# 10/16/26 - "Life, Finds a Way"

# Home Lab Documentation: LAB-DC01 Stabilization & LAB-WIN11 Domain Join

**Date:** October 16, 2026  
**Environment:** Oracle VirtualBox 7.2.20  
**Domain:** `lab.local`  
**Domain Controller:** `LAB-DC01`  
**Client:** `LAB-WIN11`

---

## 1. Objective

The primary objective for this session was to stabilize the `LAB-DC01` domain controller following repeated freezes and then proceed with joining `LAB-WIN11` to the `lab.local` Active Directory domain.

A secondary objective was to determine whether the previously observed instability was related to recent Windows Updates.

---

# 2. Initial Problem

Prior to this session, `LAB-DC01` had experienced repeated progressive freezes.

### Observed Behavior

- Increasing system latency.
- Windows becoming increasingly unresponsive.
- Delayed or failed response to `Ctrl+Alt+Delete`.
- VirtualBox itself remaining responsive while the guest OS became unusable.
- ACPI shutdowns occasionally becoming stuck at:
  - `Shutting down service: Update Orchestrator Service`
  - `Getting Windows ready. Don't turn off your computer.`
- Forced power-off eventually being required.
- `dcdiag /q` subsequently reporting an unexpected shutdown event.

The host system remained responsive during these incidents, making a host-wide system failure less likely.

VirtualBox had also been upgraded from **7.2.14 to 7.2.20** during the troubleshooting process.

### Domain Controller Configuration

`LAB-DC01` was configured with:

- **8 GB RAM**
- **4 vCPUs**
- **100% execution cap**
- **80 GB VDI**
- **SATA storage**
- **NAT + Host-Only networking**

The increase from 2 to 4 vCPUs appeared to improve the duration of usable operation, but did **not** eliminate the freezing.

---

# 3. Windows Update Investigation

A major clue came from the timing of the instability.

`LAB-DC01` had previously been processing accumulated Windows Updates. During shutdown/reboot operations, the system repeatedly became stuck while interacting with the **Update Orchestrator Service** and subsequently displayed the Windows **"Getting Windows ready"** state.

Because the system was becoming progressively less usable following the update activity, the decision was made to revert `LAB-DC01` to a previously known-good VirtualBox snapshot.

## Result After Reverting the Snapshot

After reverting:

- `LAB-DC01` successfully booted.
- The system remained usable.
- The previous severe latency/freezing was no longer immediately present.
- The server remained stable for approximately **30 minutes during active use**.

This provides **strong circumstantial evidence** that the post-update state was involved in the instability, although it does not conclusively prove that a specific Windows Update was responsible.

> **Assessment:** Suspected Windows Update-related instability. Exact root cause remains unconfirmed.

---

# 4. Server Manager Manageability Alert

During validation, Server Manager displayed the following alert for `LAB-DC01`:

> **Online - Cannot get role and feature data**

The server was displayed using both of its configured addresses:

```text
10.0.2.15
192.168.10.10
```

An attempt was made to investigate the management stack.

One command initially generated:

```text
Get-Service : Cannot find any service with service name 'ServerManager'.
```

This was determined **not to be a server fault**.

`ServerManager` is not a Windows service, so the command was inappropriate for testing Server Manager itself.

The server subsequently became progressively less responsive again during troubleshooting, so further investigation of the Server Manager alert was intentionally deferred.

## Decision

The manageability alert was **not treated as a blocker** because the actual Active Directory, DNS, and domain-controller health checks could be independently validated.

This was the correct priority:

> Establish whether the core domain infrastructure works before chasing a management-console warning.

---

# 5. Domain Controller Health Validation

After reverting to the stable snapshot, the following validation checks were performed.

## 5.1 Active Directory Diagnostic

### Command

```powershell
dcdiag /q
```

### Result

**PASS**

No outstanding AD diagnostic failures were reported.

---

## 5.2 DNS Diagnostic

### Command

```powershell
dcdiag /test:dns /q
```

### Result

**PASS**

No outstanding DNS diagnostic failures were reported.

---

## 5.3 Domain Controller Discovery

### Command

```powershell
nltest /dsgetdc:lab.local
```

### Result

**PASS**

The domain controller was successfully discovered for:

```text
lab.local
```

with `LAB-DC01` serving as the domain controller.

---

## 5.4 DNS Resolution

### Command

```powershell
nslookup lab.local
```

### Result

**PASS**

The domain name successfully resolved through the lab DNS infrastructure.

This confirmed that the client and domain controller were capable of resolving the Active Directory domain correctly.

---

# 6. Network Configuration

The lab network remains divided between the VirtualBox NAT network and the dedicated Host-Only lab network.

## `LAB-DC01`

| Adapter | Address |
|---|---|
| NAT | `10.0.2.15` |
| Lab Host-Only | `192.168.10.10/24` |

## `LAB-WIN11`

| Adapter | Address | DNS |
|---|---|---|
| NAT | `10.0.2.15` | NAT-provided |
| Lab Host-Only | `192.168.10.20/24` | `192.168.10.10` |

The important configuration for Active Directory communication is the Host-Only network and the use of `192.168.10.10` as the client's DNS server.

Earlier troubleshooting had already established that the client could:

- Resolve `lab.local`.
- Resolve `LAB-DC01`.
- Communicate with the domain controller.

Those conditions remained valid for today's domain join.

---

# 7. Domain Join

Once all health checks passed, `LAB-WIN11` was joined to:

```text
lab.local
```

The domain join completed successfully.

This was the **first successful domain join in the lab**.

**Result: PASS**

---

# 8. Authentication Testing

After rebooting the client, authentication was tested using both the administrative and standard employee accounts.

## Administrative Account

```text
LAB\ynot.admin
```

**Result: PASS**

The account successfully authenticated against the domain.

## Standard Employee Account

```text
LAB\tony.employee
```

**Result: PASS**

The account successfully authenticated against the domain.

---

# 9. What Today's Testing Proved

Today's successful domain join is significantly more important than simply getting a Windows machine to log in.

It demonstrates that the following pieces of the lab are functioning together:

```text
                 LAB.LOCAL
                     │
                     ▼
               LAB-DC01
             AD DS + DNS
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
    Domain Discovery       DNS Resolution
          │                     │
          └──────────┬──────────┘
                     ▼
                LAB-WIN11
                     │
                     ▼
             Domain Authentication
```

Specifically, we have demonstrated:

- Active Directory Domain Services functioning.
- DNS functioning.
- Domain controller discovery functioning.
- Client-to-DC network communication functioning.
- Domain join functioning.
- Domain authentication functioning.
- Administrative domain authentication functioning.
- Standard-user domain authentication functioning.

The lab has officially progressed from:

> **"AD is installed."**

to:

> **"I have a functioning domain with a domain-joined Windows workstation."**

That is a meaningful milestone.

---

# 10. Snapshots / Recovery Points

A snapshot was taken after successfully establishing the domain-joined state.

### Snapshot Name

```text
Domain Joined - Baseline
```

Both virtual machines were then shut down for the evening.

## Current Known-Good Recovery State

### `LAB-DC01`

- Stable AD/DNS configuration.
- Health checks passing.
- Domain operational.

### `LAB-WIN11`

- Windows 11 Pro.
- Successfully joined to `lab.local`.
- `LAB\ynot.admin` authentication confirmed.
- `LAB\tony.employee` authentication confirmed.

---

# 11. Outstanding Issues

The following items remain intentionally unresolved.

## Server Manager Manageability Warning

```text
Online - Cannot get role and feature data
```

This should be investigated later, but it is **not currently preventing normal AD functionality**.

---

## Windows Update Instability

The exact cause of the previous freezing has not been definitively identified.

Current evidence strongly suggests that the post-update state contributed to the problem because reverting to the earlier snapshot restored stability.

However, this should be documented as:

> **Suspected Windows Update-related instability, not confirmed root cause.**

---

## Hypervisor / NEM Investigation

Previous troubleshooting established that Windows' hypervisor was active and VirtualBox was operating through the Windows hypervisor compatibility path.

This remains a potential performance factor but was **not demonstrated to be the root cause** of the freezes.

No destructive or invasive changes were made to address this.

---

# 12. Current Lab Status

### Overall Status: 🟢 Functional

| Component | Status |
|---|:---:|
| VirtualBox 7.2.20 | ✅ |
| `LAB-DC01` boots | ✅ |
| AD DS | ✅ |
| DNS | ✅ |
| `dcdiag /q` | ✅ |
| DNS diagnostic | ✅ |
| DC discovery | ✅ |
| `LAB-WIN11` networking | ✅ |
| Domain join | ✅ |
| `LAB\ynot.admin` login | ✅ |
| `LAB\tony.employee` login | ✅ |
| Recovery snapshot | ✅ |
| Server Manager manageability alert | ⚠️ Deferred |
| Windows Update instability | ⚠️ Suspected, unresolved root cause |

---

# 13. Next Session

The next session should **not** begin by changing the DC configuration.

### Recommended Order

1. Boot `LAB-DC01`.
2. Confirm it remains stable.
3. Boot `LAB-WIN11`.
4. Confirm domain authentication still works.
5. Verify the computer object for `LAB-WIN11` in Active Directory.
6. Begin configuring **Group Policy**.
7. Establish baseline security policies for the workstation.
8. Begin building the lab toward Windows event logging and SOC-style investigation.

The next major goal should be moving from **infrastructure construction** into **security operations practice**.

---

# Session Conclusion

**October 16, 2026 successfully established the first functioning domain-joined workstation in the home lab.**

The previous instability was mitigated by reverting `LAB-DC01` to a known-good snapshot.

The resulting environment:

- Passed Active Directory validation.
- Passed DNS validation.
- Successfully discovered the domain controller.
- Successfully joined `LAB-WIN11` to `lab.local`.
- Successfully authenticated both a domain administrator and a standard domain user.
- Was preserved with a post-domain-join recovery snapshot.

## Checkpoint Reached

> **Domain infrastructure operational + Windows 11 endpoint successfully domain joined.**

### Life, finds a way. 🦖
