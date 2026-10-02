Phase 6:
Time Service
-Configured Windows Time Service to synchronize with external NTP servers.
-Verified successful synchronization.

Stability Investigation
-Observed:
--Multiple VM freezes
--Forced VirtualBox resets
--Kernel-Power Event ID 41
--DFS Replication Event IDs 2212 and 6016

Investigation concluded:
-Kernel-Power:
--Expected due to forced VM reset.

DFSR:
-Recovered automatically after unexpected shutdown.
--"No user action required."

Corrective actions:
--Increased virtual CPUs from 2 → 4
--Disabled Server Manager automatic startup
--Confirmed improved stability through extended testing.
