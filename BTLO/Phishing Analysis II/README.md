# Phishing Analysis II — BTLO

> **Platform:** Blue Team Labs Online  
> **Category:** Email Forensics / Phishing Analysis  
> **Difficulty:** Easy  
> **Status:** ✅ Completed  
> **Date:** 2026-05-07  
> **Time Spent:** 🔄 Belum diisi  

---

## 📌 Prolog

Lanjutan dari Phishing Analysis pertama, kali ini materinya lebih dalam — bukan hanya baca header, tapi juga decode body email, analisis HTML source, dan temukan artefak tersembunyi di dalam konten email itu sendiri. Email phishing-nya lebih sophisticated: lolos SPF/DKIM/DMARC, pakai SafeLinks Microsoft, dan ada hidden Facebook profile di dalam HTML.

---

## 🎯 Scenario

Sebuah email phishing diterima oleh target yang berpura-pura menjadi notifikasi resmi dari Amazon. Email ini mengklaim bahwa akun korban telah dikunci dan meminta korban untuk mengklik tombol "Review Account" yang mengarah ke situs phishing.

---

## ❓ Questions

1. What is the sending email address?
2. What is the recipient email address?
3. What is the subject line of the email?
4. What company is the attacker trying to imitate?
5. What is the date and time the email was sent?
6. What is the URL of the main call-to-action button?
7. Look at the URL using URL2PNG. What is the first sentence displayed?
8. What encoding scheme is being used?
9. What is the URL used to retrieve the company's logo?
10. What is the Facebook username based on the URL?

---

## 🔍 Answer & Walkthrough

### Initial Preparation

Buka file `.eml` di text editor untuk baca raw header dan HTML body secara langsung. Tools tambahan: Thunderbird untuk tampilan rendered, URL2PNG untuk preview URL secara aman.

---

### 1. What is the sending email address?

Terlihat di header field `From:` — nama display-nya "Amazn" (typo, bukan "Amazon"), tapi yang penting adalah alamat email-nya yang pakai domain mencurigakan, bukan domain Amazon resmi.

```
From: Amazn <amazon@zyevantoby.cn>
```

**Jawaban:** `amazon@zyevantoby.cn`

---

### 2. What is the recipient email address?

Dari field `To:` di header.

```
To: saintington73 <saintington73@outlook.com>
```

**Jawaban:** `saintington73@outlook.com`

---

### 3. What is the subject line of the email?

```
Subject: Your Account has been locked
```

**Jawaban:** `Your Account has been locked`

---

### 4. What company is the attacker trying to imitate?

Jelas dari konten email — nama "Amazon", logo Amazon, tombol bergaya Amazon, dan footer `Copyright © 1999-2021 Amazon. All rights reserved.` Semua elemen visual dirancang meniru tampilan email resmi Amazon.

**Jawaban:** `Amazon`

---

### 5. What is the date and time the email was sent?

```
Date: Wed, 14 Jul 2021 01:40:32 +0900
```

**Jawaban:** `Wed, 14 Jul 2021 01:40:32 +0900`

---

### 6. What is the URL of the main call-to-action button?

Tombol "Review Account" di HTML email mengandung dua URL — satu yang wrapped oleh Microsoft SafeLinks (ini yang muncul sebagai `href`), dan satu lagi URL aslinya yang tersimpan di atribut `originalSrc`.

**SafeLinks URL (href):**
```
https://emea01.safelinks.protection.outlook.com/?url=https%3A%2F%2Famaozn.zzyuchengzhika.cn%2F%3Fmailtoken%3Dsaintington73%40outlook.com&data=04...
```

**URL asli (originalSrc):**
```
https://amaozn.zzyuchengzhika.cn/?mailtoken=saintington73@outlook.com
```

> 🚩 Typosquatting klasik: **amaozn** (bukan amazon) — huruf 'a' dan 'o' ditukar.

**Jawaban:** `https://amaozn.zzyuchengzhika.cn/?mailtoken=saintington73@outlook.com`

---

### 7. Look at the URL using URL2PNG. What is the first sentence displayed?

Cek URL dengan URL2PNG untuk screenshot tanpa membuka langsung. Domain sudah tidak aktif saat dicek.

**Jawaban:** `this web page could not be loaded`

---

### 8. What encoding scheme is being used?

Terlihat di header body email:

```
Content-Transfer-Encoding: base64
```

**Jawaban:** `Base64`

---

### 9. What is the URL used to retrieve the company's logo?

