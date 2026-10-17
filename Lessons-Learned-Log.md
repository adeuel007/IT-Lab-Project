Some of the biggest lessons from the build so far: (This will be a living document)

-Multi-homed Domain Controllers require careful DNS registration configuration.
---I ended up with multiple adapters pointing to an unintended configuration. 
---The objective was network segmentation with 1 adapter set to handle NAT for internet access, and the other pointed at the domain controller for inter-VM communication.

-Not every Event Viewer error represents an active problem. Context matters.
--Event viewer's purpose is to log -EVERY- event that happens on a system. Not all of them are a fire that needs to be put out because it has a warning attached to it.
--Sometimes, it's just telling you that you restarted the machine, and it took that personal...

-dcdiag is a starting point for investigation, not the final answer.
--I was introduced to this command as this is the first AD Domain controller I've configured. 
--It is single handedly one of the most useful system tools I am actively using.
--It's used to diagnose the health of Active Directory Domain Services and troubleshoot issues with domain controllers.
--It performs a comprehensive suite of over 20 tests to verify critical functions such as DNS configuration, replication, connectivity, and security descriptors.
--Command Operators I've used with dcdiag = dcdiag /s:<ServerName> [tests a specific domain], dcdiag /q filters output to show only errors,  and dcdiag /test:DNS to test specific services, like DNS.

-VM stability issues can cascade into misleading service warnings.
--Initially I provisioned 2 CPU Processors to run Windows Server 2022 (Evaluation Edition). That progressively became a bottleneck for performance and stability and so i made the decision to increase it to 4 CPUs.
--I've noticed less crashing instances and will continue to monitor performance in case another increase is needed. I have, however, memorized Event ID 41 in Windows Event viewer. Windows doesn't like it when you 
--restart a machine when it doesn't expect you to.

-A clean snapshot strategy saves enormous amounts of time when experimenting.
--Pretty self-explanatory.

-Separating Internet-facing and domain networks closely mirrors enterprise designs.

---8/7/26---
Troubleshooting due to windows update freezing VM. Force restart did not resolve. Rolled back to previous checkpoint. Verified state of the Domain Controller: Learned OUs and Groups were missing.
Commands ran:
Get-ADORganizationalUnit -Filter * | Select-Object Name, DistinguishedName
--Trying to locate missing OU's.--

Command wasn't recognized so ran:
Get-Module -ListAvailable ActiveDirectory
--Populated the ActiveDirectory module. which means it exists--

Ran:
Import-Module ActiveDirectory
--supposed to import the module. it did not. so i fetched the commands directly with:

Get-Command -Module ActiveDirectory | Where-Object Name -like :*OrganizationalUnit*"
--which imported 4 previously unloaded cmdlets.--

Explicit invoke to find the OUs, ran:
ActiveDirectory\Get-ADOrganizationslUnits -Filter * | Select-Object Name, DistinguishedName
--Nothing Populated...--

used dcdiag with various options to confirm AD Health, DNS Health, Verified Replication, and configured Windows Time. 
Determined Snapshot was taken before Custom OU's and Groups were created. Will repopulate and save again.


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



10/16/26 - "Life, finds a way" 

Home Lab Documentation: LAB-DC01 Stabilization & LAB-WIN11 Domain Join
Date: October 16, 2026
Environment: Oracle VirtualBox 7.2.20
Domain: lab.local
Domain Controller: LAB-DC01
Client: LAB-WIN11
1. Objective
The primary objective for this session was to stabilize the LAB-DC01 domain controller following repeated freezes and then proceed with joining LAB-WIN11 to the lab.local Active Directory domain.
A secondary objective was to determine whether the previously observed instability was related to recent Windows Updates.
2. Initial Problem
Prior to this session, LAB-DC01 had experienced repeated progressive freezes.
Observed behavior included:
- Increasing system latency.
- Windows becoming increasingly unresponsive.
- Delayed or failed response to Ctrl+Alt+Delete.
- VirtualBox itself remaining responsive while the guest OS became unusable.
- ACPI shutdowns occasionally becoming stuck at:
  - Shutting down service: Update Orchestrator Service
  - Getting Windows ready. Don't turn off your computer.
- Forced power-off eventually being required.
- dcdiag /q subsequently reporting an unexpected shutdown event.
The host system remained responsive during these incidents, making a host-wide system failure less likely.
VirtualBox had also been upgraded from 7.2.14 to 7.2.20 during the troubleshooting process.
The domain controller was subsequently configured with:
- 8 GB RAM
- 4 vCPUs
- 100% execution cap
- 80 GB VDI
- SATA storage
- NAT + Host-Only networking
The increase from 2 to 4 vCPUs appeared to improve the duration of usable operation, but did not eliminate the freezing.
3. Windows Update Investigation
A major clue came from the timing of the instability.
LAB-DC01 had previously been processing accumulated Windows Updates. During shutdown/reboot operations, the system repeatedly became stuck while interacting with the Update Orchestrator Service and subsequently displayed the Windows "Getting Windows ready" state.
Because the system was becoming progressively less usable following the update activity, the decision was made to revert LAB-DC01 to a previously known-good VirtualBox snapshot.
Result
After reverting the snapshot:
- LAB-DC01 successfully booted.
- The system remained usable.
- The previous severe latency/freezing was no longer immediately present.
- The server remained stable for approximately 30 minutes during active use.
This provides strong circumstantial evidence that the post-update state was involved in the instability, although it does not conclusively prove that a specific Windows Update was responsible.
4. Server Manager Manageability Alert
During validation, Server Manager displayed the following alert for LAB-DC01:
Online - Cannot get role and feature data

The server was displayed using both of its configured addresses:
10.0.2.15
192.168.10.10

An attempt was made to investigate the management stack.
One command initially generated:
Get-Service : Cannot find any service with service name 'ServerManager'.

