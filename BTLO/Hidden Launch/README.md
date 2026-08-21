# Hidden Launch — BTLO

> **Platform:** Blue Team Labs Online
> **Category:** Digital Forensics / macOS Forensics
> **Difficulty:** Medium
> **Status:** 🔄 In Progress
> **Date:** 2026-08-21
> **Time Spent:** ~0 jam

---

## 📌 Prolog

Rekonstruksi serangan macOS yang stealthy, dari initial execution sampai credential theft, persistence, hidden processes, data collection, sampai final exfiltration. Tugasnya: hubungin semua evidence, ungkap timeline si attacker, dan temukan data yang dicuri sebelum jejaknya hilang.

Tools yang dipakai: MAC (macOS Artifacts), FSEvents, UnifiedLogIterator.

---

## 🎯 Scenario

Sebuah command yang keliatannya rutin, dieksekusi dari website berbahaya, jadi pintu masuk buat intrusion macOS yang stealthy. Attacker bergerak dari initial execution ke credential theft, menyalahgunakan trusted system component buat dapetin akses tambahan, membangun beberapa persistence mechanism yang disamarkan sebagai Apple services, dan deploy hidden process yang jalan diam-diam di dalam user session korban. Di saat yang sama, data sensitif dikumpulin dari seluruh sistem dan disiapkan buat exfiltration. Misinya: rekonstruksi timeline attacker, identifikasi breadcrumb yang ditinggalkan, hubungin tiap stage intrusion, dan temukan exfiltration package terakhir sebelum jejaknya hilang.

---

## ❓ Questions

1. The victim was tricked into executing commands from a malicious site. What is the full URL of the malicious site he visited?
2. Shortly after, the victim ran a command in the terminal, believing it was legitimate. What time was it executed?
3. That command was run from a specific terminal session. What is the PID of that session?
4. As a result, a binary was dropped and executed right after. What is the inode number of that first dropped binary?
5. The binary presented a fake prompt asking the victim to enter their password, and captured it. What time did this happen?
6. The attacker abused a TCC/AppleEvents trust relationship to silently gain automation control without prompting the user. What time was this permission grant recorded?
7. The attacker began dropping more files into the temp directory for persistence. What are the names of the two files used for this purpose?
8. Inspecting the persistence mechanism for the first file, reveals it executes a hidden script. What is the SHA1 hash of this script?
9. That script executes another file every 5 seconds. What is the SHA1 hash of this file?
10. The attacker began staging files for exfiltration. How many files did he manage to stage in total?
11. He then compressed all staged files into a single archive. What is the name of that zip file?

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

- [ ] Trace initial execution dari malicious site sampai dropped binary pertama
- [ ] Analisis fake password prompt & TCC/AppleEvents abuse buat automation control
- [ ] Petain persistence mechanism (LaunchAgents/LaunchDaemons) yang nyamar jadi Apple services
- [ ] Identifikasi hidden script & file yang di-invoke tiap 5 detik
- [ ] Hitung staged files & analisis final exfiltration archive

---

## 📚 References

- 🔄 Belum diisi.

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
