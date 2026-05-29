# Follina (Challenges) — BTLO

> **Platform:** Blue Team Labs Online  
> **Category:** Malware Analysis  
> **Difficulty:** Medium  
> **Status:** ✅ Completed  
> **Date:** 2026-05-29  
> **Time Spent:** ~2 jam  

---

## 📌 Prolog

Challenge ini meminta analisis sample dokumen Word berbahaya yang mengeksploitasi CVE-2022-30190 — aka **Follina**. Vulnerability yang sempat jadi headline besar di 2022 karena tidak butuh macro, tidak butuh klik lebih dari sekadar buka file. Sample diekstrak dari `Challenge.zip` (password: `infected`), dan karena VM Alpine Linux pakai BusyBox unzip yang tidak support flag `-P`, file diekstrak langsung di Mac lalu dipindah ke VM via shared folder UTM.

---

## 🎯 Scenario

Tim kamu mendapat alert tentang RCE vulnerability yang aktif dieksploitasi. Kamu ditugaskan menganalisis sample untuk mengumpulkan informasi bagi weekend team.

---

## ❓ Questions

1. What is the SHA1 hash of the sample?
2. What is the full file type of the sample?
3. What is the URL extracted from the sample?
4. What is the name of the XML file that contains the malicious URL?
5. What is the minimum number of bytes required to trigger the payload?
6. What process is killed by the payload?
7. Which processes are used to create a detection rule using Event ID 4688?
8. What is the MITRE ATT&CK Technique ID for the execution method?
9. What is the CVE for this vulnerability?

---

## 🔍 Answer & Walkthrough

### 1. What is the SHA1 hash of the sample?

Setelah file `simple.doc` tersedia di VM, jalankan:

```bash
sha1sum simple.doc
```

SHA1 dipakai sebagai fingerprint untuk identifikasi malware di platform threat intelligence seperti VirusTotal dan any.run.

**Jawaban:** `06727ffda60359236a8029e0b3e8a0fd11c23313`

---

### 2. What is the full file type of the sample?

Upload hash ke VirusTotal → tab **Details** → **File type**. Meskipun ekstensinya `.doc`, file ini sebenarnya format Office Open XML — format berbasis ZIP yang berisi XML files. Ini penting karena menentukan teknik analisis yang digunakan.

**Jawaban:** `Office Open XML Document`

---

### 3. What is the URL extracted from the sample?

**Langkah 1 — Konfirmasi bahwa file adalah ZIP**

```bash
file simple.doc
# Output: simple.doc: Zip archive data, at least v2.0 to extract
```

Bisa juga cek magic bytes:

```bash
xxd simple.doc | head -2
# File ZIP selalu diawali 50 4B 03 04 (ASCII: PK)
```

**Langkah 2 — Extract dan cari URL**

```bash
unzip simple.doc -d sample_extracted
grep -r "Target=" sample_extracted/
```

Di `word/_rels/document.xml.rels` ditemukan entry dengan `TargetMode="External"` yang mengarah ke URL eksternal. `TargetMode="External"` berarti dokumen akan kontak server luar saat dibuka — red flag.

**Jawaban:** `https://www.xmlformats.com/office/word/2022/wordprocessingDrawing/RDF821.html`

---

### 4. What is the name of the XML file that contains the malicious URL?

Dari hasil grep di Q3, URL ditemukan di path `word/_rels/document.xml.rels`. File `.rels` menyimpan relasi antar komponen dokumen Office — sering digunakan malware untuk menyembunyikan external URL karena tidak langsung terlihat saat dokumen dibuka.

**Jawaban:** `document.xml.rels`

---

### 5. What is the minimum number of bytes required to trigger the payload?

URL dari Q3 dicari di URLScan.io. Ditemukan scan lama (2022) yang masih capture HTML aslinya sebelum domain di-takeover. Di source HTML terdapat ratusan baris komentar `//AAAA...` sebagai padding, dengan ukuran file: **7,457 bytes**.

Berdasarkan riset CVE-2022-30190, Microsoft MSDT hanya memproses dan mengeksekusi payload jika ukuran file HTML melebihi threshold ini. Baris padding `//AAAA...` memang sengaja untuk memastikan file selalu melewati batas tersebut.

**Jawaban:** `4096`

---

### 6. What process is killed by the payload?

Di source HTML yang sama (URLScan lama), terdapat payload JavaScript:

```javascript
window.location.href = "ms-msdt:/id PCWDiagnostic ...";
```

Payload mengandung Base64 encoded PowerShell. Setelah di-decode:

```powershell
$cmd = "c:\windows\system32\cmd.exe";
Start-Process $cmd -windowstyle hidden -ArgumentList "/c taskkill /f /im msdt.exe";
```

Malware mematikan `msdt.exe` setelah berhasil menggunakannya — supaya dialog MSDT tidak muncul ke user dan serangan berjalan silent.

