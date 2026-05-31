# FakeGPT Lab — CyberDefenders

> **Platform:** CyberDefenders  
> **Category:** Malware Analysis  
> **Difficulty:** Easy  
> **Status:** ✅ Completed  
> **Challenge Status:** Retired  
> **Date:** 2026-05-31  
> **Time Spent:** ~30 menit  

---

## 📌 Prolog

Analisis code dan behavior dari Chrome extension berbahaya untuk mengidentifikasi mekanisme data theft, exfiltration tersembunyi via tag `<img>`, dan teknik anti-analysis.

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

### Preparation

Setelah extract file ZIP dari soal, ditemukan 6 file: beberapa file JavaScript, satu GIF, satu JSON, dan satu HTML — strukturnya menyerupai web app sederhana yang di-bundle sebagai browser extension.

Langkah pertama adalah membaca `manifest.json` untuk memahami konfigurasi extension: metadata seperti nama, versi, dan deskripsi. Yang langsung menarik perhatian adalah bagian `permissions` — extension ini meminta akses ke beberapa area yang cukup sensitif, salah satunya `cookies`. Setelah memahami struktur extension secara keseluruhan, analisis dilanjutkan ke file `app.js` dan `loader.js` sebagai file utama yang menjadi fokus pengerjaan soal.

![](./assets/manifest.png)

---

### 1. Which encoding method does the browser extension use to obscure target URLs?

Di `app.js` line 9, terdapat string yang di-encode:

```javascript
d3d3LmZhY2Vib29rLmNvbQ==
```

String ini berfungsi sebagai kondisi penentu apakah payload exfiltration akan aktif — jika URL yang sedang dikunjungi user sesuai dengan hasil decode string ini, maka eksekusi berlanjut ke tahap selanjutnya. Encoding yang digunakan adalah **Base64**.

![](./assets/soal-1/app_js_line9.png)

**Jawaban:** `Base64`

---

### 2. Which website does the extension monitor for data theft?

Hasil decode dari string Base64 di atas:

```
d3d3LmZhY2Vib29rLmNvbQ== → www.facebook.com
```

Extension secara spesifik menarget Facebook sebagai trigger lokasi exfiltration. Ketika user mengunjungi URL ini, seluruh mekanisme pencurian data aktif.

![](./assets/soal-2/decoded_url.png)

**Jawaban:** `www.facebook.com`

---

### 3. Which type of HTML element is utilized by the extension to send stolen data?

Teknik yang digunakan adalah **Pixel/Image-based Exfiltration** — data hasil exfiltration tidak dikirim via XHR atau fetch request biasa, melainkan disisipkan sebagai parameter di dalam URL `src` dari elemen `<img>`. Teknik ini lebih sulit dideteksi oleh network monitoring tool karena terlihat seperti request gambar biasa.

![](./assets/soal-3/img_exfil.png)

**Jawaban:** `img`

---

### 4. What is the first specific condition in the code that triggers the extension to deactivate itself?

Extension mengecek kondisi berikut untuk mendeteksi apakah browser sedang berjalan di lingkungan sandbox atau virtual environment:

```javascript
navigator.plugins.length === 0
```

Browser dalam mode sandbox biasanya tidak memuat plugin apapun, sehingga nilai `navigator.plugins.length` akan bernilai `0`. Jika kondisi ini terpenuhi, extension menghentikan eksekusinya dan berperilaku normal — teknik klasik sandbox evasion.

![](./assets/soal-4/sandbox_check.png)

**Jawaban:** `navigator.plugins.length === 0`

---

### 5. Which event does the extension capture to track user input submitted through forms?

Credential harvesting dipicu oleh event `submit` — ketika user menekan tombol login/submit di halaman Facebook, extension menangkap input dari form tersebut sebelum data dikirim ke server.

![](./assets/soal-5/submit_event.png)

**Jawaban:** `submit`

---

### 6. Which API or method does the extension use to capture and monitor user keystrokes?

Selain credential harvesting via form submit, extension juga menjalankan keylogger yang merekam setiap keystroke selama user berada di halaman Facebook. Event listener yang digunakan adalah `keydown` — merekam setiap tombol yang ditekan secara real-time.

Keylogger ini berfungsi sebagai fallback: jika exfiltration via form submit tidak berhasil, data keystroke tetap terekam dan di-exfiltrate secara terpisah ke C2.

![](./assets/soal-6/keydown_event.png)

**Jawaban:** `keydown`

---

### 7. What is the domain where the extension transmits the exfiltrated data?

Domain C2 tempat seluruh data hasil exfiltration dikirimkan ditemukan langsung di dalam code.

![](./assets/soal-7/c2_domain.png)

**Jawaban:** `Mo.Elshaheedy.com`

---

### 8. Which function in the code is used to exfiltrate user credentials?

Fungsi yang menangani seluruh proses credential exfiltration — dari pengambilan username/email dan password dari form, pembungkusan ke format JSON, enkripsi, hingga pengiriman ke C2:

```javascript
exfiltrateCredentials(username, password)
```

![](./assets/soal-8/exfiltrate_func.png)

**Jawaban:** `exfiltrateCredentials`

---

### 9. Which encryption algorithm is applied to secure the data before sending?

Sebelum dikirim ke C2, data credential dienkripsi menggunakan **AES**. Key enkripsinya di-hardcode langsung di dalam code:

```javascript
SuperSecretKey123
```

![](./assets/soal-9/aes_encrypt.png)

**Jawaban:** `AES`

---

