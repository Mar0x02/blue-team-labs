# Reveal Lab — CyberDefenders

> **Platform:** CyberDefenders
> **Category:** Endpoint Forensics
> **Difficulty:** Easy
> **Status:** 🔄 In Progress
> **Date:** 2026-08-13
> **Time Spent:** ~45 menit

---

## 📌 Prolog

Merekonstruksi multi-stage attack dengan menganalisis Windows memory dump pakai Volatility 3 — identifikasi proses mencurigakan, command line, dan korelasi temuan dengan threat intelligence.

**Tools:** Volatility 3

**Tactics yang tercakup:** Stealth | Discovery

**Catatan:** lab ini berstatus *Retired* di platform CyberDefenders.

---

## 🎯 Scenario

Lo berperan sebagai forensic investigator di sebuah institusi finansial. SIEM lo flag aktivitas mencurigakan di salah satu workstation yang punya akses ke data finansial sensitif. Curiga ada breach, lo dapet memory dump dari mesin yang dicurigai kompromise.

Tugas lo: analisis memory buat nyari tanda-tanda kompromise, trace asal-usul anomalinya, dan assess scope insiden ini biar bisa di-*contain* secara efektif.

---

## ❓ Questions

1. Identifying the name of the malicious process helps in understanding the nature of the attack. What is the name of the malicious process?
2. Knowing the parent process ID (PPID) of the malicious process aids in tracing the process hierarchy and understanding the attack flow. What is the parent PID of the malicious process?
3. Determining the file name used by the malware for executing the second-stage payload is crucial for identifying subsequent malicious activities. What is the file name that the malware uses to execute the second-stage payload?
4. Identifying the shared directory on the remote server helps trace the resources targeted by the attacker. What is the name of the shared directory being accessed on the remote server?
5. What is the MITRE ATT&CK sub-technique ID that describes the execution of a second-stage payload using a Windows utility to run the malicious file?
6. Identifying the username under which the malicious process runs helps in assessing the compromised account and its potential impact. What is the username that the malicious process runs under?
7. Knowing the name of the malware family is essential for correlating the attack with known threats and developing appropriate defenses. What is the name of the malware family?

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

- [ ] Lengkapi jawaban 7 pertanyaan di atas via Volatility (pstree, cmdline, netscan, dan plugin relevan lainnya)
- [ ] Cross-check malware family & IOC (IP/hash/shared directory) ke VirusTotal/ThreatFox
- [ ] Petakan MITRE sub-technique buat eksekusi second-stage payload (pertanyaan #5)

---

## 📚 References

- [CyberDefenders — Reveal Lab](https://cyberdefenders.org/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
