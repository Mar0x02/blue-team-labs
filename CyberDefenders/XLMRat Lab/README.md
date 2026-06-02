# XLMRat Lab — CyberDefenders

> **Platform:** CyberDefenders  
> **Category:** Network Forensics / Malware Analysis  
> **Difficulty:** Medium  
> **Status:** 🔄 In Progress  
> **Date:** -  
> **Time Spent:** -  
> **Tools:** CyberChef, Wireshark, VirusTotal, Python3, PowerShell  

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

> 🔄 *Belum diisi — akan dilengkapi setelah pengerjaan selesai.*

### 1. What is the URL from which the first malware stage was installed?

> 🔄 *Belum diisi.*

**Jawaban:** `...`

---

### 2. Which hosting provider owns the associated IP address?

> 🔄 *Belum diisi.*

**Jawaban:** `...`

---

### 3. What is the SHA256 of the malware executable?

> 🔄 *Belum diisi.*

**Jawaban:** `...`

---

### 4. What is the malware family label based on Alibaba?

> 🔄 *Belum diisi.*

**Jawaban:** `...`

---

### 5. What is the timestamp of the malware's creation?

> 🔄 *Belum diisi.*

**Jawaban:** `...`

---

### 6. Which LOLBin is leveraged for stealthy process execution?

> 🔄 *Belum diisi.*

**Jawaban:** `...`

---

### 7. List the names of the files dropped by the script.

> 🔄 *Belum diisi.*

**Jawaban:** `...`

---

## 🚨 Key Findings / IOCs

> 🔄 *Belum diisi.*

| Tipe | Value | Keterangan |
|------|-------|------------|
| URL | `...` | Download URL first stage malware |
| IP Address | `...` | IP attacker / hosting provider |
| File Hash | `...` | SHA256 malware executable |
| LOLBin | `...` | Binary yang dipakai untuk stealthy execution |

---

## 🗺️ MITRE ATT&CK Mapping

> 🔄 *Belum diisi.*

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Execution | ... | ... | ... |
| Defense Evasion | ... | ... | ... |

---

## 📋 Summary — Attacker Behavior & Todo

> 🔄 *Belum diisi — akan dilengkapi setelah analisis selesai.*

### Todo / Follow-up

- [ ] Selesaikan analisis PCAP di Wireshark — ekstrak payload dari traffic
- [ ] De-obfuscate script menggunakan CyberChef / Python3
- [ ] Lookup SHA256 di VirusTotal — identifikasi malware family dan label Alibaba
- [ ] Identifikasi LOLBin yang dipakai dan path-nya
- [ ] List semua file yang di-drop script
- [ ] Mapping semua teknik ke MITRE ATT&CK

---

## 📚 References

- [MITRE ATT&CK — LOLBins / Living Off the Land](https://attack.mitre.org/techniques/T1218/)
- [LOLBAS Project](https://lolbas-project.github.io/)
- [VirusTotal](https://www.virustotal.com/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
