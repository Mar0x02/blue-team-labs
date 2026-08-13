# 3CX Supply Chain Lab — CyberDefenders

> **Platform:** CyberDefenders
> **Category:** Threat Intelligence
> **Difficulty:** Easy
> **Status:** 🔄 In Progress
> **Date:** 2026-08-13
> **Time Spent:** ~45 menit

---

## 📌 Prolog

Merekonstruksi 3CX supply chain attack dengan menganalisis compromised MSI dan DLL artifact buat identifikasi TTPs dan atribusi insiden ke threat actor.

**Tools:** VirusTotal

**Tactics yang tercakup:** Execution | Stealth | Discovery

**Catatan:** lab ini berstatus *Retired* di platform CyberDefenders.

---

## 🎯 Scenario

Sebuah perusahaan multinasional besar sangat bergantung sama software 3CX buat komunikasi telepon, jadi ini komponen krusial dalam operasional bisnis mereka. Setelah update terbaru ke 3CX Desktop App, antivirus alert nunjukkin beberapa instance software ini ke-wipe di sebagian workstation, sementara yang lain nggak kena. Tim IT awalnya nganggep ini false positive dan skip alert-nya, sampai akhirnya nyadar ada performance yang menurun dan network traffic aneh ke server yang nggak dikenal. Karyawan mulai lapor masalah di app 3CX, dan tim IT security ngidentifikasi pola komunikasi nggak wajar yang terkait sama update software terbaru.

Sebagai threat intelligence analyst, tugas lo adalah investigasi kemungkinan supply chain attack ini. Objektif lo: ungkap gimana caranya attacker berhasil compromise app 3CX, identifikasi threat actor yang terlibat, dan assess seberapa luas dampak insiden ini.

---

## ❓ Questions

1. Understanding the scope of the attack and identifying which versions exhibit malicious behavior is crucial for making informed decisions if these compromised versions are present in the organization. How many versions of 3CX running on Windows have been flagged as malware?
2. Determining the age of the malware can help assess the extent of the compromise and track the evolution of malware families and variants. What's the UTC creation time of the .msi malware?
3. Executable files (.exe) are frequently used as primary or secondary malware payloads, while dynamic link libraries (.dll) often load malicious code or enhance malware functionality. Analyzing files deposited by the Microsoft Software Installer (.msi) is crucial for identifying malicious files and investigating their full potential. Which malicious DLLs were dropped by the .msi file?
4. Recognizing the persistence techniques used in this incident is essential for current mitigation strategies and future defense improvements. What is the MITRE Technique ID employed by the .msi files to load the malicious DLL?
5. Recognizing the malware type (threat category) is essential to your investigation, as it can offer valuable insight into the possible malicious actions you'll be examining. What is the threat category of the two malicious DLLs?
6. As a threat intelligence analyst conducting dynamic analysis, it's vital to understand how malware can evade detection in virtualized environments or analysis systems. This knowledge will help you effectively mitigate or address these evasive tactics. What is the MITRE ID for the virtualization/sandbox evasion techniques used by the two malicious DLLs?
7. When conducting malware analysis and reverse engineering, understanding anti-analysis techniques is vital to avoid wasting time. Which hypervisor is targeted by the anti-analysis techniques in the ffmpeg.dll file?
8. Identifying the cryptographic method used in malware is crucial for understanding the techniques employed to bypass defense mechanisms and execute its functions fully. What encryption algorithm is used by the ffmpeg.dll file?
9. As an analyst, you've recognized some TTPs involved in the incident, but identifying the APT group responsible will help you search for their usual TTPs and uncover other potential malicious activities. Which group is responsible for this attack?

---

## 🔍 Answer & Walkthrough

> 🔄 *Belum diisi — akan dilengkapi setelah pengerjaan selesai.*

---

## 🚨 Key Findings / IOCs

> 🔄 *Belum diisi.*

| Tipe | Value | Keterangan |
|------|-------|------------|
| IP Address | `...` | ... |
| File Hash | `...` | ... |
| Domain | `...` | ... |

---

## 🗺️ MITRE ATT&CK Mapping

> 🔄 *Belum diisi.*

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| ... | ... | ... | ... |

---

## 📋 Summary — Attacker Behavior & Todo

> 🔄 *Belum diisi — akan dilengkapi setelah analisis selesai.*

### Attacker Behavior

🔄 Belum diisi.

### Todo / Follow-up

- [ ] Lengkapi jawaban 9 pertanyaan di atas via VirusTotal (MSI/DLL analysis, behavior tab, relations)
- [ ] Cross-check DLL hash & threat category ke VirusTotal/Malpedia
- [ ] Petakan MITRE technique buat persistence & sandbox evasion (pertanyaan #4 & #6)
- [ ] Identifikasi APT group yang bertanggung jawab (pertanyaan #9) via threat intel report publik

---

## 📚 References

- [CyberDefenders — 3CX Supply Chain Lab](https://cyberdefenders.org/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
