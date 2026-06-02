# XLMRat Lab — CyberDefenders

> **Platform:** CyberDefenders  
> **Category:** Network Forensics / Malware Analysis  
> **Difficulty:** Medium  
> **Status:** ✅ Completed  
> **Date:** 2026-06-03  
> **Time Spent:** ~2 jam  
> **Tools:** CyberChef, Wireshark, tshark, VirusTotal, Python3, PowerShell  

---

## 📌 Prolog

Analisis network traffic untuk mengidentifikasi cara pengiriman malware, de-obfuscate script yang digunakan, dan mapping teknik attacker ke MITRE ATT&CK — dengan fokus pada stealthy execution dan reflective code loading.

---

## 🎯 Scenario

Sebuah mesin telah ditandai karena adanya suspicious network traffic. Tugasmu adalah menganalisis file PCAP untuk menentukan metode serangan, mengidentifikasi payload berbahaya, dan menelusuri timeline kejadian. Fokus pada bagaimana attacker mendapatkan akses, tools atau teknik apa yang digunakan, dan bagaimana malware beroperasi setelah compromise.

---

## ❓ Questions

1. The attacker successfully executed a command to download the first stage of the malware. What is the URL from which the first malware stage was installed?
2. Which hosting provider owns the associated IP address?
3. By analyzing the malicious scripts, two payloads were identified: a loader and a secondary executable. What is the SHA256 of the malware executable?
4. What is the malware family label based on Alibaba?
5. What is the timestamp of the malware's creation?
6. Which LOLBin is leveraged for stealthy process execution in this script? Provide the full path.
7. The script is designed to drop several files. List the names of the files dropped by the script.

---

## 🔍 Answer & Walkthrough

### 1. What is the URL from which the first malware stage was installed?

Analisis dimulai dengan tshark untuk cek semua TCP stream yang ada di PCAP — ada 3 stream. Follow stream 0: terlihat victim `10.1.9.101` melakukan koneksi ke IP `45.126.209.4` port `222` dengan path `xlm.txt`. Di response-nya, content sudah di-obfuscate (**T1027.010**) — ini adalah first stage malware.

![Stream 0 — koneksi victim ke C2 path xlm.txt](./assets/soal-1.png)

**Jawaban:** `http://45.126.209.4:222/mdm.jpg`

---

### 2. Which hosting provider owns the associated IP address?

Cek IP `45.126.209.4` di VirusTotal tab **Details** atau lewat ipinfo.io. Hosting provider yang tercatat di bawah IP tersebut adalah ReliableSite.

![VirusTotal / ipinfo — hosting provider](./assets/soal-2.png)

**Jawaban:** `ReliableSite.Net`

---

### 3. What is the SHA256 of the malware executable?

Follow stream 1 — response dari C2 saat download `mdm.jpg` berisi hexstring panjang, bukan binary image biasa. Extract semua file dari PCAP, hasilnya dua file: `xlm.txt` dan `mdm.jpg`.

Sebelum lanjut, cek magic byte `mdm.jpg`:

![Magic byte mdm.jpg — bukan file JPG biasa](./assets/soal-3.png)

Magic byte menunjukkan ini adalah Windows PE executable, bukan JPEG. Buat script Python sederhana untuk konversi hexstring ke file binary:

```python
with open("mdm.jpg", "r") as f:
    hex_data = f.read().strip()

with open("mdm.bin", "wb") as f:
    f.write(bytes.fromhex(hex_data))
```

Hasil binary di-hash SHA256:

```
sha256sum mdm.bin
```

**Jawaban:** `1eb7b02e18f67420f42b1d94e74f3b6289d92672a0fb1786c30c03d68e81d798`

---

### 4. What is the malware family label based on Alibaba?

Copy SHA256 dari soal 3, paste ke VirusTotal. Di tab **Detection**, cari baris **Alibaba**.

![VirusTotal — Alibaba label](./assets/soal-4.png)

**Jawaban:** `AsyncRat`

---

### 5. What is the timestamp of the malware's creation?

Masih di VirusTotal, buka tab **Details** lalu scroll ke section **History**. Cek field **Creation Time**.

![VirusTotal Details — Creation Time](./assets/soal-5.png)

**Jawaban:** `2023-10-30 15:08`

---

### 6. Which LOLBin is leveraged for stealthy process execution?

Follow stream dari packet download `mdm.jpg`. Di bagian bawah hexstring, ada PowerShell script. Di dalam script tersebut terlihat path ke binary Microsoft .NET yang digunakan untuk menjalankan malware tanpa spawn proses mencurigakan.

![Stream mdm.jpg — PowerShell script dengan RegSvcs](./assets/soal-6.png)

`RegSvcs.exe` adalah signed binary Microsoft yang legitimate — dipakai sebagai LOLBin untuk bypass deteksi (**T1218.009**).

**Jawaban:** `C:\Windows\Microsoft.NET\Framework\v4.0.30319\RegSvcs.exe`

---

### 7. List the names of the files dropped by the script.

Dari analisis script di dalam `mdm.jpg`, ada tiga file yang secara eksplisit di-drop ke sistem korban, masing-masing punya peran berbeda dalam execution chain.

