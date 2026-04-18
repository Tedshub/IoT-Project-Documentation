# ☂️ Sistem Kanopi Pintar Berbasis IoT
### Deteksi Hujan Menggunakan Sensor FC-37 dengan Servo SG-90 dan Monitoring melalui Bot Telegram

---

## 📋 Deskripsi Proyek

Sistem ini menggunakan **ESP32** sebagai mikrokontroler utama yang terhubung dengan **sensor hujan FC-37** untuk mendeteksi curah hujan. Ketika hujan terdeteksi, **servo SG-90** akan secara otomatis menutup kanopi, dan sistem akan mengirimkan notifikasi ke **Bot Telegram** dengan pesan *"Hujan terdeteksi! Kanopi sedang menutup."* Ketika hujan berhenti, kanopi akan terbuka kembali secara otomatis dan notifikasi dikirim kembali ke Telegram. Jika hujan terus berlanjut, notifikasi akan dikirim ulang setiap **5 detik sekali**.

---

## 🧰 Komponen yang Dibutuhkan

| No | Komponen | Keterangan |
|----|----------|------------|
| 1 | ESP32 | Mikrokontroler utama dengan WiFi built-in |
| 2 | Sensor Hujan FC-37 | Mendeteksi adanya air hujan pada permukaan sensor |
| 3 | Servo SG-90 | Motor penggerak buka/tutup kanopi |
| 4 | Kabel Jumper | Penghubung antar komponen |
| 5 | Adaptor & Kabel USB | Sumber daya dan upload program |

---

## 🔌 Wiring / Rangkaian

### Diagram Koneksi

```
ESP32                  FC-37 (Sensor Hujan)
-----                  --------------------
3.3V    ------------> VCC
GND     ------------> GND
GPIO34  ------------> DO (Digital Output)


ESP32                  SERVO SG-90
-----                  -----------
5V (VIN) -----------> VCC (Kabel Merah)
GND      -----------> GND (Kabel Coklat/Hitam)
GPIO13   -----------> Signal (Kabel Oranye/Kuning)
```

### Tabel Pin Detail

| Komponen | Pin Komponen | Pin ESP32 |
|----------|-------------|-----------|
| FC-37 | VCC | 3.3V |
| FC-37 | GND | GND |
| FC-37 | DO | GPIO34 |
| Servo SG-90 | VCC (Merah) | 5V (VIN) |
| Servo SG-90 | GND (Coklat/Hitam) | GND |
| Servo SG-90 | Signal (Oranye/Kuning) | GPIO13 |

> ⚠️ **Catatan Penting:**
> - Sensor FC-37 menggunakan logika **LOW aktif** — output DO akan bernilai **LOW (0)** ketika **hujan terdeteksi**, dan **HIGH (1)** saat **tidak hujan**. Pastikan sesuai dengan modul FC-37 yang kamu gunakan.
> - Servo SG-90 membutuhkan tegangan **5V**. Gunakan pin **VIN** pada ESP32 (bukan 3.3V) agar servo bergerak dengan tenaga yang cukup.
> - Jika servo bergetar tidak stabil, coba tambahkan kapasitor 100µF antara VCC dan GND servo.

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
   Kanopi Pintar IoT
   ```

6. **Masukkan username bot** — username harus diakhiri dengan kata `bot`.
   Contoh:
   ```
   KanopiPintar_bot
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
| `ESP32Servo` | Untuk mengontrol servo motor pada ESP32 |
| `WiFi` | Sudah bawaan board ESP32, tidak perlu install ulang |

**Cara install:**
1. Buka Arduino IDE
2. Klik menu **Sketch** → **Include Library** → **Manage Libraries...**
3. Di kolom pencarian, ketik nama library di atas satu per satu
4. Klik **Install**

---

## 📝 Kode Program

Buat project baru di Arduino IDE (`File > New`), lalu salin kode berikut:

