# EROS — BTLO

> **Platform:** Blue Team Labs Online  
> **Category:** Incident Response  
> **Tags:** AI, CLI, WebApp  
> **Difficulty:** Easy  
> **Status:** ✅ Completed  
> **Date:** 2026-06-07  
> **Time Spent:** ~X jam  

---

## 📌 Prolog

Lab ini adalah investigasi Incident Response (IR) ringan yang berfokus pada kompromi sebuah platform kencan butik. Skenario mencakup analisis forensik pada web application yang sedang berjalan secara live — kombinasi antara AI, CLI, dan WebApp sebagai attack surface yang harus ditelusuri.

---

## 🎯 Scenario

Kami adalah perusahaan jasa mak comblang (SME) yang baru saja menjadi korban serangan siber. Klien kami cukup high-profile, jadi implikasinya besar. Tugas kita adalah mengkonfirmasi root cause serangan dan menjawab beberapa pertanyaan penting untuk keperluan internal reporting.

Perlu diperhatikan: aplikasi sedang berjalan live di environment yang disediakan. Kita bebas membuat akun dan mengeksplorasi fungsionalitasnya untuk memahami permukaan serangan sebelum mulai investigasi.

---

## ❓ Questions

1. What type of attack caused the initial access?
2. At what time did the first successful attack occur?
3. What is the username of the account conducting the attack?
4. What is the name of the file & function that requires securing to prevent this specific initial access mechanism occurring again?
5. What is the username of the account the TA added to the OS?
6. What time was the username added by the TA?
7. We need to understand our risk here and consider regulatory reporting. How many users on the exist within the App that the TA may have had access to?
8. To support future detections, we need to add this IOC to the watchlist and threat intel records. What email address was associated with the TA?

---

## 🔍 Answer & Walkthrough

Investigasi dimulai dengan mengecek nginx access log untuk menemukan endpoint yang mencurigakan. Dari sana ditemukan endpoint `/admin/users/4273/make-admin` yang mengindikasikan adanya privilege escalation. Dengan melakukan trace pada `user_id=4273` di access log, ditemukan dua IP mencurigakan — `95.181.232.10` dan `158.173.24.3` — yang berinteraksi dengan profil tersebut.

`95.181.232.10` teridentifikasi sebagai attacker yang melakukan registrasi akun baru pada `2026-02-13 01:03:14`, mendapat `user_id=4273`. Identitas dikonfirmasi via database: `username=noone`, `email=noone@protonmail.com`.

Setelah registrasi, akun `noone` digunakan untuk berinteraksi dengan AI Dating Coach chatbot. Di `ai_interactions.log`, ditemukan bahwa attacker mengirim prompt injection payload — berpura-pura lupa admin credentials. AI kemudian membocorkan username admin `ErosAdmin2024` yang berasal dari function `get_system_prompt()` di `/opt/eros-io/app/routes/ai.py`. Credentials ini di-embed langsung ke system prompt tanpa sanitasi input.

Dengan credentials tersebut, attacker login sebagai admin, lakukan privilege escalation via `/admin/users/4273/make-admin`, eksekusi command melalui `/admin/diagnostics`, dan tambah user baru `eros-io` di OS level sebagai persistence.

---

### 1. What type of attack caused the initial access?

Attacker menggunakan AI Dating Coach chatbot sebagai vector. Mereka menyusun prompt yang berpura-pura lupa admin credentials, sehingga AI merespons dengan membocorkan password `ErosAdmin2024` yang ter-embed di system prompt.

![Initial Access - Prompt Injection](./assets/soal-1-3/1.get-initial-acces-attacker.png)

**Jawaban:** `Prompt Injection`

---

### 2. At what time did the first successful attack occur?

Ditemukan di `ai_interactions.log` — timestamp saat attacker berhasil mengekstrak credentials admin melalui chatbot.

**Jawaban:** `2026-02-13 01:12:35`

---

### 3. What is the username of the account conducting the attack?

Dikonfirmasi dari database setelah trace `user_id=4273` di access log.

**Jawaban:** `noone`

---

### 4. What is the name of the file & function that requires securing to prevent this specific initial access mechanism occurring again?

Function `get_system_prompt()` di file `ai.py` menyematkan credentials admin secara hardcode langsung ke dalam system prompt. Input dari user juga di-combine langsung ke prompt tanpa filter atau sanitasi, sehingga attacker bisa menyisipkan instruksi arbitrary ke AI.