This was determined not to be a server fault. ServerManager is not a Windows service and the command was therefore inappropriate for testing Server Manager itself.
The server subsequently became progressively less responsive again during troubleshooting, so further investigation of the Server Manager alert was intentionally deferred.
Decision
The manageability alert was not treated as a blocker because the actual Active Directory, DNS, and domain-controller health checks could be independently validated.
This was the correct priority: establish whether the core domain infrastructure works before chasing a management-console warning.
5. Domain Controller Health Validation
After reverting to the stable snapshot, the following validation checks were performed.
5.1 Active Directory diagnostic
Command:
dcdiag /q

Result
PASS
No outstanding AD diagnostic failures were reported.
5.2 DNS diagnostic
Command:
dcdiag /test:dns /q

Result
PASS
No outstanding DNS diagnostic failures were reported.
5.3 Domain Controller Discovery
Command:
nltest /dsgetdc:lab.local

Result
PASS
The domain controller was successfully discovered for:
lab.local

with LAB-DC01 serving as the domain controller.
5.4 DNS Resolution
Command:
nslookup lab.local

Result
PASS
The domain name successfully resolved through the lab DNS infrastructure.
This confirmed that the client and domain controller were capable of resolving the Active Directory domain correctly.
6. Network Configuration
The lab network remains divided between the VirtualBox NAT network and the dedicated Host-Only lab network.
LAB-DC01
NAT adapter:
10.0.2.15

Lab Host-Only adapter:
192.168.10.10/24

LAB-WIN11
NAT adapter:
10.0.2.15

Lab Host-Only adapter:
192.168.10.20/24
DNS:
192.168.10.10

The important configuration for Active Directory communication is the Host-Only network and the use of 192.168.10.10 as the client's DNS server.
Earlier troubleshooting had already established that the client could:
- Resolve lab.local.
- Resolve LAB-DC01.
- Communicate with the domain controller.
Those conditions remained valid for today's domain join.
7. Domain Join
Once all health checks passed, LAB-WIN11 was joined to:
lab.local

The domain join completed successfully.
This was the first successful domain join in the lab.
8. Authentication Testing
After rebooting the client, authentication was tested using both the administrative and standard employee accounts.
Administrative account
LAB\ynot.admin

Result: PASS
The account successfully authenticated against the domain.
Standard employee account
LAB\tony.employee

Result: PASS
The account successfully authenticated against the domain.
9. What Today's Testing Proved
Today's successful domain join is significantly more important than simply getting a Windows machine to log in.
It demonstrates that the following pieces of the lab are functioning together:
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

Specifically, we have demonstrated:
- Active Directory Domain Services functioning.
- DNS functioning.
- Domain controller discovery functioning.
- Client-to-DC network communication functioning.
- Domain join functioning.
- Domain authentication functioning.
- Administrative domain authentication functioning.
- Standard-user domain authentication functioning.
The lab has officially progressed from "AD is installed" to "I have a functioning domain with a domain-joined Windows workstation."
That is a meaningful milestone.
10. Snapshots / Recovery Points
A snapshot was taken after successfully establishing the domain-joined state.
Recommended snapshot description:
Domain Joined - Baseline

Both virtual machines were then shut down for the evening.
The current known-good recovery state therefore consists of:
LAB-DC01
Stable AD/DNS configuration
Health checks passing
Domain operational

LAB-WIN11
Windows 11 Pro
Successfully joined to lab.local
LAB\ynot.admin authentication confirmed
LAB\tony.employee authentication confirmed

11. Outstanding Issues
The following items remain intentionally unresolved:
Server Manager manageability warning
Online - Cannot get role and feature data

This should be investigated later, but it is not currently preventing normal AD functionality.
Windows Update instability
The exact cause of the previous freezing has not been definitively identified.
Current evidence strongly suggests that the post-update state contributed to the problem because reverting to the earlier snapshot restored stability.
However, this should be documented as:
Suspected Windows Update-related instability, not confirmed root cause.

Hypervisor/NEM investigation
Previous troubleshooting established that Windows' hypervisor was active and VirtualBox was operating through the Windows hypervisor compatibility path. This remains a potential performance factor but was not demonstrated to be the root cause of the freezes.
No destructive or invasive changes were made to address this.
12. Current Lab Status
Overall status: 🟢 Functional
Component	Status
VirtualBox 7.2.20	✅
LAB-DC01 boots	✅
AD DS	✅
DNS	✅
dcdiag /q	✅
DNS diagnostic	✅
DC discovery	✅
LAB-WIN11 networking	✅
Domain join	✅
LAB\ynot.admin login	✅
LAB\tony.employee login	✅
Recovery snapshot	✅
Server Manager manageability alert	⚠️ Deferred
Windows Update instability	⚠️ Suspected, unresolved root cause


13. Next Session
The next session should not begin by changing the DC configuration.
Recommended order:
1. Boot LAB-DC01.
2. Confirm it remains stable.
3. Boot LAB-WIN11.
4. Confirm domain authentication still works.
5. Verify the computer object for LAB-WIN11 in Active Directory.
6. Begin configuring Group Policy.
7. Establish baseline security policies for the workstation.
8. Begin building the lab toward Windows event logging and SOC-style investigation.
The next major goal should be moving from infrastructure construction into security operations practice.
Session conclusion
October 16, 2026 successfully established the first functioning domain-joined workstation in the home lab.
The previous instability was mitigated by reverting LAB-DC01 to a known-good snapshot. The resulting environment passed Active Directory and DNS validation, successfully joined LAB-WIN11 to lab.local, and successfully authenticated both a domain administrator and a standard domain user.
Checkpoint reached: Domain infrastructure operational + Windows 11 endpoint successfully domain joined.