**Jawaban:** `Conted.bat, Conted.ps1, Conted.vbs`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| IP C2 | `45.126.209.4` | Command & Control server attacker |
| Port | `222` | Port non-standard yang digunakan C2 |
| URL (Stage 1) | `http://45.126.209.4:222/xlm.txt` | First stage — obfuscated loader |
| URL (Stage 2) | `http://45.126.209.4:222/mdm.jpg` | Second stage — payload executable |
| Hosting Provider | `ReliableSite.Net` | Provider yang menampung IP C2 |
| SHA256 | `1eb7b02e18f67420f42b1d94e74f3b6289d92672a0fb1786c30c03d68e81d798` | Hash malware executable (mdm.jpg → binary) |
| Malware Family | `AsyncRat` | Label dari Alibaba via VirusTotal |
| LOLBin | `C:\Windows\Microsoft.NET\Framework\v4.0.30319\RegSvcs.exe` | Dipakai untuk stealthy process execution |
| Files Dropped | `Conted.bat, Conted.ps1, Conted.vbs` | File yang di-drop ke sistem korban |
| Victim | `10.1.9.101` | IP mesin yang terinfeksi |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Execution | User Execution | T1204 | Victim mengakses `xlm.txt` dari C2 |
| Execution | Command and Scripting Interpreter: PowerShell | T1059.001 | PowerShell dipakai di `xlm.txt` dan script dalam `mdm.jpg` |
| Defense Evasion | Command Obfuscation | T1027.010 | Content `xlm.txt` di-obfuscate sebelum dikirim |
| Defense Evasion | Obfuscated Files or Information: Embedded Payloads | T1027.009 | `mdm.jpg` menyimpan payload exe dalam bentuk hexstring |
| Defense Evasion | Signed Binary Proxy Execution: Regsvcs/Regasm | T1218.009 | `RegSvcs.exe` dipakai sebagai LOLBin |
| Command and Control | Ingress Tool Transfer | T1105 | Download `mdm.jpg` (payload) dari C2 server |
| Persistence | Scheduled Task/Job | T1053 | `Conted.vbs` mendaftarkan malware ke scheduled task |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

Victim `10.1.9.101` mengakses `xlm.txt` dari C2 `45.126.209.4:222` — kemungkinan dipicu oleh XLM macro di dokumen Office (sesuai nama challenge), meskipun initial access tidak terlihat langsung dari PCAP. Ini menandai tahap **Execution (T1204)**.

`xlm.txt` berisi PowerShell command yang di-obfuscate (**T1027.010, T1059.001**). Setelah di-decode, command tersebut mengunduh `mdm.jpg` dari C2 yang sama — **Ingress Tool Transfer (T1105)**. File ini bukan image; magic byte-nya menunjukkan Windows PE executable yang disembunyikan dalam hexstring (**T1027.009 - Embedded Payloads**).

Script di dalam `mdm.jpg` kemudian men-drop tiga file ke sistem:
- **`Conted.ps1`** — mengkonversi hexstring ke file binary executable (AsyncRat payload)
- **`Conted.bat`** — memanggil `powershell.exe` untuk mengeksekusi `Conted.ps1`
- **`Conted.vbs`** — mendaftarkan malware ke Scheduled Task agar selalu berjalan saat sistem restart (**T1053**)

Untuk eksekusi payload, attacker memanfaatkan `RegSvcs.exe` sebagai LOLBin (**T1218.009**) — binary Microsoft yang sudah di-sign, sehingga tidak memicu alert dari AV/EDR konvensional. Payload akhir yang berjalan adalah **AsyncRat**, sebuah Remote Access Trojan open-source yang umum dipakai attacker untuk persistent remote access.

### Todo / Follow-up

- [x] Analisis PCAP dengan tshark — identifikasi semua TCP stream
- [x] De-obfuscate content `xlm.txt`
- [x] Extract dan identifikasi `mdm.jpg` sebagai executable via magic byte
- [x] Konversi hexstring ke binary menggunakan Python
- [x] Lookup SHA256 di VirusTotal — identifikasi AsyncRat dan label Alibaba
- [x] Identifikasi `RegSvcs.exe` sebagai LOLBin
- [x] List file yang di-drop: `Conted.bat`, `Conted.ps1`, `Conted.vbs`
- [x] Mapping semua teknik ke MITRE ATT&CK
- [ ] Analisis lebih lanjut `Conted.ps1` — lihat payload AsyncRat yang di-generate
- [ ] Investigasi initial access — bagaimana `xlm.txt` pertama kali di-trigger (kemungkinan XLM macro di dokumen Office)
- [ ] Setup lab untuk simulasi AsyncRat C2 communication secara live

---

## 📚 References

- [MITRE ATT&CK — T1218.009: Regsvcs/Regasm](https://attack.mitre.org/techniques/T1218/009/)
- [MITRE ATT&CK — T1027.010: Command Obfuscation](https://attack.mitre.org/techniques/T1027/010/)
- [MITRE ATT&CK — T1027.009: Embedded Payloads](https://attack.mitre.org/techniques/T1027/009/)
- [MITRE ATT&CK — T1053: Scheduled Task](https://attack.mitre.org/techniques/T1053/)
- [LOLBAS — RegSvcs.exe](https://lolbas-project.github.io/lolbas/Binaries/Regsvcs/)
- [AsyncRat — GitHub](https://github.com/NYAN-x-CAT/AsyncRAT-C-Sharp)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
