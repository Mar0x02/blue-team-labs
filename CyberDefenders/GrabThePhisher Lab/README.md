# GrabThePhisher Lab — CyberDefenders

> **Platform:** CyberDefenders  
> **Category:** Threat Intelligence  
> **Difficulty:** Easy  
> **Status:** ✅ Completed  
> **Date:** 2026-08-08  
> **Time Spent:** ~14 menit  

---

## 📌 Prolog

Lab ini berfokus pada analisis sebuah phishing kit untuk cryptocurrency, dengan tujuan mengidentifikasi metode exfiltration yang dipakai, mengekstrak IOCs penting, dan mengumpulkan threat intelligence terkait pelaku di baliknya menggunakan local logs dan Telegram API.

---

## 🎯 Scenario

Sebuah platform decentralized finance (DeFi) menerima banyak keluhan dari user terkait unauthorized fund withdrawal. Investigasi forensik menemukan sebuah phishing site yang menyamar sebagai PancakeSwap exchange, yang memancing korban untuk memasukkan wallet seed phrase mereka. Phishing kit ini di-host di server yang sudah dikompromikan, dan mengeksfiltrasi kredensial korban lewat sebuah Telegram bot.

Tugasnya adalah melakukan threat intelligence analysis terhadap infrastruktur phishing ini, mengidentifikasi indicators of compromise (IoCs), dan melacak online presence dari attacker — termasuk alias dan Telegram identifier mereka — untuk memahami tactics, techniques, and procedures (TTPs) yang dipakai.

---

## ❓ Questions

1. Which wallet is used for asking the seed phrase?
2. What is the file name that has the code for the phishing kit?
3. In which language was the kit written?
4. What service does the kit use to retrieve the victim's machine information?
5. How many seed phrases were already collected?
6. Could you please provide the seed phrase associated with the most recent phishing incident?
7. Which medium was used for credential dumping?
8. What is the token for accessing the channel?
9. What is the Chat ID for the phisher's channel?
10. What are the allies of the phish kit developer?

---

## 🔍 Answer & Walkthrough

### Starting Point — Ekstrak & Orientasi Project

Ekstrak file yang disediakan lab, hasilnya berupa sebuah project web dengan framework Next.js. Project ini adalah web phishing yang meniru tampilan exchange crypto **PancakeSwap** untuk memancing korban connect wallet dan submit seed phrase mereka.

Fokus utama ada di folder `metamask` — karena wallet yang ditarget adalah Metamask. Di dalamnya ada dua file: `index.html` (halaman phishing form) dan `metamask.php` (backend logic penerima data).

---

### 1. Which wallet is used for asking the seed phrase?

Nama folder dan isi kode di `metamask.php` (baris `<b>Wallet:</b> Metamask`) sama-sama menunjukkan wallet yang ditarget.

**Jawaban:** `Metamask`

---

### 2. What is the file name that has the code for the phishing kit?

File PHP inilah yang menangani seluruh logic phishing kit — mulai dari geolocation lookup sampai exfiltration ke Telegram.

![metamask.php](./assets/phishing%20kit.png)

**Jawaban:** `metamask.php`

---

### 3. In which language was the kit written?

Dari extension file dan syntax-nya (`<?php ... ?>`), kit ini ditulis pakai PHP.

**Jawaban:** `PHP`

---

### 4. What service does the kit use to retrieve the victim's machine information?

Baris pertama `metamask.php` melakukan request ke `api.sypexgeo.net` menggunakan IP korban (`$_SERVER['REMOTE_ADDR']`) untuk resolve country dan city — layanan geolocation pihak ketiga bernama **Sypex Geo**.

```php
$request = file_get_contents("http://api.sypexgeo.net/json/".$_SERVER['REMOTE_ADDR']);
```

**Jawaban:** `Sypex Geo`

---

### 5. How many seed phrases were already collected?

Data yang berhasil dicuri disimpan lokal ke `log/log.txt` lewat `file_put_contents()` dengan flag `FILE_APPEND`. Buka file log tersebut — ada 3 baris entry seed phrase yang sudah ter-exfiltrate.

**Jawaban:** `3`

---

### 6. Could you please provide the seed phrase associated with the most recent phishing incident?

Karena tiap entry di `log.txt` ditambahkan secara append (entry terbaru ada di baris paling bawah), seed phrase dari incident paling recent ada di baris terakhir file log.

