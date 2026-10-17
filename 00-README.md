
## IT Home-Lab

This project is practice for managing an enterprise network simulation with an Active Directory Home Lab. It uses Windows Server 2022 (Eval) inside Oracle VirtualBox VM's to establish a domain controller and endpoint(s) to manage, network and troubleshoot common issues.

## Lab Environment

**Hypervisor:** Oracle VirtualBox 7.2.14

**Client:** Windows 11 (VirtualBox VM)

**Server:** Windows Server 2022 Evaluation (VirtualBox VM)

## Roadmap

- 01-planning
    - Details Active Directory Domain Controller VM settings and configurations. 

- 02-WindowsServer22 Install
    - Windows Server2022 Installation with Desktop GUI
    - Administrative account configurations
    - VirtualBox Guest Additions installed for enhanced performance.

- 03-Network Configuration
    - Dual Network Architecture configuration 

- 04-Active Directory Deployment
    - AD DS installation on server and appropriate roles installed/configured
    - Promoted Server to Domain Controller

- 05-DNS Configuration
    - DNS configuration, validation, troubleshooting, and verification process

- 06-Stability Investigation
    - Assessment of VM Stability following VM Freezes and Windows Service misconfigurations
    - Adjustments made accordingly to VM resource allocations

- 07-OU and Group Design
    - Active Directory Structure imformation including Operational Units and Groups established

- 08-Client Endpoint
    - Windows 11 Client VM Configuration

## Lessons Learned

What did you learn while building this project? What challenges did you face and how did you overcome them?

Some Major Highlights from the project so far:
- Multi-Homed Domain Controllers require careful DNS registration
- Not every Event Viewer error represents an active problem. Context matters!
- The dcdiag tool is a STARTING point for an investigation, not the final answer.
- VM Stability can cascade into misleading services warnings.
- A clean snapshot strategy saves an enormous amount of time while experimenting!
- Separating internet-facing and domain networks closely mirrors enterprise design.
