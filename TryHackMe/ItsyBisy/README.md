# ItsyBisy — TryHackMe

> **Platform:** TryHackMe  
> **Category:** Network Analysis / SIEM / Log Analysis  
> **Difficulty:** Medium  
> **Status:** 🔄 In Progress  
> **Date:** 2026-06-15  
> **Time Spent:** ~X jam  

---

## 📌 Prolog

Challenge ini datang dari alert IDS yang mendeteksi potensi komunikasi C2 dari salah satu user di departemen HR. Investigasi dilakukan menggunakan Kibana — menelusuri HTTP connection logs selama satu minggu untuk menemukan file yang diakses, URL C2, dan secret code yang tersembunyi di dalamnya.

---

## 🎯 Scenario

Selama monitoring rutin, analis SOC bernama John mendeteksi alert dari IDS yang mengindikasikan kemungkinan komunikasi C2 dari user bernama Browne di departemen HR. Ditemukan sebuah file mencurigakan yang diakses dan mengandung pola malicious dengan format `THM:{ ________ }`.

Sebagai langkah investigasi, log koneksi HTTP selama satu minggu berhasil dikumpulkan dan di-ingest ke dalam index `connection_logs` di Kibana. Karena keterbatasan resource, hanya connection logs yang tersedia — tidak ada artifact lain.

Tugas kita: telusuri log koneksi jaringan user tersebut, temukan link dan konten file yang diakses, lalu jawab pertanyaan-pertanyaan berikut.

---

## ❓ Questions

1. How many events were returned for the month of March 2022?
2. What is the IP associated with the suspected user in the logs?
3. The user's machine used a legit windows binary to download a file from the C2 server. What is the name of the binary?
4. The infected machine connected with a famous filesharing site in this period, which also acts as a C2 server used by the malware authors to communicate. What is the name of the filesharing site?
5. What is the full URL of the C2 to which the infected host is connected?
6. A file was accessed on the filesharing site. What is the name of the file accessed?
7. The file contains a secret code with the format THM{_____}.

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

- [TryHackMe — ItsyBisy Room](https://tryhackme.com/room/itsybisy)
- [Kibana — Discover Tab Documentation](https://www.elastic.co/guide/en/kibana/current/discover.html)
- [MITRE ATT&CK — T1105: Ingress Tool Transfer](https://attack.mitre.org/techniques/T1105/)
- [MITRE ATT&CK — T1071: Application Layer Protocol](https://attack.mitre.org/techniques/T1071/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
