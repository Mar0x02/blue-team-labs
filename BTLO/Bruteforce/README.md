# Bruteforce — BTLO

> **Platform:** Blue Team Labs Online  
> **Category:** Log Analysis / Incident Response  
> **Difficulty:** Easy  
> **Status:** ✅ Completed  
> **Date:** 2026-05-29  
> **Time Spent:** ~1 jam  

---

## 📌 Prolog

Challenge ini muncul waktu lagi eksplorasi kategori Log Analysis di BTLO. Premisnya simpel — ada file CSV dari Windows Security Event Log, dan tugasnya adalah menganalisis pola serangan RDP Brute Force dari log tersebut. Menariknya, BTLO kasih kebebasan soal tool yang dipakai — bisa Python, Excel, atau apapun yang bisa parse CSV.

---

## 🎯 Scenario

Salah satu system administrator mendeteksi jumlah event **Audit Failure** yang sangat besar di Windows Security Event Log. Tugas kita adalah menganalisis log tersebut untuk mengidentifikasi detail serangan brute force yang sedang berlangsung terhadap layanan RDP.

---

## ❓ Questions

1. How many Audit Failure events are there?
2. What is the username of the local account being targeted?
3. What is the failure reason related to the Audit Failure logs?
4. What is the Windows Event ID associated with these logon failures?
5. What is the source IP conducting this attack?
6. What country is this IP address associated with?
7. What is the range of source ports used by the attacker?

---

## 🔍 Answer & Walkthrough

File lab berformat CSV — export dari Windows Security Event Log berisi ribuan event dengan field seperti `Keywords`, `Event ID`, `Message`, dll. Semua event yang relevan adalah event **4625 (Logon Failure)** dengan keyword `Audit Failure`.

Ada dua pendekatan yang dipakai: Python script untuk automasi penuh, dan Excel untuk cross-check manual.

### Cara 1 — Python Script

Script ini baca file CSV sekaligus, lalu pakai **regex** untuk extract field dari kolom `Message` yang multi-line. Satu run langsung jawab semua pertanyaan.

```python
import re
from collections import Counter

FILE = "BTLO_Bruteforce_Challenge.csv"

with open(FILE, "r", encoding="utf-8", errors="ignore") as f:
    content = f.read()

# Q1: Count Audit Failure events
q1 = len(re.findall(r"Audit Failure", content))

# Q2: Username targeted
target_accounts = re.findall(r"Account For Which Logon Failed:.*?Account Name:\s+(\S+)", content, re.DOTALL)
q2 = Counter(target_accounts).most_common(1)[0][0]

# Q3: Failure reason
reasons = re.findall(r"Failure Reason:\s+(.+)", content)
q3 = Counter(r.strip() for r in reasons).most_common(1)[0][0]

# Q4: Event ID
event_ids = re.findall(r"Audit Failure,.*?,(\d{4}),", content)
q4 = Counter(event_ids).most_common(1)[0][0]

# Q5: Source IP
ips = re.findall(r"Source Network Address:\s+([\d.]+)", content)
q5 = Counter(ips).most_common(1)[0][0]

# Q7: Source port range
ports = [int(p) for p in re.findall(r"Source Port:\s+(\d+)", content) if int(p) > 0]
q7 = f"{min(ports)}-{max(ports)}"

print(f"Q1 - Audit Failure count : {q1}")
print(f"Q2 - Targeted username   : {q2}")
print(f"Q3 - Failure reason      : {q3}")
print(f"Q4 - Event ID            : {q4}")
print(f"Q5 - Source IP           : {q5}")
print(f"Q6 - Country             : Vietnam (via iplocation.net)")
print(f"Q7 - Source port range   : {q7}")
```

Output:

```
Q1 - Audit Failure count : 3103
Q2 - Targeted username   : administrator
Q3 - Failure reason      : Unknown user name or bad password.
Q4 - Event ID            : 4625
Q5 - Source IP           : 113.161.192.227
Q6 - Country             : Vietnam
Q7 - Source port range   : 49162-65534
```

---

### Cara 2 — Excel / Google Sheets

Buka file CSV di Excel, lalu gunakan pendekatan per pertanyaan:

