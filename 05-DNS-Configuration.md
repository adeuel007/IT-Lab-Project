Phase 5:
DNS Configuration

Performed validation using:
-nslookup
-dcdiag /test:dns

Identified issue:
-The Domain Controller registered both:
--192.168.10.10
--10.0.2.15

Corrective action:
-Disabled DNS registration on the NAT adapter.
-Forced DNS re-registration.

Verified final DNS record:
--LAB-DC01.lab.local
---192.168.10.10
