# Log Analysis - Sysmon — BTLO

> **Platform:** Blue Team Labs Online (BTLO)  
> **Category:** Log Analysis / Threat Hunting  
> **Difficulty:** Medium  
> **Status:** 🔄 In Progress  
> **Date:** 2026-05-30  
> **Time Spent:** 🔄 Belum diisi  

---

## 📌 Prolog

Challenge ini menyediakan Sysmon logs dari sebuah endpoint yang sudah dikompromis. Tugasnya adalah menganalisis log tersebut untuk menemukan langkah-langkah dan teknik yang digunakan attacker. Tools yang tersedia: Text Editor, PowerShell, Linux CLI.

---

## 🎯 Scenario

Kamu diberikan Sysmon logs dari sebuah endpoint yang telah dikompromis. Analisis log tersebut untuk menemukan langkah-langkah dan teknik yang digunakan attacker.

---

## ❓ Questions

1. What is the file that gave access to the attacker?
2. What is the powershell cmdlet used to download the malware file and what is the port?
3. What is the name of the environment variable set by the attacker?
4. What is the process used as a LOLBIN to execute malicious commands?
5. Malware executed multiple same commands at a time, what is the first command executed?
6. Looking at the dependency events around the malware, can you able to figure out the language the malware is written in?
7. Malware then downloads a new file, find out the full URL of the file download.
8. What is the port the attacker attempts to get reverse shell?

---

## 🔍 Answer & Walkthrough

> 🔄 *Belum diisi — akan dilengkapi setelah pengerjaan selesai.*

---

## 🚨 Key Findings / IOCs

> 🔄 *Belum diisi.*

| Tipe | Value | Keterangan |
|------|-------|------------|
| File | `...` | ... |
| PowerShell Cmdlet | `...` | ... |
| Port | `...` | ... |
| Environment Variable | `...` | ... |
| LOLBIN | `...` | ... |
| URL | `...` | ... |

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

- [Sysmon Events Reference — Microsoft](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [MITRE ATT&CK — LOLBINs](https://attack.mitre.org/techniques/T1218/)
- [LOLBAS Project](https://lolbas-project.github.io/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
