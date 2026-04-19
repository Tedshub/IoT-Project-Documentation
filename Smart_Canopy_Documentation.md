# ☂️ IoT-Based Smart Canopy System
### Rain Detection Using FC-37 Sensor with SG-90 Servo and Monitoring via Telegram Bot

---

## 📋 Project Description

This system uses an **ESP32** as the main microcontroller connected to an **FC-37 rain sensor** to detect rainfall. When rain is detected, the **SG-90 servo** will automatically close the canopy, and the system will send a notification to a **Telegram Bot** with the message *"Rain detected! Canopy is closing."* When the rain stops, the canopy will open again automatically and a notification is sent back to Telegram. If rain continues, notifications are resent every **5 seconds**.

---

## 🧰 Required Components

| No | Component | Description |
|----|-----------|-------------|
| 1 | ESP32 | Main microcontroller with built-in WiFi |
| 2 | FC-37 Rain Sensor | Detects the presence of rainwater on the sensor surface |
| 3 | SG-90 Servo | Motor for opening/closing the canopy |
| 4 | Jumper Wires | Connections between components |
| 5 | Adapter & USB Cable | Power supply and program upload |

---

## 🔌 Wiring / Circuit

### Wiring Diagram

![Smart Canopy Wiring Diagram](img/Image_2.jpg)

### Connection Diagram

```
ESP32                  FC-37 (Rain Sensor)
-----                  -------------------
3.3V    ------------> VCC
GND     ------------> GND
GPIO34  ------------> DO (Digital Output)


ESP32                  SG-90 SERVO
-----                  -----------
5V (VIN) -----------> VCC (Red Wire)
GND      -----------> GND (Brown/Black Wire)
GPIO13   -----------> Signal (Orange/Yellow Wire)
```

### Detailed Pin Table

| Component | Component Pin | ESP32 Pin |
|-----------|--------------|-----------|
| FC-37 | VCC | 3.3V |
| FC-37 | GND | GND |
| FC-37 | DO | GPIO34 |
| SG-90 Servo | VCC (Red) | 5V (VIN) |
| SG-90 Servo | GND (Brown/Black) | GND |
| SG-90 Servo | Signal (Orange/Yellow) | GPIO13 |

> ⚠️ **Important Notes:**
> - The FC-37 sensor uses **active LOW** logic — the DO output will be **LOW (0)** when **rain is detected**, and **HIGH (1)** when **no rain**. Make sure this matches the FC-37 module you are using.
> - The SG-90 servo requires **5V**. Use the **VIN** pin on the ESP32 (not 3.3V) so the servo moves with sufficient power.
> - If the servo vibrates unstably, try adding a 100µF capacitor between the servo's VCC and GND.

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
   Smart IoT Canopy
   ```

6. **Enter the bot username** — the username must end with the word `bot`.
   Example:
   ```
   SmartCanopy_bot
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
| `ESP32Servo` | For controlling the servo motor on ESP32 |
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
#include <ESP32Servo.h>

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
#define PIN_RAIN_SENSOR  34   // FC-37 Digital Output pin
#define PIN_SERVO        13   // SG-90 Signal pin

// =============================================
// SERVO ANGLE CONFIGURATION
// =============================================
#define ANGLE_CLOSED   90    // Servo angle when canopy is CLOSED (raining)
#define ANGLE_OPEN      0    // Servo angle when canopy is OPEN (clear weather)

// =============================================
// VARIABLES
// =============================================
WiFiClientSecure client;
UniversalTelegramBot bot(BOT_TOKEN, client);
Servo canopyServo;

bool rainActive        = false;
unsigned long lastSend = 0;
const unsigned long SEND_INTERVAL = 5000; // 5 seconds

// =============================================
// SETUP
// =============================================
void setup() {
  Serial.begin(115200);

  pinMode(PIN_RAIN_SENSOR, INPUT);

  // Initialize servo
  canopyServo.attach(PIN_SERVO);
  canopyServo.write(ANGLE_OPEN); // Initial position: canopy open
  delay(500);

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
  bot.sendMessage(CHAT_ID,
    "✅ Smart Canopy System ACTIVE.\nCanopy is in open position and ready to monitor the weather.",
    "");
}

// =============================================
// LOOP
// =============================================
void loop() {
  int sensorValue = digitalRead(PIN_RAIN_SENSOR);

  // FC-37: LOW = rain detected
  if (sensorValue == LOW) {

    // Close the canopy
    canopyServo.write(ANGLE_CLOSED);

    // Send Telegram notification (with 5-second interval)
    unsigned long now = millis();
    if (!rainActive || (now - lastSend >= SEND_INTERVAL)) {
      Serial.println("🌧️ Rain detected! Closing canopy...");
      bot.sendMessage(CHAT_ID,
        "🌧️ *Rain Detected!*\nCanopy is closing automatically.",
        "Markdown");
      lastSend   = now;
      rainActive = true;
    }

  } else {
    // No rain — open canopy again
    if (rainActive) {
      canopyServo.write(ANGLE_OPEN);
      Serial.println("☀️ Rain stopped. Opening canopy...");
      bot.sendMessage(CHAT_ID,
        "☀️ Rain has stopped.\nCanopy is opening automatically.",
        "");
      rainActive = false;
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

> 🔧 **Servo Calibration:** The `ANGLE_CLOSED` and `ANGLE_OPEN` values can be adjusted (0–180°) depending on the mechanical construction of your canopy. Test them first before mounting on the actual canopy.

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

### How the System Works

```
[FC-37 Rain Sensor]
        |
        | (Digital signal LOW when raining)
        ▼
[ESP32]
        |
        |-- Move SERVO → CLOSED position (90°)
        |
        |-- Send HTTP Request to Telegram API
        |         via WiFi (SSL/HTTPS)
        ▼
[Telegram Server]
        |
        ▼
[Telegram Bot] ──► [Your Chat / Phone]

When rain stops:
[FC-37] → HIGH ──► [ESP32] → Servo OPEN (0°) + Notification "Rain has stopped"
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
   - When rain is **first** detected → servo closes the canopy + message is sent immediately
   - If rain **continues** → message is resent every **5 seconds**
   - When rain **stops** → servo opens the canopy + confirmation message is sent to Telegram

6. **Servo Control:**
   - Servo moves to **90°** → canopy **closed** (when raining)
   - Servo moves to **0°** → canopy **open** (when clear)
   - Angles can be adjusted to fit the mechanical construction of the canopy

### Connection Tips

- Make sure the ESP32 is within a **stable WiFi signal range**.
- Use a **2.4 GHz** WiFi network (ESP32 does not support 5 GHz).
- If using a mobile hotspot, make sure mobile data is active and stable.
- Make sure the FC-37 sensor is **installed outdoors** and can be directly exposed to rainwater.

---

## 📁 Project Structure

```
Smart-Canopy-IoT/
│
├── README.md                  ← This documentation
└── smart_canopy/
    └── smart_canopy.ino       ← Arduino program code
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

> 💡 **Created for an IoT-based automatic canopy system using ESP32, FC-37 Rain Sensor, SG-90 Servo, and Telegram Bot.**
