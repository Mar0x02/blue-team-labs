# Network Analysis - Ransomware — BTLO

> **Platform:** Blue Team Labs Online
> **Category:** Network Forensics / PCAP Analysis
> **Difficulty:** Medium
> **Status:** 🔄 In Progress
> **Date:** 2026-08-21
> **Time Spent:** ~0 jam

---

## 📌 Prolog

ABC Industries kerja siang malam selama sebulan buat nyiapin tender document untuk proyek besar yang bakal amankan masa depan finansial perusahaan mereka. Sialnya, perusahaan kena serangan ransomware — diduga dilakukan kompetitor — dan versi final dari tender document tersebut ke-encrypt. Sekarang mereka butuh expert yang bisa decrypt dokumen krusial ini. Yang tersedia cuma network traffic, ransom note, dan encrypted tender document itu sendiri.

Tools yang dipakai: Wireshark / TShark / TCPDump.

---

## 🎯 Scenario

ABC Industries kerja siang malam selama sebulan buat nyiapin tender document untuk proyek prestisius yang bakal amankan masa depan finansial perusahaan. Perusahaan kena serangan ransomware, diduga dilakukan oleh kompetitor, dan versi final dari tender document tersebut ter-encrypt. Saat ini mereka butuh expert yang bisa decrypt dokumen kritis ini. Artifact yang tersedia: network traffic, ransom note, dan encrypted tender document.

---

## ❓ Questions

1. What is the operating system of the host from which the network traffic was captured? (Look at Capture File Properties, copy the details exactly)
2. What is the full URL from which the ransomware executable was downloaded?
3. Name the ransomware executable file?
4. What is the MD5 hash of the ransomware?
5. What is the name of the ransomware?
6. What is the encryption algorithm used by the ransomware, according to the ransom note?
7. What is the domain beginning with 'd' that is related to ransomware traffic?
8. Decrypt the Tender document and submit the flag

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

- [ ] Analisis network traffic (pcap) untuk trace initial access & delivery ransomware executable
- [ ] Identifikasi ransomware family dari ransom note & encryption algorithm
- [ ] Decrypt tender document dan submit flag

---

## 📚 References

- 🔄 Belum diisi.

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
