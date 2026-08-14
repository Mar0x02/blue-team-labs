# BlackEnergy Lab — CyberDefenders

> **Platform:** CyberDefenders
> **Category:** Endpoint Forensics
> **Difficulty:** Medium
> **Status:** 🔄 In Progress
> **Date:** 2026-08-15
> **Time Spent:** ~1 jam

---

## 📌 Prolog

Ngembangin practical skills soal Windows memory forensics pakai Volatility, dengan cara detect malware indicators, analisis suspicious process, dan identifikasi code injection serta unauthorized DLL di compromised system.

**Tools:** Volatility

**Tactics yang tercakup:** Privilege Escalation | Stealth

**Catatan:** lab ini berstatus *Retired* di platform CyberDefenders.

---

## 🎯 Scenario

Sebuah perusahaan multinasional kena cyber attack yang berujung ke pencurian sensitive data. Serangan ini pakai varian **BlackEnergy v2** malware yang belum pernah keliatan sebelumnya (previously unseen variant). Tim security perusahaan udah dapetin memory dump dari mesin yang terinfeksi, dan butuh keahlian lo sebagai SOC analyst buat analisis dump tersebut biar ngerti scope dan impact dari serangan ini.

---

## ❓ Questions

1. Which volatility profile would be best for this machine?
2. How many processes were running when the image was acquired?
3. What is the process ID of cmd.exe?
4. What is the name of the most suspicious process?
5. Which process shows the highest likelihood of code injection?
6. There is an odd file referenced in the recent process. Provide the full path of that file.
7. What is the name of the injected DLL file loaded from the recent process?
8. What is the base address of the injected DLL?

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

- [ ] Tentuin Volatility profile yang cocok buat image ini (pertanyaan #1)
- [ ] Enumerasi process list — jumlah process & PID `cmd.exe` (pertanyaan #2 & #3)
- [ ] Identifikasi process paling mencurigakan (pertanyaan #4)
- [ ] Cek indikasi code injection di process — `malfind` atau teknik serupa (pertanyaan #5)
- [ ] Trace file path aneh yang direferensiin process tersebut (pertanyaan #6)
- [ ] Identifikasi nama & base address DLL yang di-inject (pertanyaan #7 & #8)

---

## 📚 References

- [CyberDefenders — BlackEnergy Lab](https://cyberdefenders.org/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
