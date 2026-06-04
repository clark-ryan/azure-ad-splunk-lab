# Splunk & Sysmon Setup

## Environment
- Splunk Server: Ubuntu Server (Azure VM)
  - IP Address: 192.168.100.10
  - Splunk Port: 8000 (web UI), 9997 (receiving)
- Windows 10 Workstation: 192.168.100.100
- Active Directory DC: 192.168.100.5
- Both Windows machines have Splunk Universal Forwarder and Sysmon installed

---

## Splunk Server Setup

1. Set a static IP on the Ubuntu VM (192.168.100.10) by editing `/etc/netplan/installer-config.yaml` or manually setting it up in Azure 
2. Downloaded Splunk Enterprise (Linux .deb) from splunk.com
3. Installed Splunk: `sudo dpkg -i splunk.deb`
4. Started Splunk and accepted the license agreement
5. Enabled Splunk to start on boot: `sudo ./splunk enable boot-start -user splunk`
6. Logged into the Splunk web UI at `http://192.168.100.10:8000`
7. Created a new index named `endpoint` under Settings > Indexes
8. Enabled a receiving port under Settings > Forwarding and Receiving > Configure Receiving > Port `9997`

---

## Sysmon Setup (Windows Machines)

1. Downloaded Sysmon from Sysinternals
2. Downloaded the Olaf Hartong Sysmon config on github: `sysmonconfig.xml`
3. Opened PowerShell as Administrator and ran:
```powershell
.\Sysmon64.exe -i ..\sysmonconfig.xml
```
4. Accepted the license agreement — Sysmon installed and started automatically

---

## Splunk Universal Forwarder Setup (Windows Machines)

1. Downloaded Splunk Universal Forwarder (64-bit Windows) from splunk.com
2. Ran the MSI installer and configured:
   - Receiving indexer: `192.168.100.10:9997`
3. Created a new `inputs.conf` file under:
   `C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`