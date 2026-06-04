# azure-ad-splunk-lab

I simulated an internal corporate network in Microsoft Azure demonstrating Active Directory, Splunk SIEM, identity and access management, and cloud infrastructure.

![Topology](screenshots/network-topology.png)

The lab models a company's internal network where a domain controller manages users and devices across a private network. All endpoints forward logs to a Splunk SIEM to monitor for unauthorized activity that could compromise the environment. Threat simulation was performed using Kali Linux and Atomic Red Team to validate detections end-to-end.

## Skills Demonstrated

| Capability | Evidence |
|---|---|
| Active Directory | [AD Setup](active-directory/setup-notes/ad-setup.md) — domain controller, OUs, users, domain join |
| SIEM & Log Management | [Splunk Config](splunk/configs/inputs.conf.example) — Universal Forwarder, Windows Event Logs, Sysmon |
| Threat Detection | [Alert Rules](splunk/alerts/suspicious-activity-alerts.md) — brute force, account creation, suspicious processes |
| Attack Simulation | [Attack & Detection](atomic-red-team/attack-simulation.md) — Hydra RDP brute force, Atomic Red Team T1136.001 |
| Cloud & Firewall | [Challenges](notes/challenges.md) — Azure NSG rules, port management, DNS troubleshooting |

## Repository Contents

- [`active-directory/`](active-directory/) — domain controller setup, GPO configuration, users and OUs
- [`splunk/`](splunk/) — forwarder configs and Splunk detection rules
- [`atomic-red-team/`](atomic-red-team/) — attack simulation results and Splunk detections
- [`assets/`](assets/) — network topology diagram and screenshots
- [`notes/`](notes/) — challenges encountered and lessons learned

## License
[MIT](LICENSE)