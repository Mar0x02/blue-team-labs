# Secrets (Challenges) — BTLO

> **Platform:** Blue Team Labs Online  
> **Category:** Cryptography / Web Security  
> **Difficulty:** Easy  
> **Status:** ✅ Completed  
> **Date:** 2026-05-29  
> **Time Spent:** ~1 jam  

---

## 📌 Prolog

Challenge ini berkutat seputar JWT — token yang sering dipakai untuk autentikasi di web app modern. Tugasnya sederhana: decode token yang diberikan, crack secret-nya, lalu buat ulang token baru dengan hak istimewa yang lebih rendah. Bagus untuk memahami kenapa JWT bukan mekanisme keamanan yang sempurna kalau implementasinya ceroboh.

---

## 🎯 Scenario

Kamu adalah seorang senior cyber security engineer. Saat shift kamu, terdapat aksi dengan hak istimewa tinggi dari sumber yang tidak dikenal dan terindikasi sebagai aktivitas berbahaya. Tim telah mengintersep tiket yang melakukan aksi-aksi tersebut — kamu adalah orang yang membuat secret untuk tiket-tiket ini. Perbaiki ini dan kirimkan tiket dengan hak istimewa rendah agar bisa memastikan kamu layak untuk posisi ini.

Token yang diberikan:

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJmbGFnIjoiQlRMe180X0V5ZXN9IiwiaWF0Ijo5MDAwMDAwMCwibmFtZSI6IkdyZWF0RXhwIiwiYWRtaW4iOnRydWV9.jbkZHll_W17BOALT95JQ17glHBj9nY-oWhT1uiahtv8
```

---

## ❓ Questions

1. Sebutkan nama token ini?
2. Apa struktur dari token ini?
3. Apa hint yang kamu temukan dari token ini?
4. Apa Secret-nya?
5. Bisakah kamu membuat tiket tanda tangan terverifikasi baru dengan hak istimewa rendah?

---

## 🔍 Answer & Walkthrough

### 1. Sebutkan nama token ini?

Token ini menggunakan format `xxxxx.yyyyy.zzzzz` — tiga bagian dipisah titik, di-encode Base64URL. Format standar JSON Web Token.

**Jawaban:** `JWT`

---

### 2. Apa struktur dari token ini?

JWT terdiri dari tiga bagian yang masing-masing punya peran berbeda: Header berisi metadata token (tipe dan algoritma), Payload berisi klaim/data, dan Signature digunakan untuk verifikasi integritas.

**Jawaban:** `Header.Payload.Signature`

---

### 3. Apa hint yang kamu temukan dari token ini?

JWT hanya di-encode Base64URL — **bukan dienkripsi**. Siapapun bisa decode dan baca isinya tanpa perlu tahu secret.

**Langkah 1 — Decode Header**

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```

Decode Base64 → `{"typ":"JWT","alg":"HS256"}`

**Langkah 2 — Decode Payload**

```
eyJmbGFnIjoiQlRMe180X0V5ZXN9IiwiaWF0Ijo5MDAwMDAwMCwibmFtZSI6IkdyZWF0RXhwIiwiYWRtaW4iOnRydWV9
```

Decode Base64 →

```json
{
  "flag": "BTL{_4_Eyes}",
  "iat": 90000000,
  "name": "GreatExp",
  "admin": true
}
```

Dari payload terbaca langsung field `flag` yang berisi hint. Nilai `_4_Eyes` merujuk pada **Four Eyes Principle** — konsep keamanan di mana aksi sensitif memerlukan persetujuan dua orang berbeda (dual control).

