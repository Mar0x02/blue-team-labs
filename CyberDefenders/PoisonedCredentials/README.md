# PoisonedCredentials — CyberDefenders

> **Platform:** CyberDefenders  
> **Category:** Network Forensics  
> **Difficulty:** Easy  
> **Status:** ✅ Completed  
> **Date:** 2026-05-16  
> **Time Spent:** ~1 jam  

---

## 📌 Prolog

Lab analisis PCAP untuk serangan LLMNR/NBT-NS poisoning — salah satu teknik credential harvesting paling umum di internal network. Serangan ini memanfaatkan typo dari user biasa dan protokol yang tidak punya mekanisme verifikasi. Tool utama yang dipakai attacker di dunia nyata untuk ini: **Responder**. Lab ini bagus untuk memahami bagaimana credential bisa bocor hanya karena salah ketik nama server.

---

## 🎯 Scenario

Tim keamanan mendeteksi lonjakan aktivitas jaringan yang mencurigakan. Ada indikasi serangan LLMNR dan NBT-NS poisoning terjadi di dalam jaringan — protokol ini dieksploitasi untuk menginterasep traffic dan mengkompromasi kredensial pengguna. Tugasmu: investigasi network log dan periksa captured network traffic.

---

## ❓ Questions

1. In the context of the incident described in the scenario, the attacker initiated their actions by taking advantage of benign network traffic from legitimate machines. Can you identify the specific mistyped query made by the machine with the IP address 192.168.232.162?
2. We are investigating a network security incident. To conduct a thorough investigation, we need to determine the IP address of the rogue machine. What is the IP address of the machine acting as the rogue entity?
3. As part of our investigation, identifying all affected machines is essential. What is the IP address of the second machine that received poisoned responses from the rogue machine?
4. We suspect that user accounts may have been compromised. What is the username of the account that the attacker compromised?
5. As part of our investigation, we aim to understand the extent of the attacker's activities. What is the hostname of the machine that the attacker accessed via SMB?

---

## 🔍 Answer & Walkthrough

### 1. What is the mistyped query made by 192.168.232.162?

```
ip.addr == 192.168.232.162 && (llmnr || nbns)
```

Filter traffic dari IP tersebut khusus protokol LLMNR dan NBNS. Di kolom Info langsung terlihat query `FILESHAARE` yang dikirim ke broadcast `224.0.0.252`. Typo-nya: huruf A double — `FILESHAARE` (harusnya `FILESHARE`). Karena nama ini tidak ada di DNS, sistem otomatis fallback ke LLMNR dan broadcast ke seluruh jaringan.

**Jawaban:** `FILESHAARE`

---

### 2. What is the IP address of the rogue machine?

```
llmnr
```

Amati pola response — IP `192.168.232.215` terus-terusan mengirim LLMNR response ke setiap host yang melakukan query. Karakteristik rogue machine dalam serangan ini: dia merespons *semua* query di jaringan, seolah-olah dia adalah server yang dicari.

Alur yang terjadi:
1. `192.168.232.162` broadcast: *"siapa FILESHAARE?"*
2. `192.168.232.215` (attacker) menjawab: *"aku FILESHAARE, koneksi ke aku"*
3. Korban percaya → kirim autentikasi NTLM → hash ter-capture

**Jawaban:** `192.168.232.215`

---

### 3. What is the IP of the second machine that received poisoned responses?

```
llmnr
```

Scroll hasil filter — cari IP lain selain `192.168.232.162` yang juga menerima poisoned response dari `192.168.232.215`. Ditemukan `192.168.232.176` melakukan query `prinetr` (typo dari `printer`) dan langsung direspons oleh rogue machine. Pola sama persis: typo → LLMNR broadcast → attacker jawab.

**Jawaban:** `192.168.232.176`

---

### 4. What is the username of the compromised account?

```
ntlmssp
```

Filter traffic NTLMSSP, lalu cari packet **NTLMSSP_AUTH** (fase ke-3 dalam NTLM handshake). Expand Session Id → field `Acct` menunjukkan `janesmith`, domain `cybercactus.local`, host `WORKSTATION`.

