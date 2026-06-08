# Shadow Trace — TryHackMe

> **Platform:** TryHackMe  
> **Category:** Malware Analysis / Incident Response  
> **Difficulty:** Medium  
> **Status:** ✅ Completed  
> **Date:** 2026-06-07  
> **Time Spent:** ~X jam  

---

## 📌 Prolog

Ini adalah malam shift tengah malam. Kamu satu-satunya analyst yang ada di SOC ketika seorang manager menelepon dengan urgent: sebuah file mencurigakan ditemukan di mesin pengguna dan butuh review segera.

File dibuka dan investigasi dimulai. Ada yang tidak wajar dari file yang mengaku sebagai company updater ini — dan di waktu yang hampir bersamaan, EDR mulai melempar beberapa alert.

Tugas kita: analisis file tersebut, kumpulkan semua yang bisa dipakai untuk mengidentifikasinya, ekstrak potential IOCs, lalu korelasikan dan analisis alert-alert EDR untuk memahami aktivitas berbahaya yang sedang terjadi. Semua harus diselesaikan sebelum ancaman menyebar lebih jauh.

**Learning Objectives:**
- Extract IOCs from suspicious binaries
- Correlate alerts with malicious activity
- Perform basic triage actions

**Prerequisites:**
- Introduction to Malware Analysis
- MAL: Malware Introductory
- Malware Classification

---

## 🎯 Scenario

Analisis binary yang ada di `C:\Users\DFIRUser\Desktop\windows-update.exe` di mesin yang disediakan. Beberapa tools analisis tersedia di `C:\Users\DFIRUser\DFIR Tools`.

---

## 🗂️ Labs

| Lab | Deskripsi | Status |
|-----|-----------|--------|
| [File Analysis](./File-Analysis/) | Analisis static dan ekstraksi IOC dari binary `windows-update.exe` | ✅ |
| [Alert Analysis](./Alert-Analysis/) | Korelasi dan analisis EDR alerts dari proses `powershell.exe` dan `chrome.exe` | ✅ |

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
