# Phase 2: Windows Server 2022 Installation

## Objective

Install and prepare Windows Server 2022 on the `LAB-DC01` virtual machine as the foundation for the Active Directory lab.

---

## Completed Tasks

### 1. Windows Server Installation

- Installed Windows Server 2022 Evaluation.
- Selected **Desktop Experience** to provide the graphical user interface (GUI).
- Confirmed the server was operational following installation.

### 2. Administrator Account Configuration

- Configured the local Administrator account.
- Applied strict password guidelines when establishing the administrative password.
- Established administrative access for subsequent server configuration.

**Security note:** Administrative credentials should be stored securely and never committed to a public GitHub repository.

### 3. VirtualBox Guest Additions

- Installed Oracle VirtualBox Guest Additions.
- Enabled guest integration features to improve usability and interaction between the host and virtual machine.

### 4. Clipboard Integration

- Configured VirtualBox clipboard integration to support copying and pasting between the host machine and the guest operating system.

### 5. Snapshot and Recovery Strategy

- Created VM snapshots at major project milestones.
- Established recovery points to help preserve known-good configurations and simplify recovery during troubleshooting.

Snapshots are particularly useful when experimenting with server roles, networking, Windows updates, and Active Directory configuration.

---

## Phase 2 Status

**Status:** Complete

Windows Server 2022 was successfully installed and prepared for its role as the lab's domain controller.

## Result

The server environment was ready for the next stage of configuration: networking, Active Directory Domain Services, DNS, and domain creation.

Since then, `LAB-DC01` has been configured as the domain controller for `lab.local`, and the Windows 11 client `LAB-WIN11` has successfully joined the domain.

---

## Lessons Learned

- **Establish recovery points early.** Snapshots provide a way to return to a known-good state after failed configuration changes.
- **Prepare the operating system before adding roles.** Establishing administrative access and basic VM integration first makes subsequent configuration easier.
- **Protect administrative credentials.** Strong passwords and secure credential storage are foundational to responsible server administration.
