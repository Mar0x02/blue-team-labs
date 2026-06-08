# Learn Sigma — LetsDefend

> **Platform:** LetsDefend  
> **Category:** Threat Detection / Sigma Rules  
> **Difficulty:** Easy  
> **Status:** 🔄 In Progress  
> **Date:** 2026-06-08  
> **Time Spent:** ~X jam  

---

## 🎯 Scenario

Organisasi mendeteksi infeksi ransomware pada salah satu sistem kritisnya. Ransomware ini mencari file berharga — dokumen sensitif, configuration file — lalu mengenkripsinya dengan algoritma enkripsi kuat. Investigasi mengungkapkan bahwa ransomware kemungkinan menggunakan Windows utility `bitsadmin.exe` untuk mengunduh payload tambahan atau berkomunikasi dengan C2 server.

Tugas kita: review Sigma rule yang sudah ada, jawab pertanyaan terkait, dan pahami bagaimana berbagai section rule (selection, condition, fields, tags, logsource) bekerja bersama untuk mendeteksi aktivitas berbahaya ini.

**File Location:** `C:\Users\LetsDefend\Desktop\ChallengeFile\proc_creation_win_bitsadmin_download.yml`

---

## ❓ Questions

1. Which executable file was specifically targeted by this Sigma rule?
2. What command-line option is used to indicate a file transfer in this rule?
3. What logical expression in the condition field combined the criteria to trigger this rule?
4. Which specific field did this rule capture that shows the command being executed?
5. Which single ATT&CK tactic tag is listed first in this rule?
6. What is the primary category of events that this Sigma rule was written to monitor?
7. What specific command-line argument did this rule look for to identify HTTP-based downloads?
8. Which command-line option must be present to create a new transfer using bitsadmin?

---

## 🔍 Answer & Walkthrough

> 🔄 *Belum diisi — akan dilengkapi setelah pengerjaan selesai.*

### 1. Which executable file was specifically targeted by this Sigma rule?

**Jawaban:** `...`

---

### 2. What command-line option is used to indicate a file transfer in this rule?

**Jawaban:** `...`

---

### 3. What logical expression in the condition field combined the criteria to trigger this rule?

**Jawaban:** `...`

---

### 4. Which specific field did this rule capture that shows the command being executed?

**Jawaban:** `...`

---

### 5. Which single ATT&CK tactic tag is listed first in this rule?

**Jawaban:** `...`

---

### 6. What is the primary category of events that this Sigma rule was written to monitor?

**Jawaban:** `...`

---

### 7. What specific command-line argument did this rule look for to identify HTTP-based downloads?

**Jawaban:** `...`

---

### 8. Which command-line option must be present to create a new transfer using bitsadmin?

**Jawaban:** `...`

---

## 🚨 Key Findings / IOCs

> 🔄 *Belum diisi.*

| Tipe | Value | Keterangan |
|------|-------|------------|
| Executable | `...` | Target binary di Sigma rule |
| Command-line | `...` | Argumen yang dideteksi |
| Domain/URL | `...` | Indikator C2 jika ada |

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

- [Sigma Rules Documentation](https://sigmahq.io/)
- [MITRE ATT&CK - T1197 BITS Jobs](https://attack.mitre.org/techniques/T1197/)
- [bitsadmin.exe abuse — LOLBAS](https://lolbas-project.github.io/lolbas/Binaries/Bitsadmin/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
