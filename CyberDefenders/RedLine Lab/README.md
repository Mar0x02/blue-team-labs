# RedLine Lab — CyberDefenders

> **Platform:** CyberDefenders
> **Category:** Endpoint Forensics
> **Difficulty:** Easy
> **Status:** 🔄 In Progress
> **Date:** 2026-08-12
> **Time Spent:** ~45 menit

---

## 📌 Prolog

Analisis memory dump pakai Volatility buat identifikasi proses mencurigakan, network IOC, memory protection, dan infrastruktur command-and-control attacker.

**Tools:** Volatility | Strings

**Tactics yang tercakup:** Privilege Escalation | Stealth | Command and Control

---

## 🎯 Scenario

Sebagai anggota tim Security Blue Team, tugas lo adalah menganalisis memory dump menggunakan tools Redline dan Volatility. Tujuannya adalah menelusuri langkah-langkah yang diambil attacker di mesin yang udah dikompromikan, dan menentukan gimana caranya mereka berhasil bypass Network Intrusion Detection System (NIDS).

Investigasi lo akan mengidentifikasi malware family spesifik yang dipakai dalam serangan ini beserta karakteristiknya. Selain itu, tugas lo juga mengidentifikasi dan memitigasi jejak yang ditinggalkan attacker.

---

## ❓ Questions

1. What is the name of the suspicious process?
2. What is the child process name of the suspicious process?
3. What is the memory protection applied to the suspicious process memory region?
4. What is the name of the process responsible for the VPN connection?
5. What is the attacker's IP address?
6. What is the full URL of the PHP file that the attacker visited?
7. What is the full path of the malicious executable?

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

🔄 Belum diisi.

### Todo / Follow-up

- [ ] Lengkapi jawaban 7 pertanyaan di atas via Volatility (pstree, malfind, netscan, cmdline) dan strings
- [ ] Cross-check malicious executable path & IP attacker ke VirusTotal/ThreatFox
- [ ] Analisis gimana attacker bypass NIDS di skenario ini

---

## 📚 References

- [CyberDefenders — RedLine Lab](https://cyberdefenders.org/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
