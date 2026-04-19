# 🔔 IoT-Based Project Activity Vibration Detection System
### Using SW-420 Sensor and Buzzer with Monitoring via Telegram Bot

---

## 📋 Project Description

This system utilizes an **ESP32** as the main microcontroller connected to an **SW-420 vibration sensor** to detect vibrations. When a vibration is detected, a **buzzer** sounds as a local alarm, and simultaneously the system sends a notification to a **Telegram Bot** with the message *"Vibration Detected"*. If vibration continues, notifications are resent every **5 seconds**.

---

## 🧰 Required Components

| No | Component | Description |
|----|-----------|-------------|
| 1 | ESP32 | Main microcontroller with built-in WiFi |
| 2 | SW-420 Vibration Sensor | Detects vibration/tremors |
| 3 | Buzzer | Audible alarm when vibration is detected |
| 4 | Jumper Wires | Connections between components |
| 5 | Adapter & USB Cable | Power supply and program upload |

---

## 🔌 Wiring / Circuit

### Wiring Diagram

![Vibration Detection Wiring Diagram](img/Image_3.jpg)

### Connection Diagram

```
ESP32                  SW-420
-----                  ------
3.3V    ------------> VCC
GND     ------------> GND
GPIO34  ------------> DO (Digital Output)


ESP32                  BUZZER
-----                  ------
GPIO26  ------------> (+) Positive
GND     ------------> (-) Negative
```

### Detailed Pin Table

| Component | Component Pin | ESP32 Pin |
|-----------|--------------|-----------|
| SW-420 | VCC | 3.3V |
| SW-420 | GND | GND |
| SW-420 | DO | GPIO34 |
| Buzzer | (+) | GPIO26 |
| Buzzer | (-) | GND |

> ⚠️ **Note:** The SW-420 sensor uses **active LOW** logic — the output will be LOW (0) when **no vibration** is present, and HIGH (1) when **vibration is detected**. Make sure this configuration matches the SW-420 module you are using, as some modules may have inverted logic.

---

## 💻 Software Setup

### 1. Install Arduino IDE