**Jawaban:** `father also recycle embody balance concert mechanic believe owner pair muffin hockey`

---

### 7. Which medium was used for credential dumping?

Fungsi `sendTel()` mengirim `$message` (berisi wallet phrase, IP, geo, dan user agent korban) ke Telegram Bot API via `api.telegram.org/bot<token>/sendMessage`.

**Jawaban:** `Telegram`

---

### 8. What is the token for accessing the channel?

Token bot Telegram hardcoded langsung di variabel `$token` dalam fungsi `sendTel()`.

**Jawaban:** `5457463144:AAG8t4k7e2ew3tTi0IBShcWbSia0Irvxm10`

---

### 9. What is the Chat ID for the phisher's channel?

Chat ID juga hardcoded, satu baris di atas `$token`, di variabel `$id`.

**Jawaban:** `5442785564`

---

### 10. What are the allies of the phish kit developer?

Di dalam comment block `metamask.php`, developer kit meninggalkan pesan "gift" buat sesama attacker beserta alias-nya.

**Jawaban:** `j1j1b1s@m3r0`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| File | `metamask.php` | Backend phishing kit yang menangani exfiltration |
| Service/Domain | `api.sypexgeo.net` | Third-party geolocation service dipakai kit |
| Telegram Bot Token | `5457463144:AAG8t4k7e2ew3tTi0IBShcWbSia0Irvxm10` | Token bot untuk kirim data curian |
| Telegram Chat ID | `5442785564` | Channel/grup tujuan exfiltration |
| Threat Actor Alias | `j1j1b1s@m3r0` | Alias developer kit, ditinggal di komentar kode |
| Log File | `log/log.txt` | Lokasi penyimpanan lokal hasil curian, berisi 3 entry seed phrase |
| Seed Phrase (terbaru) | `father also recycle embody balance concert mechanic believe owner pair muffin hockey` | Data korban paling recent yang ter-exfiltrate |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Initial Access | Phishing | T1566 | Web phishing menyamar sebagai exchange PancakeSwap |
| Initial Access | Phishing: Spearphishing Link | T1566.002 | Delivery attack lewat link menuju site phishing |
| Initial Access | Phishing: Spearphishing via Service | T1566.003 | Delivery attack lewat platform/service pihak ketiga |
| Defense Evasion | Impersonation | T1656 | Site meniru identitas visual PancakeSwap untuk menipu korban |
| Exfiltration | Exfiltration Over Webhook | T1567.004 | Data korban (seed phrase, IP, geo, user agent) dikirim ke Telegram Bot API |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

Kit ini adalah phishing web berbasis Next.js yang menyamar sebagai exchange crypto PancakeSwap, dengan fokus utama mencuri seed phrase wallet Metamask. Ketika korban submit seed phrase-nya lewat form phishing, data dikirim ke backend `metamask.php`.

Di backend, sebelum data diproses, kit lebih dulu melakukan lookup IP korban ke layanan **Sypex Geo** untuk dapat info country dan city. Data ini digabung dengan seed phrase (`$_POST["data"]`), IP address, dan user agent korban jadi satu pesan HTML. Pesan ini kemudian dikirim lewat fungsi `sendTel()` ke **Telegram Bot API** — memanfaatkan bot token dan chat ID yang sudah di-hardcode — sekaligus disimpan lokal ke `log/log.txt` sebagai backup.

Dari log tersebut, sudah ada 3 seed phrase korban yang berhasil dicuri. Developer kit ini juga meninggalkan jejak alias `j1j1b1s@m3r0` di komentar kode, kemungkinan sebagai signature atau ajakan ke sesama "hustler"/attacker lain yang memakai kit yang sama.

### Todo / Follow-up

- [ ] Cross-check alias `j1j1b1s@m3r0` di forum underground/Telegram untuk profiling threat actor lebih lanjut
- [ ] Pelajari Telegram Bot API sebagai exfiltration channel — bagaimana mendeteksi traffic semacam ini di network monitoring
- [ ] Cek isi `index.html` untuk lihat bagaimana form phishing meniru UI PancakeSwap/Metamask connect wallet
- [ ] Pelajari teknik deteksi hardcoded credential (bot token, chat ID) di static analysis phishing kit lain

---

## 📚 References

- [CyberDefenders — GrabThePhisher Lab](https://cyberdefenders.org/)
- [MITRE ATT&CK](https://attack.mitre.org/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
