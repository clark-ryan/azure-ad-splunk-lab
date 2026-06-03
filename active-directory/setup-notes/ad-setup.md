# Active Directory Setup

## Environment

- OS: Windows Server 2022 (Azure VM)
    - Domain: clarklabs.local
    - IP Address: 192.168.100.5
    - DNS: 192.168.100.5 (forwards to 8.8.8.8)
- OS: Windows 10 Enterprise
    - Domain: clarklabs.local
    - IP Address: 192.168.100.100
    - Domain Controller IP: 192.168.100.5
    - DNS Server: 192.168.100.5

## Steps

1. Set a static IP on the Windows Server VM
2. Opened Server Manager → Manage → Add Roles and Features
3. Selected Active Directory Domain Services (AD DS) and installed
4. After install, clicked the flag icon → Promote this server to a domain controller
5. Selected "Add a new Forest" and named the domain clarklabs.local
6. Set a DSRM password and completed the wizard — server restarted automatically
7. Logged back in; domain prefix on login screen confirmed AD DS was active
8. Opened Active Directory Users and Computers via Server Manager → Tools
9. Created two Organizational Units: IT and HR
10. Created user Jenny Smith (j.smith) under IT and Terry Smith (t.smith) under HR
11. On the Windows 10 VM, changed DNS to point to the DC (192.168.100.5)
12. Joined the Windows 10 machine to clarklabs.local using administrator credentials
13. Restarted and logged in as j.smith to confirm domain authentication worked