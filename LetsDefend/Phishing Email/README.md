# Phishing Email — LetsDefend

> **Platform:** LetsDefend  
> **Category:** Email Forensics / Phishing Analysis  
> **Difficulty:** Beginner  
> **Status:** ✅ Completed  
> **Date:** 2026-06-11  
> **Time Spent:** ~X jam  

---

## 🎯 Scenario

Alamat email bocor dan menerima email yang mengaku dari PayPal — ditulis dalam Bahasa Jerman. Tugas: analisis email mencurigakan tersebut dan tentukan apakah ini phishing atau bukan.

File: `C:\Users\LetsDefend\Desktop\Files\PhishingChallenge.zip` | Password: `infected`

---

## ❓ Questions

1. What is the return path of the email?
2. What is the domain name of the url in this mail?
3. Is the domain mentioned in the previous question suspicious?
4. What is the body SHA-256 of the domain?
5. Is this email a phishing email?

---

## 🔍 Answer & Walkthrough

### 1. What is the return path of the email?

Extract `PhishingChallenge.zip`, buka file `.eml`, lalu view **Message Source**. Di header, field `Return-Path` menunjukkan alamat yang akan menerima notifikasi bounce jika email gagal terkirim ke target.

**Jawaban:** `bounce@rjttznyzjjzydnillquh.designclub.uk.com`

---

### 2. What is the domain name of the url in this mail?

Dari message source, ada `href` link dan button yang mengarahkan victim ke sebuah URL. Domain-nya adalah milik Google Storage — tampak legitim di permukaan.

**Jawaban:** `storage.googleapis.com`

---

### 3. Is the domain mentioned in the previous question suspicious?

Meski `storage.googleapis.com` adalah domain legitim milik Google, ia digunakan untuk meng-host HTML file dengan nama yang di-masquerade: `aemmfcy1vxeo.htm1`. Memanfaatkan domain terpercaya untuk menyimpan payload adalah taktik umum phishing agar lolos dari URL filter.

**Jawaban:** `Yes`

---

### 4. What is the body SHA-256 of the domain?

Cek body SHA-256 dari domain `storage.googleapis.com` via VirusTotal.

**Jawaban:** `13945ecc33afee74ac7f72e1d5bb73050894356c4bf63d02a1a53e76830567f5`

---

### 5. Is this email a phishing email?

Kombinasi spoofed sender, subject mencurigakan, dan link ke HTML masqueraded sudah cukup untuk mengonfirmasi ini phishing.

**Jawaban:** `Yes`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| Email (Target) | `krystyalia@gmail.com` | Alamat target penerima |
| Return Path | `bounce@rjttznyzjjzydnillquh.designclub.uk.com` | Alamat notif bounce |
| Sender | `IHKH@MFEWW@kodehexa.net` | Spoofed sender address |
| Sent Via | `foresthillrestaurant[.]com` | Domain pengirim sebenarnya |
| Subject | `*P.A.Y.P.A.L#` | Subject masquerade sebagai PayPal |
| Domain | `storage.googleapis.com` | Domain legitim yang dipakai host payload |
| URL | `https://storage.googleapis.com/hqyoqzatqthj/aemmfcy1vxeo.htm1` | Link phishing ke HTML masqueraded |
| File Hash (SHA-256) | `13945ecc33afee74ac7f72e1d5bb73050894356c4bf63d02a1a53e76830567f5` | Body SHA-256 domain |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Initial Access | Phishing: Spearphishing Link | T1566.002 | Link ke storage.googleapis.com yang meng-host HTML phishing |
| Stealth | Social Engineering: Impersonation | T1684.001 | Email mengimpersonasi PayPal |
| Stealth | Social Engineering: Email Spoofing | T1684.002 | Sender address di-spoof |

---

## 📋 Summary — Attacker Behavior

Email phishing ini menyamar sebagai notifikasi dari PayPal — dikirim dalam Bahasa Jerman ke target `krystyalia@gmail.com` pada `Mon, 15 Aug 2022 07:35:02`.

Dari message source, beberapa red flag langsung terlihat: subject `*P.A.Y.P.A.L#` yang aneh, sender `IHKH@MFEWW@kodehexa.net` yang jelas di-spoof, dan email sebenarnya dikirim via `foresthillrestaurant[.]com`. SPF pass — tapi itu hanya berarti email valid dari domain tersebut, bukan berarti aman.

Penyerang menyematkan link unsubscribe via `mktomail.com` (platform marketing milik Adobe) untuk menambah kesan legitimasi. Tapi inti serangannya ada di href button yang mengarahkan victim ke `storage.googleapis.com` — domain Google yang terpercaya — dengan URL yang menunjuk ke file `aemmfcy1vxeo.htm1`, HTML file dengan nama acak yang di-masquerade. Teknik ini efektif karena domain Google sering lolos dari filter URL berbasis reputasi.

---

## 📚 References

- [LetsDefend — Phishing Email Challenge](https://app.letsdefend.io)
- [MITRE ATT&CK - T1566.002 Spearphishing Link](https://attack.mitre.org/techniques/T1566/002/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
