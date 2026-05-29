# WebStrike — CyberDefenders

> **Platform:** CyberDefenders  
> **Category:** Network Forensics  
> **Difficulty:** Easy  
> **Status:** ✅ Completed  
> **Date:** 2026-05-29  
> **Time Spent:** ~1 jam  

---

## 📌 Prolog

Lab analisis PCAP dengan skenario web server yang dikompromi. Flow-nya cukup klasik — file upload vulnerability → web shell → reverse shell → exfiltration. Bagus untuk latihan membaca HTTP traffic dan TCP stream di Wireshark sekaligus memahami attack chain dari awal sampai akhir.

---

## 🎯 Scenario

File mencurigakan ditemukan di web server milik perusahaan, memicu alarm di intranet. Tim Development menandai anomali tersebut dan mencurigai adanya aktivitas berbahaya. Tim jaringan menangkap traffic kritis dan menyiapkan file PCAP untuk ditinjau.

Tugasmu: analisis PCAP tersebut untuk mengungkap bagaimana file itu muncul dan menentukan sejauh mana aktivitas tidak sah yang terjadi.

---

## ❓ Questions

1. Identifying the geographical origin of the attack facilitates the implementation of geo-blocking measures and the analysis of threat intelligence. From which city did the attack originate?
2. Knowing the attacker's User-Agent assists in creating robust filtering rules. What's the attacker's Full User-Agent?
3. We need to determine if any vulnerabilities were exploited. What is the name of the malicious web shell that was successfully uploaded?
4. Identifying the directory where uploaded files are stored is crucial for locating the vulnerable page and removing any malicious files. Which directory is used by the website to store the uploaded files?
5. Which port, opened on the attacker's machine, was targeted by the malicious web shell for establishing unauthorized outbound communication?
6. Recognizing the significance of compromised data helps prioritize incident response actions. Which file was the attacker attempting to exfiltrate?

---

## 🔍 Answer & Walkthrough

### 1. From which city did the attack originate?

Filter HTTP traffic di Wireshark dan identifikasi IP yang melakukan request ke web server. IP penyerang `117.11.88.124` — whois APNIC menunjukkan network China Unicom Tianjin province (UNICOM-TJ).

**Jawaban:** `Tianjin`

---

### 2. What's the attacker's Full User-Agent?

Buka salah satu HTTP request dari IP `117.11.88.124`, lihat header `User-Agent`. Penyerang menggunakan Firefox 115 di sistem Linux x86_64.

**Jawaban:** `Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0`

---

### 3. What is the name of the malicious web shell that was successfully uploaded?

Cari POST request di traffic HTTP. Ditemukan upload ke `/reviews/upload.php`. Penyerang menggunakan teknik **double extension** — menyamarkan file PHP sebagai gambar agar lolos validasi. Server tetap mengeksekusinya sebagai PHP karena membaca ekstensi terakhir (`.php`).

**Jawaban:** `image.jpg.php`

---

### 4. Which directory is used by the website to store the uploaded files?

Dari HTTP traffic, setelah upload berhasil penyerang mengakses web shell langsung di path `/reviews/uploads/image.jpg.php`.

**Jawaban:** `/reviews/uploads/`

---

### 5. Which port was targeted by the web shell for establishing unauthorized outbound communication?

Lihat TCP stream setelah web shell dieksekusi. Terdapat paket TCP SYN dari server korban (`24.49.63.79`) ke mesin penyerang (`117.11.88.124:8080`) — reverse shell yang diinisiasi oleh web shell.

**Jawaban:** `8080`

---

### 6. Which file was the attacker attempting to exfiltrate?

Follow TCP stream pada koneksi port 8080. Penyerang menjalankan serangkaian command recon secara berurutan: `whoami` → `uname -a` → `pwd` → `ls /home` → `cat /etc/passwd`. Target eksfiltrasi adalah `/etc/passwd` yang berisi daftar semua user di sistem.

**Jawaban:** `/etc/passwd`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| IP (Attacker) | `117.11.88.124` | China Unicom, Tianjin, CN |
| IP (Victim) | `24.49.63.79` | Web server shoporoma.com |
| Web Shell | `image.jpg.php` | Uploaded via `/reviews/upload.php` |
| Upload Path | `/reviews/uploads/` | Direktori tempat web shell tersimpan |
| C2 Port | `8080` | Port reverse shell di mesin penyerang |
| Exfiltration Target | `/etc/passwd` | File user system yang dicoba dieksfiltrasi |
| User-Agent | `Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0` | UA penyerang |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Initial Access | Exploit Public-Facing Application | T1190 | Upload web shell via unrestricted file upload |
| Execution | Server Software Component: Web Shell | T1505.003 | `image.jpg.php` dieksekusi di server |
| Persistence | Server Software Component: Web Shell | T1505.003 | Web shell tersimpan permanen di `/reviews/uploads/` |
| Command & Control | Non-Standard Port | T1571 | Reverse shell ke port 8080 |
| Exfiltration | Exfiltration Over C2 Channel | T1041 | `cat /etc/passwd` via reverse shell |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

**5W Triage:**

- **What** — Unrestricted File Upload. Web shell PHP diupload dengan teknik double extension (`image.jpg.php`) untuk bypass validasi upload.
- **Who** — Victim: `24.49.63.79` (shoporoma.com) | Attacker: `117.11.88.124` (China Unicom, Tianjin, CN)
- **When** — 12 Mei 2026, pukul 09.00
- **Where** — Upload form di `/reviews/upload.php`, web shell stored di `/reviews/uploads/`
- **Why** — Server PHP mengeksekusi file berdasarkan ekstensi **terakhir** — teknik double extension memanfaatkan celah ini untuk bypass filter yang hanya mengecek ekstensi pertama/tengah.

**Attack Chain:**

| # | Fase | Detail |
|---|------|--------|
| 1 | Recon | Penyerang dari Tianjin (117.11.88.124) browsing dan explorasi `/reviews/` |
| 2 | Initial Access | Upload `image.jpg.php` via form di `/reviews/upload.php` |
| 3 | Execution | Web shell dieksekusi via `/reviews/uploads/image.jpg.php` |
| 4 | Persistence | File web shell tersimpan permanen di direktori uploads |
| 5 | C2 | Reverse shell dari server korban (`24.49.63.79`) ke `117.11.88.124:8080` |
| 6 | Exfiltration | `whoami` → `uname -a` → `pwd` → `ls /home` → `cat /etc/passwd` |

### Todo / Follow-up

- [ ] Pelajari cara detect double extension attack di level WAF / server config
- [ ] Coba reproduce upload bypass dengan Burp Suite di lab environment
- [ ] Review konfigurasi Apache/Nginx yang aman untuk mencegah eksekusi file di upload directory
- [ ] Eksplorasi Wireshark filter lebih lanjut: `http.request.method == "POST"`, `tcp.stream`

---

## 📚 References

- [MITRE ATT&CK T1505.003 — Web Shell](https://attack.mitre.org/techniques/T1505/003/)
- [MITRE ATT&CK T1190 — Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)
- [OWASP — Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