### 10. What does the extension access to store or manipulate session-related data?

Kembali ke `manifest.json` — dari 7 permission yang diminta extension ini, salah satunya adalah `cookies`. Ini mengindikasikan bahwa extension memiliki kemampuan untuk mengakses dan memanipulasi session cookie pengguna, yang bisa dimanfaatkan sebagai vektor exfiltration tambahan di luar credential harvesting.

![](./assets/soal-10/manifest_permissions.png)

**Jawaban:** `cookies`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| Domain C2 | `Mo.Elshaheedy.com` | Server tujuan exfiltration data |
| Target URL (encoded) | `d3d3LmZhY2Vib29rLmNvbQ==` | URL trigger yang di-encode Base64 di app.js |
| Target URL (decoded) | `www.facebook.com` | Situs yang dimonitor untuk credential harvesting |
| Hardcoded Key | `SuperSecretKey123` | Key AES yang ditulis langsung di dalam code |
| Permission Sensitif | `cookies` | Salah satu dari 7 permission di manifest.json |
| Exfiltration Method | `<img src="...">` | Pixel/Image-based Exfiltration |
| Fungsi Exfiltration | `exfiltrateCredentials(username, password)` | Fungsi utama pencurian credential |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Persistence | Browser Extensions | T1176 | Extension berbahaya yang menyamar sebagai ChatGPT |
| Defense Evasion | Virtualization/Sandbox Evasion | T1497 | Cek `navigator.plugins.length === 0` untuk deteksi sandbox |
| Defense Evasion | Obfuscated Files or Information | T1027 | URL target di-encode dengan Base64 |
| Credential Access | Input Capture: Keylogging | T1056.001 | Keylogger via event `keydown` saat di halaman Facebook |
| Collection | Input Capture | T1056 | Credential harvesting via event `submit` |
| Exfiltration | Exfiltration Over C2 Channel | T1041 | Data dikirim ke `Mo.Elshaheedy.com` via image request |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

Extension ini menyamar sebagai "ChatGPT" dan dirancang sebagai malicious browser extension dengan kemampuan sandbox evasion. Saat pertama dijalankan, extension mengecek kondisi `navigator.plugins.length === 0` — jika browser terdeteksi berjalan di lingkungan sandbox atau virtual machine, extension langsung menonaktifkan diri dan berperilaku normal agar tidak terdeteksi saat analisis dinamis.

Jika lolos dari pemeriksaan sandbox, extension mulai memonitor URL yang dikunjungi user. Target spesifiknya adalah `www.facebook.com`, yang alamatnya di-encode Base64 (`d3d3LmZhY2Vib29rLmNvbQ==`) di dalam code untuk menghindari deteksi static analysis.

Ketika user mengunjungi Facebook, dua mekanisme exfiltration aktif secara bersamaan:

1. **Credential Harvesting** — Event `submit` ditangkap saat user login. Username/email dan password diambil dari form, dibungkus dalam format JSON, dienkripsi dengan AES (key hardcoded: `SuperSecretKey123`), lalu dikirim ke C2 via fungsi `exfiltrateCredentials()`.

2. **Keylogging** — Setiap keystroke direkam via event `keydown` selama user berada di halaman Facebook. Data ini di-exfiltrate secara terpisah sebagai fallback jika credential harvesting gagal.

Teknik exfiltration yang digunakan adalah **Pixel/Image-based Exfiltration** — data dikirim bukan via XHR, melainkan melalui request gambar dengan `<img src="...">`, yang lebih sulit terdeteksi oleh network monitoring. Seluruh data dikirimkan ke domain C2 `Mo.Elshaheedy.com`.

Selain itu, permission `cookies` di `manifest.json` mengindikasikan kemampuan potensial untuk mengakses dan memanipulasi session cookie — vektor exfiltration tambahan yang mungkin digunakan di skenario serangan yang lebih luas.

**Attack chain:**  
`Install extension` → `Sandbox check` → `Monitor URL` → `Facebook detected` → `Credential harvest (submit)` + `Keylogger (keydown)` → `AES encrypt` → `Image-based exfiltration` → `C2: Mo.Elshaheedy.com`

### Todo / Follow-up

- [ ] Pelajari lebih dalam teknik **Pixel/Image-based Exfiltration** dan cara deteksinya di level network — kenapa request via `<img>` lebih susah dideteksi dibanding XHR
- [ ] Eksplorasi teknik **sandbox evasion berbasis `navigator.plugins`** dan cara counter-detection dari sisi analyst
- [ ] Pelajari cara analisis malicious browser extension secara lebih mendalam menggunakan **ExtAnalysis** dan **CRX Viewer**
- [ ] Review implikasi permission `cookies` di browser extension — seberapa jauh akses yang bisa diperoleh attacker

---

## 📚 References

- [CyberDefenders — FakeGPT Lab](https://cyberdefenders.org/blueteam-ctf-challenges/fakegpt/)
- [MITRE ATT&CK — T1176 Browser Extensions](https://attack.mitre.org/techniques/T1176/)
- [MITRE ATT&CK — T1056.001 Keylogging](https://attack.mitre.org/techniques/T1056/001/)
- [MITRE ATT&CK — T1497 Virtualization/Sandbox Evasion](https://attack.mitre.org/techniques/T1497/)
- [MITRE ATT&CK — T1041 Exfiltration Over C2 Channel](https://attack.mitre.org/techniques/T1041/)
- [MITRE ATT&CK — T1027 Obfuscated Files or Information](https://attack.mitre.org/techniques/T1027/)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