Di dalam HTML body, ada tag `<IMG>` yang memuat logo Amazon dari Squarespace CDN — bukan dari server Amazon. Ini indikasi tambahan bahwa email ini bukan dari Amazon resmi.

```
https://images.squarespace-cdn.com/content/52e2b6d3e4b06446e8bf13ed/1500584238342-OX2L298XVSKF8AO6I3SV/amazon-logo?format=750w&content-type=image%2Fpng
```

**Jawaban:** `https://images.squarespace-cdn.com/content/52e2b6d3e4b06446e8bf13ed/1500584238342-OX2L298XVSKF8AO6I3SV/amazon-logo?format=750w&content-type=image%2Fpng`

---

### 10. What is the Facebook username based on the URL?

Di dalam HTML email, teks "Amazon Support Team" di footer sebenarnya adalah hyperlink yang mengarah bukan ke Amazon, melainkan ke profil Facebook pribadi attacker.

```
https://www.facebook.com/amir.boyka.7
```

**Jawaban:** `amir.boyka.7`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| Email (Sender) | `amazon@zyevantoby.cn` | Domain phishing yang meniru Amazon |
| Email (Target) | `saintington73@outlook.com` | Korban |
| IP Sender | `45.156.23.138` | Originating IP dari header `Received-SPF` |
| Domain | `zyevantoby.cn` | Domain pengirim phishing |
| Domain (Phishing) | `amaozn.zzyuchengzhika.cn` | Landing page phishing — typosquatting "amazon" |
| URL | `https://amaozn.zzyuchengzhika.cn/?mailtoken=saintington73@outlook.com` | URL tombol "Review Account" |
| Facebook | `https://www.facebook.com/amir.boyka.7` | Profil FB attacker yang tersembunyi di HTML |
| Logo CDN | Squarespace CDN (`52e2b6d3e4b06446e8bf13ed`) | Logo Amazon diambil dari CDN pihak ketiga |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Initial Access | Phishing: Spearphishing Link | T1566.002 | Email berisi link "Review Account" ke situs phishing |
| Credential Access | Steal Web Session Cookie / Credentials | T1539 | Landing page dirancang untuk harvest credential Amazon |
| Defense Evasion | Masquerading | T1036 | Domain `zyevantoby.cn`, nama display "Amazn", typosquatting `amaozn` |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

Attacker menyiapkan domain `zyevantoby.cn` dengan konfigurasi SPF, DKIM, dan DMARC yang valid sehingga email bisa melewati filter Microsoft Exchange. Email dikirim dengan display name "Amazn" dan visual yang meniru Amazon — logo, warna tombol oranye, footer copyright Amazon.

Target (`saintington73@outlook.com`) menerima email dengan subject "Your Account has been locked" yang menciptakan urgensi palsu. Tombol "Review Account" mengarah ke `amaozn.zzyuchengzhika.cn` — domain typosquatting dengan mailtoken target yang di-embed di URL, kemungkinan untuk personalisasi landing page phishing.

Menariknya, attacker meninggalkan jejak di HTML email: link "Amazon Support Team" di footer sebenarnya mengarah ke profil Facebook pribadi `amir.boyka.7` — kemungkinan kelalaian yang bisa digunakan sebagai attribution.

**Attack chain:**  
`Domain setup (SPF/DKIM/DMARC)` → `Email spoofing "Amazon"` → `Social engineering (account locked)` → `Redirect ke phishing site` → `Credential harvesting`

### Todo / Follow-up

- [ ] Pelajari cara kerja **Microsoft SafeLinks** — bagaimana cara attacker membypass atau memanfaatkannya, dan bagaimana investigator bisa decode URL yang di-wrap
- [ ] Eksplorasi teknik **typosquatting detection** — tools dan teknik untuk identifikasi domain yang meniru brand terkenal
- [ ] Review SPF/DKIM/DMARC pass di email phishing — kenapa email bisa lolos filter meski jelas bukan dari Amazon
- [ ] Cek teknik attribution dari **Facebook profile leak** di dalam HTML email — ini termasuk OSINT yang valuable

---

## 📚 References

- [BTLO — Phishing Analysis 2](https://blueteamlabs.online/home/challenge/phishing-analysis-2-a1091574b8)
- [MITRE ATT&CK — T1566.002 Spearphishing Link](https://attack.mitre.org/techniques/T1566/002/)
- [MITRE ATT&CK — T1036 Masquerading](https://attack.mitre.org/techniques/T1036/)
- [Microsoft SafeLinks documentation](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/safe-links-about)
- [URL2PNG](https://www.url2png.com/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
