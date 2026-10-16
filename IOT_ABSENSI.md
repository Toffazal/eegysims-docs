# DOKUMENTASI API IOT ABSENSI RFID STANDALONE (EEGYO SIMS)

Dokumentasi ini menjelaskan secara lengkap integrasi antara perangkat keras scanner absensi (*ESP32, ESP8266, Modul GSM SIM800L/A7670, RFID RC522/PN532, RTC DS3231, LCD 16x2 I2C*) dengan backend sistem **Eegyo SIMS**.

---

## DAFTAR ISI
1. [Arsitektur & Konsep Sistem](#1-arsitektur--konsep-sistem)
2. [Mekanisme Autentikasi Perangkat](#2-mekanisme-autentikasi-perangkat)
3. [Daftar Endpoint](#3-daftar-endpoint)
   - [3.1 GET /api/iot/ping](#31-get-apiiotping-heartbeat--sync-rtc)
   - [3.2 GET /api/iot/check-card/{rfid}](#32-get-apiiotcheck-cardrfid-mode-cekregistrasi)
   - [3.3 POST /api/iot/absen](#33-post-apiiotabsen-endpoint-utama-tap-rfid)
   - [3.4 POST /api/iot/sync-offline](#34-post-apiiotsync-offline-batch-sinkronisasi-microsd)
4. [Logika & Aturan Bisnis Absensi](#4-logika--aturan-bisnis-absensi)
5. [Standar Output Kontrol Hardware (LCD, Buzzer, LED)](#5-standar-output-kontrol-hardware-lcd-buzzer-led)
6. [Contoh Implementasi Firmware (C++ / Arduino IDE / ESP32)](#6-contoh-implementasi-firmware-c--arduino-ide--esp32)

---

## 1. Arsitektur & Konsep Sistem

Sistem IoT Absensi Eegyo SIMS dirancang dengan prinsip **Edge Computing + Offline-First**:
- **Otomatis Mengenali Kartu**: Sistem otomatis membedakan apakah kartu milik **Siswa** atau **Pegawai/Guru** tanpa perlu tombol pemilihan manual di alat.
- **Otomatis Masuk & Pulang**: Tap pertama di hari tersebut dicatat sebagai **Masuk**. Tap berikutnya setelah interval toleransi (> 10 menit) dicatat sebagai **Pulang**.
- **Edge-Ready Hardware Control**: Respon dari API menyertakan teks tampilan LCD baris 1 & 2 (`lcd_line1`, `lcd_line2`), sinyal bunyi buzzer (`buzzer`), dan status indikator LED (`led`), sehingga mikrokontroler tidak perlu memproses logika tampilan yang rumit.
- **Dukungan Offline-First**: Jika koneksi internet/GSM terputus, data scan disimpan sementara di MicroSD card lokal perangkat dan dapat dikirim sekaligus (batch) melalui endpoint `sync-offline` saat jaringan kembali online.

---

## 2. Mekanisme Autentikasi Perangkat

Setiap request dari mikrokontroler scanner wajib diautentikasi untuk memastikan data hanya dikirim oleh mesin scanner resmi sekolah yang terdaftar di menu **Manajemen Scanner RFID**.

### Header Autentikasi
Header berikut wajib disertakan pada setiap request HTTP:

| Nama Header | Tipe Data | Keterangan |
|-------------|-----------|------------|
| `device_key` | String | Kunci rahasia scanner yang digenerate di sistem admin (*contoh: `sk_eegyo_3f8b91a...`*). Alternatif nama header: `Device-Key` atau `X-Device-Key`. |

### Parameter Perangkat
ID unik perangkat wajib disertakan di **Payload (Body)** atau melalui header `X-Device-Id`:

| Nama Parameter | Tipe Data | Keterangan |
|----------------|-----------|------------|
| `device_id` | String | ID unik scanner (*contoh: `SCANNER-GERBANG-01`*). |

> **Catatan Keamanan:** Jika `device_key` salah, scanner dinonaktifkan oleh admin, atau `device_id` tidak ditemukan, server akan mengembalikan respon error autentikasi (`401` / `403` / `404`).

---

## 3. Daftar Endpoint

Base URL sistem mengikuti domain/IP server Eegyo SIMS Anda, misalnya:
`http://sekolah-anda.com/api/iot` atau `http://192.168.1.100/api/iot`

---

### 3.1 GET `/api/iot/ping` (Heartbeat & Sync RTC)
Digunakan saat ESP32 pertama kali booting atau secara periodik (misal setiap 1-5 menit) untuk memeriksa status koneksi, status libur sekolah, dan menyinkronkan waktu RTC DS3231 dengan waktu server.

#### Request
- **Method:** `GET`
- **Headers:**
  ```http
  device_key: sk_eegyo_xxxxxxxxxxxxxx
  ```
- **Query Params (Opsional):**
  - `device_id` (String): ID scanner untuk mencatat timestamp `last_ping` dan IP scanner di database.

#### Contoh Request URL:
```
GET /api/iot/ping?device_id=SCANNER-GERBANG-01
```

#### Response Sukses (200 OK):
```json
{
  "success": true,
  "device_id": "SCANNER-GERBANG-01",
  "device_nama": "Scanner Gerbang Utama",
  "lokasi": "Pintu Gerbang Depan",
  "server_time": "06:45:12",
  "server_date": "2026-09-13",
  "server_day": "Senin",
  "timestamp": 1789343112,
  "datetime": "2026-09-13 06:45:12",
  "school_name": "SMK Negeri 1 Contoh",
  "school_status": "aktif",
  "jam_masuk": "07:15",
  "message": "Koneksi ke server Eegyo SIMS berhasil.",
  "lcd_line1": "SMK Negeri 1 Con",
  "lcd_line2": "06:45 13/09/2026"
}
```

---

### 3.2 GET `/api/iot/check-card/{rfid}` (Mode Cek/Registrasi)
Digunakan untuk mengecek identitas pemilik kartu tanpa mencatat presensi. Sangat berguna untuk:
- Mode registrasi kartu baru di alat teknisi.
- Debugging pembacaan UID kartu di Serial Monitor.

#### Request
- **Method:** `GET`
- **Headers:**
  ```http
  device_key: sk_eegyo_xxxxxxxxxxxxxx
  ```
- **Query Params (Opsional):** `device_id=SCANNER-GERBANG-01`
- **Path Parameter:**
  - `{rfid}` : UID kartu RFID yang dibaca (misal: `1A2B3C4D`).

#### Contoh Response Kartu Terdaftar (200 OK):
```json
{
  "success": true,
  "tipe": "siswa",
  "id": 142,
  "nama": "Ahmad Fauzi",
  "identitas": "0081234567",
  "info": "XII RPL 1",
  "rfid": "1A2B3C4D",
  "message": "Kartu terdaftar sebagai Siswa Aktif.",
  "lcd_line1": "Ahmad Fauzi",
  "lcd_line2": "Siswa: XII RPL 1"
}
```

#### Contoh Response Kartu Tidak Terdaftar (404 Not Found):
```json
{
  "success": false,
  "message": "Kartu RFID belum terdaftar pada sistem.",
  "rfid": "99AA88BB",
  "lcd_line1": "TIDAK TERDAFTAR",
  "lcd_line2": "UID: 99AA88BB"
}
```

---

### 3.3 POST `/api/iot/absen` (Endpoint Utama Tap RFID)
Endpoint utama yang dipanggil saat ada kartu RFID yang ditempelkan ke reader.

#### Request
- **Method:** `POST`
- **Headers:**
  ```http
  Content-Type: application/json
  device_key: sk_eegyo_xxxxxxxxxxxxxx
  ```
- **Body JSON:**
```json
{
  "device_id": "SCANNER-GERBANG-01",
  "rfid": "1A2B3C4D",
  "scanned_at": "2026-09-13 07:10:00"
}
```
*Keterangan parameter:*
- `device_id` *(Wajib)*: ID perangkat scanner.
- `rfid` *(Wajib)*: String UID kartu RFID. Karakter spasi, strip, atau titik dua otomatis dibersihkan oleh server.
- `scanned_at` *(Opsional)*: Waktu saat kartu ditap (`YYYY-MM-DD HH:mm:ss`). Berguna saat pengiriman data tertunda atau menggunakan RTC lokal. Jika tidak dikirim, server menggunakan waktu `now()`.

---

#### Contoh-Contoh Response:

##### 1. Siswa - Presensi Masuk (Tepat Waktu)
```json
{
  "success": true,
  "message": "Presensi masuk berhasil dicatat (pukul 07:10).",
  "tipe": "siswa",
  "nama": "Ahmad Fauzi",
  "status": "Masuk",
  "waktu": "07:10",
  "terlambat": false,
  "lcd_line1": "Ahmad Fauzi",
  "lcd_line2": "Masuk 07:10 OK",
  "buzzer": "success",
  "led": "green"
}
```

##### 2. Siswa - Presensi Masuk (Terlambat)
Sistem otomatis membuat catatan poin pelanggaran keterlambatan dan mengirim notifikasi WhatsApp ke Wali Kelas & Orang Tua (Ibu).
```json
{
  "success": true,
  "message": "Presensi masuk berhasil dicatat (Terlambat pukul 07:25).",
  "tipe": "siswa",
  "nama": "Muhammad Rizky",
  "status": "Masuk",
  "waktu": "07:25",
  "terlambat": true,
  "lcd_line1": "Muhammad Rizky",
  "lcd_line2": "TERLAMBAT 07:25",
  "buzzer": "warning",
  "led": "red"
}
```

##### 3. Presensi Pulang Berhasil
```json
{
  "success": true,
  "message": "Presensi pulang berhasil dicatat (pukul 14:05).",
  "tipe": "siswa",
  "nama": "Ahmad Fauzi",
  "status": "Pulang",
  "waktu": "14:05",
  "terlambat": false,
  "lcd_line1": "Ahmad Fauzi",
  "lcd_line2": "Pulang 14:05 OK",
  "buzzer": "success",
  "led": "green"
}
```

##### 4. Ditolak: Belum 10 Menit Sejak Jam Masuk
Mencegah *double tapping* kartu yang tidak disengaja.
```json
{
  "success": false,
  "message": "Belum 10 menit sejak jam absen masuk.",
  "lcd_line1": "Ahmad Fauzi",
  "lcd_line2": "Tunggu 10 Menit",
  "buzzer": "warning",
  "led": "red"
}
```

##### 5. Ditolak: Sudah Absen Pulang
```json
{
  "success": false,
  "message": "Kamu sudah absen pulang hari ini (pukul 14:05).",
  "lcd_line1": "Ahmad Fauzi",
  "lcd_line2": "Pulang 14:05",
  "buzzer": "warning",
  "led": "red"
}
```

##### 6. Ditolak: Kartu Tidak Dikenal
```json
{
  "success": false,
  "message": "Kartu RFID belum terdaftar di sistem SIMS.",
  "lcd_line1": "AKSES DITOLAK",
  "lcd_line2": "Kartu Tdk Dikenal",
  "buzzer": "error",
  "led": "red"
}
```

##### 7. Ditolak: Sekolah Libur / Tidak Aktif
```json
{
  "success": false,
  "message": "Sekolah sedang tidak aktif atau dalam masa libur.",
  "lcd_line1": "AKSES DITUTUP",
  "lcd_line2": "Sekolah Libur",
  "buzzer": "error",
  "led": "red"
}
```

##### 8. Ditolak: Lisensi Aplikasi Tidak Valid / Kedaluwarsa
Jika lisensi sekolah sudah habis atau tidak valid, mesin scanner dinonaktifkan dari server.
```json
{
  "success": false,
  "message": "Lisensi aplikasi tidak valid atau telah kedaluwarsa.",
  "lcd_line1": "LISENSI MATI",
  "lcd_line2": "Hubungi Vendor",
  "buzzer": "error",
  "led": "red"
}
```

---

### 3.4 POST `/api/iot/sync-offline` (Batch Sinkronisasi MicroSD)
Ketika alat berada dalam kondisi *offline* (ketiadaan sinyal WiFi atau GSM), ESP32 menyimpan setiap pembacaan kartu ke file log MicroSD lokal. Saat koneksi internet tersambung kembali, alat mengirimkan kumpulan data tersebut sekaligus ke endpoint ini.

#### Request
- **Method:** `POST`
- **Headers:**
  ```http
  Content-Type: application/json
  device_key: sk_eegyo_xxxxxxxxxxxxxx
  ```
- **Body JSON:**
```json
{
  "device_id": "SCANNER-GERBANG-01",
  "scans": [
    {
      "rfid": "1A2B3C4D",
      "scanned_at": "2026-09-13 06:55:00"
    },
    {
      "rfid": "5E6F7A8B",
      "scanned_at": "2026-09-13 07:01:22"
    },
    {
      "rfid": "9C0D1E2F",
      "scanned_at": "2026-09-13 07:18:45"
    }
  ]
}
```

#### Response (200 OK):
```json
{
  "success": true,
  "message": "Proses sinkronisasi offline selesai. Berhasil: 3, Gagal/Skip: 0.",
  "summary": {
    "total": 3,
    "success": 3,
    "failed": 0
  }
}
```

> **Keunggulan Teknis:** Server memproses setiap item secara internal memanggil sub-request transaksi absensi lengkap (termasuk status keterlambatan sesuai timestamp `scanned_at` masing-masing saat kartu di-tap, bukan waktu saat disinkronkan).

---

## 4. Logika & Aturan Bisnis Absensi

### Alur Siswa
1. **Pemeriksaan Status Sekolah:** Jika tanggal/hari tersebut dinyatakan libur pada sistem, akses ditolak.
2. **Belum Absen:** Menjadi data **Presensi Masuk** (`status: Hadir`).
3. **Keterlambatan Siswa:**
   - Dibandingkan terhadap batas `jamMasuk` sekolah (default `07:15`).
   - Jika waktu masuk > batas jam masuk, sistem secara otomatis:
     1. Menambahkan catatan **Pelanggaran Siswa** (*Datang Terlambat*, default +5 poin).
     2. Mengirim WhatsApp ke Wali Kelas berisi notifikasi siswa terlambat & nama scanner.
     3. Mengirim WhatsApp ke nomor Ibu siswa.
4. **Sudah Absen Masuk:**
   - Jika tap ulang < 10 menit $\rightarrow$ Ditolak (`Tunggu 10 Menit`).
   - Jika tap ulang $\ge$ 10 menit $\rightarrow$ Dicatat sebagai **Presensi Pulang** (`keluar: HH:mm`).
   - Jika notifikasi pulang diaktifkan, kirim WA ke Orang Tua.
5. **Sudah Pulang:** Tap berikutnya ditolak (`Pulang HH:mm`).

### Alur Pegawai / Guru
1. **Belum Absen:** Menjadi data **Presensi Masuk Pegawai**. Sistem mengirim notifikasi WA ke Kepala Sekolah.
2. **Sudah Absen Masuk:**
   - Interval < 10 menit $\rightarrow$ Ditolak (`Tunggu 10 Menit`).
   - Interval $\ge$ 10 menit $\rightarrow$ Dicatat sebagai **Presensi Pulang Pegawai**. Sistem mengirim notifikasi WA kepulangan pegawai ke Kepala Sekolah.
3. **Sudah Pulang:** Tap berikutnya ditolak.

---

## 5. Standar Output Kontrol Hardware (LCD, Buzzer, LED)

API mengembalikan nilai panduan kontrol komponen agar program mikrokontroler tetap ringkas:

### 1. Tampilan Layar (LCD 16x2 / OLED)
- `lcd_line1`: Teks baris 1 (Maksimal 16 karakter, biasanya nama siswa/guru atau status error).
- `lcd_line2`: Teks baris 2 (Maksimal 16 karakter, jam masuk/pulang atau instruksi).

### 2. Sinyal Bunyi Buzzer (`buzzer`)
| Nilai | Arti | Rekomendasi Pola Nada pada Mikrokontroler |
|-------|------|-------------------------------------------|
| `"success"` | Sukses Tepat Waktu | Beep pendek 1x (100ms ON). |
| `"warning"` | Berhasil tapi Terlambat / Peringatan | Beep sedang 2x (150ms ON, 100ms OFF, 150ms ON). |
| `"error"` | Gagal / Ditolak / Tidak Dikenal | Beep panjang 1x atau 3x cepat (400ms ON). |

### 3. Indikator LED (`led`)
| Nilai | Status Warna | Rekomendasi Aksi Pin GPIO |
|-------|--------------|---------------------------|
| `"green"` | Hijau | Nyalakan LED Hijau selama 1.5 - 2 detik. |
| `"red"` | Merah | Nyalakan LED Merah / Kedipkan 2x. |

---

## 6. Contoh Implementasi Firmware (C++ / Arduino IDE / ESP32)

Berikut adalah template kode C++ siap pakai untuk mikrokontroler ESP32 menggunakan library `HTTPClient` dan `ArduinoJson` (v6 atau v7):

```cpp
#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>
#include <LiquidCrystal_I2C.h>

// -------------------------------------------------------------
// KONFIGURASI JARINGAN & PERANGKAT
// -------------------------------------------------------------
const char* WIFI_SSID     = "NamaWiFiSekolah";
const char* WIFI_PASSWORD = "PasswordWiFi";

const char* API_URL       = "http://192.168.1.100/api/iot/absen";
const char* DEVICE_ID     = "SCANNER-GERBANG-01";
const char* DEVICE_KEY    = "sk_eegyo_3f8b91a7c4d5e6f7";

// PIN HARDWARE
const int PIN_BUZZER      = 25;
const int PIN_LED_GREEN   = 26;
const int PIN_LED_RED     = 27;

LiquidCrystal_I2C lcd(0x27, 16, 2);

void setup() {
  Serial.begin(115200);
  
  pinMode(PIN_BUZZER, OUTPUT);
  pinMode(PIN_LED_GREEN, OUTPUT);
  pinMode(PIN_LED_RED, OUTPUT);

  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("SIMS RFID READY");
  lcd.setCursor(0, 1);
  lcd.print("Hubungkan WiFi..");

  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("WiFi Terhubung!");
  lcd.setCursor(0, 1);
  lcd.print(WiFi.localIP());
  delay(1500);
  tampilkanStandby();
}

void loop() {
  // Simulasikan saat pembaca RFID mendeteksi UID kartu
  // Misalnya string uid didapat dari modul RC522:
  // kirimPresensi("1A2B3C4D");
}

void tampilkanStandby() {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("TEMPELKAN KARTU");
  lcd.setCursor(0, 1);
  lcd.print("Eegyo SIMS Ready");
}

void eksekusiIndikator(String buzzerType, String ledColor) {
  if (ledColor == "green") {
    digitalWrite(PIN_LED_GREEN, HIGH);
  } else {
    digitalWrite(PIN_LED_RED, HIGH);
  }

  if (buzzerType == "success") {
    digitalWrite(PIN_BUZZER, HIGH);
    delay(120);
    digitalWrite(PIN_BUZZER, LOW);
  } else if (buzzerType == "warning") {
    digitalWrite(PIN_BUZZER, HIGH); delay(100);
    digitalWrite(PIN_BUZZER, LOW);  delay(80);
    digitalWrite(PIN_BUZZER, HIGH); delay(100);
    digitalWrite(PIN_BUZZER, LOW);
  } else { // error
    digitalWrite(PIN_BUZZER, HIGH);
    delay(400);
    digitalWrite(PIN_BUZZER, LOW);
  }

  delay(1500); // Tahan tampilan di LCD sejenak
  digitalWrite(PIN_LED_GREEN, LOW);
  digitalWrite(PIN_LED_RED, LOW);
  tampilkanStandby();
}

void kirimPresensi(String rfidUid) {
  if (WiFi.status() != WL_CONNECTED) {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("KONEKSI PUTUS");
    lcd.setCursor(0, 1);
    lcd.print("Simpan MicroSD");
    // TODO: Simpan ke file log di MicroSD jika offline
    return;
  }

  HTTPClient http;
  http.begin(API_URL);
  
  // Set Header Keamanan
  http.addHeader("Content-Type", "application/json");
  http.addHeader("device_key", DEVICE_KEY);

  // Buat Payload JSON
  StaticJsonDocument<256> docReq;
  docReq["device_id"] = DEVICE_ID;
  docReq["rfid"]      = rfidUid;
  // docReq["scanned_at"] = "2026-09-13 07:15:00"; // Jika menggunakan RTC lokal

  String requestBody;
  serializeJson(docReq, requestBody);

  int httpCode = http.POST(requestBody);

  if (httpCode > 0) {
    String responseString = http.getString();
    
    StaticJsonDocument<512> docRes;
    DeserializationError err = deserializeJson(docRes, responseString);

    if (!err) {
      const char* l1     = docRes["lcd_line1"] | "HASIL SCAN";
      const char* l2     = docRes["lcd_line2"] | "";
      const char* buzzer = docRes["buzzer"]    | "error";
      const char* led    = docRes["led"]       | "red";

      // Tampilkan ke LCD sesuai instruksi server
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print(l1);
      lcd.setCursor(0, 1);
      lcd.print(l2);

      eksekusiIndikator(String(buzzer), String(led));
    } else {
      Serial.println("Gagal parsing respon JSON");
    }
  } else {
    Serial.printf("HTTP Request Gagal, error: %s\n", http.errorToString(httpCode).c_str());
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("GAGAL SERVER");
    lcd.setCursor(0, 1);
    lcd.print("Error Timeout");
    eksekusiIndikator("error", "red");
  }

  http.end();
}
```

---

## 7. Troubleshooting & Tanya Jawab

| Gejala Masalah | Penyebab Umum | Solusi |
|----------------|---------------|--------|
| **HTTP 401 Unauthorized** | Header `device_key` kosong atau salah. | Periksa konfigurasi `DEVICE_KEY` di firmware mikrokontroler. Pastikan nilainya sama persis dengan yang ada di web admin. |
| **HTTP 404 Scanner Tidak Ditemukan** | Nilai `device_id` belum didaftarkan di admin. | Daftarkan nama ID scanner di menu **Admin > Scanner RFID**. |
| **HTTP 403 Scanner Dinonaktifkan** | Scanner berstatus "Nonaktif" di database. | Masuk ke menu admin dan klik tombol aktifkan pada scanner terkait. |
| **Respon: "Belum 10 menit..."** | Siswa menempelkan kartu berulang kali dalam rentang waktu singkat. | Ini adalah mekanisme proteksi double-tapping. Siswa hanya perlu menunggu jeda 10 menit untuk absen pulang. |
| **Jam Server Tidak Sesuai** | Waktu server dan RTC alat selisih jauh. | Gunakan endpoint `GET /api/iot/ping` saat boot untuk mengoreksi waktu RTC lokal DS3231. |
