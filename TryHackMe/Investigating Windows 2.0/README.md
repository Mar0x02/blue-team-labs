# Investigating Windows 2.0 — TryHackMe

> **Platform:** TryHackMe  
> **Category:** Digital Forensics / Malware Analysis / Threat Hunting  
> **Difficulty:** Medium  
> **Status:** ✅ Completed  
> **Date:** 2026-06-20  
> **Time Spent:** ~4 jam  

---

## 📌 Prolog

Challenge lanjutan dari seri Investigating Windows. Kali ini investigasi lebih dalam — melibatkan analisis registry, scheduled tasks, WMI persistence, process monitoring dengan Sysinternals, sampai binary scanning dengan Loki dan Yara. Prerequisite yang disarankan: room Core Windows Processes, Sysinternals, dan Yara. Akses ke mesin via RDP.

---

## 🎯 Scenario

Sebuah mesin Windows dicurigai telah terkompromi. Investigasi dilakukan langsung dari dalam mesin menggunakan berbagai tools forensik: registry analysis, Sysinternals (Process Monitor, Process Explorer), Loki scanner, dan Yara rules.

Tugas kita adalah mengidentifikasi persistence mechanism yang digunakan attacker, script berbahaya yang berjalan di background, binary yang ter-flag sebagai malicious, hingga melengkapi Yara rule untuk mendeteksi binary yang luput dari scan Loki.

---

## ❓ Questions

1. What registry key contains the same command that is executed within a scheduled task?
2. What analysis tool will immediately close if/when you attempt to launch it?
3. What is the full WQL Query associated with this script?
4. What is the script language?
5. What is the name of the other script?
6. What is the name of the software company visible within the script?
7. What 2 websites are associated with this software company? (answer, answer)
8. Search online for the name of the script from Q5 and one of the websites from the previous answer. What attack script comes up in your search?
9. What is the location of this file within the local machine?
10. Which 2 processes open and close very quickly every few minutes? (answer, answer)
11. What is the parent process for these 2 processes?
12. What is the first operation for the first of the 2 processes?
13. Inspect the properties for the 1st occurrence of this process. In the Event tab what are the 4 pieces of information displayed? (answer, answer, answer, answer)
14. Inspect the disk operations, what is the name of the unusual process?
15. Run Loki. Inspect the output. What is the name of the module after `Init`?
16. Regarding the 2nd warning, what is the name of the eventFilter?
17. For the 4th warning, what is the class name?
18. What binary alert has the following 4d5a90000300000004000000ffff0000b8000000 as FIRST_BYTES?
19. According to the results, what is the description listed for reason 1?
20. Which binary alert is marked as APT Cloaked?
21. What are the matches? (str1, str2)
22. Which binary alert is associated with somethingwindows.dmp found in C:\TMP?
23. Which binary is encrypted that is similar to a trojan?
24. There is a binary that can masquerade itself as a legitimate core Windows process/image. What is the full path of this binary?
25. What is the full path location for the legitimate version?
26. What is the description listed for reason 1?
27. There is a file in the same folder location that is labeled as a hacktool. What is the name of the file?
28. What is the name of the Yara Rule MATCH?
29. Which binary didn't show in the Loki results?
30. Complete the yar rule file located within the Tools folder on the Desktop. What are 3 strings to complete the rule in order to detect the binary Loki didn't hit on? (answer, answer, answer)

---

## 🔍 Answer & Walkthrough

### 1–5. Autoruns: Persistence & WMI Investigation

Mulai dari `autoruns.exe`, buka tab **Everything** supaya semua entry persistence muncul sekaligus. Langsung ada beberapa yang suspicious:

- **`HKCU\Environment\UserInitMprLogonScript`** → menjalankan `C:\TMP\mim.exe` setiap kali user logon
- **`HKLM\SOFTWARE\Microsoft\Windows\Current Version\Run`** → dua entry:
  - `BadrClient`: `wscript.exe "C:\badr\start-badr.vbs" //B //Nologo`
  - `UpdateSvc`: `C:\TMP\p.exe -s \\10.34.2.3 'net user' > C:\TMP\o2.txt`
