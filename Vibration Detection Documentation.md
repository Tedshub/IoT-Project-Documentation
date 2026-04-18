# 🔔 Sistem Deteksi Getaran Aktivitas Proyek Berbasis IoT
### Menggunakan Sensor SW-420 dan Buzzer dengan Monitoring melalui Bot Telegram

---

## 📋 Deskripsi Proyek

Sistem ini memanfaatkan **ESP32** sebagai mikrokontroler utama yang terhubung dengan **sensor getar SW-420** untuk mendeteksi getaran. Ketika getaran terdeteksi, **buzzer** akan berbunyi sebagai alarm lokal, dan secara bersamaan sistem akan mengirimkan notifikasi ke **Bot Telegram** dengan pesan *"Getaran Terdeteksi"*. Jika getaran terus berlanjut, notifikasi akan dikirim ulang setiap **5 detik sekali**.

---

## 🧰 Komponen yang Dibutuhkan

| No | Komponen | Keterangan |
|----|----------|------------|
| 1 | ESP32 | Mikrokontroler utama dengan WiFi built-in |
| 2 | Sensor Getar SW-420 | Mendeteksi getaran/vibrasi |
| 3 | Buzzer | Alarm bunyi saat getaran terdeteksi |
| 4 | Kabel Jumper | Penghubung antar komponen |
| 5 | Adaptor & Kabel USB | Sumber daya dan upload program |

---

## 🔌 Wiring / Rangkaian

### Diagram Koneksi

```
ESP32                  SW-420
-----                  ------
3.3V    ------------> VCC
GND     ------------> GND
GPIO34  ------------> DO (Digital Output)


ESP32                  BUZZER
-----                  ------
GPIO26  ------------> (+) Positif
GND     ------------> (-) Negatif
```

### Tabel Pin Detail

| Komponen | Pin Komponen | Pin ESP32 |
|----------|-------------|-----------|
| SW-420 | VCC | 3.3V |
| SW-420 | GND | GND |
| SW-420 | DO | GPIO34 |
| Buzzer | (+) | GPIO26 |
| Buzzer | (-) | GND |

> ⚠️ **Catatan:** Sensor SW-420 menggunakan logika **LOW aktif** — output akan bernilai LOW (0) ketika **tidak ada getaran**, dan HIGH (1) ketika **getaran terdeteksi**. Pastikan konfigurasi ini sesuai dengan modul SW-420 yang kamu gunakan, karena beberapa modul mungkin memiliki logika terbalik.

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
   Sistem Deteksi Getaran
   ```

6. **Masukkan username bot** — username harus diakhiri dengan kata `bot`.
   Contoh:
   ```
   DeteksiGetaran_bot
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
#define PIN_SENSOR  34   // Pin Digital Output sensor SW-420
#define PIN_BUZZER  26   // Pin Buzzer

// =============================================
// VARIABEL
// =============================================
WiFiClientSecure client;
UniversalTelegramBot bot(BOT_TOKEN, client);

bool getaranAktif      = false;
unsigned long lastSend = 0;
const unsigned long INTERVAL_KIRIM = 5000; // 5 detik

// =============================================
// SETUP
// =============================================
void setup() {
  Serial.begin(115200);

  pinMode(PIN_SENSOR, INPUT);
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
  bot.sendMessage(CHAT_ID, "✅ Sistem Deteksi Getaran AKTIF dan siap memantau.", "");
}

// =============================================
// LOOP
// =============================================
void loop() {
  int nilaiSensor = digitalRead(PIN_SENSOR);

  // SW-420: HIGH = ada getaran
  if (nilaiSensor == HIGH) {
    // Aktifkan buzzer
    digitalWrite(PIN_BUZZER, HIGH);

    // Kirim notifikasi Telegram (dengan jeda 5 detik)
    unsigned long sekarang = millis();
    if (!getaranAktif || (sekarang - lastSend >= INTERVAL_KIRIM)) {
      Serial.println("⚠️ Getaran terdeteksi! Mengirim notifikasi...");
      bot.sendMessage(CHAT_ID, "⚠️ *Getaran Terdeteksi!*\nSensor mendeteksi adanya getaran/aktivitas.", "Markdown");
      lastSend    = sekarang;
      getaranAktif = true;
    }
  } else {
    // Tidak ada getaran — matikan buzzer
    if (getaranAktif) {
      digitalWrite(PIN_BUZZER, LOW);
      Serial.println("✅ Getaran berhenti.");
      bot.sendMessage(CHAT_ID, "✅ Getaran berhenti.", "");
      getaranAktif = false;
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

Berikut penjelasan bagaimana ESP32 berkomunikasi dengan Bot Telegram:

### Cara Kerja Sistem

```
[SW-420 Sensor]
      |
      | (Sinyal Digital HIGH saat getaran)
      ▼
[ESP32]
      |
      |-- Aktifkan BUZZER (alarm lokal)
      |
      |-- Kirim HTTP Request ke Telegram API
      |         via WiFi (SSL/HTTPS)
      ▼
[Telegram Server]
      |
      ▼
[Bot Telegram] ──► [Chat / HP kamu]
```

### Alur Komunikasi

1. **ESP32 terhubung ke WiFi** — ESP32 menggunakan koneksi WiFi bawaan untuk mengakses internet.

2. **Library `UniversalTelegramBot`** — Library ini menyederhanakan pengiriman pesan ke Telegram. Di balik layar, ia membuat permintaan HTTPS ke endpoint:
   ```
   https://api.telegram.org/bot<TOKEN>/sendMessage
   ```

3. **Token Bot** — Digunakan sebagai autentikasi agar hanya ESP32 kamu yang bisa mengirim pesan atas nama bot tersebut.

4. **Chat ID** — Menentukan ke siapa pesan dikirim. Tanpa Chat ID yang benar, pesan tidak akan sampai.

5. **Logika pengiriman:**
   - Saat getaran **pertama kali** terdeteksi → pesan langsung dikirim
   - Jika getaran **terus berlanjut** → pesan dikirim ulang setiap **5 detik sekali** (agar tidak spam berlebihan)
   - Saat getaran **berhenti** → pesan konfirmasi "Getaran berhenti" dikirim, dan buzzer dimatikan

### Tips Koneksi

- Pastikan ESP32 berada dalam **jangkauan sinyal WiFi** yang stabil.
- Gunakan WiFi 2.4 GHz (ESP32 tidak mendukung 5 GHz).
- Jika menggunakan hotspot HP, pastikan data seluler aktif dan stabil.

---

## 📁 Struktur Project

```
Sistem-Deteksi-Getaran/
│
├── README.md               ← Dokumentasi ini
└── deteksi_getaran/
    └── deteksi_getaran.ino ← Kode program Arduino
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

> 💡 **Dibuat untuk keperluan monitoring proyek berbasis IoT menggunakan ESP32, SW-420, dan Bot Telegram.**
