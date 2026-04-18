# 💧 Sistem Water Tank Pintar Berbasis IoT
### Monitoring Volume Air Menggunakan Sensor HC-SR04 dan Buzzer dengan Notifikasi melalui Bot Telegram

---

## 📋 Deskripsi Proyek

Sistem ini menggunakan **ESP32** sebagai mikrokontroler utama yang terhubung dengan **sensor ultrasonik HC-SR04** untuk mengukur jarak permukaan air di dalam tangki. Sistem akan secara terus-menerus memantau ketinggian air dan memberikan **peringatan** apabila jarak antara sensor dengan permukaan air melebihi batas yang ditentukan (default: **15 cm**), yang mengindikasikan bahwa volume air sudah menipis. Saat kondisi tersebut terjadi, **buzzer** akan berbunyi sebagai alarm lokal dan notifikasi dikirim ke **Bot Telegram**. Jika kondisi air rendah terus berlanjut, notifikasi akan dikirim ulang setiap **5 detik sekali**.

> 📐 **Prinsip Kerja:** Sensor HC-SR04 dipasang di bagian atas tangki menghadap ke bawah. Semakin jauh jarak yang terukur, semakin sedikit volume air di dalam tangki. Jika jarak terukur **≥ 15 cm**, berarti air sudah hampir habis dan sistem akan memberi peringatan.

---

## 🧰 Komponen yang Dibutuhkan

| No | Komponen | Keterangan |
|----|----------|------------|
| 1 | ESP32 | Mikrokontroler utama dengan WiFi built-in |
| 2 | Sensor HC-SR04 | Sensor ultrasonik pengukur jarak permukaan air |
| 3 | Buzzer | Alarm bunyi saat volume air menipis |
| 4 | Kabel Jumper | Penghubung antar komponen |
| 5 | Adaptor & Kabel USB | Sumber daya dan upload program |

---

## 🔌 Wiring / Rangkaian

### Diagram Koneksi

```
ESP32                  HC-SR04
-----                  -------
5V (VIN) -----------> VCC
GND      -----------> GND
GPIO5    -----------> TRIG
GPIO18   -----------> ECHO


ESP32                  BUZZER
-----                  ------
GPIO26  ------------> (+) Positif
GND     ------------> (-) Negatif
```

### Tabel Pin Detail

| Komponen | Pin Komponen | Pin ESP32 |
|----------|-------------|-----------|
| HC-SR04 | VCC | 5V (VIN) |
| HC-SR04 | GND | GND |
| HC-SR04 | TRIG | GPIO5 |
| HC-SR04 | ECHO | GPIO18 |
| Buzzer | (+) Positif | GPIO26 |
| Buzzer | (-) Negatif | GND |

> ⚠️ **Catatan Penting:**
> - Sensor HC-SR04 membutuhkan tegangan **5V**. Gunakan pin **VIN** pada ESP32 (bukan 3.3V).
> - Pin **ECHO** pada HC-SR04 mengeluarkan sinyal **5V**, sedangkan ESP32 hanya toleran **3.3V**. Untuk keamanan, disarankan menggunakan **voltage divider** (pembagi tegangan) pada pin ECHO menggunakan resistor **1kΩ** dan **2kΩ** sebelum masuk ke GPIO18. Namun dalam praktik sederhana, koneksi langsung sering berhasil — gunakan dengan risiko sendiri.
> - Sensor dipasang **di bagian atas tangki**, menghadap ke bawah menuju permukaan air.
> - Pastikan tidak ada hambatan/penghalang antara sensor dan permukaan air.

### Skema Voltage Divider ECHO (Opsional tapi Disarankan)

```
HC-SR04 ECHO (5V) ---[1kΩ]---+---[2kΩ]--- GND
                              |
                           GPIO18 ESP32 (~3.3V)
```

---

## 💻 Persiapan Software

### 1. Install Arduino IDE

