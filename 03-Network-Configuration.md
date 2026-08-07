Phase 3:
Networking
--Designed a dual-network architecture.

Adapter 1
--Purpose:
---Internet access

Configuration:
--NAT
---Provides:
----Windows Updates
----Internet connectivity
----External DNS lookups

Adapter 2
--Purpose:
---Private Active Directory network

Configuration:
--Host-Only Adapter

Addressing:
--192.168.10.10 /24

Gateway:
None

-This network is intentionally isolated and will be used by all future lab machines.

-/*Network Troubleshooting*/-
Initially experienced:
--APIPA address (169.254.x.x)
--Host-to-VM ping failures
--DNS registration issues

Troubleshooting performed:
--Verified VirtualBox Host-Only Adapter configuration
--Created dedicated Host-Only network
--Assigned static IPv4 addressing
--Tested connectivity using:
---ipconfig
---ping
---arp -a
---Test-NetConnection

Eventually confirmed:
-Host-only communication working
-Internet connectivity working
-DNS resolution functioning