- **Scheduled Tasks**: `\BADR`, `\BadrClient`, `\Clean file system` (kontennya mirip netcat), dan `\falshupdate` yang trigger PowerShell setiap 2 menit

![Registry & Scheduled Task](./assets/1.cek-registry-impact.png)
![Schtask dengan command yang sama](./assets/1.cek-schtask-with-same-cmnd.png)

Di bagian **WMI Database Entries** ada 2 script:
- Script pertama: `Kill Process` — WQL-nya `SELECT * FROM Win32_ProcessStartTrace WHERE ProcessName = 'procexp64.exe'`. Setiap kali `procexp64.exe` (Process Explorer) dibuka, langsung di-kill.
- Script kedua: `LaunchBeaconingBackdoor` — nama yang cukup terang-terangan untuk sebuah persistence mechanism. Kedua script ini ditulis dalam VBScript.

![WMI Kill Process & LaunchBeaconingBackdoor](./assets/2.3.4.jawbaan-nya-terkait-kill-process.png)

**Jawaban:**

1. `HKCU\Environment\UserInitMprLogonScript`
2. `procexp64.exe`
3. `SELECT * FROM Win32_ProcessStartTrace WHERE ProcessName = 'procexp64.exe'`
4. `VBScript`
5. `LaunchBeaconingBackdoor`

---

### 6–9. Script Analysis & OSINT

Buka file `LaunchBeaconingBackdoor` di Notepad — yang muncul adalah file temporary bernama `WMIB1F5.tmp`. Di dalamnya terlihat nama perusahaan **Motobit Software** beserta dua websitenya.

![Company Name](./assets/5.company-name.png)
![LaunchBeaconingBackdoor Script](./assets/6.another-ecript.png)
![Website 1](./assets/7.website-name-1.png)
![Website 2](./assets/7.website-name-2.png)

Coba Google dengan query `LaunchBeaconingBackdoor motobit.com` — langsung ketemu repo GitHub dengan file bernama `WMIBackdoor.ps1`. Ini adalah APT Simulator tool, bukan malware murni, tapi tetap tidak bisa langsung dipercaya hanya karena ada di repo publik. File tersebut ada di folder `C:\TMP` di mesin ini.

![GitHub Search Result](./assets/8.githu-repo-about-malware.png)
![File di C:\TMP](./assets/9.cek-in-folder.png)
![Folder malware](./assets/9.get-folder-malware-name.png)

**Jawaban:**

6. `Motobit Software`
7. `http://www.motobit.com`, `http://Motobit.cz`
8. `WMIBackdoor.ps1`
9. `C:\TMP`

---

### 10–14. Process Monitoring

Ada dua process yang terus muncul dan langsung tutup setiap beberapa menit: `mim.exe` yang dijalankan via `cmd.exe`, dan `powershell.exe`.

![mim.exe process](./assets/10.mim-exe.png)
![powershell.exe process](./assets/10.powershell-exe.png)

Untuk menangkap detail process ini, digunakan **Process Monitor** (`procmon.exe`/`procmon64.exe`) — bukan Process Explorer, karena `procexp64.exe` langsung di-kill oleh WMI script. Supaya bisa membuka Process Explorer, rename dulu binary-nya sehingga nama file tidak match dengan WQL filter di WMI.

Parent process dari kedua proses tersebut adalah `svchost.exe`.

![Parent Process](./assets/11.parent-malware.png)

Di Process Monitor, buka properties untuk kemunculan pertama `mim.exe`. Operasi pertama yang tercatat adalah **Process Start**. Di tab **Event**, ada 4 field yang ditampilkan: Parent PID, Command line, Current directory, dan Environment.

![Jalankan Procmon](./assets/12.run-procmon.png)
![Event Properties - Process](./assets/12.event-properties-process.png)
![Event Properties - Conhost Process](./assets/12.event-properties-conhost-process.png)
![Event Properties - PowerShell](./assets/12.event-properties-powershell.png)
![Event Properties - Conhost](./assets/12.13.event-properties-conhost.png)

