# Piggy (Investigation) — BTLO

> **Platform:** Blue Team Labs Online  
> **Category:** PCAP Analysis  
> **Difficulty:** Medium  
> **Status:** ✅ Completed  
> **Date:** 2026-05-29  
> **Time Spent:** ~2 jam  

---

## 📌 Prolog

Lab investigasi PCAP dari BTLO. Dikasih beberapa file PCAP yang berisi traffic dari sistem yang diduga terinfeksi. Tugas utamanya: analisis traffic, identifikasi pola mencurigakan, dan correlate temuan ke threat intel lewat OSINT. Lab ini cukup padat karena satu host bisa punya lebih dari satu ancaman aktif secara bersamaan.

---

## 🎯 Scenario

Sebuah sistem internal (`10.0.9.171`) terdeteksi melakukan aktivitas jaringan yang mencurigakan. Tim SOC mengumpulkan beberapa file PCAP dari periode infeksi. Analisis menunjukkan adanya kombinasi ancaman: transfer data terenkripsi dalam volume besar, koneksi ke infrastructure yang terkait banking trojan, koneksi ke mining pool via port non-standard, dan aktivitas DNS yang tidak lazim. Investigasi dilakukan per-PCAP untuk mengurai tiap lapisan ancaman.

---

## ❓ Questions

1. How much data was transferred in total? (Format: XXXX M)
2. Review the IPs the infected system has communicated with. Perform OSINT searches to identify the malware family tied to this infrastructure.
3. Review the two IPs that are communicating on an unusual port. What are the two ASN numbers these IPs belong to?
4. Perform OSINT checks. What malware category have these IPs been attributed to historically?
5. What ATT&CK technique is most closely related to this activity?
6. Go to View > Time Display Format > Seconds Since Beginning of Capture. How long into the capture was the first TXT record query made?
7. What is the ATT&CK subtechnique relating to this activity?

---

## 🔍 Answer & Walkthrough

### 1. How much data was transferred in total? (Format: XXXX M)

Yang dihitung bukan semua traffic — hanya **encrypted payload** saja. Untuk menyaring ini di Wireshark, gunakan filter berikut tergantung protokolnya:
- TLS: `ssl.app_data`
- SSH: `ssh.encrypted_packet`

Setelah filter aktif, buka **Statistics → Capture File Properties** dan lihat nilai **Displayed bytes**. Konversi ke MB:

```
1,128,631,478 bytes ÷ 1,000,000 = 1128 M
```

**Jawaban:** `1128 M`

---

### 2. Identify the malware family tied to the infrastructure

Buka **Statistics → Conversations → IPv4**, catat semua IP yang berkomunikasi dengan host korban `10.0.9.171`. Fokus ke IP dengan traffic terbanyak atau yang paling mencurigakan.

Cek tiap IP ke OSINT:
- **VirusTotal** → `virustotal.com/gui/ip-address/[IP]`
- **AbuseIPDB** → `abuseipdb.com/check/[IP]`
- **ThreatFox** → `threatfox.abuse.ch`

IP `31.184.253.37` langsung ketahuan:
- Flagged: Malicious (8), Phishing (3), Malware (1) di VirusTotal
- 49 reports di AbuseIPDB
- Tab Community VirusTotal menyebut C2 TrickBot

**Jawaban:** `TrickBot`

---

### 3. Two IPs communicating on unusual ports — their ASN numbers

Buka **Statistics → Conversations → TCP**, perhatikan kolom Port A dan Port B. Cari yang bukan port standard (80, 443, 22, 53).

Ditemukan dua koneksi unusual:
- `10.0.9.171` ↔ `194.233.171.171` via **port 8080**
- `10.0.9.171` ↔ `104.236.57.24` via **port 8000**

Cek ASN keduanya di `ipinfo.io`:
- `194.233.171.171` → **AS63949** (Linode/Akamai Cloud)
- `104.236.57.24` → **AS14061** (DigitalOcean)

**Jawaban:** `AS63949 dan AS14061`

---

### 4. What malware category have these IPs been attributed to historically?

Cek kedua IP di VirusTotal → tab Detection. Keduanya sama-sama di-flag sebagai **Miner**. Konfirmasi juga via tab Community. Kedua IP ini adalah mining pool server yang menerima koneksi dari sistem korban untuk keperluan cryptomining.

**Jawaban:** `Miner`

---

### 5. What ATT&CK technique is most closely related to this activity?

