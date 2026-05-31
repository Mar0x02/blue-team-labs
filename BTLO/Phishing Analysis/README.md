# Phishing Analysis — BTLO

> **Platform:** Blue Team Labs Online  
> **Category:** Email Forensics / Phishing Analysis  
> **Difficulty:** Easy  
> **Status:** ✅ Completed  
> **Date:** 2026-05-31  
> **Time Spent:** ~1 jam  

---

## 📌 Prolog

Challenge email forensics — nganalisis email phishing yang diterusin user ke SOC. Materinya cukup straightforward: baca raw email header, trace originating IP, dan identifikasi teknik yang dipakai penyerang. Tools-nya ringan: Text Editor, Thunderbird, URL2PNG, dan WHOis.

---

## 🎯 Scenario

Seorang pengguna menerima email phishing dan meneruskannya ke SOC. Tugasnya adalah menginvestigasi email beserta attachment-nya untuk mengumpulkan artefak yang berguna guna mendukung proses analisis insiden.

---

## ❓ Questions

1. Who is the primary recipient of this email?
2. What is the subject of this email?
3. What is the date and time the email was sent?
4. What is the Originating IP?
5. Perform reverse DNS on this IP address, what is the resolved host?
6. What is the name of the attached file?
7. What is the URL found inside the attachment?
8. What service is this webpage hosted on?
9. Using URL2PNG, what is the heading text on this page?

---

## 🔍 Answer & Walkthrough

### 1. Who is the primary recipient of this email?

Langsung cek header email pada field **To:** — tidak perlu analisis lebih lanjut.

**Jawaban:** `kinnar1975@yahoo.co.uk`

---

### 2. What is the subject of this email?

Email yang diterima John Smith bukan email phishing langsung, melainkan email bounce/NDR (Non-Delivery Report) yang dikirim otomatis oleh Mail Delivery System karena email aslinya gagal terkirim ke `kinnar1975@yahoo.co.uk`. Subjectnya adalah subject dari bounce tersebut.

**Jawaban:** `Undeliverable: Website contact form submission`

---

### 3. What is the date and time the email was sent?

Ditemukan di header email pada field **Sent:** yang mencatat kapan Mail Delivery System mengirim notifikasi kegagalan ke John Smith.

**Jawaban:** `18 March 2021 04:14`

---

### 4. What is the Originating IP?

Buka file attachment `Website contact form submission.eml` dan baca raw email header-nya. Ada dua IP yang muncul di sini yang bisa bikin bingung:

- `91.90.123.43` → muncul di field `X-PHP-Script` sebagai `REMOTE_ADDR` — ini adalah IP penyerang yang mengisi contact form
- `103.9.171.10` → muncul di field `X-Originating-IP` — ini IP server hosting yang mengirim email

Yang dimaksud sebagai originating IP adalah IP yang tercatat secara resmi oleh mail server, bukan IP penyerang.

```
X-Originating-IP: 103.9.171.10
```

**Jawaban:** `103.9.171.10`

---

### 5. Perform reverse DNS on this IP address, what is the resolved host?

Lookup IP `103.9.171.10` di [whois.domaintools.com](https://whois.domaintools.com). Hasilnya konsisten dengan apa yang ada di raw header pada field `Received:`:

```
Received: from c5s2-1e-syd.hosting-services.net.au ([103.9.171.10])
```

**Jawaban:** `c5s2-1e-syd.hosting-services.net.au`

---

### 6. What is the name of the attached file?

Di raw header email, bagian `Content-Disposition` tidak punya field `filename=` yang eksplisit — attachment bertipe `message/rfc822` (embedded email) tanpa nama file yang dideklarasikan. Email client secara otomatis menggunakan **Subject** dari email yang di-attach sebagai nama file.

**Jawaban:** `Website contact form submission.eml`

---

### 7. What is the URL found inside the attachment?

Buka isi attachment dan cari di bagian body email — URL disisipkan pada field **Services Required** dari contact form yang disalahgunakan penyerang. Teknik ini dikenal sebagai **contact form abuse**: penyerang mengeksploitasi form di website legitimate untuk menyisipkan spam/phishing link agar lolos spam filter.

**Jawaban:** `https://35000usdperwwekpodf.blogspot.sg?p=9swghttps://35000usdperwwekpodf.blogspot.co.il?o=0hnd`

---

### 8. What service is this webpage hosted on?

Langsung identifiable dari domain URL: `35000usdperwwekpodf.blogspot.sg` dan `35000usdperwwekpodf.blogspot.co.il`. Keduanya pakai subdomain **blogspot** — platform blog gratis milik Google yang sering disalahgunakan karena domain-nya relatif dipercaya spam filter.

**Jawaban:** `blogspot`

---

### 9. Using URL2PNG, what is the heading text on this page?

Gunakan **URL2PNG** untuk screenshot URL tanpa harus mengunjunginya langsung. Hasilnya: blog sudah di-takedown oleh Google.

**Jawaban:** `Blog has been removed`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| IP Address | `103.9.171.10` | Originating IP — server hosting markgardner.com.au |
| IP Address | `91.90.123.43` | IP penyerang yang mengisi contact form |
| Domain | `www.markgardner.com.au` | Website legitimate yang contact form-nya disalahgunakan |
| Domain | `35000usdperwwekpodf.blogspot.sg` | Phishing URL (sudah di-takedown) |
| Domain | `35000usdperwwekpodf.blogspot.co.il` | Phishing URL (sudah di-takedown) |
| Email | `kinnar1975@yahoo.co.uk` | Target penerima email phishing |
| Email | `johnsmith123@gmail.com` | SOC member / pelapor |
| Hostname | `c5s2-1e-syd.hosting-services.net.au` | Reverse DNS dari originating IP |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Initial Access | Phishing | T1566 | Email phishing via contact form abuse |
| Initial Access | Phishing: Spearphishing Link | T1566.002 | Link blogspot disisipkan di body email |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

Penyerang menggunakan teknik **contact form abuse** — mengisi form kontak di website legitimate `www.markgardner.com.au` dengan menyisipkan spam link pada kolom *Services Required*. Website secara otomatis mengirim notifikasi pengisian form ke `kinnar1975@yahoo.co.uk` melalui server hosting `103.9.171.10`. Email gagal terkirim karena mailbox target sudah dinonaktifkan (error 554.30), sehingga Mail Delivery System mengirim bounce/NDR ke `johnsmith123@gmail.com`. Link yang disisipkan mengarah ke blogspot dengan iming-iming "earn $6500/day" — classic financial fraud via get-rich-quick scheme. Blog phishing-nya sendiri sudah di-takedown oleh Google.

### Todo / Follow-up

- [ ] Pelajari lebih dalam tentang contact form abuse dan bagaimana cara mendeteksinya di level email gateway
- [ ] Eksplorasi teknik analisis email header lebih lanjut — terutama cara membedakan originating IP vs relay IP di multi-hop email
- [ ] Review SPF/DKIM/DMARC sebagai mitigasi terhadap email spoofing dan form abuse

---

## 📚 References

- [BTLO — Phishing Analysis Challenge](https://blueteamlabs.online/home/challenge/phishing-analysis-f92ef500ce)
- [MITRE ATT&CK — T1566 Phishing](https://attack.mitre.org/techniques/T1566/)
- [MITRE ATT&CK — T1566.002 Spearphishing Link](https://attack.mitre.org/techniques/T1566/002/)
- [URL2PNG](https://www.url2png.com/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