Saat cek disk operations di Process Monitor, ada entry dengan nama process **No Process** — ini terjadi karena process-nya sudah terminate sebelum Process Monitor sempat mengidentifikasinya. Perilaku ini sendiri sudah suspicious.

![Disk Inspection](./assets/14.disk-inspection.png)

Untuk analisis yang lebih cepat, `ProcessHacker.exe` juga dicoba — tapi ini lebih tricky karena ada sesuatu yang menyebabkannya terus diminta untuk ditutup. Triknya: buka sebelum malware trigger berikutnya, analisis secepat mungkin.

**Jawaban:**

10. `mim.exe`, `powershell.exe`
11. `svchost.exe`
12. `Process Start`
13. `Parent PID`, `Command line`, `Current directory`, `Environment`
14. `No Process`

---

### 15–28. Loki Scan

Jalankan **Loki IOC Scanner** dari folder Tools di Desktop. Total 682 Yara rules di-load. Setelah proses `Init` selesai, modul pertama yang dijalankan adalah **WMIScan** — langsung mendeteksi dua event yang kita temukan di Autoruns: `ProcessStartTrigger` dan `LaunchBeaconingBackdoor`. Detail di Loki lebih lengkap dari di Autoruns.

![Module setelah Init](./assets/15.after-init-module.png)
![WMI Warning ke-2: ProcessStartTrigger](./assets/16.warning-name-2nd.png)
![Warning ke-4: __FilterToConsumerBinding](./assets/17.alert-function-name.png)

Setelah scan WMI, Loki scan memory — semua process di memory aman. Lanjut ke disk scan, mulai banyak alert:

- **`C:\inetpub\wwwroot\b.js`** — Yara rule: WebShell, score 70, VirusTotal 28/62
- **`C:\inetpub\wwwroot\tests.jsp`** — WebShell dengan parameter `cmd`
- **`C:\TMP\mim-out.txt`** — bukan binary, tapi log output Mimikatz, score 80
- **`C:\TMP\nbtscan.exe`** — match nama IOC, pernah dipakai APT Group. FIRST_BYTES: `4d5a90000300000004000000ffff0000b8000000`. Reason 1: *Known Bad / Dual use classics*

![nbtscan first bytes](./assets/18.first-byte-bin-alert.png)
![Description reason 1](./assets/19.desc-of-first-byte.png)

- **`C:\TMP\p.exe`** — marked **APT Cloaked**. Matches: `psexesvc.exe` dan `Sysinternals PsExec`

![APT Cloaked](./assets/20.marked-apt-cloaked.png)
![Matches](./assets/21.matches-str.png)

- **`C:\TMP\schtasks-backdoor.ps1`** — malware family `shellrunner`, terkait dengan `somethingwindows.dmp` di `C:\TMP`

![schtasks-backdoor.ps1](./assets/22.png)

- **`C:\TMP\xCmd.exe`** — encrypted, VirusTotal 61/71, family label: DarkComet trojan

![xCmd.exe](./assets/23.png)

- **`C:\Users\Public\svchost.exe`** — masquerade sebagai core Windows process. Legitimate version ada di `C:\Windows\System32`. Reason 1: *Stuff running where it normally shouldn't*

![svchost.exe masquerade](./assets/24.png)
![Legitimate path](./assets/25.png)
![Reason description](./assets/26.png)

- **`C:\Users\Public\en-US.js`** — hacktool CactusTorch, VirusTotal 34/53. Yara Rule Match: **CACTUSTORCH**

![en-US.js hacktool](./assets/27.png)
![CACTUSTORCH Yara match](./assets/28.png)

Di folder `\\AppData\\Local\\Microsoft\\Windows\\Explorer\\` ditemukan 31 file yang pattern-nya match dengan pola simpan malware APT — perlu diverifikasi satu per satu mana yang legitimate.

