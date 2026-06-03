# Challenges

## 1. Azure Firewall Rules
One of the biggest challenges was Azure's predefined firewall rules on virtual machines. Early on, I couldn't RDP into the machines at all until I figured out the ports were blocked. Later, when setting up Splunk, port 9997 was blocked so endpoints weren't forwarding data to the Splunk server — I spent a lot of time troubleshooting before realizing the fix was simply opening that port for the local network (192.168.100.0/24). The same issue came up when joining machines to Active Directory. A bit of research identified the required ports, and once opened, everything worked.

## 2. DNS Misconfiguration Locking Me Out
After getting the ports sorted, I changed the Windows machine's DNS to point to the Active Directory server — but the AD server hadn't been configured to forward DNS requests it couldn't resolve. This caused the Windows machine to lose internet connectivity entirely. Since I was working in Azure with no physical access, and RDP depends on network connectivity, I was effectively locked out. After researching AD DNS forwarding, I configured the server to forward unresolved requests to 8.8.8.8, which restored connectivity and resolved the issue.

## 3. No Way to Verify Network Connectivity via Ping
Windows machines block ICMP traffic by default, and Azure's firewall adds another layer on top of that. Even after allowing ICMP at the network level, the Windows firewall rules made it difficult to get ping working between machines. As a result, I had no reliable way to verify that my private network was functioning until I successfully joined the Windows machine to the domain — which itself confirmed connectivity.