- **Q1** → `=COUNTIF(A:A,"Audit Failure")` pada kolom `Keywords`
- **Q2** → `Ctrl+F` → cari `Account For Which Logon Failed` → baca `Account Name` di bawahnya
- **Q3** → `Ctrl+F` → cari `Failure Reason:` → baca nilainya
- **Q4** → kolom `Event ID` sudah ada → `=MODE(D:D)` untuk yang paling sering muncul
- **Q5** → `Ctrl+F` → cari `Source Network Address:` → baca IP-nya
- **Q6** → lookup IP `113.161.192.227` di [iplocation.net](https://www.iplocation.net/ip-lookup)
- **Q7** → extract semua `Source Port:` ke kolom baru → `=MIN()` dan `=MAX()`

---

### 1. How many Audit Failure events are there?

Hitung kemunculan string `Audit Failure` di kolom `Keywords` — langsung pakai `COUNTIF` atau `re.findall`.

**Jawaban:** `3103`

---

### 2. What is the username of the local account being targeted?

Di dalam kolom `Message`, ada section `Account For Which Logon Failed:` diikuti `Account Name:`. Semua event mengarah ke satu username yang sama.

**Jawaban:** `administrator`

---

### 3. What is the failure reason related to the Audit Failure logs?

Field `Failure Reason:` di dalam message log menyebutkan alasan gagalnya autentikasi secara konsisten di semua event.

**Jawaban:** `Unknown user name or bad password.`

---

### 4. What is the Windows Event ID associated with these logon failures?

Event ID untuk failed logon di Windows adalah standar — bisa dilihat langsung di kolom `Event ID` pada CSV.

**Jawaban:** `4625`

---

### 5. What is the source IP conducting this attack?

Field `Source Network Address:` di message log menunjukkan satu IP yang muncul di seluruh 3103 event — konsisten dari sumber tunggal.

**Jawaban:** `113.161.192.227`

---

### 6. What country is this IP address associated with?

Lookup IP `113.161.192.227` di [iplocation.net](https://www.iplocation.net/ip-lookup) — hasilnya menunjuk ke Vietnam.

**Jawaban:** `Vietnam`

---

### 7. What is the range of source ports used by the attacker?

Extract semua nilai `Source Port:` (exclude port 0), lalu ambil nilai minimum dan maksimum. Port range yang lebar ini tipikal dari tool brute force yang spawn koneksi secara masif dari ephemeral ports.

**Jawaban:** `49162-65534`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| IP Address | `113.161.192.227` | Source IP penyerang — Vietnam |
| Username | `administrator` | Target akun lokal yang diserang |
| Event ID | `4625` | Windows failed logon event |
| Port Range | `49162-65534` | Ephemeral ports dari tool brute force |
| Total Events | `3103` | Jumlah percobaan login yang gagal |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Credential Access | Brute Force: Password Guessing | T1110.001 | Percobaan login berulang ke akun `administrator` via RDP |
| Initial Access | Exploit Public-Facing Application | T1190 | RDP (port 3389) exposed ke internet sebagai attack surface |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

Penyerang dari IP `113.161.192.227` (Vietnam) melancarkan RDP brute force attack terhadap akun `administrator` secara masif — tercatat **3103 percobaan gagal** dalam satu sesi. Serangan dilakukan dari ephemeral ports `49162–65534`, pola yang konsisten dengan tool brute force otomatis. Failure reason `Unknown user name or bad password` mengindikasikan password-based attack, bukan credential stuffing dengan username enumeration terpisah.

Semua hit berasal dari satu IP tunggal — tidak ada rotasi IP, yang berarti tidak ada upaya untuk evade IP-based blocking.

### Todo / Follow-up

- [ ] Coba analisis timeline event — apakah ada pola burst/interval yang bisa identify tool yang dipakai
- [ ] Pelajari cara setup Windows Event Log forwarding ke SIEM (misalnya Splunk/Elastic) untuk deteksi real-time
- [ ] Explore teknik mitigasi: Account Lockout Policy, RDP rate limiting, VPN gateway untuk restrict RDP exposure
- [ ] Cari tahu apakah IP ini ada di threat intel feed (AbuseIPDB, VirusTotal, dll)

---

## 📚 References

- [Microsoft Event ID 4625 — An account failed to log on](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625)
- [MITRE ATT&CK T1110.001 — Brute Force: Password Guessing](https://attack.mitre.org/techniques/T1110/001/)
- [iplocation.net — IP Geolocation Lookup](https://www.iplocation.net/ip-lookup)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
