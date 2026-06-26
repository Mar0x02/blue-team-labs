# Spectrum — BTLO

> **Platform:** [Blue Team Labs Online](https://blueteamlabs.online/home/challenge/spectrum-d6ff2a32b9)  
> **Category:** Digital Forensics / Steganography  
> **Difficulty:** Medium  
> **Status:** ✅ Completed  
> **Date:** 2026-05-22  
> **Time Spent:** ~2 jam  
> **Tags:** `Steganography` `File Carving` `Audio Analysis` `Spectrogram`

---

## 📌 Prolog

Challenge ini memadukan beberapa teknik forensik sekaligus — file carving dari disk image, metadata analysis, audio steganography, cracking ZIP password, dan decoding Base58. Masing-masing langkah menghasilkan clue untuk langkah berikutnya. Yang bikin menarik adalah penggunaan spectrogram sebagai media steganografi: data disembunyikan bukan di dalam file, tapi sebagai pola visual yang hanya terlihat saat audio divisualisasikan dalam domain frekuensi.

---

## 🎯 Scenario

Scotland Yard telah mencegat informasi tentang salah satu transaksi narkoba terbesar yang akan terjadi di kota London. Seseorang yang diduga terlibat dalam transaksi tersebut telah ditangkap. Satu-satunya barang yang ada di dalam kepemilikannya hanyalah sebuah USB flash drive.

Sayangnya, salah satu analis junior tidak berhasil menemukan sesuatu yang mencurigakan. Sebelum tersangka dilepas, tim meminta seorang ahli Digital Forensics untuk memeriksa apakah ada informasi tentang transaksi tersebut sebelum terjadi.

---

## ❓ Questions

1. What time is the meeting happening?
2. What are the supposed coordinates for the deal?
3. Looking into these coordinates, what is the name of this location?

---

## 🔍 Answer & Walkthrough

### Overview: Flow Pengerjaan

```
image.dd (disk image USB flash drive)
    ↓ foremost -i image.dd -o output/
Recover file → f0048900.jpg, f0048140.jpg, f0000240_brown.zip
    ↓ exiftool f0048900.jpg
Metadata Artist: "steghide password: cheese on toast"
    ↓ john-jumbo (zip2john + john)
Password ZIP: garfield
    ↓ unzip f0000240_brown.zip
4 file WAV: brown.wav, location.wav, wahwah.wav, white.wav
    ↓ Audacity → Spectrogram view
white.wav → koordinat tersembunyi secara visual di spectrogram
location.wav → "NICE TRY, NOTHING TO HEAR HERE!" (jebakan)
    ↓ steghide extract white.wav -p "cheese on toast"
stardate.txt → isi: 56inrkS7AcAXatqrFM
    ↓ CyberChef → From Base58
15:01:00 + emit (= "time" dibalik → clue untuk reverse waktu)
    ↓ Reverse: 00:10:51
```

---

### Step 1: File Carving dengan foremost

Photorec hanya menemukan 3 file dari disk image, sementara foremost berhasil recover file tambahan `f0048900.jpg` yang jadi kunci seluruh chain ini. Perbedaannya: foremost melakukan scanning berdasarkan file signature saja tanpa bergantung pada struktur filesystem, sehingga lebih agresif dalam menemukan file di area yang mungkin dilewati Photorec.

```bash
foremost -i image.dd -o output/
```

> Best practice forensik: selalu gunakan lebih dari satu tool file carving agar tidak ada file yang terlewat.

---

### Step 2: Extract Password dari Metadata

Dari file `f0048900.jpg` yang ditemukan foremost:

```bash
exiftool f0048900.jpg
```

Di field `Artist` tersembunyi password steghide:

```
Artist: steghide password: cheese on toast
```

---

### Step 3: Crack Password ZIP

File `f0000240_brown.zip` terenkripsi. Crack dengan john-jumbo:

```bash
zip2john f0000240_brown.zip > hash.txt
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

Password ZIP: `garfield`

Setelah di-unzip, ditemukan 4 file WAV: `brown.wav`, `location.wav`, `wahwah.wav`, `white.wav`.

---

### 1. What time is the meeting happening?

Buka semua file WAV di Audacity dan cek spectrogram masing-masing. `location.wav` adalah jebakan — hanya berisi teks "NICE TRY, NOTHING TO HEAR HERE!" di spectrogram-nya.

Untuk `white.wav`, ekstrak data tersembunyi dengan steghide menggunakan password yang sudah didapat:

```bash
steghide extract -sf white.wav -p "cheese on toast"
```

Output: `stardate.txt` dengan isi `56inrkS7AcAXatqrFM`.

Decode di CyberChef dengan **From Base58** → menghasilkan `15:01:00 emit`.

Kata `emit` adalah `time` yang dibalik — petunjuk bahwa waktu yang didapat juga perlu di-reverse. Reverse `15:01:00` → `00:10:51`.

**Jawaban:** `00:10:51`

---

### 2. What are the supposed coordinates for the deal?

Buka `white.wav` di Audacity, lalu ganti view ke **Spectrogram** (klik dropdown pada track → pilih Spectrogram). Koordinat GPS tersembunyi secara visual sebagai pola frekuensi yang membentuk teks.

Teknik ini disebut **audio steganography via spectrogram** — data disembunyikan bukan di level bit file, melainkan sebagai gambar yang hanya terlihat di domain frekuensi.

**Jawaban:** `51.505278, 0.055278`

---

### 3. Looking into these coordinates, what is the name of this location?

Input koordinat `51.505278, 0.055278` ke Google Maps atau [gps-coordinates.net](https://www.gps-coordinates.net/).

**Jawaban:** `London City Airport`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| File | `f0048900.jpg` | File tersembunyi di disk image, hanya ditemukan foremost |
| Credential | `cheese on toast` | Password steghide, tersembunyi di metadata EXIF |
| Credential | `garfield` | Password ZIP |
| Encoded | `56inrkS7AcAXatqrFM` | Waktu meeting dalam Base58 + reversed |
| Koordinat | `51.505278, 0.055278` | Lokasi transaksi, tersembunyi di spectrogram |
| Lokasi | `London City Airport` | Tujuan transaksi narkoba |
| Waktu | `00:10:51` | Waktu meeting setelah di-decode dan di-reverse |

---

## 🗺️ MITRE ATT&CK Mapping

> 🔄 *Belum diisi.*

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| ... | ... | ... | ... |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

Pelaku menyembunyikan informasi transaksi dalam disk image USB dengan menggunakan berlapis-lapis teknik obfuskasi. File kunci (`f0048900.jpg`) disembunyikan di area disk yang tidak terdeteksi Photorec, berisi password steghide di metadata EXIF. Password tersebut digunakan untuk mengekstrak file dari audio WAV (`white.wav`) yang sekaligus menyimpan koordinat GPS secara visual di spectrogram-nya. Waktu meeting di-encode dalam Base58 kemudian di-reverse agar tidak langsung terbaca. `location.wav` sengaja dibuat sebagai decoy untuk menyesatkan investigator. Keseluruhan chain ini dirancang agar setiap lapisan butuh tool yang berbeda, sehingga investigator yang hanya mengandalkan satu pendekatan akan gagal menemukan informasi lengkap.

### Todo / Follow-up

- [ ] Pelajari lebih dalam teknik steganografi di domain frekuensi (spectrogram steganography)
- [ ] Eksplorasi tools audio steganography selain steghide (OpenStego, SilentEye)
- [ ] Latihan file carving lebih lanjut — kenali perbedaan hasil foremost vs Photorec vs binwalk
- [ ] Pelajari encoding scheme lain yang sering dipakai: Base32, Base85, Base58

---

## 📚 References

- [Audacity — Spectrogram View](https://support.audacityteam.org/audio-analysis/spectrogram-view)
- [steghide Documentation](http://steghide.sourceforge.net/documentation.php)
- [foremost — File Carving Tool](https://foremost.sourceforge.net/)
- [CyberChef — From Base58](https://gchq.github.io/CyberChef/)
- [BTLO — Spectrum Challenge](https://blueteamlabs.online/home/challenge/spectrum-d6ff2a32b9)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