![Vulnerable Function - get_system_prompt](./assets/soal-4/1.cek-ai-func.png)

**Jawaban:** `ai.py:get_system_prompt`

---

### 5. What is the username of the account the TA added to the OS?

Setelah berhasil masuk sebagai admin, attacker mengakses `/admin/diagnostics` untuk melakukan RCE, kemudian menambahkan user baru di OS level sebagai persistence.

![OS Account Creation](./assets/soal-5-6/1.cek-user-create.png)

**Jawaban:** `eros-io`

---

### 6. What time was the username added by the TA?

**Jawaban:** `2026-02-13 01:28:33`

---

### 7. How many users within the App that the TA may have had access to?

Langkah pertama adalah menemukan lokasi file SQL-nya, kemudian cek nama tabel — ternyata `users` bukan `user`. Setelah itu query langsung untuk menghitung total record.

![User Table Query](./assets/soal-7-8/1.cek-user-table.png)

**Jawaban:** `4277`

---

### 8. What email address was associated with the TA?

Dikonfirmasi dari tabel `users` di database saat query untuk menjawab soal 7.

**Jawaban:** `noone@protonmail.com`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| Username | `noone` | Akun attacker di aplikasi |
| Email | `noone@protonmail.com` | Email TA — tambahkan ke watchlist |
| IP Address | `95.181.232.10` | IP attacker (registrasi + serangan) |
| IP Address | `158.173.24.3` | IP kedua yang berinteraksi dengan profil TA |
| Password | `ErosAdmin2024` | Admin credentials yang bocor via AI |
| OS Username | `eros-io` | Akun persistence yang dibuat TA di OS |
| File | `/opt/eros-io/app/routes/ai.py` | File vulnerable |
| Function | `get_system_prompt` | Function yang perlu di-patch |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Initial Access | Exploit Public-Facing Application | T1190 | Exploit AI chatbot yang publicly accessible |
| Initial Access | LLM Prompt Injection | AML.T0051 | Manipulasi AI via crafted prompt untuk extract credentials |
| Credential Access | Unsecured Credentials | T1552 | Credentials admin ter-embed di system prompt AI |
| Privilege Escalation | Valid Accounts | T1078 | Gunakan credentials `ErosAdmin2024` untuk akses admin panel |
| Privilege Escalation | Account Manipulation | T1098 | Make-admin pada `user_id=4273` |
| Execution | Command and Scripting Interpreter | T1059 | RCE via `/admin/diagnostics` |
| Persistence | Create Account: Local Account | T1136.001 | Tambah user `eros-io` di OS level |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

Eros adalah dating platform berbasis Flask/Nginx yang dikompromi melalui **Prompt Injection** pada fitur AI Dating Coach. Attacker (`noone` / `noone@protonmail.com`) mendaftar akun baru untuk mendapat akses ke chatbot, lalu memanipulasi AI agar membocorkan credentials admin `ErosAdmin2024` yang ter-embed di system prompt tanpa sanitasi.

Dengan credentials tersebut, attacker login sebagai admin → privilege escalation via `/admin/users/4273/make-admin` → RCE via `/admin/diagnostics` → tambah user `eros-io` di OS sebagai persistence. Total **4277 user** berpotensi terekspos.

Root cause: `get_system_prompt()` di `/opt/eros-io/app/routes/ai.py` menyimpan sensitive credentials langsung di system prompt dan tidak memfilter input user sebelum di-inject ke prompt AI.

### Todo / Follow-up

- [ ] Pelajari lebih dalam tentang LLM Prompt Injection dan teknik mitigasinya (input sanitization, instruction hierarchy, system prompt separation)
- [ ] Eksplorasi ATLAS framework (AML.T0051) untuk pemetaan AI-specific attack techniques
- [ ] Pelajari cara audit Flask app — telusuri endpoint-endpoint admin dan bagaimana privilege escalation bisa terjadi via web panel
- [ ] Simulasikan RCE via diagnostic endpoint — pahami kenapa endpoint semacam itu berbahaya jika tidak di-protect dengan ketat

---

## 📚 References

- [MITRE ATT&CK - T1190 Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)
- [MITRE ATLAS - AML.T0051 LLM Prompt Injection](https://atlas.mitre.org/techniques/AML.T0051)
- [MITRE ATT&CK - T1136.001 Create Account: Local Account](https://attack.mitre.org/techniques/T1136/001/)
- [OWASP LLM Top 10 - LLM01: Prompt Injection](https://owasp.org/www-project-top-10-for-large-language-model-applications/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
