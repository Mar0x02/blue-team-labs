# ThePackages — BTLO

> **Platform:** Blue Team Labs Online  
> **Category:** OSINT / Google Dorking  
> **Difficulty:** Easy  
> **Status:** 🔄 In Progress  
> **Date:** -  
> **Time Spent:** -  

---

## 📌 Prolog

🔄 *Belum diisi — akan dilengkapi setelah pengerjaan selesai.*

---

## 🎯 Scenario

Pihak berwenang sedang mencari seorang hacker yang berencana menjual sebuah perangkat canggih, dan pengirimannya akan dilakukan di lokasi yang dirahasiakan namun ramai. Penting untuk mengetahui produk apa itu dan mendapatkan lebih banyak informasi tentang hacker tersebut. Challenge ini dapat diselesaikan menggunakan Google dan perintah-perintahnya (Google Dorking).

**File:** `Authorities Info` (password: `btlo`)

---

## ❓ Questions

1. What is the name of the WiFi hacking device? (Format: Wifi X)
2. What is the name of the website containing the information of delivery location? (Format: WebsiteName)
3. 🔄 *Belum diisi*

---

## 🔍 Answer & Walkthrough

### 1. What is the name of the WiFi hacking device?

Google Dorking dengan keyword yang dikombinasikan dari clue "fruit" dan konteks WiFi hacking device:

```
"wifi" "hacking device" "fruit" "pineapple" "device" for sale
```

WiFi Pineapple adalah device buatan Hak5 yang umum dipakai untuk serangan MITM dan penetration testing. Keyword "related to a fruit" di clue merujuk ke kata "Pineapple".

**Jawaban:** `Wifi Pineapple`

---

### 2. What is the name of the website containing the information of delivery location?

Dari additional info, si hacker dikenal sebagai good developer. Dorking dengan username yang ditemukan dari file attachment:

```
"jllerenac" inurl:github.com
```

GitHub adalah platform paling umum yang dipakai developer untuk profil publik, termasuk info lokasi.

**Jawaban:** `GitHub`

---

### 3. 🔄 *Belum diisi*

> 🔄 *Akan dilengkapi setelah pengerjaan selesai.*

---

## 🚨 Key Findings / IOCs

> 🔄 *Belum diisi.*

| Tipe | Value | Keterangan |
|------|-------|------------|
| Username | `jllerenac` | Username hacker ditemukan dari attachment |
| Device | `WiFi Pineapple` | Device yang dijual hacker |

---

## 🗺️ MITRE ATT&CK Mapping

> 🔄 *Belum diisi.*

---

## 📋 Summary — Attacker Behavior & Todo

> 🔄 *Belum diisi — akan dilengkapi setelah analisis selesai.*

### Todo / Follow-up

- [ ] Selesaikan sisa pertanyaan challenge
- [ ] Lengkapi attacker profile dari OSINT yang ditemukan
- [ ] Dokumentasikan semua dork query yang dipakai

---

## 📚 References

- [Google Hacking Database (GHDB)](https://www.exploit-db.com/google-hacking-database)
- [Hak5 WiFi Pineapple](https://shop.hak5.org/products/wifi-pineapple)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
