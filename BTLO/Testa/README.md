# Testa — BTLO

> **Platform:** [Blue Team Labs Online](https://blueteamlabs.online/home/investigation/testa-ba3a8c3fd0)  
> **Category:** OT Security / ICS Forensics  
> **Difficulty:** Medium  
> **Status:** ✅ Completed  
> **Date:** 2026-05-28  
> **Time Spent:** ~2 jam  
> **Tags:** `Modbus TCP` `OT Security` `ICS` `Wireshark` `PCAP Analysis`

---

## 📌 Prolog

Ini pengalaman pertama ngerjain challenge OT/ICS — dunia yang cukup berbeda dari IT forensics biasa. Protokolnya asing (Modbus TCP), port-nya non-standard, dan Wireshark perlu di-configure manual sebelum bisa decode traffic-nya. Yang menarik adalah bagaimana serangan ini direkonstruksi dari packet capture: tiga fase jelas — reconnaissance, manipulasi fisik, lalu emergency shutdown — semuanya terlihat di traffic jaringan. Cukup eye-opening untuk melihat betapa mudahnya memanipulasi sistem OT yang tidak punya authentication sama sekali.

---

## 🎯 Scenario

Jurassic Resourcing adalah perusahaan minyak independen upstream dan midstream yang beroperasi di kawasan Teluk Persia. Terminal JRL-OT-01 di Bandar Al-Hajar menerima minyak mentah melalui pipa utama PP-101, menyimpannya di tangki penyimpanan, dan memuatnya ke kapal tanker di Dermaga B-01 untuk ekspor.

Operasi pengiriman minyak di JRL-OT-01 telah dihentikan dan tim investigasi perlu memahami apakah sistem telah disusupi. Selidiki packet capture yang tersedia.

---

## ❓ Questions

1. Which unauthorised source IP appears on the OT segment?
2. What Modbus function codes does the unauthorised host use, and which are write operations?
3. What value did the unauthorised host attempt to write to Terminal TLCS holding register 40000?
4. How many cargo tank valves did the attacker open?
5. At what time does the ship-side SHORE_STOP_ACTIVE coil transition from 0 to 1 bringing down operations for the facility?

---

## 🔍 Answer & Walkthrough

### Peta IP di Network

| IP | Peran |
|----|-------|
| `172.20.0.1` | **Unauthorized** — reconnaissance via HTTP REST API |
| `172.20.0.2` | Ship Cargo Controller (legitimate, tapi dicompromise) |
| `172.20.0.3` | OT Terminal TLCS (victim) — Web HMI port 8080, Modbus port 5020 |
| `172.20.0.4` | Engineering / ship-side Modbus client (legitimate) |
| `172.20.0.5` | **Unauthorized** — Modbus write operations via port 5020 |

### Setup Wireshark untuk Modbus

Traffic Modbus berjalan di port non-standard (5020, 5021), sehingga Wireshark tidak auto-detect. Konfigurasi manual:

1. Klik kanan packet TCP port 5020 → **Decode As** → pilih **Modbus/TCP**
2. Ulangi untuk port 5021
3. Filter: `modbus && tcp.port == 5020`

---

### 1. Which unauthorised source IP appears on the OT segment?

Filter HTTP request dan lihat siapa yang mengakses REST API OT system:

```
ip.src == 172.20.0.1 && http.request
```

IP `172.20.0.1` melakukan reconnaissance dengan mengakses:
- `GET /api/tags` — enumerate semua tag dan variable OT system
- `GET /api/events?limit=60` — lihat event log operasional
- `GET /api/batch/last` — lihat informasi batch terakhir

Meskipun berada di subnet yang sama, aktivitasnya tidak authorized.

**Jawaban:** `172.20.0.1`

---

### 2. What Modbus function codes does the unauthorised host use, and which are write operations?

Filter Modbus traffic dari IP unauthorized `172.20.0.5`:

```
modbus && ip.src == 172.20.0.5
```

Function codes yang ditemukan:

| FC | Nama | Tipe |
|----|------|------|
| FC1 | Read Coils | Read |
| FC4 | Read Input Registers | Read |
| FC5 | Write Single Coil | **Write** |
| FC16 | Write Multiple Registers | **Write** |

FC5 digunakan untuk membuka/menutup valve. FC16 digunakan untuk mengubah nilai holding register.

**Jawaban:** `FC1(R),FC4(R),FC5(W),FC16(W)`

---

### 3. What value did the unauthorised host attempt to write to Terminal TLCS holding register 40000?

Filter FC16 dari `172.20.0.5` dan inspect payload packet:

```
modbus && ip.src == 172.20.0.5 && modbus.func_code == 16
```

Pada timestamp `2026-04-30 12:51:53`, attacker menulis nilai **9000** ke holding register 40000. System log mencatat: `"Unknown command register value: 9000"` — nilai ini tidak dikenali sistem dan ditolak.

**Jawaban:** `9000`

---

### 4. How many cargo tank valves did the attacker open?

Cek HTTP response `/api/tags` dari `172.20.0.3` dan lihat status field `tanks`, lalu konfirmasi dengan packet Modbus FC5 dari `172.20.0.5`. Attacker berhasil membuka valve C1, C2, C3, dan C4 melalui serangkaian Write Single Coil command.

**Jawaban:** `4`

---

### 5. At what time does the ship-side SHORE_STOP_ACTIVE coil transition from 0 to 1?

Filter traffic di port 5021 (ship-side Modbus) di sekitar timeframe shutdown:

```
modbus && tcp.port == 5021 && ip.src == 172.20.0.2
```

Packet dari `172.20.0.2` (Ship Cargo Controller) ke `172.20.0.4` membawa SHORE_STOP signal. Event ini tercatat di `/api/events` sebagai `"Shore stop received from MV STEGOSAURUS — HH level alarm"` dan mengubah mode terminal ke **ESD (Emergency Shutdown)**.

**Jawaban:** `12:52:37:255230`

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| Unauthorized IP | `172.20.0.1` | Reconnaissance via HTTP REST API |
| Unauthorized IP | `172.20.0.5` | Modbus write operations |
| Compromised Host | `172.20.0.2` | Ship Cargo Controller — kirim SHORE_STOP |
| Target | `172.20.0.3` | OT Terminal TLCS (victim) |
| Modbus Register | `40000` | TLCS command register — diwrite nilai 9000 |
| Valves Opened | `C1, C2, C3, C4` | 4 cargo tank valves dibuka paksa |
| Shutdown Timestamp | `2026-04-30 12:52:37:255230` | SHORE_STOP_ACTIVE transition 0→1 |
| Ship Name | `MV STEGOSAURUS` | Kapal yang terlibat dalam shutdown |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Discovery | Network Service Discovery | T0840 | Reconnaissance via REST API (`/api/tags`, `/api/events`) |
| Collection | Point & Tag Identification | T0861 | Enumerate OT tags dan variabel sistem |
| Impair Process Control | Unauthorized Command Message | T0855 | FC16 write ke command register 40000 |
| Impair Process Control | Manipulation of Control | T0831 | FC5 membuka 4 cargo tank valve |
| Inhibit Response Function | Activate Firmware Update Mode | T0800 | Trigger ESD via SHORE_STOP signal |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

Serangan ini terkoordinasi dalam tiga fase yang jelas. **Fase 1 — Reconnaissance:** `172.20.0.1` mengakses HTTP REST API OT system untuk memetakan tag, event log, dan status operasional terminal. **Fase 2 — Manipulation:** `172.20.0.5` mengirimkan Modbus write command — FC16 mencoba menulis nilai 9000 ke command register (ditolak sistem), lalu FC5 berhasil membuka 4 cargo tank valve secara paksa. Karena Modbus tidak punya authentication, command langsung diterima tanpa verifikasi. **Fase 3 — Shutdown:** `172.20.0.2` (Ship Cargo Controller yang sudah dicompromise) mengirimkan SHORE_STOP signal ke terminal tepat 44 detik setelah fase manipulasi, men-trigger ESD dan menghentikan seluruh operasi loading. Kombinasi tiga aktor berbeda dalam satu serangan terkoordinasi menunjukkan ini bukan opportunistic attack — ada perencanaan dan akses internal.

### Todo / Follow-up

- [ ] Pelajari lebih dalam protokol Modbus — function codes, data model, dan cara penyerangannya
- [ ] Eksplorasi ICS security framework: IEC 62443 dan NIST SP 800-82
- [ ] Latihan analisis traffic OT lainnya: DNP3, EtherNet/IP, PROFINET
- [ ] Buat detection rule untuk mendeteksi Modbus write dari IP yang tidak authorized (anomaly-based)

---

## 📚 References

- [MITRE ATT&CK for ICS](https://attack.mitre.org/matrices/ics/)
- [Modbus Protocol Specification](https://modbus.org/docs/Modbus_Application_Protocol_V1_1b3.pdf)
- [CISA ICS Advisory Resources](https://www.cisa.gov/ics)
- [BTLO — Testa Investigation](https://blueteamlabs.online/home/investigation/testa-ba3a8c3fd0)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
