# Phase 5: DNS Configuration

## Objective

Configure and validate DNS on `LAB-DC01` to ensure the Domain Controller registers its private lab-network address for the `lab.local` domain.

---

## 1. DNS Validation

Performed DNS validation using the following tools:

```powershell
nslookup
dcdiag /test:dns
```

These tools were used to investigate name resolution and verify DNS health within the Active Directory environment.

---

## 2. Issue Identified

The Domain Controller had registered both network interface addresses in DNS:

| Interface | IPv4 Address | Purpose |
|---|---|---|
| Host-Only | `192.168.10.10` | Private Active Directory lab network |
| NAT | `10.0.2.15` | Internet connectivity |

The NAT address was being registered alongside the private lab address.

This could cause clients to receive an address that is not appropriate for domain communication, potentially resulting in connectivity or domain-discovery issues.

---

## 3. Corrective Action

Applied the following changes:

1. Disabled DNS registration on the NAT adapter.
2. Forced DNS re-registration to update the Domain Controller's DNS records.
3. Rechecked the resulting DNS record to confirm the intended address was registered.

This retained NAT connectivity while preventing the NAT interface from registering its address in the lab's DNS configuration.

---

## 4. Verification

Verified the final DNS record for the Domain Controller.

**Expected result:**

```text
LAB-DC01.lab.local
    192.168.10.10
```

The Domain Controller's fully qualified domain name (FQDN) resolved to its private lab-network address.

**Result:** DNS registration corrected and the intended address verified.

---

## Lessons Learned

- **Multi-homed servers require deliberate DNS configuration.** Not every network interface should automatically register its address in DNS.
- **DNS records influence domain connectivity.** Incorrect or unintended address registrations can interfere with clients locating domain services.
- **Verify changes after applying them.** A configuration change is not complete until the resulting DNS records have been checked.

---

## Phase 5 Status

**Status:** Complete

DNS registration was corrected so that `LAB-DC01.lab.local` resolves to `192.168.10.10`, the Domain Controller's private lab-network address.

This supports reliable DNS resolution and Active Directory communication between the Domain Controller and domain-joined clients.
