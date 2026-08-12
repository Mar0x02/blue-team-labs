# Red Stealer Lab — CyberDefenders

> **Platform:** CyberDefenders
> **Category:** Threat Intelligence
> **Difficulty:** Easy
> **Status:** 🔄 In Progress
> **Date:** 2026-08-12
> **Time Spent:** ~30 menit

---

## 📌 Prolog

Investigasi executable mencurigakan menggunakan VirusTotal dan MalwareBazaar untuk ekstrak IOC, identifikasi infrastruktur C2, mapping MITRE ATT&CK technique, dan mekanisme privilege escalation yang dipakai malware.

**Tools:** Whois | VirusTotal | MalwareBazaar | ThreatFox | ANY.RUN

**Tactics yang tercakup:** Execution | Persistence | Privilege Escalation | Stealth | Defense Impairment | Discovery | Collection | Impact

---

## 🎯 Scenario

Lo bagian dari tim Threat Intelligence di SOC (Security Operations Center). Sebuah file executable ditemukan di komputer rekan kerja, dan dicurigai berhubungan dengan Command and Control (C2) server — indikasi kemungkinan infeksi malware.

Tugas lo adalah investigasi executable ini dengan menganalisis hash-nya. Tujuannya adalah mengumpulkan dan menganalisis data yang berguna buat anggota SOC lain, termasuk tim Incident Response, supaya bisa merespons perilaku mencurigakan ini secara efisien.

---

## ❓ Questions

1. Categorizing malware enables a quicker and clearer understanding of its unique behaviors and attack vectors. What category has Microsoft identified for that malware in VirusTotal?
2. Clearly identifying the name of the malware file improves communication among the SOC team. What is the file name associated with this malware? (Don't include the file extension in the name.)
3. Knowing the exact timestamp of when the malware was first observed can help prioritize response actions. Newly detected malware may require urgent containment and eradication compared to older, well-documented threats. What is the UTC timestamp of the malware's first submission to VirusTotal?
4. Understanding the techniques used by malware helps in strategic security planning. What is the MITRE ATT&CK technique ID for the malware's data collection from the system before exfiltration?
5. Following execution, which social media-related domain names did the malware resolve via DNS queries?
6. Once the malicious IP addresses are identified, network security devices such as firewalls can be configured to block traffic to and from these addresses. Can you provide the IP address and destination port the malware communicates with?
7. YARA rules are designed to identify specific malware patterns and behaviors. Using MalwareBazaar, what's the name of the YARA rule created by "Varp0s" that detects the identified malware?
8. Understanding which malware families are targeting the organization helps in strategic security planning for the future and prioritizing resources based on the threat. Can you provide the different malware alias associated with the malicious IP address according to ThreatFox?
9. By identifying the malware's imported DLLs, we can configure security tools to monitor for the loading or unusual usage of these specific DLLs. Can you provide the DLL utilized by the malware for privilege escalation?

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

- [ ] Lengkapi jawaban 9 pertanyaan di atas via VirusTotal, MalwareBazaar, ThreatFox, dan ANY.RUN
- [ ] Cross-check IOC (IP, domain, hash) ke ThreatFox buat cek malware family alias
- [ ] Analisis sandbox report di ANY.RUN buat konfirmasi behavior privilege escalation & data collection

---

## 📚 References

- [CyberDefenders — Red Stealer Lab](https://cyberdefenders.org/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