Di folder PowerGUI temp, ada beberapa instance `mk.ps1` dengan deteksi execute encode base84 dan indikasi Mimikatz — kemungkinan ini adalah mekanisme Replication atau Persistence tambahan.

**Jawaban:**

15. `WMIScan`
16. `ProcessStartTrigger`
17. `__FilterToConsumerBinding`
18. `nbtscan.exe`
19. `Known Bad / Dual use classics`
20. `p.exe`
21. `psexesvc.exe`, `Sysinternals PsExec`
22. `schtasks-backdoor.ps1`
23. `x.Cmd.exe`
24. `C:\Users\Public\svchost.exe`
25. `C:\Windows\System32`
26. `Stuff running where it normally shouldn't`
27. `en-US.js`
28. `CACTUSTORCH`

---

### 29–30. Custom Yara Rule — mim.exe

`mim.exe` yang terus-menerus muncul di cmd.exe ternyata **tidak terdeteksi oleh Loki**. Kalau diperhatikan lebih lama, window `cmd.exe`-nya sempat menampilkan nama **Mimikatz** sebelum tutup — jadi kita tahu ini Mimikatz yang di-rename.

Buka Yara rules di `Desktop\Tools\yara-v4.0.4-1544-win64`, lihat `test.yar`. Ada rule Mimikatz di sana. Gunakan `strings.exe` atau `strings64.exe` dari Sysinternals untuk dump readable strings dari `C:\TMP\mim.exe`, lalu `findstr` untuk cocokkan dengan string yang ada di `test.yar`.

Tiga string yang perlu ditambahkan ke rule agar bisa detect `mim.exe`:

![Yara rules check](./assets/30.yara-rules-cek.png)

**Jawaban:**

