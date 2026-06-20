# Investigating Windows 2.0 — TryHackMe

> **Platform:** TryHackMe  
> **Category:** Digital Forensics / Malware Analysis / Threat Hunting  
> **Difficulty:** Medium  
> **Status:** 🔄 In Progress  
> **Date:** 2026-06-20  
> **Time Spent:** ~X jam  

---

## 📌 Prolog

Challenge lanjutan dari seri Investigating Windows. Kali ini investigasi lebih dalam — melibatkan analisis registry, scheduled tasks, WMI persistence, process monitoring dengan Sysinternals, sampai binary scanning dengan Loki dan Yara. Prerequisite yang disarankan: room Core Windows Processes, Sysinternals, dan Yara. Akses ke mesin via RDP.

---

## 🎯 Scenario

Sebuah mesin Windows dicurigai telah terkompromi. Investigasi dilakukan langsung dari dalam mesin menggunakan berbagai tools forensik: registry analysis, Sysinternals (Process Monitor, Process Explorer), Loki scanner, dan Yara rules.

Tugas kita adalah mengidentifikasi persistence mechanism yang digunakan attacker, script berbahaya yang berjalan di background, binary yang ter-flag sebagai malicious, hingga melengkapi Yara rule untuk mendeteksi binary yang luput dari scan Loki.

---

## ❓ Questions

1. What registry key contains the same command that is executed within a scheduled task?
2. What analysis tool will immediately close if/when you attempt to launch it?
3. What is the full WQL Query associated with this script?
4. What is the script language?
5. What is the name of the other script?
6. What is the name of the software company visible within the script?
7. What 2 websites are associated with this software company? (answer, answer)
8. Search online for the name of the script from Q5 and one of the websites from the previous answer. What attack script comes up in your search?
9. What is the location of this file within the local machine?
10. Which 2 processes open and close very quickly every few minutes? (answer, answer)
11. What is the parent process for these 2 processes?
12. What is the first operation for the first of the 2 processes?
13. Inspect the properties for the 1st occurrence of this process. In the Event tab what are the 4 pieces of information displayed? (answer, answer, answer, answer)
14. Inspect the disk operations, what is the name of the unusual process?
15. Run Loki. Inspect the output. What is the name of the module after `Init`?
16. Regarding the 2nd warning, what is the name of the eventFilter?
17. For the 4th warning, what is the class name?
18. What binary alert has the following 4d5a90000300000004000000ffff0000b8000000 as FIRST_BYTES?
19. According to the results, what is the description listed for reason 1?
20. Which binary alert is marked as APT Cloaked?
21. What are the matches? (str1, str2)
22. Which binary alert is associated with somethingwindows.dmp found in C:\TMP?
23. Which binary is encrypted that is similar to a trojan?
24. There is a binary that can masquerade itself as a legitimate core Windows process/image. What is the full path of this binary?
25. What is the full path location for the legitimate version?
26. What is the description listed for reason 1?
27. There is a file in the same folder location that is labeled as a hacktool. What is the name of the file?
28. What is the name of the Yara Rule MATCH?
29. Which binary didn't show in the Loki results?
30. Complete the yar rule file located within the Tools folder on the Desktop. What are 3 strings to complete the rule in order to detect the binary Loki didn't hit on? (answer, answer, answer)

---

## 🔍 Answer & Walkthrough

> 🔄 *Belum diisi — akan dilengkapi setelah pengerjaan selesai.*

---

## 🚨 Key Findings / IOCs

> 🔄 *Belum diisi.*

| Tipe | Value | Keterangan |
|------|-------|------------|
| Registry Key | `...` | ... |
| File | `...` | ... |
| Domain | `...` | ... |

---

## 🗺️ MITRE ATT&CK Mapping

> 🔄 *Belum diisi.*

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| ... | ... | ... | ... |

---

## 📋 Summary — Attacker Behavior & Todo

> 🔄 *Belum diisi — akan dilengkapi setelah analisis selesai.*

### Attacker Behavior

### Todo / Follow-up

- [ ] ...

---

## 📚 References

- [TryHackMe — Core Windows Processes](https://tryhackme.com/room/btwindowsinternals)
- [TryHackMe — Sysinternals](https://tryhackme.com/room/btsysinternalssg)
- [TryHackMe — Yara](https://tryhackme.com/room/yara)
- [Loki IOC Scanner](https://github.com/Neo23x0/Loki)
- [MITRE ATT&CK](https://attack.mitre.org/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
