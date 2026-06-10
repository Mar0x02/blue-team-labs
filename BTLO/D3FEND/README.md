# D3FEND — BTLO

> **Platform:** Blue Team Labs Online  
> **Category:** Framework  
> **Difficulty:** Easy  
> **Status:** ✅ Completed  
> **Date:** 2026-05-09  
> **Time Spent:** ~30 menit  

---

## 📌 Prolog

Challenge ini memperkenalkan MITRE D3FEND — framework defensive yang menjadi "pasangan" dari ATT&CK. Kalau ATT&CK mengkatalogkan teknik serangan, D3FEND mengkatalogkan teknik pertahanan dan memetakan hubungan keduanya. Lab ini lebih ke knowledge exploration: buka framework-nya, pahami strukturnya, dan jawab pertanyaan seputar teknik-teknik defensif yang ada di dalamnya.

---

## 🎯 Scenario

D3FEND adalah katalog teknik-teknik defensive cybersecurity beserta hubungannya dengan teknik offensive/adversary yang telah dirilis. Mari kita lihat apa yang ada di dalamnya.

> **Catatan:** Jika ada beberapa jawaban, urutkan secara alfabetis.

---

## ❓ Questions

1. What is the corresponding name for the ID `D3-SDM`?
2. What are the five general tactics used to classify each defensive method? (In the order they appear)
3. What open-source project retrieves Azure Sentinel rules that are mapped to MITRE ATT&CK Framework and generates the related MITRE D3FEND defenses?
4. What does 'File Access Pattern Analysis' mean?
5. What does 'Local Resource Access' artifact mean?

---

## 🔍 Answer & Walkthrough

### 1. What is the corresponding name for the ID `D3-SDM`?

Search ID `D3-SDM` langsung di D3FEND Lookup — [d3fend.mitre.org](https://d3fend.mitre.org). Hasilnya menampilkan nama teknik beserta definisinya. D3-SDM adalah teknik defensive yang melacak perubahan pada state atau konfigurasi critical system-level processes, masuk ke tactic **Detect**.

**Jawaban:** `System Daemon Monitoring`

---

### 2. What are the five general tactics used to classify each defensive method?

Lihat urutan tactic di tree/matrix utama halaman D3FEND. Kelima tactic ini menjadi backbone struktur framework:

- **Harden** — Memperkuat sistem agar lebih sulit diserang (hardening konfigurasi, credential management)
- **Detect** — Mendeteksi aktivitas mencurigakan atau berbahaya (monitoring, analysis)
- **Isolate** — Mengisolasi aset atau sistem dari ancaman (network isolation, sandboxing)
- **Deceive** — Menipu attacker agar mengira berhasil menyerang (honeypot, decoy files)
- **Evict** — Mengusir attacker yang sudah masuk ke dalam sistem (credential eviction, process termination)

**Jawaban:** `Harden, Detect, Isolate, Deceive, Evict`

---

### 3. What open-source project retrieves Azure Sentinel rules mapped to ATT&CK and generates D3FEND defenses?

Google dorking `github Azure Sentinel ATT&CK D3FEND rules` mengarah ke repo [Intellisec-Solutions/Sentinel2D3FEND](https://github.com/Intellisec-Solutions/Sentinel2D3FEND). Project ini secara otomatis mengambil Azure Sentinel detection rules yang sudah di-mapping ke ATT&CK, lalu men-generate teknik-teknik D3FEND yang relevan sebagai rekomendasi pertahanan — berguna untuk SOC team yang ingin mengintegrasikan D3FEND ke workflow Azure Sentinel.

**Jawaban:** `Sentinel2D3FEND`

---

### 4. What does 'File Access Pattern Analysis' mean?

Search `File Access Pattern Analysis` di D3FEND Lookup, cek bagian **Definition**. Teknik ini masuk ke tactic **Detect** — dengan memantau file mana saja yang diakses oleh suatu process, defender bisa mendeteksi perilaku mencurigakan seperti akses ke file sistem yang tidak seharusnya disentuh.

**Jawaban:** `Analyzing the files accessed by a process to identify unauthorized activity.`

---

### 5. What does 'Local Resource Access' artifact mean?

Di popup **File Access Pattern Analysis**, klik bagian artifact → muncul definisi Local Resource Access. Disebut "ephemeral" karena artifact ini bersifat sementara — hanya ada selama proses request-response berlangsung. Dalam konteks D3FEND, artifact ini adalah objek yang dianalisis oleh teknik File Access Pattern Analysis.

**Jawaban:** `Ephemeral digital artifact comprising a request of a local resource and any response from that resource.`

---

## 🚨 Key Findings / IOCs

> 🔄 *Tidak relevan — challenge ini berbasis framework exploration, bukan investigasi insiden.*

---

## 🗺️ MITRE ATT&CK Mapping

> 🔄 *Tidak relevan untuk challenge ini.*

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

> 🔄 *Tidak relevan — challenge ini berfokus pada pemahaman D3FEND Framework, bukan analisis serangan.*

### Todo / Follow-up

- [ ] Eksplorasi lebih dalam relasi antara teknik D3FEND dengan teknik ATT&CK — coba mapping beberapa TTP umum ke teknik defensifnya
- [ ] Coba gunakan Sentinel2D3FEND untuk generate rekomendasi defensif dari detection rules yang sudah ada
- [ ] Pelajari tactic **Evict** lebih dalam — jarang dibahas tapi penting untuk incident response

---

## 📚 References

- [MITRE D3FEND](https://d3fend.mitre.org/)
- [BTLO — D3FEND Challenge](https://blueteamlabs.online/home/challenge/d3fend-6c9dcd4b79)
- [Sentinel2D3FEND — GitHub](https://github.com/Intellisec-Solutions/Sentinel2D3FEND)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
