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
