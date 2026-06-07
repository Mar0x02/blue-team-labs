# Shadow Trace — File Analysis

> **Parent:** [Shadow Trace](../)  
> **Lab:** File Analysis  
> **Target:** `C:\Users\DFIRUser\Desktop\windows-update.exe`  
> **Status:** 🔄 In Progress  

---

## 🎯 Scenario

Analisis binary `windows-update.exe` yang ditemukan di mesin pengguna. File ini mengaku sebagai company updater — tugas kita membuktikan sebaliknya dan mengekstrak semua IOC yang bisa digunakan untuk deteksi dan reporting.

Tools tersedia di `C:\Users\DFIRUser\DFIR Tools`.

---

## ❓ Questions

1. What is the architecture of the binary file windows-update.exe?
2. What is the hash (sha-256) of the file windows-update.exe?
3. Identify the URL within the file to use it as an IOC
4. With the URL identified, can you spot a domain that can be used as an IOC?
5. Input the decoded flag from the suspicious domain
6. What library related to socket communication is loaded by the binary?

---

## 🔍 Answer & Walkthrough

> 🔄 *Belum diisi — akan dilengkapi setelah pengerjaan selesai.*

---

### 1. What is the architecture of the binary file windows-update.exe?

**Jawaban:** `...`

---

### 2. What is the hash (sha-256) of the file windows-update.exe?

**Jawaban:** `...`

---

### 3. Identify the URL within the file to use it as an IOC

**Jawaban:** `...`

---

### 4. With the URL identified, can you spot a domain that can be used as an IOC?

**Jawaban:** `...`

---

### 5. Input the decoded flag from the suspicious domain

**Jawaban:** `...`

---

### 6. What library related to socket communication is loaded by the binary?

**Jawaban:** `...`

---

## 🚨 Key Findings / IOCs

> 🔄 *Belum diisi.*

| Tipe | Value | Keterangan |
|------|-------|------------|
| SHA-256 | `...` | Hash windows-update.exe |
| URL | `...` | URL embedded dalam binary |
| Domain | `...` | Domain IOC |
| Library | `...` | Socket library yang di-load |

---

## 📚 References

- [PE File Format](https://learn.microsoft.com/en-us/windows/win32/debug/pe-format)
- [VirusTotal](https://www.virustotal.com/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
