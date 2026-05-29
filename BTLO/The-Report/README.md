# The Report — BTLO

> **Platform:** Blue Team Labs Online  
> **Category:** Threat Intelligence  
> **Difficulty:** Easy  
> **Status:** ✅ Completed  
> **Date:** 2026-05-29  
> **Time Spent:** ~2 jam  

---

## 📌 Prolog

Challenge ini berbeda dari yang lain — bukan soal analisis artifact atau PCAP, tapi baca dan ekstrak intel dari threat report nyata. Sumber yang dipakai adalah **Red Canary Threat Detection Report 2022**, laporan tahunan yang menganalisis ancaman paling prevalens sepanjang 2021 dari data pelanggan Red Canary. Relevan banget untuk dipelajari karena banyak teknik dan threat actor yang dibahas masih aktif sampai sekarang.

---

## 🎯 Scenario

Kamu bekerja di SOC yang baru dibentuk dan belum sepenuhnya operasional. Sebagai bagian dari pengumpulan intel awal, kamu ditugaskan untuk **mempelajari threat report yang dirilis tahun 2022** dan menyarankan output yang berguna bagi SOC — mulai dari teknik deteksi, prioritas monitoring, hingga threat actor awareness.

Artifact: **Red Canary Threat Detection Report 2022** (`2022_ThreatDetectionReport_RedCanary.pdf`)

---

## ❓ Questions

1. Name the supply chain attack related to Java logging library in the end of 2021.
2. Mention the MITRE Technique ID which affected more than 50% of the customers.
3. Submit the names of 2 vulnerabilities belonging to Exchange Servers.
4. Submit the CVE of the zero day vulnerability of a driver which led to RCE and gain SYSTEM privileges.
5. Mention the 2 adversary groups that leverage SEO to gain initial access.
6. In the detection rule, what should be mentioned as parent process if we are looking for execution of malicious JS files? *(Hint: Not CMD)*
7. Ransomware gangs started using affiliate model to gain initial access. Name the precursors used by affiliates of Conti ransomware group.
8. The main target of coin miners was outdated software. Mention the 2 outdated software mentioned in the report.
9. Name the ransomware group which threatened to conduct DDoS if they didn't pay ransom.
10. What is the security measure we need to enable for RDP connections in order to safeguard from ransomware attacks?

---

## 🔍 Answer & Walkthrough

Semua jawaban didapat langsung dari membaca **Red Canary Threat Detection Report 2022**. Challenge ini melatih kemampuan membaca dan mengekstrak informasi kritis dari threat report — skill yang sangat relevan untuk threat intelligence analyst.

---

### 1. Name the supply chain attack related to Java logging library in the end of 2021.

**Log4j** adalah Java logging library yang sangat populer dan dipakai sebagai dependency di ribuan aplikasi. Desember 2021, ditemukan vulnerability **RCE (Remote Code Execution)** yang dijuluki **Log4Shell**. Ini supply chain attack karena siapa pun yang pakai Log4j sebagai dependency langsung terdampak — target utama awalnya adalah server **VMware Horizon** yang internet-facing.

**Jawaban:** `Log4j`

---

### 2. Mention the MITRE Technique ID which affected more than 50% of the customers.

**T1059: Command and Scripting Interpreter** adalah satu-satunya teknik yang memengaruhi lebih dari 50% pelanggan — tepatnya **53.4%**, menjadikannya teknik #1 paling prevalens di laporan. Dua sub-technique utamanya:
- **T1059.001 — PowerShell**: 35.0% pelanggan
- **T1059.003 — Windows Command Shell**: 28.1% pelanggan

**Jawaban:** `T1059`

---

### 3. Submit the names of 2 vulnerabilities belonging to Exchange Servers.

Dua vulnerability besar yang menarget Microsoft Exchange Server di 2021:

**ProxyLogon** (CVE-2021-26855, -26857, -26858, -27065) — diumumkan Maret 2021, memungkinkan RCE pada Exchange. Dieksploitasi oleh **HAFNIUM** (diduga disponsori China) untuk deploy web shells dan mencuri data, serta digunakan untuk menyebarkan **DearCry ransomware**.

**ProxyShell** (CVE-2021-31207, -34523, -34473) — diumumkan Juli 2021, memungkinkan RCE **tanpa autentikasi**. Adversaries menggunakannya untuk deploy web shells, reconnaissance, lateral movement, dan ransomware deployment. Dampaknya bertahan hingga Desember 2021.

**Jawaban:** `ProxyLogon, ProxyShell`

---

### 4. Submit the CVE of the zero day vulnerability of a driver which led to RCE and gain SYSTEM privileges.

