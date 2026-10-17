# Phase 6: Time Service and Stability Investigation

## Objective

Configure Windows Time Service to synchronize with external Network Time Protocol (NTP) servers, investigate recurring virtual machine instability, and apply corrective actions to improve the reliability of the Active Directory lab.

## 1. Windows Time Service Configuration

### Completed Tasks

- Configured Windows Time Service to synchronize with external NTP servers.
- Verified successful time synchronization.

### Results

Time synchronization was confirmed, helping ensure consistent system time for Active Directory authentication and domain operations.

Accurate time is important in an Active Directory environment because Kerberos authentication relies on synchronized clocks between domain members and domain controllers.

## 2. Stability Investigation

### Observed Symptoms

During lab operation, `LAB-DC01` experienced recurring stability issues:

- Multiple virtual machine freezes.
- Forced VirtualBox resets to recover from unresponsive states.
- Kernel-Power Event ID 41 in the Windows Event Log.
- DFS Replication (DFSR) Event IDs 2212 and 6016.

These events were investigated to distinguish expected recovery behavior following unexpected shutdowns from potential underlying stability issues.

### Event Log Findings

| Event | Investigation Finding |
|---|---|
| Kernel-Power Event ID 41 | Consistent with the virtual machine being forcibly reset or shut down without a clean Windows shutdown. This event records the unexpected shutdown; it does not independently identify the root cause. |
| DFSR Event ID 2212 | DFS Replication detected an unexpected shutdown and performed recovery activity. |
| DFSR Event ID 6016 | Observed during the DFS Replication investigation. Its specific cause should be interpreted using the complete event message and surrounding logs rather than the event ID alone. |

DFSR recovered automatically following the unexpected shutdown. The recovery message indicated that no user action was required.

**Investigation note:** These events helped explain some of the behavior observed after forced resets, but they did not establish the original cause of the virtual machine freezes.

## 3. Corrective Actions

The following changes were made to improve virtual machine stability:

| Action | Purpose |
|---|---|
| Increased virtual CPUs from 2 to 4 | Allocated additional virtual processing capacity to the server. |
| Disabled automatic Server Manager startup | Reduced unnecessary automatic activity when Windows Server starts or a user signs in. |
| Conducted extended stability testing | Evaluated whether the configuration changes improved reliability over longer operating periods. |

### Results

- Improved stability was observed during extended testing.
- The virtual machine was more reliable following the configuration changes.
- The original root cause of the freezes remains unconfirmed.

Although stability improved after these changes, further investigation would be necessary to determine whether CPU allocation, Server Manager startup behavior, Windows updates, or another factor contributed to the original issue.

## 4. Lessons Learned

- **Time synchronization matters:** Consistent system time supports reliable Kerberos authentication and Active Directory operations.
- **Unexpected shutdowns leave evidence:** Kernel-Power and DFSR events can help explain what Windows does after an unclean shutdown.
- **An event is not automatically a root cause:** Event logs must be interpreted alongside the system's behavior and surrounding events.
- **Recovery is not the same as prevention:** Automatic DFSR recovery restored service, but it did not prevent the underlying virtual machine instability.
- **Change one variable at a time when practical:** Isolating configuration changes makes it easier to identify which changes actually improve stability.
- **Snapshots support recovery, not diagnosis:** VirtualBox snapshots provide a useful recovery point, but event logs and controlled testing are still needed to investigate recurring failures.

## Phase Status

**Completed:** External NTP synchronization was configured and verified. Stability issues were investigated, corrective changes were applied, and improved reliability was observed during extended testing.

**Remaining consideration:** The original cause of the virtual machine freezes has not been conclusively identified.