29. `mim.exe`
30. `mk.ps1`, `mk.exe`, `v2.0.50727`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| Registry Key | `HKCU\Environment\UserInitMprLogonScript` | Logon persistence — eksekusi `mim.exe` tiap logon |
| Registry Key | `HKLM\SOFTWARE\Microsoft\Windows\Current Version\Run` | `BadrClient` & `UpdateSvc` persistence |
| File | `C:\TMP\mim.exe` | Mimikatz, tidak terdeteksi Loki |
| File | `C:\TMP\mim-out.txt` | Output dump Mimikatz, score 80 |
| File | `C:\TMP\p.exe` | PsExec, flagged APT Cloaked |
| File | `C:\TMP\nbtscan.exe` | NetBIOS scanner, IOC match APT |
| File | `C:\TMP\schtasks-backdoor.ps1` | Shellrunner family |
| File | `C:\TMP\xCmd.exe` | Encrypted, DarkComet trojan family |
| File | `C:\Users\Public\svchost.exe` | Masquerade sebagai core Windows process |
| File | `C:\Users\Public\en-US.js` | CactusTorch hacktool |
| File | `C:\inetpub\wwwroot\b.js` | WebShell, VirusTotal 28/62 |
| File | `C:\inetpub\wwwroot\tests.jsp` | WebShell dengan parameter `cmd` |
| WMI Event | `ProcessStartTrigger` | Filter kill `procexp64.exe` |
| WMI Event | `__FilterToConsumerBinding` | Binding class untuk WMI persistence |
| IP | `10.34.2.3` | Target lateral movement via `p.exe` |
| Domain | `http://www.motobit.com` | Referensi di WMI backdoor script |
| Domain | `http://Motobit.cz` | Referensi di WMI backdoor script |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Persistence | Logon Script (Windows) | T1037.001 | `UserInitMprLogonScript` → `mim.exe` |
| Persistence | Registry Run Keys | T1547.001 | `BadrClient` & `UpdateSvc` di HKLM Run |
| Persistence | Scheduled Task | T1053.005 | `\BADR`, `\Clean file system`, `\falshupdate` |
| Persistence | WMI Event Subscription | T1546.003 | `ProcessStartTrigger` & `LaunchBeaconingBackdoor` |
| Defense Evasion | Masquerading | T1036.005 | `svchost.exe` di `C:\Users\Public\` |
| Defense Evasion | Indicator Removal | T1070 | WMI kill `procexp64.exe` saat dibuka |
| Credential Access | OS Credential Dumping | T1003.001 | `mim.exe` (Mimikatz) dump LSASS |
| Execution | Windows Management Instrumentation | T1047 | WMI eksekusi backdoor script |
| Discovery | Network Service Scanning | T1046 | `nbtscan.exe` untuk NetBIOS scanning |
| Lateral Movement | Remote Services | T1021 | `p.exe` (PsExec) ke `\\10.34.2.3` |
| Command & Control | Web Shell | T1505.003 | `b.js` & `tests.jsp` di IIS webroot |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

**Initial Access:** Attacker masuk melalui WebShell (`b.js` dan `tests.jsp`) yang di-deploy ke `C:\inetpub\wwwroot` — memanfaatkan IIS yang terekspos.

**Persistence:** Multiple layer persistence dipasang — registry logon script (`UserInitMprLogonScript → mim.exe`), dua Run key (`BadrClient` via wscript dan `UpdateSvc` via PsExec), scheduled tasks (`\BADR`, `\falshupdate` setiap 2 menit, `\Clean file system` yang fungsinya mirip netcat), dan WMI Event Subscription (`LaunchBeaconingBackdoor` via `WMIBackdoor.ps1`).

**Defense Evasion:** WMI event `ProcessStartTrigger` aktif kill `procexp64.exe` setiap kali dibuka — attacker tahu analis akan pakai Process Explorer. Selain itu, `svchost.exe` palsu disimpan di `C:\Users\Public\` untuk masquerade sebagai core Windows process.

**Credential Access:** `mim.exe` (Mimikatz yang di-rename) dijalankan secara berkala via scheduled task, outputnya disimpan ke `mim-out.txt`. Ini yang tidak terdeteksi Loki karena filename-nya di-obfuscate.

**Discovery:** `nbtscan.exe` digunakan untuk NetBIOS scanning — kemungkinan recon terhadap jaringan internal.

**Lateral Movement:** `p.exe` (PsExec yang di-rename, flagged APT Cloaked) dijalankan dengan command `net user` ke target `\\10.34.2.3`. `xCmd.exe` juga tersedia sebagai alternatif remote command execution.

**Persistence via Replication:** Beberapa instance `mk.ps1` ditemukan di folder temp PowerGUI — kemungkinan mekanisme replication untuk maintain presence.

### Todo / Follow-up

- [ ] Pelajari WMI Event Subscription lebih dalam — consumer types (CommandLineEventConsumer vs ActiveScriptEventConsumer)
- [ ] Latihan tulis custom Yara rule dari scratch, tidak hanya edit yang sudah ada
- [ ] Dalami cara kerja `strings.exe` + `findstr` untuk binary analysis manual
- [ ] Coba reproduce rename trick untuk bypass WMI kill — experiment di lab sendiri
- [ ] Review APT Simulator repo untuk memahami TTP mana yang di-simulate di room ini
- [ ] Pelajari cara ProcessHacker bisa di-force-open saat ada kill mechanism — timing attack atau method lain

---

## 📚 References

- [TryHackMe — Core Windows Processes](https://tryhackme.com/room/btwindowsinternals)
- [TryHackMe — Sysinternals](https://tryhackme.com/room/btsysinternalssg)
- [TryHackMe — Yara](https://tryhackme.com/room/yara)
- [Loki IOC Scanner](https://github.com/Neo23x0/Loki)
- [MITRE ATT&CK — WMI Event Subscription T1546.003](https://attack.mitre.org/techniques/T1546/003/)
- [MITRE ATT&CK — Masquerading T1036.005](https://attack.mitre.org/techniques/T1036/005/)
- [MITRE ATT&CK — OS Credential Dumping T1003](https://attack.mitre.org/techniques/T1003/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
