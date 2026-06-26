# ILOVEYOU — BTLO

> **Platform:** [Blue Team Labs Online](https://blueteamlabs.online/home/challenge/iloveyou-b9b3e99c9b)  
> **Category:** Malware Analysis  
> **Difficulty:** Medium  
> **Status:** ✅ Completed  
> **Date:** 2026-05-19  
> **Time Spent:** ~2 jam  
> **Tags:** `VBScript` `Worm` `Static Analysis`  
> ⚠️ **Peringatan:** Challenge ini mengandung real malware — kerjakan di dalam virtual machine!

---

## 📌 Prolog

ILOVEYOU adalah salah satu malware paling terkenal dalam sejarah cybersecurity. Worm VBScript ini menyebar pada 4 Mei 2000, menginfeksi lebih dari 50 juta komputer hanya dalam hitungan jam dengan kerugian diestimasi $10 miliar. Dibuat oleh Onel de Guzman dari Filipina, malware ini menyebar via email dengan subject "ILOVEYOU" dan attachment `LOVE-LETTER-FOR-YOU.TXT.vbs`. Challenge ini mengajak kita menganalisis kodenya secara statis — sebuah cara yang baik untuk memahami bagaimana worm klasik bekerja dari sisi teknis.

---

## 🎯 Scenario

"ILOVEYOU" — tiga kata ajaib yang berdampak besar bagi kehidupan kebanyakan orang. Di sisi lain, tiga kata ini tidak membutuhkan perkenalan bagi siapapun yang berkecimpung di industri Infosec. Mari kita mengenang sejarah dengan menganalisis malware **ILOVEYOU**.

---

## ❓ Questions

1. What is the text present as part of email when the victim received this malware?
2. What is the domain name that was added as the browser's homepage?
3. The malware replicated itself into 3 locations, what are they?
4. What is the name of the file that looks for the filesystem?
5. Which file extensions, beginning with m, does this virus target?
6. What is the name of the file generated when the malware identifies any Internet Relay Chat service?
7. What is the name of the password stealing trojan that is downloaded by the malware?
8. What is the name of the email service that is targeted by the malware?
9. What is the registry entry responsible for reading the contacts of the logged in email account?
10. What is the value that is stored in the registry to remember that an email was already sent to a user?

---

## 🔍 Answer & Walkthrough

Sebelum menjawab pertanyaan, penting untuk memahami struktur malware ini. ILOVEYOU terdiri dari beberapa subroutine utama:

```
Korban buka LOVE-LETTER-FOR-YOU.TXT.vbs
    ↓
main() dipanggil
    ↓
├── regruns()       → persistence + download trojan
├── html()          → buat LOVE-LETTER-FOR-YOU.HTM
├── spreadtoemail() → kirim ke semua kontak Outlook
└── listadriv()     → telusuri semua drive → infectfiles()
```

Semua jawaban diperoleh dari static analysis — buka script di text editor dan telusuri tiap subroutine.

---

### 1. What is the text present as part of email when the victim received this malware?

Cari di fungsi `spreadtoemail()` bagian `male.Body`. Di sana terlihat teks yang di-set sebagai body email yang dikirim ke semua kontak Outlook korban.

**Jawaban:** `kindly check the attached LOVELETTER coming from me.`

---

### 2. What is the domain name that was added as the browser's homepage?

Cari di fungsi `regruns()` bagian `regcreate` untuk key `StartPage`. Malware secara acak memilih salah satu dari 4 URL di domain yang sama untuk dijadikan homepage Internet Explorer — tujuannya men-download trojan BAROK. Setelah download selesai, homepage di-reset ke `about:blank` untuk menghapus jejak.

**Jawaban:** `www.skyinet.net`

---

### 3. The malware replicated itself into 3 locations, what are they?

Cari di fungsi `main()` bagian `c.Copy`. Malware menggunakan `GetSpecialFolder(0)` untuk Windows folder dan `GetSpecialFolder(1)` untuk System32 folder. Nama file sengaja dibuat menyerupai file sistem Windows (teknik masquerading):

```
C:\Windows\System32\MSKernel32.vbs
C:\Windows\Win32DLL.vbs
C:\Windows\System32\LOVE-LETTER-FOR-YOU.TXT.vbs
```

**Jawaban:** `C:\Windows\System32\MSKernel32.vbs, C:\Windows\Win32DLL.vbs, C:\Windows\System32\LOVE-LETTER-FOR-YOU.TXT.vbs`

---

### 4. What is the name of the file that looks for the filesystem?

Cari di fungsi `regruns()` bagian `fileexist`. Sebelum men-download trojan, malware mengecek keberadaan file ini di System32 folder sebagai mekanisme **anti-reinfection** — jika sudah ada, berarti sistem sudah pernah terinfeksi dan trojan tidak perlu di-download ulang.

**Jawaban:** `WinFAT32.exe`

---

### 5. Which file extensions, beginning with m, does this virus target?

Cari di fungsi `infectfiles()` bagian pengecekan extension. Untuk file `.mp3` dan `.mp2`, malware tidak menghapusnya — melainkan menyembunyikan file asli (hidden attribute) lalu membuat file `.vbs` baru dengan nama yang sama. Korban tidak sadar file musiknya sudah "digantikan".

**Jawaban:** `mp3, mp2`

---

### 6. What is the name of the file generated when the malware identifies any Internet Relay Chat service?

Cari di fungsi `infectfiles()` bagian pengecekan file mIRC (`mirc32.exe`, `mlink32.exe`, `mirc.ini`, dll). Ketika ditemukan instalasi mIRC, malware membuat file ini yang berisi script mIRC untuk otomatis mengirim `LOVE-LETTER-FOR-YOU.HTM` ke semua user yang join IRC channel via DCC.

**Jawaban:** `script.ini`

---

### 7. What is the name of the password stealing trojan that is downloaded by the malware?

Cari di fungsi `html()` pada meta tag yang di-generate. Di sana ada string `CONTENT=BAROK VBS - LOVELETTER` yang mengungkap nama trojan yang di-download dari `skyinet.net` dalam bentuk file `WIN-BUGSFIX.exe`.

**Jawaban:** `BAROK`

---

### 8. What is the name of the email service that is targeted by the malware?

Cari di fungsi `spreadtoemail()` bagian `CreateObject`. Malware menggunakan `WScript.CreateObject("Outlook.Application")` dan MAPI untuk mengakses address book dan mengirim email ke seluruh kontak korban secara otomatis.

**Jawaban:** `Outlook`

---

### 9. What is the registry entry responsible for reading the contacts of the logged in email account?

Cari di fungsi `spreadtoemail()` bagian `RegRead`. WAB (Windows Address Book) adalah registry key yang menyimpan informasi kontak email. Malware membaca key ini untuk mendapatkan daftar target, sekaligus menulis ke key yang sama setelah email terkirim untuk mencegah duplikasi.

**Jawaban:** `HKEY_CURRENT_USER\Software\Microsoft\WAB\`

---

### 10. What is the value that is stored in the registry to remember that an email was already sent to a user?

Cari di fungsi `spreadtoemail()` bagian `RegWrite` setelah email berhasil dikirim. Sebelum mengirim ke kontak manapun, malware selalu mengecek nilai ini terlebih dahulu — jika sudah `1`, email tidak akan dikirim ulang ke kontak yang sama.

**Jawaban:** `1`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| File | `LOVE-LETTER-FOR-YOU.TXT.vbs` | File utama malware |
| File | `WIN-BUGSFIX.exe` | Trojan BAROK yang di-download |
| File | `WinFAT32.exe` | Marker anti-reinfection |
| Domain | `www.skyinet.net` | Server download trojan |
| Registry | `HKLM\...\Run\MSKernel32` | Persistence startup |
| Registry | `HKLM\...\RunServices\Win32DLL` | Persistence startup |
| Registry | `HKCU\Software\Microsoft\WAB\` | Tracking kontak yang sudah dikirim |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Execution | Scripting: Visual Basic | T1059.005 | Seluruh malware ditulis dalam VBScript |
| Persistence | Registry Run Keys | T1547.001 | `MSKernel32` dan `Win32DLL` di Run/RunServices |
| Defense Evasion | Masquerading | T1036 | Nama file menyerupai file sistem Windows |
| Collection | Data from Local System | T1005 | Infeksi dan pengumpulan file di semua drive |
| Lateral Movement | Replication Through Removable Media | T1091 | Menyebar ke network drive |
| Exfiltration | Exfiltration Over C2 Channel | T1041 | Trojan BAROK mencuri dan mengirim credential |
| Impact | Data Destruction | T1485 | File jpg/jpeg dihapus permanen |
| Command and Control | Ingress Tool Transfer | T1105 | Download `WIN-BUGSFIX.exe` dari skyinet.net |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

ILOVEYOU mengeksekusi empat fase serangan secara bersamaan begitu script dijalankan. **Persistence** dilakukan via registry Run/RunServices agar worm tetap aktif setiap kali Windows startup. **Spreading** terjadi di tiga vektor sekaligus: Outlook mengirim email ke semua kontak korban, mIRC menyebarkan HTM file ke semua user di channel, dan worm menimpa file di seluruh filesystem termasuk network drive. **Payload destruktif** menghapus semua file `.jpg`/`.jpeg` secara permanen dan menyembunyikan file `.mp3`/`.mp2`. **Credential theft** dilakukan via trojan BAROK yang di-download dari `skyinet.net` dan berjalan diam-diam di background. Mekanisme anti-duplikasi via registry WAB memastikan satu kontak hanya menerima satu email — cukup cerdas untuk worm tahun 2000.

### Todo / Follow-up

- [ ] Pelajari lebih dalam teknik VBScript untuk mengakses Windows API dan filesystem
- [ ] Eksplorasi cara kerja Windows Address Book (WAB) dan MAPI
- [ ] Coba jalankan di VM dengan network monitoring untuk melihat traffic ke `skyinet.net`
- [ ] Buat detection rule (Sigma/YARA) berdasarkan IOC yang ditemukan

---

## 📚 References

- [MITRE ATT&CK — T1059.005 Visual Basic](https://attack.mitre.org/techniques/T1059/005/)
- [MITRE ATT&CK — T1547.001 Registry Run Keys](https://attack.mitre.org/techniques/T1547/001/)
- [BTLO — ILOVEYOU Challenge](https://blueteamlabs.online/home/challenge/iloveyou-b9b3e99c9b)
- [Wikipedia — ILOVEYOU](https://en.wikipedia.org/wiki/ILOVEYOU)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