Aktivitas yang teridentifikasi: koneksi ke mining pool via unusual port → **cryptomining**. Cari di `attack.mitre.org` dengan keyword "resource hijacking" atau "cryptomining".

Teknik yang match: **T1496 - Resource Hijacking** di bawah Tactic **Impact** — attacker mengeksploitasi resource komputasi korban untuk mining cryptocurrency.

**Jawaban:** `T1496 - Resource Hijacking`

---

### 6. How long into the capture was the first TXT record query made?

Ubah format waktu dulu: **View → Time Display Format → Seconds Since Beginning of Capture**.

Kemudian filter DNS TXT record query:

```
dns.qry.type == 16
```

> `16` adalah DNS record type code untuk TXT.

Packet pertama yang muncul adalah **No. 1709** dengan timestamp `8.527712` detik sejak capture dimulai. Domain yang di-query: `mlckdhokhvhtcmevvcgbggcviwxgim.sandbox.alphasoc.xyz` — subdomain panjang dan random, ciri khas DNS tunneling.

**Jawaban:** `8.527712`

---

### 7. What is the ATT&CK subtechnique relating to this activity?

Domain yang di-query punya subdomain sangat panjang dan acak, dan menggunakan TXT record — kombinasi ini adalah signature dari **DNS Tunneling**. TXT record dipilih karena bisa membawa data lebih besar dibanding record lain, ideal untuk C2 communication atau data exfiltration yang bersembunyi di traffic DNS.

Cari di `attack.mitre.org` dengan keyword "DNS" atau "DNS tunneling":

**T1071.004 - Application Layer Protocol: DNS** (Tactic: Command and Control)

**Jawaban:** `T1071.004 - Application Layer Protocol: DNS`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| IP Address | `10.0.9.171` | Host korban |
| IP Address | `31.184.253.37` | TrickBot C2 server |
| IP Address | `194.233.171.171` | Mining pool — AS63949 (Linode/Akamai) |
| IP Address | `104.236.57.24` | Mining pool — AS14061 (DigitalOcean) |
| Domain | `mlckdhokhvhtcmevvcgbggcviwxgim.sandbox.alphasoc.xyz` | DNS Tunneling C2 |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Impact | Resource Hijacking | T1496 | Cryptomining via koneksi ke mining pool |
| Command and Control | Application Layer Protocol: DNS | T1071.004 | DNS Tunneling menggunakan TXT record |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

Host `10.0.9.171` terinfeksi setidaknya dua ancaman berbeda secara bersamaan:

1. **TrickBot** — banking trojan yang berkembang jadi modular botnet. Sistem berhasil diidentifikasi berkomunikasi dengan C2 di `31.184.253.37` yang sudah banyak di-flag di threat intel. TrickBot sering jadi initial access sebelum ransomware (Ryuk/Conti) di-deploy.

2. **Cryptominer** — sistem konek ke dua mining pool (`194.233.171.171` dan `104.236.57.24`) via port 8080 dan 8000. Kedua IP di-flag sebagai Miner di VirusTotal. Attacker memanfaatkan resource CPU/GPU korban untuk mining cryptocurrency tanpa sepengetahuan korban.

3. **DNS Tunneling** — terdeteksi query TXT record ke domain dengan subdomain random dan sangat panjang, mulai ~8.5 detik setelah capture dimulai. Ini kemungkinan channel C2 tersembunyi yang bypass firewall dengan menyamar sebagai traffic DNS biasa.

### Todo / Follow-up

- [ ] Deep dive cara kerja TrickBot sebagai modular botnet — modul apa saja yang biasa di-deploy
- [ ] Pelajari TrickBot → Ryuk/Conti ransomware kill chain
- [ ] Praktik identifikasi DNS Tunneling di Wireshark — ciri-ciri subdomain dan query pattern
- [ ] Pelajari semua DNS record type yang sering disalahgunakan: TXT, CNAME, MX
- [ ] Explore tools khusus DNS Tunneling detection: dnstap, dnscat2 forensics

---

## 📚 References

- [MITRE ATT&CK T1496 - Resource Hijacking](https://attack.mitre.org/techniques/T1496/)
- [MITRE ATT&CK T1071.004 - Application Layer Protocol: DNS](https://attack.mitre.org/techniques/T1071/004/)
- [VirusTotal](https://virustotal.com)
- [AbuseIPDB](https://abuseipdb.com)
- [ThreatFox](https://threatfox.abuse.ch)
- [ipinfo.io](https://ipinfo.io)
- [Blue Team Labs Online](https://blueteamlabs.online)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
