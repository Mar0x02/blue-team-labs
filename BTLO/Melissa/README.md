# Melissa — BTLO

> **Platform:** [Blue Team Labs Online](https://blueteamlabs.online/home/challenge/melissa-2e0e37b4d1)  
> **Category:** Malware Analysis / Macro Virus  
> **Difficulty:** Easy  
> **Status:** ✅ Completed  
> **Date:** 2026-05-24  
> **Time Spent:** ~1 jam  
> **Tags:** `VBA Macro` `oledump` `Static Analysis` `Email Worm`  
> ⚠️ **Peringatan:** Challenge ini mengandung real malware — kerjakan di dalam virtual machine!

---

## 📌 Prolog

Setelah ILOVEYOU, kini giliran Melissa — macro virus yang lebih tua, muncul setahun sebelumnya di 1999. Kalau ILOVEYOU ditulis sebagai VBScript standalone, Melissa bersembunyi di dalam dokumen Word sebagai macro VBA. Cara analisisnya berbeda: butuh `oledump` untuk membongkar stream OLE file terlebih dahulu sebelum bisa membaca kode macro-nya. Challenge ini relatif straightforward, tapi bagus untuk membangun intuisi analisis macro Office malware.

---

## 🎯 Scenario

Melissa, juga dikenal sebagai W97M.Melissa.A (Symantec) atau Virus:W32/Melissa (F-Secure), adalah macro virus yang berasal dari 26 Maret 1999. Virus ini menyebar melalui dokumen Microsoft Word dan email, dan sempat melumpuhkan server email di seluruh dunia dengan kerugian diestimasi $80 juta.

---

## ❓ Questions

1. Submit the stream number that contains the Melissa macro in the LIST.DOC file
2. After identifying which version of word, Melissa will enable all macros from registry
3. What is the email service targeted by Melissa?
4. How many number of email addresses were collected?
5. What is the string used by Melissa to identify whether a PC is infected or not and decide whether to collect email addresses or not?
6. What is the variable responsible for identifying the email username of the infected PC?
7. What is the text in email body used for spreading Melissa?
8. What is the text that is inserted by Melissa in an open word document?

---

## 🔍 Answer & Walkthrough

### Step 1 — Identifikasi Stream dengan oledump

```bash
python3 oledump.py LIST.DOC
```

Output menampilkan semua stream di dalam file OLE. Stream yang bertanda **M** mengindikasikan adanya macro VBA:

```
1:       106 '\x01CompObj'
2:       576 '\x05DocumentSummaryInformation'
3:       512 '\x05SummaryInformation'
4:      4113 '1Table'
5:       331 'Macros/PROJECT'
6:        26 'Macros/PROJECTwm'
7: M    6544 'Macros/VBA/Melissa'   ← target
8:      3946 'Macros/VBA/_VBA_PROJECT'
...
14:     9253 'WordDocument'
```

### Step 2 — Extract dan Analisis Macro

```bash
# Extract semua macro
olevba LIST.DOC > result.txt

# Atau extract stream spesifik
python3 oledump.py LIST.DOC -s 7 -v
```

Flag `-s 7` untuk select stream 7, flag `-v` untuk decompress VBA code. Buka `result.txt` di text editor dan analisis line by line.

---

### 1. Submit the stream number that contains the Melissa macro in the LIST.DOC file

Stream nomor **7** bernama `Macros/VBA/Melissa` (6544 bytes) — satu-satunya stream yang ditandai huruf **M** yang berarti mengandung macro VBA aktif.

**Jawaban:** `7`

---

### 2. After identifying which version of word, Melissa will enable all macros from registry

Cari di `result.txt` bagian registry key yang diakses Melissa. Di line 11 dan 13, ditemukan referensi ke:

```
HKEY_CURRENT_USER\Software\Microsoft\Office\9.0\Word\Security
```

Versi **9.0** adalah Microsoft Word 2000.

**Jawaban:** `9.0`

---

### 3. What is the email service targeted by Melissa?

Line 19 macro script:

```vba
Set UngaDasOutlook = CreateObject("Outlook.Application")
```

Melissa menggunakan Microsoft Outlook via MAPI untuk mengakses address book dan mengirim email ke semua kontak korban.

**Jawaban:** `Outlook`

---

### 4. How many number of email addresses were collected?

Line 32 macro script mengandung kondisi pembatas loop:

```vba
If x > 50 Then oo = AddyBook.AddressEntries.Count
```

Melissa membatasi pengiriman ke maksimal **50** email per address list — mekanisme untuk menghindari deteksi berdasarkan volume email yang tidak wajar.

**Jawaban:** `50`

---

### 5. What is the string used by Melissa to identify whether a PC is infected or not?

Line 21 macro script:

```vba
If System.PrivateProfileString("", "HKEY_CURRENT_USER\Software\Microsoft\Office\", "Melissa?") <> "... by Kwyjibo" Then
```

Melissa menyimpan string `... by Kwyjibo` di registry sebagai marker infeksi. Jika string ini belum ada, Melissa mulai mengumpulkan kontak dan mengirim email. Mekanisme anti-reinfection yang sama dengan ILOVEYOU, bedanya ILOVEYOU pakai registry WAB.

**Jawaban:** `... by Kwyjibo`

---

### 6. What is the variable responsible for identifying the email username of the infected PC?

Line 34 macro script:

```vba
BreakUmOffASlice.Subject = "Important Message From " & Application.UserName
```

Variable `Application.UserName` mengambil nama user yang sedang login di Microsoft Office, lalu digunakan sebagai bagian dari subject email — teknik social engineering agar penerima percaya email datang dari orang yang dikenal.

**Jawaban:** `Application.Username`

---

### 7. What is the text in email body used for spreading Melissa?

Line 35 macro script:

```vba
BreakUmOffASlice.Body = "Here is that document you asked for ... don't show anyone else ;-)"
```

Body email ini merupakan social engineering klasik — membuat penerima percaya bahwa dokumen yang dilampirkan memang mereka minta, sehingga lebih tergoda untuk membuka attachment.

**Jawaban:** `Here is that document you asked for ... don't show anyone else ;-)`

---

### 8. What is the text that is inserted by Melissa in an open word document?

Line 93 macro script mengandung kondisi trigger:

```vba
If Day(Now) = Minute(Now) Then Selection.TypeText "Twenty-two points, plus triple-word-score, plus fifty points for using all my letters. Game's over. I'm outta here."
```

Teks ini disisipkan ke dokumen Word yang sedang terbuka ketika angka hari sama dengan angka menit saat ini — sebuah easter egg dari pembuat virus yang terinspirasi dari permainan Scrabble.

**Jawaban:** `Twenty-two points, plus triple-word-score, plus fifty points for using all my letters. Game's over. I'm outta here.`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| File | `LIST.DOC` | Dokumen Word pembawa macro virus |
| Stream | `Macros/VBA/Melissa` (stream 7) | Lokasi macro di dalam file OLE |
| Registry Key | `HKCU\Software\Microsoft\Office\9.0\Word\Security` | Diakses untuk disable macro security |
| Registry Key | `HKCU\Software\Microsoft\Office\` key `Melissa?` | Marker anti-reinfection |
| Infection Marker | `... by Kwyjibo` | String penanda komputer sudah terinfeksi |
| Email Limit | `50` | Batas maksimal email per address list |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Execution | Command and Scripting Interpreter: Visual Basic | T1059.005 | Macro VBA dieksekusi saat dokumen dibuka |
| Persistence | Office Application Startup: Office Template Macros | T1137.001 | Macro menyebar ke dokumen Word lain |
| Defense Evasion | Modify Registry | T1112 | Ubah registry Word Security untuk enable semua macro |
| Lateral Movement | Phishing: Spearphishing Attachment | T1566.001 | Kirim dokumen terinfeksi ke 50 kontak Outlook |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

Begitu dokumen `LIST.DOC` dibuka dan macro diizinkan berjalan, Melissa langsung mengecek registry untuk marker infeksi `... by Kwyjibo`. Jika belum ada, ia memodifikasi registry Word Security agar semua macro otomatis diizinkan, lalu mengakses Microsoft Outlook via MAPI untuk mengirim dirinya ke maksimal 50 kontak pertama di address book — menggunakan nama user yang sedang login sebagai bagian dari subject email agar terlihat legitimate. Selain menyebar via email, Melissa juga menginfeksi template `Normal.dot` agar setiap dokumen Word baru yang dibuat di komputer tersebut juga terinfeksi. Easter egg berupa teks Scrabble disisipkan ke dokumen yang sedang terbuka ketika kondisi waktu tertentu terpenuhi.

### Todo / Follow-up

- [ ] Pelajari struktur file OLE lebih dalam — bagaimana stream dan storage diorganisasikan
- [ ] Eksplorasi `oledump` lebih lanjut: plugin detection, deobfuscation VBA
- [ ] Bandingkan teknik spreading Melissa vs ILOVEYOU — persamaan dan perbedaan mekanisme anti-reinfection-nya
- [ ] Buat YARA rule untuk mendeteksi string `... by Kwyjibo` atau macro stream Melissa

---

## 📚 References

- [MITRE ATT&CK — T1059.005 Visual Basic](https://attack.mitre.org/techniques/T1059/005/)
- [MITRE ATT&CK — T1137.001 Office Template Macros](https://attack.mitre.org/techniques/T1137/001/)
- [oledump — Didier Stevens](https://blog.didierstevens.com/programs/oledump-py/)
- [Wikipedia — Melissa (computer virus)](https://en.wikipedia.org/wiki/Melissa_(computer_virus))
- [BTLO — Melissa Challenge](https://blueteamlabs.online/home/challenge/melissa-2e0e37b4d1)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