Cara decode bisa pakai [jwt.io](https://jwt.io) (paste token, langsung terbaca), atau manual via Python:

```python
import base64, json

token = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJmbGFnIjoiQlRMe180X0V5ZXN9IiwiaWF0Ijo5MDAwMDAwMCwibmFtZSI6IkdyZWF0RXhwIiwiYWRtaW4iOnRydWV9.jbkZHll_W17BOALT95JQ17glHBj9nY-oWhT1uiahtv8"

header_b64, payload_b64, sig = token.split('.')
header = json.loads(base64.urlsafe_b64decode(header_b64 + "=="))
payload = json.loads(base64.urlsafe_b64decode(payload_b64 + "=="))

print("Header:", header)
print("Payload:", payload)
```

**Jawaban:** `_4_Eyes`

---

### 4. Apa Secret-nya?

Secret JWT yang pendek bisa di-crack dengan hashcat menggunakan mask attack (brute force per-karakter).

**Langkah 1 — Simpan JWT ke file**

```bash
echo "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJmbGFnIjoiQlRMe180X0V5ZXN9IiwiaWF0Ijo5MDAwMDAwMCwibmFtZSI6IkdyZWF0RXhwIiwiYWRtaW4iOnRydWV9.jbkZHll_W17BOALT95JQ17glHBj9nY-oWhT1uiahtv8" > jwt.txt
```

**Langkah 2 — Brute force 4 karakter dengan hashcat**

```bash
hashcat -a 3 -m 16500 jwt.txt "?a?a?a?a"
```

**Langkah 3 — Lihat hasil**

```bash
hashcat -m 16500 jwt.txt "?a?a?a?a" --show
# Output: <token>:bT!0
```

| Parameter | Fungsi |
|-----------|--------|
| `-a 3` | Mode brute force (mask attack) |
| `-m 16500` | Hash type = JWT (HMAC-SHA256) |
| `?l` | Huruf kecil a-z |
| `?u` | Huruf besar A-Z |
| `?d` | Angka 0-9 |
| `?s` | Karakter spesial |
| `?a` | Semua karakter (l+u+d+s) |
| `--increment` | Coba dari panjang 1 sampai maksimum |
| `--show` | Tampilkan hasil yang sudah ter-crack |

Secret 4 karakter dengan `?a?a?a?a` selesai dalam hitungan detik di GPU. Kalau panjang tidak diketahui, pakai `--increment --increment-min=1`.

**Jawaban:** `bT!0`

---

### 5. Bisakah kamu membuat tiket tanda tangan terverifikasi baru dengan hak istimewa rendah?

Setelah secret diketahui (`bT!0`), buat ulang token dengan mengubah `admin: true` → `admin: false`.

```python
import hmac, hashlib, base64, json

secret = "bT!0"

header = {"typ": "JWT", "alg": "HS256"}
payload = {
    "flag": "BTL{_4_Eyes}",
    "iat": 90000000,
    "name": "GreatExp",
    "admin": False  # diubah dari True ke False (low privilege)
}

def b64url(data):
    return base64.urlsafe_b64encode(
        json.dumps(data, separators=(',',':')).encode()
    ).rstrip(b'=').decode()

h = b64url(header)
p = b64url(payload)
msg = f"{h}.{p}"

sig = hmac.new(secret.encode(), msg.encode(), hashlib.sha256).digest()
b64sig = base64.urlsafe_b64encode(sig).rstrip(b'=').decode()

print(f"{msg}.{b64sig}")
```

Token yang dihasilkan sudah terverifikasi dengan secret yang sama, hanya nilai `admin` yang diubah jadi `false`.

**Jawaban:**

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJmbGFnIjoiQlRMe180X0V5ZXN9IiwiaWF0Ijo5MDAwMDAwMCwibmFtZSI6IkdyZWF0RXhwIiwiYWRtaW4iOmZhbHNlfQ.nMXNFvttCvtDcpswOQA8u_LpURwv6ZrCJ-ftIXegtX4
```

---

## 🚨 Key Findings / IOCs

| Tipe | Value | Keterangan |
|------|-------|------------|
| JWT Secret | `bT!0` | Secret lemah, 4 karakter, di-crack dengan hashcat mask attack |
| Flag / Hint | `BTL{_4_Eyes}` | Ditemukan di payload JWT setelah decode Base64 |
| Claim berbahaya | `"admin": true` | Privilege escalation via manipulasi payload JWT |

---

## 🗺️ MITRE ATT&CK Mapping

| Tactic | Technique | ID | Keterangan |
|--------|-----------|----|------------|
| Credential Access | Brute Force: Password Cracking | T1110.002 | Hashcat mask attack untuk crack JWT secret |
| Privilege Escalation | Exploitation for Privilege Escalation | T1068 | Modifikasi claim `admin` di payload JWT |

---

## 📋 Summary — Attacker Behavior & Todo

### Attacker Behavior

Token JWT dengan secret lemah (`bT!0`, 4 karakter) diintersep. Karena payload JWT hanya di-encode Base64URL — bukan dienkripsi — attacker bisa langsung baca isi payload dan menemukan field `admin: true`. Secret di-crack dengan hashcat dalam hitungan detik. Dengan secret di tangan, attacker bisa forge token baru dengan klaim apapun — termasuk mempertahankan atau mengubah privilege sesuai keinginan.

### Todo / Follow-up

- [ ] Pelajari JWT algorithm confusion attack (`alg: none` dan RS256 → HS256 confusion)
- [ ] Eksplorasi tool `jwt_tool` untuk automated JWT testing
- [ ] Review best practice JWT: panjang secret minimum, rotasi secret, penggunaan `exp` claim
- [ ] Coba implementasi JWT yang benar di Python/Node sebagai counter-exercise

---

## 📚 References

- [jwt.io — JWT debugger dan dokumentasi](https://jwt.io)
- [MITRE ATT&CK T1110.002 — Password Cracking](https://attack.mitre.org/techniques/T1110/002/)
- [Hashcat mask attack documentation](https://hashcat.net/wiki/doku.php?id=mask_attack)
- [PortSwigger — JWT attacks](https://portswigger.net/web-security/jwt)

---

*Writeup ini dibuat sebagai bagian dari perjalanan belajar Blue Team / SOC Analyst.*
