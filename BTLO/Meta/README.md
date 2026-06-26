# Meta — BTLO

> **Platform:** [Blue Team Labs Online](https://blueteamlabs.online/home/challenge/meta-b976cec9e2)  
> **Category:** OSINT / Digital Forensics  
> **Difficulty:** Easy  
> **Status:** ✅ Completed  
> **Date:** 2026-05-07  
> **Time Spent:** ~1 jam  
> **Tags:** `ExifTool` `Reverse Image Search`

---

## 🎯 Scenario

Dua gambar diunggah oleh seorang kriminal yang sedang dalam pelarian, dengan caption **"I'm roaming free. You will never catch me"**. Tugas kita adalah membuktikan bahwa ia salah dengan menganalisis metadata dan konten visual dari gambar tersebut untuk menentukan lokasi keberadaannya.

---

## ❓ Questions

1. What is the camera model?
2. When was the picture taken?
3. What does the comment on the first image say?
4. Where could the criminal be?

---

## 🔍 Answer & Walkthrough

### 1. What is the camera model?

Jalankan ExifTool pada gambar pertama dan simpan outputnya ke file untuk memudahkan analisis:

```bash
exiftool uploaded_1.JPG > res.txt
cat res.txt
```

Di antara berbagai field metadata, ditemukan:

```
Camera Model Name : Canon EOS 550D
```

**Jawaban:** `Canon EOS 550D`

---

### 2. When was the picture taken?

Masih dari output ExifTool yang sama, field timestamp original:

```
Date/Time Original : 2021:11:02 13:20:23
```

**Jawaban:** `2021:11:02 13:20:23`

---

### 3. What does the comment on the first image say?

Dalam output ExifTool juga terdapat field `Comment`:

```
Comment : relying on altered metadata to catch me?
```

Menariknya, field GPS Position yang ada di metadata ternyata tidak valid — longitude 279° W melebihi batas maksimum 180°, yang mengonfirmasi bahwa metadata GPS sengaja dipalsukan.

**Jawaban:** `relying on altered metadata to catch me?`

---

### 4. Where could the criminal be?

Karena metadata GPS sudah dimanipulasi, satu-satunya cara menentukan lokasi adalah lewat **konten visual** gambar kedua. Gunakan **Google Lens** pada `uploaded_2.png`.

Hasil pencarian menunjukkan arsitektur khas **Newari/Nepal** — bata merah, atap pagoda bertingkat, ornamen tradisional — yang mengidentifikasi lokasi sebagai **Kathmandu Durbar Square, Nepal**.

**Jawaban:** `Kathmandu`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| Camera | `Canon EOS 550D` | Kamera yang digunakan pelaku |
| Timestamp | `2021:11:02 13:20:23` | Waktu pengambilan foto |
| GPS | `32°40'3.87"S, 279°29'31.87"W` | Koordinat palsu — longitude melebihi 180° |
| Lokasi | `Kathmandu, Nepal` | Diidentifikasi via reverse image search |

---

## 🗺️ MITRE ATT&CK Mapping

> 🔄 *Belum diisi.*

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| ... | ... | ... | ... |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

Pelaku mengunggah dua gambar dengan koordinat GPS yang sengaja dipalsukan (longitude 279° W — angka yang tidak mungkin valid secara geografis) sebagai upaya menyesatkan investigasi. Ia bahkan meninggalkan komentar di metadata yang secara eksplisit mengakui manipulasi tersebut. Namun konten visual gambar kedua justru mengungkap lokasi sebenarnya — arsitektur khas Kathmandu Durbar Square yang dapat diidentifikasi melalui reverse image search.

### Todo / Follow-up

- [ ] Eksplorasi lebih lanjut field-field ExifTool yang bisa dimanipulasi (GPS, timestamps, author)
- [ ] Pelajari teknik OSINT geolocation dari konten visual (landmark matching)
- [ ] Coba tools alternatif seperti `exiv2` atau `mat2` untuk metadata stripping

---

## 📚 References

- [ExifTool Documentation](https://exiftool.org/)
- [Google Lens](https://lens.google.com/)
- [BTLO — Meta Challenge](https://blueteamlabs.online/home/challenge/meta-b976cec9e2)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