**CVE-2021-34527** adalah CVE dari **PrintNightmare** — zero-day pada **Windows Print Spooler service**. Vulnerability ini memungkinkan RCE dan escalation ke **SYSTEM-level privileges** (level tertinggi di Windows) karena Print Spooler berjalan sebagai SYSTEM dan gagal melakukan autentikasi dengan benar saat memuat printer driver DLL. Dieksploitasi oleh ransomware operator seperti **Vice Society** dan **Magniber**.

> Catatan: CVE-2021-34523 adalah bagian dari grup ProxyShell (Exchange) — berbeda dengan PrintNightmare.

**Jawaban:** `CVE-2021-34527`

---

### 5. Mention the 2 adversary groups that leverage SEO to gain initial access.

Laporan secara eksplisit menyebutkan keduanya bersama di bagian *User-initiated Initial Access*:

> *"Adversaries behind both Gootkit and Yellow Cockatoo abuse search engine optimization (SEO) to display malicious content at the top of a victim's search results."*

- **Yellow Cockatoo**: SEO poisoning → situs berbahaya → download file dengan nama sesuai query pencarian (misal `this-is-my-search-query.msi`) → RAT fileless (XOR + Base64 encoded)
- **Gootkit**: SEO poisoning → situs palsu yang menyediakan konten legal/finansial → download ZIP berisi JavaScript loader → Cobalt Strike → terhubung ke aktivitas Sodinokibi/REvil

SocGholish tidak masuk karena menggunakan *drive-by download* di situs legitimate yang dikompromis — bukan SEO poisoning.

**Jawaban:** `Yellow Cockatoo, Gootkit`

---

### 6. In the detection rule, what should be mentioned as parent process if we are looking for execution of malicious JS files?

**wscript.exe** adalah Windows Script Host — program bawaan Windows yang mengeksekusi file `.js` dan `.vbs`. Ketika korban double-click file JavaScript berbahaya, Windows secara default menjalankannya melalui `wscript.exe`.

Dua contoh detection rule di laporan yang menggunakan `wscript.exe` sebagai parent:

```
# Deteksi Gootkit
process == wscript.exe
&&
file_path_includes (%APPDATA%)

# Deteksi SocGholish
process == wscript.exe
&&
command_line_includes (.zip && .js)
&&
has_external_netconn
```

**Jawaban:** `wscript.exe`

---

### 7. Name the precursors used by affiliates of Conti ransomware group.

Conti mengandalkan **affiliates** — pihak ketiga yang bertugas mendapatkan initial access lalu menyerahkan akses tersebut ke operator Conti. Tiga malware precursor yang dipakai affiliates Conti berdasarkan tabel di laporan:

| Malware | Cara Pengiriman |
|---------|----------------|
| **Qbot** | Phishing email berisi attachment XLSX berbahaya |
| **Bazar** | BazarLoader/BazarBackdoor via phishing, termasuk BazaCall (korban ditipu menelepon nomor palsu) |
| **IcedID** | Banking trojan yang berfungsi sebagai loader untuk payload lanjutan |

Model affiliate membuat attribution semakin sulit karena aktor yang berbeda terlibat di tiap fase intrusi.

**Jawaban:** `Qbot, Bazar, IcedID`

---

### 8. Mention the 2 outdated software mentioned in the report targeted by coin miners.

Di bagian *Linux Coinminers*, laporan secara eksplisit menyebut:

> *"Many of the coinminers we saw exploited flaws in outdated applications like JBoss and WebLogic, so keeping systems updated will deter adversaries who are simply scanning for applications with known vulnerabilities."*

- **JBoss** (sekarang WildFly): Application server Java open-source dengan sejarah panjang vulnerability
- **Oracle WebLogic**: Enterprise Java application server yang banyak dipakai di lingkungan enterprise dan sering tidak di-update

Coinminers menarget keduanya karena cukup lakukan scanning otomatis — tidak butuh teknik yang canggih.

**Jawaban:** `JBoss, WebLogic`

---

### 9. Name the ransomware group which threatened to conduct DDoS if they didn't pay ransom.

Di bagian *Ransomware — Beyond Encryption*, laporan menyebut:

> *"An adversary known as Fancy Lazarus (no affiliation with Fancy Bear or Lazarus Group) extorted victims by threatening to conduct a distributed denial of service (DDoS) intrusion if they didn't pay."*

Ini adalah **triple extortion** — enkripsi data + ancaman kebocoran data + ancaman DDoS. Nama "Fancy Lazarus" sengaja dipilih agar terkesan menakutkan dengan menyerupai nama APT terkenal, padahal tidak ada hubungannya dengan Fancy Bear (APT28/Rusia) maupun Lazarus Group (Korea Utara).

