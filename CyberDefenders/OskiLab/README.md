# OskiLab — CyberDefenders

> **Platform:** CyberDefenders  
> **Category:** Malware Analysis / Threat Intelligence  
> **Difficulty:** Easy  
> **Status:** ✅ Completed  
> **Date:** 2026-05-25  
> **Time Spent:** ~1.5 jam  

---

## 📌 Prolog

Lab ini fokus menganalisis laporan sandbox ANY.RUN untuk memahami perilaku **Stealc** — infostealer yang masuk via file PPT berbahaya. Tidak banyak hands-on di sini, lebih ke membaca laporan sandbox secara teliti: timeline eksekusi, child process, network connections, dan MITRE mapping. Bagus untuk melatih threat intel dari laporan sandbox tanpa harus setup environment sendiri.

---

## 🎯 Scenario

Seorang akuntan menerima email dengan judul **"Urgent New Order"** dari klien. Saat mencoba membuka invoice yang terlampir, isi invoice ternyata berisi informasi pesanan yang palsu. Tidak lama setelah itu, SIEM perusahaan membangkitkan alert terkait download file yang berpotensi berbahaya. Investigasi awal menunjukkan file PPT yang terlampir kemungkinan menjadi penyebab download tersebut.

> Tugasmu: lakukan pemeriksaan mendetail terhadap file tersebut.

**File:** MD5 `12c1842c3ccafe7408c23ebf292ee3d9` | Zip password: `cyberdefenders.org`

---

## ❓ Questions

1. Menentukan waktu pembuatan malware dapat memberikan wawasan tentang asal-usulnya. Kapan waktu pembuatan malware tersebut?
2. Mengidentifikasi C2 server yang digunakan malware dapat membantu melacak kembali ke penyerang. C2 server mana yang digunakan malware dalam file PPT tersebut?
3. Mengidentifikasi tindakan awal malware setelah infeksi dapat memberikan wawasan tentang tujuan utamanya. Library apa yang pertama kali diminta oleh malware setelah infeksi?
4. RC4 key apa yang digunakan malware untuk mendekripsi string yang telah di-encode dengan base64?
5. Identifikasi teknik MITRE utama (bukan sub-teknik) yang digunakan malware untuk mencuri password pengguna.
6. Direktori mana yang menjadi target malware untuk menghapus semua file DLL?
7. Berapa detik yang dibutuhkan malware untuk melakukan self-delete setelah berhasil melakukan exfiltration data pengguna?

---

## 🔍 Answer & Walkthrough

### 1. Kapan waktu pembuatan malware?

Di ANY.RUN, buka tab **Details** atau **Static Analysis** dari submission hash MD5 `12c1842c3ccafe7408c23ebf292ee3d9`. Timestamps PE header menunjukkan compile time malware.

**Jawaban:** `2022-09-28 17:40`

---

### 2. C2 server mana yang digunakan?

Di tab **Network** laporan ANY.RUN, lihat HTTP connections yang dibuat payload (`VPN.exe`). Terdapat POST request ke satu endpoint yang konsisten dipakai untuk exfiltration.

**Jawaban:** `http://171.22.28.221/5c06c05b7b34e8e6.php`

---

### 3. Library apa yang pertama kali diminta setelah infeksi?

Setelah payload dieksekusi, malware menghubungi C2 untuk download dependency. Di laporan ANY.RUN, lihat urutan file yang di-drop atau di-download — library pertama yang diminta adalah untuk membaca database browser (tempat credentials dan cookies disimpan).

**Jawaban:** `sqlite3.dll`

---

### 4. RC4 key yang digunakan untuk dekripsi?

Di laporan ANY.RUN, bagian **Extracted Configuration** atau **Strings** — Stealc menggunakan RC4 + base64 untuk menyembunyikan konfigurasi C2 dan string sensitif dari analisis statis. Key-nya terlihat di output deobfuscation sandbox.

**Jawaban:** `5329514621441247975720749009`

---

### 5. Teknik MITRE utama untuk pencurian password?

Di tab **MITRE ATT&CK** laporan ANY.RUN, filter teknik yang berkaitan dengan credential theft. Stealc menggunakan `sqlite3.dll` untuk membaca database browser — ini masuk ke teknik **Credentials from Password Stores** dan sub-tekniknya **Credentials from Web Browsers** (T1555.003). Soal meminta teknik utama (bukan sub-teknik).

**Jawaban:** `T1555`

---

### 6. Direktori target penghapusan DLL?

Di tab **Process Tree** / **Child Processes** laporan ANY.RUN, cari command yang dijalankan `cmd.exe` pada fase cleanup. Command lengkapnya:

```
cmd.exe /c timeout /t 5 & del /f /q "C:\Users\admin\AppData\Local\Temp\VPN.exe" & del "C:\ProgramData\*.dll" & exit
```

Breakdown:
- `timeout /t 5` — tunggu 5 detik
- `del /f /q "...VPN.exe"` — hapus executable utama secara silent
- `del "C:\ProgramData\*.dll"` — hapus semua DLL yang di-download dari C2
- `exit` — tutup proses

**Jawaban:** `C:\ProgramData`

---

### 7. Berapa detik malware menunggu sebelum self-delete?

Dari command di atas, `timeout /t 5` memberikan jeda 5 detik sebelum eksekusi penghapusan — cukup waktu untuk memastikan exfiltration selesai sebelum file dihapus.

**Jawaban:** `5`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| File Hash (MD5) | `12c1842c3ccafe7408c23ebf292ee3d9` | Sample PPT berbahaya |
| C2 Server | `http://171.22.28.221/5c06c05b7b34e8e6.php` | Endpoint exfiltration dan download payload |
| Payload | `VPN.exe` | Executable utama Stealc, di-drop ke `%TEMP%` |
| RC4 Key | `5329514621441247975720749009` | Key dekripsi konfigurasi C2 |
| First DLL | `sqlite3.dll` | Library pertama yang di-download untuk baca database browser |
| Cleanup Dir | `C:\ProgramData` | Direktori yang DLL-nya dihapus saat cleanup |
| Compile Time | `2022-09-28 17:40` | Timestamp PE header malware |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Initial Access | Phishing: Spearphishing Attachment | T1566.001 | Email "Urgent New Order" dengan file PPT berbahaya |
| Execution | User Execution: Malicious File | T1204.002 | User membuka file PPT |
| Defense Evasion | Obfuscated Files or Information | T1027 | RC4 + base64 untuk sembunyikan konfigurasi C2 |
| Defense Evasion | Indicator Removal: File Deletion | T1070.004 | Self-delete post-exfiltration via `cmd.exe` |
| Credential Access | Steal Web Session Cookie | T1539 | Steal session cookies dari browser |
| Credential Access | Unsecured Credentials: Credentials In Files | T1552.001 | Credentials tersimpan dalam file |
| Credential Access | Credentials from Password Stores: Web Browsers | T1555.003 | Ekstrak username & password dari database browser via `sqlite3.dll` |
| Command & Control | Application Layer Protocol: Web Protocols | T1071.001 | C2 communication via HTTP |
| Exfiltration | Exfiltration Over C2 Channel | T1041 | Data dikirim via HTTP POST ke C2 |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

Serangan dimulai dengan phishing email bertema invoice urgent. File PPT berbahaya men-trigger download payload `VPN.exe` dari C2 server ke `%TEMP%`. Setelah dieksekusi, Stealc langsung download `sqlite3.dll` dan library lain ke `C:\ProgramData\` untuk membaca database browser. Konfigurasi C2 disembunyikan dengan enkripsi RC4 + base64 untuk lolos analisis statis.

Semua credential, cookies, dan data sensitif yang berhasil dicuri dikirim via HTTP POST ke `http://171.22.28.221/5c06c05b7b34e8e6.php`. Setelah exfiltration selesai, malware menjalankan cleanup via `cmd.exe`: tunggu 5 detik, hapus `VPN.exe`, hapus semua DLL di `C:\ProgramData\`, lalu exit — tidak meninggalkan artefak yang mudah ditemukan.

### Todo / Follow-up

- [ ] Analisis sample Stealc lebih dalam menggunakan Ghidra/IDA untuk lihat implementasi RC4
- [ ] Coba submit hash ke VirusTotal dan bandingkan detection rate dengan ANY.RUN
- [ ] Pelajari cara hunting Stealc di SIEM: query untuk deteksi `sqlite3.dll` yang di-drop di `%TEMP%` / `C:\ProgramData\`
- [ ] Review email phishing indicators: header analysis untuk identifikasi pengirim palsu
- [ ] Eksplorasi family malware Stealc lebih lanjut — varian dan infrastructure yang dipakai

---

## 📚 References

- [MITRE ATT&CK T1555 — Credentials from Password Stores](https://attack.mitre.org/techniques/T1555/)
- [MITRE ATT&CK T1566.001 — Spearphishing Attachment](https://attack.mitre.org/techniques/T1566/001/)
- [ANY.RUN — Stealc analysis](https://any.run)
- [VirusTotal — hash lookup](https://virustotal.com)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
