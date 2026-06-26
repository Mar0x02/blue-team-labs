# PowerShell Analysis - Keylogger — BTLO

> **Platform:** [Blue Team Labs Online](https://blueteamlabs.online/home/challenge/powershell-analysis-keylogger-9f4ab9a11c)  
> **Category:** Malware Analysis / PowerShell Analysis  
> **Difficulty:** Easy  
> **Status:** ✅ Completed  
> **Date:** 2026-05-19  
> **Time Spent:** ~1 jam  
> **Tags:** `PowerShell` `Keylogger` `Static Analysis`

---

## 🎯 Scenario

Sebuah PowerShell script yang mencurigakan ditemukan di salah satu endpoint milik organisasi. Tugasnya adalah menganalisis script tersebut secara statis untuk memahami apa yang dilakukannya — apakah ia merekam keystroke, mengirim data ke luar, atau keduanya.

---

## ❓ Questions

1. What is the SHA256 hash of the PowerShell script file?
2. What email address is used to send and receive emails?
3. What is the password of the email account?
4. What is the port used for SMTP?
5. What DLL is imported to help record keystrokes?
6. In what directory is the generated txt file saved?

---

## 🔍 Answer & Walkthrough

### Persiapan: Extract Archive

File diberikan dalam bentuk archive terenkripsi. Ekstrak menggunakan **7zip** dengan password `btlo` atau `infected`.

---

### 1. What is the SHA256 hash of the PowerShell script file?

Setelah file diekstrak, hitung SHA256 hash-nya:

```powershell
Get-FileHash .\script.ps1 -Algorithm SHA256
```

atau di Linux/macOS:

```bash
shasum -a 256 script.ps1
```

**Jawaban:** `e0b7a2ad2320ac32c262aeb6fe2c6c0d75449c6e34d0d18a531157c827b9754e`

---

### 2. What email address is used to send and receive emails?

Buka script di text editor dan cari bagian konfigurasi SMTP. Script ini menggunakan email yang sama sebagai sender sekaligus receiver — ditemukan langsung di variabel konfigurasi email dalam script.

**Jawaban:** `chaudhariparth454@gmail.com`

---

### 3. What is the password of the email account?

Di baris yang sama dengan konfigurasi email, password akun disimpan secara plaintext di dalam script — red flag klasik dari malware yang ditulis terburu-buru.

**Jawaban:** `yjghfdafsd5464562!`

---

### 4. What is the port used for SMTP?

Masih di bagian konfigurasi SMTP, port eksplisit dicantumkan. Port 587 adalah port standar SMTP dengan STARTTLS — umum digunakan malware untuk exfiltrasi data via email agar terlihat seperti traffic legitimate.

**Jawaban:** `587`

---

### 5. What DLL is imported to help record keystrokes?

Cari bagian `[DllImport]` atau `Add-Type` dalam script. PowerShell keylogger biasanya memanggil Windows API dari `user32.dll` untuk intercept keystroke via fungsi seperti `GetAsyncKeyState`.

**Jawaban:** `user32.dll`

---

### 6. In what directory is the generated txt file saved?

Script menyimpan hasil rekaman keystroke ke file txt. Path yang ditemukan di script menggunakan environment variable:

```
$env:temp\keylogger.txt
```

Path ini merujuk ke direktori Temp milik user yang sedang login — biasanya `C:\Users\[username]\AppData\Local\Temp\keylogger.txt`. Lokasi ini dipilih karena jarang dipantau dan jarang dibersihkan secara manual oleh user biasa.

**Jawaban:** `$env:temp\keylogger.txt`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| File Hash (SHA256) | `e0b7a2ad2320ac32c262aeb6fe2c6c0d75449c6e34d0d18a531157c827b9754e` | Hash script keylogger |
| Email | `chaudhariparth454@gmail.com` | Akun exfiltrasi data |
| Password | `yjghfdafsd5464562!` | Credential hardcoded di script |
| SMTP Port | `587` | Port exfiltrasi via Gmail SMTP |
| DLL | `user32.dll` | Digunakan untuk intercept keystroke |
| Output File | `$env:temp\keylogger.txt` | Lokasi penyimpanan hasil rekaman |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Collection | Input Capture: Keylogging | T1056.001 | Script merekam keystroke via `user32.dll` |
| Exfiltration | Exfiltration Over Web Service | T1567 | Data dikirim via Gmail SMTP port 587 |
| Defense Evasion | Scripting: PowerShell | T1059.001 | Menggunakan PowerShell untuk menghindari deteksi binary |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

Script ini adalah PowerShell keylogger sederhana yang bekerja dalam dua fase: **collection** dan **exfiltration**. Fase pertama memanfaatkan `user32.dll` via Windows API untuk merekam setiap keystroke dan menyimpannya ke `$env:temp\keylogger.txt` — lokasi yang jarang dipantau. Fase kedua mengirim hasil rekaman ke email attacker via Gmail SMTP (port 587) menggunakan credential yang di-hardcode langsung di dalam script. Tidak ada obfuscation, tidak ada enkripsi credential — menunjukkan script ini dibuat cepat atau untuk target yang dianggap tidak akan menginvestigasi terlalu dalam.

### Todo / Follow-up

- [ ] Pelajari cara PowerShell keylogger intercept keystroke via `GetAsyncKeyState` di `user32.dll`
- [ ] Eksplorasi teknik deteksi PowerShell malware: script block logging, AMSI, dan event ID 4104
- [ ] Coba buat Sigma rule untuk mendeteksi pola SMTP exfiltration dari PowerShell

---

## 📚 References

- [MITRE ATT&CK — T1056.001 Keylogging](https://attack.mitre.org/techniques/T1056/001/)
- [MITRE ATT&CK — T1059.001 PowerShell](https://attack.mitre.org/techniques/T1059/001/)
- [BTLO — PowerShell Analysis - Keylogger](https://blueteamlabs.online/home/challenge/powershell-analysis-keylogger-9f4ab9a11c)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
