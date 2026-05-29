# BEC-ky — BTLO

> **Platform:** Blue Team Labs Online  
> **Category:** Email Forensics / BEC Investigation  
> **Difficulty:** Easy  
> **Status:** ✅ Completed  
> **Date:** 2026-05-29  
> **Time Spent:** ~2 jam  
> **Tags:** Azure, Email, Phishing, BEC  

---

## 📌 Prolog

Challenge ini menarik karena bukan soal malware atau PCAP — murni email forensics dan Azure Audit Log analysis. Skenarionya sangat realistis: BEC (Business Email Compromise) yang menarget CFO lewat phishing, lalu attacker langsung eksekusi transfer dana pensiun sebelum korban sadar. Artifact yang tersedia adalah export Azure Audit Logs dan beberapa email yang dicurigai relevan.

---

## 🎯 Scenario

Organisasi menghadapi insiden finansial berdampak tinggi yang menarget **dana pensiun perusahaan**. Dalam 48 jam terakhir, sejumlah besar transfer bank yang tampak sah telah diproses — memindahkan dana dari rekening pensiun ke rekening eksternal domestik maupun internasional.

Transfer tersebut tampak *authorized* karena menggunakan akun CFO — orang yang memang punya wewenang legitimate untuk approve transaksi. Artifacts yang tersedia: **Azure Audit Logs** dan beberapa **email yang relevan**.

---

## ❓ Questions

1. What is the source address of the initial phishing email sent to Becky?
2. What type of compromise has taken place?
3. What are the two IP addresses utilised by the threat actor?
4. What is the name of the bank the fraudulent transaction was sent to?
5. What is the name of the inbox folder created during the compromise?
6. What word is the first inbox rule looking for?

---

## 🔍 Answer & Walkthrough

### Aktor yang Terlibat

| Nama | Role | Email | Keterangan |
|------|------|-------|------------|
| Becky Lorray | CFO, Tempestas Energy | `becky.lorray@tempestasenergy.com` | Target utama — akun dikompromikan |
| Sabastian Hague | Director, Flanagans Pensions | `sabastian@flanaganspensions.co.uk` | Vendor yang akunnya digunakan sebagai vektor |
| Liam Fray | Internal, Tempestas Energy | `liam.fray@tempestasenergy.com` | Penerima komunikasi internal soal pension |
| Bernard Holland | Internal, Tempestas Energy | `bernard.holland@tempestasenergy.com` | Penerima komunikasi internal soal pension |
| Chinedu Okafor | — | — | Nama fiktif yang dipakai attacker untuk withdrawal |

---

### Attack Timeline

**01 Juli 2025, 21:25** — Becky Lorray mengirim email ke Liam dan Bernard, menetapkan bahwa semua withdrawal dari dana pensiun harus melewati approval langsung darinya via email. Attacker kemungkinan sudah memantau komunikasi dan menjadikan ini sebagai celah.

**02 Juli 2025, 12:54** — Sabastian (yang akunnya sudah dikompromikan) mengirim **form template withdrawal** ke Becky berisi field: Name of Withdrawal, Employee ID, Withdrawal Amount, Reason for Withdrawal, Requested Date. Ini mempersiapkan proses yang akan dieksploitasi.

**15:40** — **Email phishing dikirim** dari akun Sabastian ke Becky — undangan palsu *"Copilot Enterprise AI Pilot Programme"* dengan link:

```
https://login.portal.microsoft.copilotweb.co/GuKdDmBu
```

Domain `copilotweb.co` bukan milik Microsoft (aslinya `microsoft.com`). Ini **credential harvesting page** yang meniru tampilan Microsoft login — teknik typosquatting domain.

**15:41** — Attacker berhasil login ke akun Becky dari IP `159.203.17.81` (MacOS, Chrome). Hanya **1 menit** setelah phishing dikirim.

**15:45** — Attacker mengakses `\\Inbox` dan `\\Sent Items` dari IP `95.181.232.30`.

**15:48** — Attacker membuat **Inbox Rule pertama** (*Default Rule*):
- Kondisi: email dari `sabastian@flanaganspensions.co.uk` yang mengandung kata **"Withdrawal"** di subject/body
- Aksi: `DeleteMessage = True` — email langsung dihapus otomatis
- Tujuan: menyembunyikan semua komunikasi withdrawal dari Becky

**15:57** — Attacker mengirim **withdrawal request** dari akun Becky ke Sabastian:
- Nama: Chinedu Okafor (EMP00128)
- Jumlah: **£110,000**
- Tujuan: International Transfer
- SWIFT/BIC: `FBNINGLA` (First Bank of Nigeria Ltd)
- Nomor Rekening: `3025819476`
- Alasan: *Pension Business Expense Reimbursement*

**15:58** — Attacker membuat **Inbox Rule kedua** (*Default Rule 2*):
- Aksi: pindahkan email ke folder **"History"** dan forward ke `sabastian@flanaganspensions.co.uk`
- Tujuan: semua komunikasi terkait di-redirect agar Becky tidak melihat balasan

Total waktu dari phishing dikirim hingga withdrawal dieksekusi: **17 menit**.

---

### 1. What is the source address of the initial phishing email sent to Becky?

Dari analisis email, phishing dikirim menggunakan akun Sabastian yang sudah dikompromikan. Alamat ini dipilih karena Becky kenal dan percaya dengan Sabastian sebagai vendor pension yang sah.

