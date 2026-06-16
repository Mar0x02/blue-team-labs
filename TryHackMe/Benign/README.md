# Benign — TryHackMe

> **Platform:** TryHackMe  
> **Category:** SIEM / Log Analysis / Threat Hunting  
> **Difficulty:** Medium  
> **Status:** 🔄 In Progress  
> **Date:** 2026-06-16  
> **Time Spent:** ~X jam  

---

## 📌 Prolog

Challenge ini mengharuskan kita menelusuri process execution logs dari sebuah host yang dicurigai terkompromi. Tools yang digunakan adalah **Splunk** — semua log sudah di-ingest ke dalam index `win_eventlogs` dan siap dianalisis. Untuk yang belum familiar dengan Splunk, TryHackMe menyediakan room splunk101 dan splunk201 sebagai prerequisite.

---

## 🎯 Scenario

IDS salah satu klien mendeteksi eksekusi proses yang mencurigakan dari host di departemen HR. Beberapa tools terkait network information gathering dan scheduled tasks ikut tereksekusi, yang semakin memperkuat dugaan adanya kompromi.

Karena keterbatasan resource, hanya **process execution logs** dengan **Event ID: 4688** yang berhasil dikumpulkan dan di-ingest ke Splunk di index `win_eventlogs`.

Jaringan klien terbagi dalam tiga segmen:

| Departemen | User |
|------------|------|
| IT | James, Moin, Katrina |
| HR | Haroon, Chris, Diana |
| Marketing | Bell, Amelia, Deepak |

---

## ❓ Questions

1. How many logs are ingested from the month of March, 2022?
2. Imposter Alert: There seems to be an imposter account observed in the logs, what is the name of that user?
3. Which user from the HR department was observed to be running scheduled tasks?
4. Which user from the HR department executed a system process (LOLBIN) to download a payload from a file-sharing host?
5. To bypass the security controls, which system process (lolbin) was used to download a payload from the internet?
6. What was the date that this binary was executed by the infected host? format (YYYY-MM-DD)
7. Which third-party site was accessed to download the malicious payload?
8. What is the name of the file that was saved on the host machine from the C2 server during the post-exploitation phase?
9. The suspicious file downloaded from the C2 server contained malicious content with the pattern THM{..........}; what is that pattern?
10. What is the URL that the infected host connected to?

---

## 🔍 Answer & Walkthrough

> 🔄 *Belum diisi — akan dilengkapi setelah pengerjaan selesai.*

---

## 🚨 Key Findings / IOCs

> 🔄 *Belum diisi.*

| Tipe | Value | Keterangan |
|------|-------|------------|
| IP Address | `...` | ... |
| File Hash | `...` | ... |
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

---

## 📚 References

- [MITRE ATT&CK — T1059: Command and Scripting Interpreter](https://attack.mitre.org/techniques/T1059/)
- [MITRE ATT&CK — T1053: Scheduled Task/Job](https://attack.mitre.org/techniques/T1053/)
- [Splunk Documentation](https://docs.splunk.com/)
- [Windows Event ID 4688 — Process Creation](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