**Jawaban:** `msdt.exe`

---

### 7. Which processes are used to create a detection rule using Event ID 4688?

Windows Event ID 4688 mencatat pembuatan proses baru (Process Creation). Untuk mendeteksi Follina:

- **ProcessName:** `msdt.exe` — proses yang dilaunch sebagai bagian exploit
- **ParentProcessName:** `winword.exe` — Word yang men-spawn msdt.exe saat dokumen berbahaya dibuka

Dalam kondisi normal, `msdt.exe` tidak seharusnya di-spawn oleh `winword.exe`. Kombinasi parent-child ini adalah anomali sangat spesifik untuk Follina — signature detection yang reliable.

**Jawaban:** `msdt.exe, winword.exe`

---

### 8. What is the MITRE ATT&CK Technique ID for the execution method?

Hash disubmit ke any.run dan VirusTotal. Di tab **Behavior** VirusTotal, ditemukan mapping MITRE ATT&CK: **T1559 — Inter-Process Communication**.

Follina menggunakan `ms-msdt://` URI scheme untuk berkomunikasi dari Word ke proses MSDT. URI scheme ini adalah mekanisme IPC — Word mengirim instruksi ke MSDT via protocol handler, yang kemudian mengeksekusi PowerShell payload.

**Jawaban:** `T1559`

---

### 9. What is the CVE for this vulnerability?

Dari tag di any.run submission dan riset OSINT. Zero-day RCE di Microsoft Support Diagnostic Tool yang memungkinkan eksekusi arbitrary code melalui dokumen Office tanpa macro, ditemukan dan aktif dieksploitasi pada Mei 2022.

**Jawaban:** `CVE-2022-30190`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| File Hash (SHA1) | `06727ffda60359236a8029e0b3e8a0fd11c23313` | Hash sample `simple.doc` |
| URL | `https://www.xmlformats.com/office/word/2022/wordprocessingDrawing/RDF821.html` | Remote template berbahaya yang di-fetch saat dokumen dibuka |
| CVE | `CVE-2022-30190` | Follina — MSDT RCE vulnerability |
| Process (victim) | `msdt.exe` | Dieksploitasi sebagai execution bridge, lalu di-kill |
| Process (parent) | `winword.exe` | Spawns msdt.exe via ms-msdt:// URI scheme |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Execution | Inter-Process Communication | T1559 | `ms-msdt://` URI scheme digunakan Word untuk spawn dan instruksi MSDT |
| Defense Evasion | Masquerading / Living off the Land | - | Menyalahgunakan binary legitimate: `msdt.exe`, `cmd.exe`, `certutil.exe`, `expand.exe` |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

Dokumen Word berbahaya dikirim ke korban. Saat dibuka, Word membaca `word/_rels/document.xml.rels` dan menemukan `TargetMode="External"` yang mengarah ke server attacker. Word fetch URL tersebut, mendapat HTML berbahaya berisi skema `ms-msdt://`, lalu spawn `msdt.exe`. MSDT memproses HTML dan mengeksekusi PowerShell payload yang di-encode dalam Base64 — tanpa macro, tanpa klik tambahan.

Payload melakukan:
1. Kill `msdt.exe` supaya tidak ada dialog mencurigakan di layar user
2. Cari file RAR di folder temp, copy ke `C:\users\public\`
3. Decode dan extract payload dari RAR menggunakan `certutil` dan `expand`
4. Eksekusi `rgb.exe` sebagai payload akhir

```
winword.exe
    └─► fetch URL (RDF821.html)
            └─► spawn msdt.exe
                    └─► eksekusi PowerShell payload
                            ├─► taskkill msdt.exe  (silent)
                            ├─► copy & decode RAR
                            └─► rgb.exe  (payload akhir)
```

### Todo / Follow-up

- [ ] Analisis `rgb.exe` — apa payload akhirnya? Ransomware? Stealer?
- [ ] Coba reproduce exploit chain di VM untuk observasi behavior langsung
- [ ] Buat YARA rule berdasarkan pattern payload PowerShell
- [ ] Pelajari teknik fuzzy hashing (SSDEEP/TLSH) untuk deteksi varian Follina
- [ ] Review patch Microsoft untuk CVE-2022-30190 — apa yang di-fix?

---

## 📚 References

- [MITRE ATT&CK T1559 — Inter-Process Communication](https://attack.mitre.org/techniques/T1559/)
- [NVD CVE-2022-30190](https://nvd.nist.gov/vuln/detail/CVE-2022-30190)
- [Microsoft Security Advisory — CVE-2022-30190](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2022-30190)
- [URLScan.io — malicious HTML sample scan](https://urlscan.io)
- [VirusTotal — sample behavior analysis](https://virustotal.com)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