**Jawaban:** `sabastian@flanaganspensions.co.uk`

---

### 2. What type of compromise has taken place?

Attacker tidak pakai malware — hanya phishing untuk ambil credentials, lalu manfaatkan kepercayaan dan akses email untuk eksekusi transfer finansial. Definisi klasik BEC.

**Jawaban:** `Business Email Compromise`

---

### 3. What are the two IP addresses utilised by the threat actor?

Dari Azure Audit Log, ada dua IP berbeda:
- `159.203.17.81` — digunakan saat login pertama (UserLoggedIn), OS: MacOS, Browser: Chrome
- `95.181.232.30` — digunakan saat mengakses mail items (MailItemsAccessed)

Dua IP berbeda mengindikasikan attacker menggunakan infrastruktur terpisah untuk initial access vs. operasi lanjutan.

**Jawaban:** `159.203.17.81, 95.181.232.30`

---

### 4. What is the name of the bank the fraudulent transaction was sent to?

Dari withdrawal request yang dikirim attacker menggunakan akun Becky — field SWIFT/BIC `FBNINGLA` adalah kode bank untuk First Bank of Nigeria Ltd.

**Jawaban:** `First Bank of Nigeria Ltd`

---

### 5. What is the name of the inbox folder created during the compromise?

Inbox Rule kedua memindahkan semua email terkait ke folder **"History"** — folder yang dibuat attacker untuk menyembunyikan balasan dari Sabastian agar Becky tidak melihatnya.

**Jawaban:** `History`

---

### 6. What word is the first inbox rule looking for?

Inbox Rule pertama (*Default Rule*) dikonfigurasi untuk mendeteksi email dari `sabastian@flanaganspensions.co.uk` yang mengandung kata ini di subject atau body — lalu langsung menghapusnya.

**Jawaban:** `Withdrawal`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| Email | `sabastian@flanaganspensions.co.uk` | Akun vendor yang dikompromikan, dipakai sebagai vektor phishing |
| URL | `https://login.portal.microsoft.copilotweb.co/GuKdDmBu` | Credential harvesting page — typosquatting domain Microsoft |
| Domain | `copilotweb.co` | Domain phishing yang meniru Microsoft |
| IP Address | `159.203.17.81` | IP login pertama attacker (MacOS, Chrome) |
| IP Address | `95.181.232.30` | IP akses email attacker |
| SWIFT/BIC | `FBNINGLA` | First Bank of Nigeria Ltd — tujuan transfer |
| Account | `3025819476` | Nomor rekening tujuan |
| Amount | `£110,000` | Jumlah yang ditransfer |
| Identity | `Chinedu Okafor` | Nama fiktif yang dipakai attacker untuk withdrawal |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Initial Access | Phishing: Spearphishing Link | T1566.002 | Email phishing berisi link credential harvesting |
| Credential Access | Steal Web Session Cookie / Credentials | T1539 | Credential harvesting via fake Microsoft login page |
| Defense Evasion | Hide Artifacts: Email Hiding Rules | T1564.008 | Inbox rules untuk delete dan redirect email terkait withdrawal |
| Collection | Email Collection | T1114 | Akses ke Inbox dan Sent Items korban |
| Impact | Financial Theft | T1657 | Transfer £110,000 ke rekening eksternal via BEC |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

Attacker terlebih dahulu mengkompromikan akun vendor (`sabastian@flanaganspensions.co.uk`) untuk mendapatkan kredibilitas. Setelah memahami proses approval withdrawal dari komunikasi internal yang terpantau, mereka mengirim phishing via akun Sabastian ke CFO Becky Lorray dengan pretext *Microsoft Copilot pilot program*. Becky mengklik link dan credentials-nya dicuri — attacker login dalam 1 menit.

Dari sana, attacker bergerak cepat: buat inbox rule untuk menyembunyikan komunikasi terkait withdrawal, kirim withdrawal request £110,000 atas nama entitas fiktif ke First Bank of Nigeria, lalu buat inbox rule kedua untuk redirect balasan ke folder tersembunyi. Seluruh operasi selesai dalam **17 menit**.

### Todo / Follow-up

- [ ] Pelajari cara baca Azure Audit Log secara lebih mendalam — field apa saja yang relevan untuk investigasi BEC
- [ ] Cari tahu cara deteksi inbox rule creation secara real-time di Microsoft 365 Defender / Sentinel
- [ ] Explore teknik mitigasi BEC: MFA enforcement, anomalous login alert, inbox rule monitoring
- [ ] Lookup IP `159.203.17.81` dan `95.181.232.30` di AbuseIPDB dan VirusTotal
- [ ] Pelajari SWIFT/BIC lookup untuk mengenali pola rekening fraud

---

## 📚 References

- [MITRE ATT&CK T1566.002 — Spearphishing Link](https://attack.mitre.org/techniques/T1566/002/)
- [MITRE ATT&CK T1564.008 — Email Hiding Rules](https://attack.mitre.org/techniques/T1564/008/)
- [MITRE ATT&CK T1657 — Financial Theft](https://attack.mitre.org/techniques/T1657/)
- [Microsoft: Investigate BEC in Microsoft 365](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/investigate-malicious-email-that-was-delivered)
- [FBI IC3: Business Email Compromise](https://www.ic3.gov/PSA/2023/PSA230609)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