- 🔗 Download: [https://www.arduino.cc/en/software/](https://www.arduino.cc/en/software/)
- 📺 Download & Install Tutorial: [https://youtu.be/lTKvZRfJRgw](https://youtu.be/lTKvZRfJRgw)

---

### 2. Install CP2102 Driver for ESP32

- 🔗 Download Driver: [https://www.silabs.com/software-and-tools/usb-to-uart-bridge-vcp-drivers?tab=downloads](https://www.silabs.com/software-and-tools/usb-to-uart-bridge-vcp-drivers?tab=downloads)
- 📺 CP2102 Driver Install Tutorial: [https://youtu.be/h-jqF8Y5iV4](https://youtu.be/h-jqF8Y5iV4)

---

### 3. Install ESP32 Board in Arduino IDE

- 📁 ESP32 Board Install Tutorial: [https://drive.google.com/file/d/1pb8Wt_b7EgaFoNqN1fqX-U0KogYZ6rVh/view?usp=sharing](https://drive.google.com/file/d/1pb8Wt_b7EgaFoNqN1fqX-U0KogYZ6rVh/view?usp=sharing)

> ⏳ **Note:** Installing the ESP32 board in Arduino IDE may take a bit longer than usual. Please be patient and wait for the process to complete fully.

---

## 🤖 Creating a Telegram Bot with BotFather

Follow these steps to create a new Telegram Bot:

1. **Open the Telegram app** on your phone or PC.

2. **Search for BotFather** — type `@BotFather` in the Telegram search bar, then click the account with the blue checkmark.

3. **Start a chat** — click the **START** button or send `/start`.

4. **Create a new bot** — send the command:
   ```
   /newbot
   ```

5. **Enter the bot name** — BotFather will ask for a display name for your bot.
   Example:
   ```
   Vibration Detection System
   ```

6. **Enter the bot username** — the username must end with the word `bot`.
   Example:
   ```
   VibrationDetection_bot
   ```

7. **Save the API Token** — BotFather will provide an **API Token** like this:
   ```
   123456789:ABCDefGhIJKlmNoPQRsTUVwxyZ
   ```
   > 🔐 **Keep this token safe!** It will be used in the ESP32 program code.

8. **Get your Chat ID** — open a browser and access the following URL (replace `TOKEN` with your bot token):
   ```
   https://api.telegram.org/bot<TOKEN>/getUpdates
   ```
   Send any message to your bot via Telegram first, then refresh the URL. Look for the `"id"` value inside the `"chat"` section of the JSON response. That is your **Chat ID**.

   Example JSON response:
   ```json
   {
     "message": {
       "chat": {
         "id": 987654321
       }
     }
   }
   ```

9. **Save that Chat ID** — it will be used together with the token in the program code.

---

## 📦 Required Libraries

Install the following libraries via the Arduino IDE **Library Manager** (`Sketch > Include Library > Manage Libraries`):

| Library | Description |
|---------|-------------|
| `UniversalTelegramBot` | For communication with the Telegram Bot |
| `ArduinoJson` | For parsing JSON data from the Telegram API |
| `WiFi` | Already bundled with the ESP32 board, no need to reinstall |

**How to install:**
1. Open Arduino IDE
2. Click **Sketch** → **Include Library** → **Manage Libraries...**
3. In the search box, type each library name above one by one
4. Click **Install**

---

## 📝 Program Code

Create a new project in Arduino IDE (`File > New`), then paste the following code:

```cpp
#include <WiFi.h>
#include <WiFiClientSecure.h>
#include <UniversalTelegramBot.h>
#include <ArduinoJson.h>

// =============================================
// CONFIGURATION — FILL IN YOUR OWN DATA
// =============================================
const char* ssid     = "YOUR_WIFI_NAME";       // Replace with your WiFi SSID
const char* password = "YOUR_WIFI_PASSWORD";   // Replace with your WiFi password

#define BOT_TOKEN  "YOUR_TELEGRAM_BOT_TOKEN"   // Replace with your BotFather token
#define CHAT_ID    "YOUR_CHAT_ID"              // Replace with your Chat ID

// =============================================
// PIN CONFIGURATION
// =============================================
#define PIN_SENSOR  34   // SW-420 Digital Output pin
#define PIN_BUZZER  26   // Buzzer pin

// =============================================
// VARIABLES
// =============================================
WiFiClientSecure client;
UniversalTelegramBot bot(BOT_TOKEN, client);

bool vibrationActive   = false;
unsigned long lastSend = 0;
const unsigned long SEND_INTERVAL = 5000; // 5 seconds

// =============================================
// SETUP
// =============================================
void setup() {
  Serial.begin(115200);

  pinMode(PIN_SENSOR, INPUT);
  pinMode(PIN_BUZZER, OUTPUT);
  digitalWrite(PIN_BUZZER, LOW);

  // Connect to WiFi
  Serial.print("Connecting to WiFi");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi Connected!");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());

  client.setInsecure(); // Bypass SSL certificate (for simplicity)

  // Send system activation confirmation message
  bot.sendMessage(CHAT_ID, "✅ Vibration Detection System ACTIVE and ready to monitor.", "");
}

// =============================================
// LOOP
// =============================================
void loop() {
  int sensorValue = digitalRead(PIN_SENSOR);

  // SW-420: HIGH = vibration detected
  if (sensorValue == HIGH) {
    // Activate buzzer
    digitalWrite(PIN_BUZZER, HIGH);

    // Send Telegram notification (with 5-second interval)
    unsigned long now = millis();
    if (!vibrationActive || (now - lastSend >= SEND_INTERVAL)) {
      Serial.println("⚠️ Vibration detected! Sending notification...");
      bot.sendMessage(CHAT_ID, "⚠️ *Vibration Detected!*\nThe sensor has detected vibration/activity.", "Markdown");
      lastSend        = now;
      vibrationActive = true;
    }
  } else {
    // No vibration — turn off buzzer
    if (vibrationActive) {
      digitalWrite(PIN_BUZZER, LOW);
      Serial.println("✅ Vibration stopped.");
      bot.sendMessage(CHAT_ID, "✅ Vibration has stopped.", "");
      vibrationActive = false;
    }
  }

  delay(100); // Small debounce
}
```

> 📝 **Before uploading, make sure you have replaced:**
> - `YOUR_WIFI_NAME` → Your WiFi SSID
> - `YOUR_WIFI_PASSWORD` → Your WiFi password
> - `YOUR_TELEGRAM_BOT_TOKEN` → Token from BotFather
> - `YOUR_CHAT_ID` → Your Telegram Chat ID

---

## 📤 How to Upload the Program to ESP32

1. **Make sure the ESP32 is connected** to your laptop/PC via USB cable.

2. **Configure the Board:**
   - Click the **Tools** menu in Arduino IDE
   - Go to **Board** → **esp32** → select **ESP32 Dev Module**

3. **Configure the Port:**
   - Click the **Tools** menu again
   - Go to **Port**
   - Select the port connected to the ESP32 (e.g., `COM8` on Windows, `/dev/ttyUSB0` on Linux/Mac)

4. **Upload the Program:**
   - Press **Ctrl+U** to start the upload

---

### ⚠️ Upload Troubleshooting

| Problem | Solution |
|---------|----------|
| Upload fails | The selected Port may be wrong. Try selecting a different available port. |
| Still fails after changing port | Press **Ctrl+U** and **hold the BOOT button** on the ESP32 simultaneously. The ESP32 usually requires this on the first program upload. |
| Still failing | Read the error message in the Arduino IDE **Output** window for further guidance. |

---

## 🔗 ESP32 and Telegram Bot Integration

Here is an explanation of how the ESP32 communicates with the Telegram Bot:

### How the System Works

```
[SW-420 Sensor]
      |
      | (Digital signal HIGH when vibration occurs)
      ▼
[ESP32]
      |
      |-- Activate BUZZER (local alarm)
      |
      |-- Send HTTP Request to Telegram API
      |         via WiFi (SSL/HTTPS)
      ▼
[Telegram Server]
      |
      ▼
[Telegram Bot] ──► [Your Chat / Phone]
```

### Communication Flow

1. **ESP32 connects to WiFi** — the ESP32 uses its built-in WiFi connection to access the internet.

2. **`UniversalTelegramBot` library** — this library simplifies sending messages to Telegram. Behind the scenes, it makes an HTTPS request to the endpoint:
   ```
   https://api.telegram.org/bot<TOKEN>/sendMessage
   ```

3. **Bot Token** — used for authentication so that only your ESP32 can send messages on behalf of that bot.

4. **Chat ID** — determines who the message is sent to. Without the correct Chat ID, the message will not be delivered.

5. **Message sending logic:**
   - When vibration is **first** detected → message is sent immediately
   - If vibration **continues** → message is resent every **5 seconds** (to avoid excessive spam)
   - When vibration **stops** → a confirmation message "Vibration has stopped" is sent, and the buzzer is turned off

### Connection Tips

- Make sure the ESP32 is within a **stable WiFi signal range**.
- Use a **2.4 GHz** WiFi network (ESP32 does not support 5 GHz).
- If using a mobile hotspot, make sure mobile data is active and stable.

---

## 📁 Project Structure

```
Vibration-Detection-System/
│
├── README.md                    ← This documentation
└── vibration_detection/
    └── vibration_detection.ino  ← Arduino program code
```

---

## 📞 References & Important Links

| Source | Link |
|--------|------|
| Arduino IDE | https://www.arduino.cc/en/software/ |
| Arduino IDE Install Tutorial | https://youtu.be/lTKvZRfJRgw |
| CP2102 Driver | https://www.silabs.com/software-and-tools/usb-to-uart-bridge-vcp-drivers?tab=downloads |
| CP2102 Driver Install Tutorial | https://youtu.be/h-jqF8Y5iV4 |
| ESP32 Board Install Tutorial | https://drive.google.com/file/d/1pb8Wt_b7EgaFoNqN1fqX-U0KogYZ6rVh/view?usp=sharing |
| BotFather Telegram | https://t.me/BotFather |

---

> 💡 **Created for IoT-based project monitoring using ESP32, SW-420 Vibration Sensor, and Telegram Bot.**
