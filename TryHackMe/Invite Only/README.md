# Invite Only — TryHackMe

> **Platform:** TryHackMe  
> **Category:** Threat Intelligence  
> **Difficulty:** Medium  
> **Status:** 🔄 In Progress  
> **Date:** 2026-06-12  
> **Time Spent:** ~X jam  

---

## 🎯 Scenario

Kamu adalah SOC analyst di tim SOC Managed Server Provider bernama TrySecureMe. Hari ini, kamu mendukung L3 analyst dalam investigasi flagged IPs, hash, URL, dan domain sebagai bagian dari aktivitas IR. Seorang L1 analyst mengeskalasi dua temuan mencurigakan di pagi hari — tugasmu adalah menganalisis lebih lanjut dan mengolahnya menjadi threat intelligence yang bisa digunakan.

**Flagged Indicators:**
- IP: `101[.]99[.]76[.]120`
- SHA256: `5d0509f68a9b7c415a726be75a078180e3f02e59866f193b0a99eee8e39c874f`

Tools yang digunakan: **TryDetectThis2.0** — aplikasi threat intelligence search yang baru dibeli tim.

---

## ❓ Questions

1. What is the name of the file identified with the flagged SHA256 hash?
2. What is the file type associated with the flagged SHA256 hash?
3. What are the execution parents of the flagged hash? List the names chronologically, using a comma as a separator. Note down the hashes for later use.
4. What is the name of the file being dropped? Note down the hash value for later use.
5. Research the second hash in question 3 and list the four malicious dropped files in the order they appear (from up to down), separated by commas.
6. Analyse the files related to the flagged IP. What is the malware family that links these files?
7. What is the title of the original report where these flagged indicators are mentioned? Use Google to find the report.
8. Which tool did the attackers use to steal cookies from the Google Chrome browser?
9. Which phishing technique did the attackers use? Use the report to answer the question.
10. What is the name of the platform that was used to redirect a user to malicious servers?

---

## 🔍 Answer & Walkthrough

> 🔄 *Belum diisi — akan dilengkapi setelah pengerjaan selesai.*

---

## 🚨 Key Findings / IOCs

> 🔄 *Belum diisi.*

| Tipe | Value | Keterangan |
|------|-------|------------|
| IP Address | `101[.]99[.]76[.]120` | Flagged IP |
| File Hash | `5d0509f68a9b7c415a726be75a078180e3f02e59866f193b0a99eee8e39c874f` | Flagged SHA256 |

---

## 🗺️ MITRE ATT&CK Mapping

> 🔄 *Belum diisi.*

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| ... | ... | ... | ... |

---

## 📋 Summary — Attacker Behavior & Todo

> 🔄 *Belum diisi — akan dilengkapi setelah analisis selesai.*

---

## 📚 References

- [MITRE ATT&CK](https://attack.mitre.org/)
- [TryHackMe — Invite Only](https://tryhackme.com/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
