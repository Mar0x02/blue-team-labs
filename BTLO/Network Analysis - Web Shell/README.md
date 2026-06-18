# Network Analysis - Web Shell — BTLO

> **Platform:** Blue Team Labs Online  
> **Category:** Network Forensics / PCAP Analysis  
> **Difficulty:** Easy  
> **Status:** 🔄 In Progress  
> **Date:** 2026-06-18  
> **Time Spent:** ~X jam  

---

## 📌 Prolog

SOC menerima alert dari SIEM untuk aktivitas *'Local to Local Port Scanning'* — sebuah IP internal mulai melakukan scanning ke sistem internal lainnya. Challenge ini meminta kita untuk menginvestigasi apakah aktivitas tersebut memang berbahaya, menggunakan tools seperti Wireshark, TCPDump, atau TShark.

---

## 🎯 Scenario

SOC menerima alert di SIEM mereka untuk aktivitas *'Local to Local Port Scanning'*, di mana sebuah IP private internal mulai melakukan scanning ke sistem internal lain. Investigasi dan tentukan apakah aktivitas ini berbahaya atau tidak. Artifact yang diberikan berupa file PCAP untuk dianalisis menggunakan tools pilihan.

---

## ❓ Questions

1. What is the IP responsible for conducting the port scan activity?
2. What is the port range scanned by the suspicious host?
3. What is the type of port scan conducted?
4. Two more tools were used to perform reconnaissance against open ports, what were they?
5. What is the name of the php file through which the attacker uploaded a web shell?
6. What is the name of the web shell that the attacker uploaded?
7. What is the parameter used in the web shell for executing commands?
8. What is the first command executed by the attacker?
9. What is the type of shell connection the attacker obtains through command execution?
10. What is the port he uses for the shell connection?

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

### Attacker Behavior

> 🔄 *Belum diisi.*

### Todo / Follow-up

- [ ] ...

---

## 📚 References

- [Wireshark Documentation](https://www.wireshark.org/docs/)
- [MITRE ATT&CK — Network Service Scanning (T1046)](https://attack.mitre.org/techniques/T1046/)
- [MITRE ATT&CK — Web Shell (T1505.003)](https://attack.mitre.org/techniques/T1505/003/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