```cpp
#include <WiFi.h>
#include <WiFiClientSecure.h>
#include <UniversalTelegramBot.h>
#include <ArduinoJson.h>
#include <ESP32Servo.h>

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
#define PIN_SENSOR_HUJAN  34   // Pin Digital Output sensor FC-37
#define PIN_SERVO         13   // Pin Signal servo SG-90

// =============================================
// KONFIGURASI SUDUT SERVO
// =============================================
#define SUDUT_TUTUP   90    // Sudut servo saat kanopi TERTUTUP (hujan)
#define SUDUT_BUKA     0    // Sudut servo saat kanopi TERBUKA (cerah)

// =============================================
// VARIABEL
// =============================================
WiFiClientSecure client;
UniversalTelegramBot bot(BOT_TOKEN, client);
Servo servoKanopi;

bool hujanAktif        = false;
unsigned long lastSend = 0;
const unsigned long INTERVAL_KIRIM = 5000; // 5 detik

// =============================================
// SETUP
// =============================================
void setup() {
  Serial.begin(115200);

  pinMode(PIN_SENSOR_HUJAN, INPUT);

  // Inisialisasi servo
  servoKanopi.attach(PIN_SERVO);
  servoKanopi.write(SUDUT_BUKA); // Posisi awal: kanopi terbuka
  delay(500);

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
  bot.sendMessage(CHAT_ID,
    "✅ Sistem Kanopi Pintar AKTIF.\nKanopi dalam posisi terbuka dan siap memantau cuaca.",
    "");
}

// =============================================
// LOOP
// =============================================
void loop() {
  int nilaiSensor = digitalRead(PIN_SENSOR_HUJAN);

  // FC-37: LOW = hujan terdeteksi
  if (nilaiSensor == LOW) {

    // Tutup kanopi
    servoKanopi.write(SUDUT_TUTUP);

    // Kirim notifikasi Telegram (dengan jeda 5 detik)
    unsigned long sekarang = millis();
    if (!hujanAktif || (sekarang - lastSend >= INTERVAL_KIRIM)) {
      Serial.println("🌧️ Hujan terdeteksi! Menutup kanopi...");
      bot.sendMessage(CHAT_ID,
        "🌧️ *Hujan Terdeteksi!*\nKanopi sedang menutup secara otomatis.",
        "Markdown");
      lastSend   = sekarang;
      hujanAktif = true;
    }

  } else {
    // Tidak hujan — buka kanopi kembali
    if (hujanAktif) {
      servoKanopi.write(SUDUT_BUKA);
      Serial.println("☀️ Hujan berhenti. Membuka kanopi...");
      bot.sendMessage(CHAT_ID,
        "☀️ Hujan berhenti.\nKanopi kembali terbuka secara otomatis.",
        "");
      hujanAktif = false;
    }
  }

  delay(100); // Debounce kecil
}
```

> 📝 **Sebelum upload, pastikan kamu sudah mengganti:**
> - `NAMA_WIFI_KAMU` → SSID WiFi kamu
> - `PASSWORD_WIFI_KAMU` → Password WiFi kamu
> - `TOKEN_BOT_TELEGRAM_KAMU` → Token dari BotFather
> - `CHAT_ID_KAMU` → Chat ID Telegram kamu

> 🔧 **Kalibrasi Servo:** Nilai `SUDUT_TUTUP` dan `SUDUT_BUKA` dapat disesuaikan (0–180°) tergantung konstruksi mekanik kanopi kamu. Uji coba terlebih dahulu sebelum dipasang pada kanopi sesungguhnya.

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
[FC-37 Sensor Hujan]
        |
        | (Sinyal Digital LOW saat hujan)
        ▼
[ESP32]
        |
        |-- Gerakkan SERVO → Posisi TUTUP (90°)
        |
        |-- Kirim HTTP Request ke Telegram API
        |         via WiFi (SSL/HTTPS)
        ▼
[Telegram Server]
        |
        ▼
[Bot Telegram] ──► [Chat / HP kamu]

Saat hujan berhenti:
[FC-37] → HIGH ──► [ESP32] → Servo BUKA (0°) + Notifikasi "Hujan berhenti"
```

### Alur Komunikasi

1. **ESP32 terhubung ke WiFi** — ESP32 menggunakan koneksi WiFi bawaan untuk mengakses internet.

2. **Library `UniversalTelegramBot`** — Library ini menyederhanakan pengiriman pesan ke Telegram. Di balik layar, ia membuat permintaan HTTPS ke endpoint:
   ```
   https://api.telegram.org/bot<TOKEN>/sendMessage
   ```

3. **Token Bot** — Digunakan sebagai autentikasi agar hanya ESP32 kamu yang bisa mengirim pesan atas nama bot tersebut.

4. **Chat ID** — Menentukan ke siapa pesan dikirim. Tanpa Chat ID yang benar, pesan tidak akan sampai.

5. **Logika pengiriman pesan:**
   - Saat hujan **pertama kali** terdeteksi → servo menutup kanopi + pesan langsung dikirim
   - Jika hujan **terus berlanjut** → pesan dikirim ulang setiap **5 detik sekali**
   - Saat hujan **berhenti** → servo membuka kanopi + pesan konfirmasi dikirim ke Telegram

6. **Kontrol Servo:**
   - Servo bergerak ke sudut **90°** → kanopi **tertutup** (saat hujan)
   - Servo bergerak ke sudut **0°** → kanopi **terbuka** (saat cerah)
   - Sudut dapat disesuaikan sesuai konstruksi mekanik kanopi

### Tips Koneksi

- Pastikan ESP32 berada dalam **jangkauan sinyal WiFi** yang stabil.
- Gunakan WiFi **2.4 GHz** (ESP32 tidak mendukung 5 GHz).
- Jika menggunakan hotspot HP, pastikan data seluler aktif dan stabil.
- Pastikan sensor FC-37 **terpasang di luar ruangan** dan dapat terkena air hujan secara langsung.

---

## 📁 Struktur Project

```
Kanopi-Pintar-IoT/
│
├── README.md                  ← Dokumentasi ini
└── kanopi_pintar/
    └── kanopi_pintar.ino      ← Kode program Arduino
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

> 💡 **Dibuat untuk keperluan sistem kanopi otomatis berbasis IoT menggunakan ESP32, Sensor Hujan FC-37, Servo SG-90, dan Bot Telegram.**
