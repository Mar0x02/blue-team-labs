# FakeGPT Lab — CyberDefenders

> **Platform:** CyberDefenders  
> **Category:** Malware Analysis  
> **Difficulty:** Easy  
> **Status:** 🔄 In Progress  
> **Challenge Status:** Retired  
> **Date:** 2026-05-31  
> **Time Spent:** 🔄 Belum diisi  

---

## 📌 Prolog

Analyze a malicious Chrome extension's code and behavior to identify data theft mechanisms, covert exfiltration via `<img>` tags, and anti-analysis techniques.

**Tactics:** Credential Access · Collection · Command and Control · Exfiltration  
**Tools:** ExtAnalysis · CRX Viewer

---

## 🎯 Scenario

Tim cybersecurity mendapat alert aktivitas mencurigakan di jaringan. Beberapa karyawan melaporkan perilaku aneh di browser mereka setelah menginstal ekstensi bernama "ChatGPT". Akun-akun mulai dikompromis dan informasi sensitif tampak bocor.

Tugas: lakukan analisis menyeluruh terhadap ekstensi ini dan identifikasi komponen-komponen berbahayanya.

---

## ❓ Questions

1. Which encoding method does the browser extension use to obscure target URLs, making them more difficult to detect during analysis?
2. Which website does the extension monitor for data theft, targeting user accounts to steal sensitive information?
3. Which type of HTML element is utilized by the extension to send stolen data?
4. What is the first specific condition in the code that triggers the extension to deactivate itself?
5. Which event does the extension capture to track user input submitted through forms?
6. Which API or method does the extension use to capture and monitor user keystrokes?
7. What is the domain where the extension transmits the exfiltrated data?
8. Which function in the code is used to exfiltrate user credentials, including the username and password?
9. Which encryption algorithm is applied to secure the data before sending?
10. What does the extension access to store or manipulate session-related data and authentication information?

---

## 🔍 Answer & Walkthrough

> 🔄 *Belum diisi — akan dilengkapi setelah pengerjaan selesai.*

---

## 🚨 Key Findings / IOCs

> 🔄 *Belum diisi.*

| Tipe | Value | Keterangan |
|------|-------|------------|
| | | |

---

## 🗺️ MITRE ATT&CK Mapping

> 🔄 *Belum diisi.*

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Credential Access | | | |
| Collection | | | |
| Command and Control | | | |
| Exfiltration | | | |

---

## 📋 Summary — Attacker Behavior & Todo

> 🔄 *Belum diisi — akan dilengkapi setelah analisis selesai.*

---

## 📚 References

- [CyberDefenders — FakeGPT Lab](https://cyberdefenders.org/blueteam-ctf-challenges/fakegpt/)
- [MITRE ATT&CK — Credential Access](https://attack.mitre.org/tactics/TA0006/)
- [MITRE ATT&CK — Exfiltration](https://attack.mitre.org/tactics/TA0010/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