- 🔗 Download: [https://www.arduino.cc/en/software/](https://www.arduino.cc/en/software/)
- 📺 Tutorial Download & Install: [https://youtu.be/lTKvZRfJRgw](https://youtu.be/lTKvZRfJRgw)

---

### 2. Install Driver CP2102 untuk ESP32

- 🔗 Download Driver: [https://www.silabs.com/software-and-tools/usb-to-uart-bridge-vcp-drivers?tab=downloads](https://www.silabs.com/software-and-tools/usb-to-uart-bridge-vcp-drivers?tab=downloads)
- 📺 Tutorial Install Driver CP2102: [https://youtu.be/h-jqF8Y5iV4](https://youtu.be/h-jqF8Y5iV4)

---

### 3. Install Board ESP32 di Arduino IDE

- 📁 Tutorial Install Board ESP32: [https://drive.google.com/file/d/1pb8Wt_b7EgaFoNqN1fqX-U0KogYZ6rVh/view?usp=sharing](https://drive.google.com/file/d/1pb8Wt_b7EgaFoNqN1fqX-U0KogYZ6rVh/view?usp=sharing)

> ⏳ **Catatan:** Install board ESP32 di Arduino IDE memang membutuhkan waktu sedikit lebih lama dari biasanya. Harap bersabar dan tunggu hingga proses selesai sepenuhnya.

---

## 🤖 Membuat Bot Telegram dengan BotFather

Ikuti langkah-langkah berikut untuk membuat Bot Telegram baru:

1. **Buka aplikasi Telegram** di HP atau PC kamu.

2. **Cari BotFather** — ketik `@BotFather` di kolom pencarian Telegram, lalu klik akun dengan centang biru.

3. **Mulai chat** — klik tombol **START** atau kirim pesan `/start`.

4. **Buat bot baru** — kirim perintah:
   ```
   /newbot
   ```

5. **Masukkan nama bot** — BotFather akan meminta nama tampilan untuk bot kamu.
   Contoh:
   ```
   Water Tank Monitor
   ```

6. **Masukkan username bot** — username harus diakhiri dengan kata `bot`.
   Contoh:
   ```
   WaterTankMonitor_bot
   ```

7. **Simpan Token API** — BotFather akan memberikan **Token API** seperti berikut:
   ```
   123456789:ABCDefGhIJKlmNoPQRsTUVwxyZ
   ```
   > 🔐 **Simpan token ini baik-baik!** Token ini akan digunakan di kode program ESP32.

8. **Dapatkan Chat ID kamu** — buka browser dan akses URL berikut (ganti `TOKEN` dengan token bot kamu):
   ```
   https://api.telegram.org/bot<TOKEN>/getUpdates
   ```
   Kirim pesan apa saja ke bot kamu terlebih dahulu melalui Telegram, lalu refresh URL tersebut. Cari nilai `"id"` di dalam bagian `"chat"` pada respons JSON. Itulah **Chat ID** kamu.

   Contoh respons JSON:
   ```json
   {
     "message": {
       "chat": {
         "id": 987654321
       }
     }
   }
   ```

9. **Simpan Chat ID tersebut** — akan digunakan bersama token di kode program.

---

## 📦 Library yang Dibutuhkan

Install library berikut melalui **Library Manager** Arduino IDE (`Sketch > Include Library > Manage Libraries`):

| Library | Keterangan |
|---------|------------|
| `UniversalTelegramBot` | Untuk komunikasi dengan Bot Telegram |
| `ArduinoJson` | Untuk parsing data JSON dari Telegram API |
| `WiFi` | Sudah bawaan board ESP32, tidak perlu install ulang |

**Cara install:**
1. Buka Arduino IDE
2. Klik menu **Sketch** → **Include Library** → **Manage Libraries...**
3. Di kolom pencarian, ketik nama library di atas satu per satu
4. Klik **Install**

> ✅ Sensor HC-SR04 dikontrol langsung menggunakan fungsi `pulseIn()` bawaan Arduino, **tidak membutuhkan library tambahan**.

---

## 📝 Kode Program

Buat project baru di Arduino IDE (`File > New`), lalu salin kode berikut:

```cpp
#include <WiFi.h>
#include <WiFiClientSecure.h>
#include <UniversalTelegramBot.h>
#include <ArduinoJson.h>

// =============================================
// KONFIGURASI — SESUAIKAN DENGAN DATA KAMU
// =============================================
const char* ssid     = "NAMA_WIFI_KAMU";       // Ganti dengan nama WiFi
const char* password = "PASSWORD_WIFI_KAMU";   // Ganti dengan password WiFi

#define BOT_TOKEN  "TOKEN_BOT_TELEGRAM_KAMU"   // Ganti dengan token BotFather
#define CHAT_ID    "CHAT_ID_KAMU"              // Ganti dengan Chat ID kamu

// =============================================
// KONFIGURASI PIN
// =============================================
#define PIN_TRIG    5    // Pin TRIG sensor HC-SR04
#define PIN_ECHO    18   // Pin ECHO sensor HC-SR04
#define PIN_BUZZER  26   // Pin Buzzer

// =============================================
// KONFIGURASI BATAS JARAK (dapat diubah)
// =============================================
// Jika jarak terukur >= BATAS_JARAK_CM,
// berarti air sudah menipis → peringatan aktif
#define BATAS_JARAK_CM  15   // Dalam satuan sentimeter

// =============================================
// VARIABEL
// =============================================
WiFiClientSecure client;
UniversalTelegramBot bot(BOT_TOKEN, client);

bool peringatanAktif            = false;
unsigned long lastSend          = 0;
unsigned long lastUkur          = 0;
const unsigned long INTERVAL_KIRIM = 5000;  // Interval kirim Telegram: 5 detik
const unsigned long INTERVAL_UKUR  = 1000;  // Interval baca sensor: 1 detik

// =============================================
// FUNGSI BACA JARAK HC-SR04
// =============================================
float bacaJarak() {
  digitalWrite(PIN_TRIG, LOW);
  delayMicroseconds(2);
  digitalWrite(PIN_TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(PIN_TRIG, LOW);

  // Timeout 30ms (~5 meter)
  long durasi = pulseIn(PIN_ECHO, HIGH, 30000);

  if (durasi == 0) return -1; // Tidak terbaca / timeout
  return (durasi / 2.0) / 29.1;
}

// =============================================
// SETUP
// =============================================
void setup() {
  Serial.begin(115200);

  pinMode(PIN_TRIG,   OUTPUT);
  pinMode(PIN_ECHO,   INPUT);
  pinMode(PIN_BUZZER, OUTPUT);
  digitalWrite(PIN_BUZZER, LOW);

  // Koneksi WiFi
  Serial.print("Menghubungkan ke WiFi");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi Terhubung!");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());

  client.setInsecure(); // Bypass SSL certificate (untuk kemudahan)

  // Kirim pesan konfirmasi sistem aktif
  String pesanAwal = "✅ Sistem Water Tank Monitor AKTIF.\n";
  pesanAwal += "Pemantauan volume air dimulai.\n";
  pesanAwal += "Batas peringatan: jarak ≥ " + String(BATAS_JARAK_CM) + " cm dari sensor.";
  bot.sendMessage(CHAT_ID, pesanAwal, "");
}

// =============================================
// LOOP
// =============================================
void loop() {
  unsigned long sekarang = millis();

  // Baca sensor setiap 1 detik
  if (sekarang - lastUkur >= INTERVAL_UKUR) {
    lastUkur = sekarang;

    float jarak = bacaJarak();

    if (jarak < 0) {
      Serial.println("⚠️ Sensor tidak terbaca. Periksa koneksi.");
      return;
    }

    Serial.print("📏 Jarak terukur: ");
    Serial.print(jarak);
    Serial.println(" cm");

    // Cek apakah air sudah menipis (jarak >= batas)
    if (jarak >= BATAS_JARAK_CM) {

      // Aktifkan buzzer
      digitalWrite(PIN_BUZZER, HIGH);

      // Kirim notifikasi Telegram dengan jeda 5 detik
      if (!peringatanAktif || (sekarang - lastSend >= INTERVAL_KIRIM)) {
        Serial.println("🚨 Air menipis! Mengirim notifikasi...");

        String pesan = "🚨 *Peringatan! Air Menipis!*\n";
        pesan += "📏 Jarak terukur: *" + String(jarak, 1) + " cm*\n";
        pesan += "⚠️ Batas peringatan: " + String(BATAS_JARAK_CM) + " cm\n";
        pesan += "Segera isi ulang tangki air!";

        bot.sendMessage(CHAT_ID, pesan, "Markdown");
        lastSend        = sekarang;
        peringatanAktif = true;
      }

    } else {
      // Air masih cukup — matikan buzzer
      if (peringatanAktif) {
        digitalWrite(PIN_BUZZER, LOW);
        Serial.println("✅ Volume air normal kembali.");

        String pesan = "✅ *Volume Air Normal*\n";
        pesan += "📏 Jarak terukur: *" + String(jarak, 1) + " cm*\n";
        pesan += "Tangki air sudah terisi dengan cukup.";

        bot.sendMessage(CHAT_ID, pesan, "Markdown");
        peringatanAktif = false;
      }
    }
  }
}
```

> 📝 **Sebelum upload, pastikan kamu sudah mengganti:**
> - `NAMA_WIFI_KAMU` → SSID WiFi kamu
> - `PASSWORD_WIFI_KAMU` → Password WiFi kamu
> - `TOKEN_BOT_TELEGRAM_KAMU` → Token dari BotFather
> - `CHAT_ID_KAMU` → Chat ID Telegram kamu

> 🔧 **Mengubah Batas Jarak Peringatan:**
> Cukup ubah nilai pada baris berikut di kode:
> ```cpp
> #define BATAS_JARAK_CM  15   // Ubah angka ini sesuai kebutuhan
> ```
> Contoh: jika tangki kamu tingginya 50 cm dan ingin peringatan saat air tersisa 10 cm dari dasar, maka isi nilai `40` (50 cm - 10 cm).

---

## 📤 Cara Upload Program ke ESP32

1. **Pastikan ESP32 terhubung** ke laptop/PC menggunakan kabel USB.

2. **Setting Board:**
   - Klik menu **Tools** pada Arduino IDE
   - Arahkan ke **Board** → **esp32** → pilih **ESP32 Dev Module**

3. **Setting Port:**
   - Klik menu **Tools** lagi
   - Arahkan ke **Port**
   - Pilih port yang terhubung dengan ESP32 (contoh: `COM8` di Windows, `/dev/ttyUSB0` di Linux/Mac)

4. **Upload Program:**
   - Tekan **Ctrl+U** untuk memulai upload

---

### ⚠️ Troubleshooting Upload

| Masalah | Solusi |
|---------|--------|
| Upload gagal | Kemungkinan Port yang dipilih salah. Coba pilih port lain yang tersedia. |
| Masih gagal setelah ganti port | Tekan **Ctrl+U** dan **tahan tombol BOOT** pada ESP32 secara bersamaan. ESP32 biasanya memerlukan ini saat pertama kali diupload program. |
| Masih gagal | Perhatikan dan baca pesan error pada jendela **Output** Arduino IDE untuk petunjuk lebih lanjut. |

---

## 🔗 Integrasi ESP32 dengan Bot Telegram

### Cara Kerja Sistem

```
[HC-SR04 Sensor Ultrasonik]
          |
          | Kirim pulsa TRIG → terima pantulan ECHO
          | Hitung jarak (cm) ke permukaan air
          ▼
[ESP32] — Jarak >= 15 cm? (air menipis)
          |
          |-- YA  → Aktifkan BUZZER (alarm lokal)
          |          Kirim notifikasi Telegram tiap 5 detik
          |
          |-- TIDAK → Buzzer tetap mati
          |            Jika sebelumnya ada peringatan:
          |            Matikan buzzer + kirim "Air normal"
          ▼
[Telegram Server]
          |
          ▼
[Bot Telegram] ──► [Chat / HP kamu]
```

### Cara Kerja Sensor HC-SR04

```
[ Sensor HC-SR04 ] ← dipasang di atas tangki
        |
        |  ↕ Jarak terukur (cm)
        |
  ══════════════  ← Permukaan air
  |              |
  |  Tangki Air  |
  |              |
  ══════════════
```

1. Pin **TRIG** mengirimkan pulsa ultrasonik selama 10 mikrosecond.
2. Gelombang suara merambat ke bawah dan memantul dari permukaan air.
3. Pin **ECHO** menerima kembali pantulan gelombang tersebut.
4. **Rumus jarak:** `Jarak (cm) = (Durasi ECHO / 2) / 29.1`
5. Semakin **besar** nilai jarak → permukaan air semakin **jauh** dari sensor → air semakin **sedikit**.

### Alur Notifikasi Telegram

1. **ESP32 terhubung ke WiFi** — menggunakan koneksi WiFi untuk mengakses internet.
2. **Library `UniversalTelegramBot`** — mengirim pesan HTTPS ke endpoint Telegram:
   ```
   https://api.telegram.org/bot<TOKEN>/sendMessage
   ```
3. **Logika pengiriman pesan:**
   - Saat air **pertama kali** terdeteksi menipis → pesan + nilai jarak dikirim langsung
   - Jika kondisi air rendah **terus berlanjut** → pesan dikirim ulang setiap **5 detik**
   - Saat air **kembali normal** → pesan konfirmasi dikirim, buzzer dimatikan
4. Setiap pesan Telegram menyertakan **nilai jarak aktual (cm)** agar mudah dipantau dari jarak jauh.

### Tips Pemasangan & Koneksi

- Pasang sensor HC-SR04 **tepat di tengah bagian atas** tutup tangki menghadap ke bawah.
- Pastikan posisi sensor **horizontal / rata** agar pembacaan akurat.
- Hindari turbulensi air yang berlebihan saat pengisian karena dapat mempengaruhi pembacaan.
- Gunakan WiFi **2.4 GHz** (ESP32 tidak mendukung 5 GHz).
- Jika menggunakan hotspot HP, pastikan data seluler aktif dan stabil.

---

## 📐 Panduan Kalibrasi Batas Jarak

Sesuaikan nilai `BATAS_JARAK_CM` berdasarkan dimensi tangki air kamu:

```
Contoh: Tinggi Tangki = 60 cm
Sensor dipasang di atas (jarak 0 cm = tangki penuh)

Estimasi Pengaturan:
├── Air penuh      : jarak ±  2 cm dari sensor
├── Air 75%        : jarak ± 15 cm dari sensor
├── Air 50%        : jarak ± 30 cm dari sensor
├── Air 25%        : jarak ± 45 cm dari sensor  ← Set BATAS_JARAK_CM = 45
└── Air hampir habis : jarak ± 55 cm dari sensor
```

> Ukur sendiri dimensi tangki kamu dan tentukan pada jarak berapa peringatan sebaiknya diberikan. Kemudian masukkan nilai tersebut ke `BATAS_JARAK_CM` pada kode program.

---

## 📁 Struktur Project

```
Water-Tank-Monitor-IoT/
│
├── README.md                      ← Dokumentasi ini
└── water_tank_monitor/
    └── water_tank_monitor.ino     ← Kode program Arduino
```

---

## 📞 Referensi & Link Penting

| Sumber | Link |
|--------|------|
| Arduino IDE | https://www.arduino.cc/en/software/ |
| Tutorial Install Arduino IDE | https://youtu.be/lTKvZRfJRgw |
| Driver CP2102 | https://www.silabs.com/software-and-tools/usb-to-uart-bridge-vcp-drivers?tab=downloads |
| Tutorial Install Driver CP2102 | https://youtu.be/h-jqF8Y5iV4 |
| Tutorial Install Board ESP32 | https://drive.google.com/file/d/1pb8Wt_b7EgaFoNqN1fqX-U0KogYZ6rVh/view?usp=sharing |
| BotFather Telegram | https://t.me/BotFather |

---

> 💡 **Dibuat untuk keperluan sistem monitoring volume air tangki berbasis IoT menggunakan ESP32, Sensor Ultrasonik HC-SR04, Buzzer, dan Bot Telegram.**