NTLM handshake terdiri dari tiga fase: NEGOTIATE → CHALLENGE → AUTH. Di fase AUTH inilah username dan NetNTLMv2 hash ter-capture oleh attacker.

**Jawaban:** `janesmith`

---

### 5. What is the hostname of the machine the attacker accessed via SMB?

```
ntlmssp.challenge.target_info
```

Filter packet NTLMSSP Challenge, expand Target Info → Attribute: NetBIOS computer name → nilai `ACCOUNTINGPC`. DNS computer name mengkonfirmasi: `AccountingPC.cybercactus.local`.

Field ini lebih akurat dibanding melihat Tree Connect path karena datang langsung dari identifikasi mesin di level autentikasi.

**Jawaban:** `ACCOUNTINGPC`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| IP (Attacker / Rogue) | `192.168.232.215` | Rogue machine yang poisoning LLMNR responses |
| IP (Victim 1) | `192.168.232.162` | Query typo: `FILESHAARE` |
| IP (Victim 2) | `192.168.232.176` | Query typo: `prinetr` |
| Username | `janesmith` | Akun yang credentialnya ter-capture |
| Domain | `cybercactus.local` | Domain internal organisasi |
| Hostname (target SMB) | `ACCOUNTINGPC` | Mesin yang diakses attacker via SMB |
| Protocol Exploited | LLMNR / NBT-NS | Protokol tanpa verifikasi yang dieksploitasi |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Credential Access | LLMNR/NBT-NS Poisoning and SMB Relay | T1557.001 | Rogue machine merespons broadcast LLMNR untuk capture NetNTLMv2 hash |
| Collection | Man-in-the-Middle | T1557 | Attacker memposisikan diri sebagai host yang dicari korban |
| Lateral Movement | Remote Services: SMB/Windows Admin Shares | T1021.002 | Akses ke `ACCOUNTINGPC` via SMB menggunakan credential `janesmith` |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

**Attack Chain:**

| # | Fase | Detail |
|---|------|--------|
| 1 | Passive Listening | `192.168.232.215` diam-diam mendengarkan semua broadcast traffic di jaringan |
| 2 | Trigger (Victim 1) | `192.168.232.162` typo query `FILESHAARE` → LLMNR broadcast ke `224.0.0.252` |
| 3 | Poisoning | Attacker menjawab broadcast seolah-olah dia adalah server yang dicari |
| 4 | Trigger (Victim 2) | `192.168.232.176` typo query `prinetr` → poisoned response yang sama |
| 5 | Credential Capture | Windows otomatis kirim NTLM auth → NetNTLMv2 hash `janesmith` ter-capture |
| 6 | Lateral Movement | Attacker akses `ACCOUNTINGPC` via SMB menggunakan credential yang sudah di-capture |

Hash yang didapat bisa dipakai dua cara: crack offline dengan hashcat (`-m 5600` untuk NetNTLMv2) untuk plaintext password, atau langsung dipakai dengan Pass-the-Hash attack tanpa perlu crack.

Kemungkinan initial access attacker ke dalam jaringan: phishing → malware → Responder berjalan di background di mesin karyawan yang terinfeksi. Karyawan tidak sadar mesinnya jadi rogue machine.

### Todo / Follow-up

- [ ] Coba reproduce serangan ini di lab dengan Responder + Wireshark
- [ ] Pelajari cara crack NetNTLMv2 hash dengan hashcat `-m 5600`
- [ ] Eksplorasi Pass-the-Hash attack setelah credential di-capture
- [ ] Review cara disable LLMNR via Group Policy dan NBT-NS via adapter settings
- [ ] Pelajari tool deteksi: cara monitor anomali LLMNR/NBNS response yang tidak wajar

---

## 📚 References

- [MITRE ATT&CK T1557.001 — LLMNR/NBT-NS Poisoning](https://attack.mitre.org/techniques/T1557/001/)
- [Responder — tool yang umum dipakai untuk serangan ini](https://github.com/lgandx/Responder)
- [Microsoft — Disable LLMNR via Group Policy](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/ee808927(v=ws.10))

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
