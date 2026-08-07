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