**Jawaban:** `Fancy Lazarus`

---

### 10. What is the security measure we need to enable for RDP connections?

Di bagian *Ransomware — Take Action*, laporan menyatakan:

> *"Internet-facing remote desktop protocol (RDP) connections without multi-factor authentication (MFA) are a common ransomware vector, making MFA for any accounts that can log in via RDP a high priority."*

RDP tanpa MFA berbahaya karena satu credential yang bocor sudah cukup untuk masuk. Dengan MFA aktif, meskipun password berhasil dicuri, adversary tetap tidak bisa login tanpa faktor kedua.

**Jawaban:** `MFA`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| Vulnerability | `Log4Shell` | RCE pada Log4j — supply chain attack, Desember 2021 |
| Vulnerability | `ProxyLogon` | RCE pada Exchange Server, dipakai HAFNIUM |
| Vulnerability | `ProxyShell` | RCE tanpa auth pada Exchange Server |
| CVE | `CVE-2021-34527` | PrintNightmare — RCE + SYSTEM privileges via Print Spooler |
| MITRE Technique | `T1059` | Command and Scripting Interpreter — 53.4% customers |
| Threat Actor | `Yellow Cockatoo` | SEO poisoning → fileless RAT |
| Threat Actor | `Gootkit` | SEO poisoning → JavaScript loader → Cobalt Strike |
| Threat Actor | `Fancy Lazarus` | Triple extortion ransomware dengan ancaman DDoS |
| Malware | `Qbot, Bazar, IcedID` | Precursors affiliates Conti ransomware |
| Software | `JBoss, WebLogic` | Target utama coinminers karena outdated |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Execution | Command and Scripting Interpreter | T1059 | Teknik #1 paling prevalens — 53.4% customers |
| Execution | Command and Scripting Interpreter: PowerShell | T1059.001 | Sub-technique #1 — 35.0% customers |
| Initial Access | Phishing | T1566 | Vektor utama pengiriman Qbot, Bazar, IcedID |
| Initial Access | Drive-by Compromise | T1189 | SEO poisoning — Yellow Cockatoo, Gootkit |
| Privilege Escalation | Exploitation for Privilege Escalation | T1068 | PrintNightmare (CVE-2021-34527) → SYSTEM |
| Defense Evasion | Hide Artifacts: Email Hiding Rules | T1564.008 | Umum dipakai di BEC post-compromise |
| Impact | Data Encrypted for Impact | T1486 | Ransomware — Conti, DearCry, Vice Society |
| Impact | Network Denial of Service | T1498 | Fancy Lazarus — ancaman DDoS sebagai triple extortion |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

Berdasarkan Red Canary Threat Detection Report 2022, lanskap ancaman 2021 didominasi oleh:

1. **Exploitation of public-facing vulnerabilities** — Log4Shell, ProxyLogon, ProxyShell, dan PrintNightmare menjadi attack vector utama karena dampaknya yang luas dan kecepatan eksploitasi oleh berbagai threat actor
2. **Scripting abuse** — T1059 (PowerShell + Windows Command Shell) tetap menjadi teknik paling prevalens karena fleksibilitasnya untuk eksekusi payload, download stager, dan operasi fileless
3. **Ransomware affiliate ecosystem** — Conti dan grup lain memisahkan initial access (Qbot, Bazar, IcedID) dari operasi enkripsi, membuat attribution lebih sulit dan operasi lebih scalable
4. **Extortion escalation** — dari single (enkripsi) ke double (+ data leak) ke triple extortion (+ DDoS threat) oleh Fancy Lazarus

### Todo / Follow-up

- [ ] Baca full Red Canary Threat Detection Report 2022 untuk konteks lebih dalam setiap teknik
- [ ] Explore detection rules untuk T1059 (PowerShell abuse) di Sigma rule repository
- [ ] Pelajari cara monitoring inbox rule creation di Microsoft 365 Defender sebagai deteksi BEC
- [ ] Cari tahu cara deteksi `wscript.exe` spawning suspicious child processes di SIEM
- [ ] Baca Red Canary Threat Detection Report 2023/2024 untuk tren terbaru

---

## 📚 References

- [Red Canary Threat Detection Report 2022](https://redcanary.com/threat-detection-report/)
- [MITRE ATT&CK T1059 — Command and Scripting Interpreter](https://attack.mitre.org/techniques/T1059/)
- [Microsoft: PrintNightmare CVE-2021-34527](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-34527)
- [Microsoft: ProxyLogon CVE-2021-26855](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-26855)
- [CISA: Log4Shell (CVE-2021-44228)](https://www.cisa.gov/news-events/cybersecurity-advisories/aa21-356a)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
