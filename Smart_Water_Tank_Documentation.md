# 💧 IoT-Based Smart Water Tank System
### Water Volume Monitoring Using HC-SR04 Sensor and Buzzer with Telegram Bot Notifications

---

## 📋 Project Description

This system uses an **ESP32** as the main microcontroller connected to an **HC-SR04 ultrasonic sensor** to measure the water surface distance inside the tank. The system continuously monitors the water level and triggers a **warning** when the distance between the sensor and the water surface exceeds a defined threshold (default: **15 cm**), indicating that the water volume is running low. When this condition occurs, a **buzzer** sounds as a local alarm and a notification is sent to a **Telegram Bot**. If the low water condition persists, notifications are resent every **5 seconds**.

> 📐 **Working Principle:** The HC-SR04 sensor is mounted at the top of the tank, facing downward. The greater the measured distance, the less water is in the tank. If the measured distance is **≥ 15 cm**, it means the water is almost empty and the system will issue a warning.

---

## 🧰 Required Components

| No | Component | Description |
|----|-----------|-------------|
| 1 | ESP32 | Main microcontroller with built-in WiFi |
| 2 | HC-SR04 Sensor | Ultrasonic sensor for measuring water surface distance |
| 3 | Buzzer | Audible alarm when water volume is low |
| 4 | Jumper Wires | Connections between components |
| 5 | Adapter & USB Cable | Power supply and program upload |

---

## 🔌 Wiring / Circuit

### Wiring Diagram

![Smart Water Tank Wiring Diagram](img/Image_1.jpg)

### Connection Diagram

```
ESP32                  HC-SR04
-----                  -------
5V (VIN) -----------> VCC
GND      -----------> GND
GPIO5    -----------> TRIG
GPIO18   -----------> ECHO


ESP32                  BUZZER
-----                  ------
GPIO26  ------------> (+) Positive
GND     ------------> (-) Negative
```

### Detailed Pin Table

| Component | Component Pin | ESP32 Pin |
|-----------|--------------|-----------|
| HC-SR04 | VCC | 5V (VIN) |
| HC-SR04 | GND | GND |
| HC-SR04 | TRIG | GPIO5 |
| HC-SR04 | ECHO | GPIO18 |
| Buzzer | (+) Positive | GPIO26 |
| Buzzer | (-) Negative | GND |

> ⚠️ **Important Notes:**
> - The HC-SR04 sensor requires **5V**. Use the **VIN** pin on the ESP32 (not 3.3V).
> - The **ECHO** pin on the HC-SR04 outputs a **5V** signal, while the ESP32 is only **3.3V** tolerant. For safety, it is recommended to use a **voltage divider** on the ECHO pin using **1kΩ** and **2kΩ** resistors before connecting to GPIO18. However, in simple practice, direct connections often work — use at your own risk.
> - Mount the sensor **at the top of the tank**, facing downward toward the water surface.
> - Ensure there are no obstacles between the sensor and the water surface.

### ECHO Voltage Divider Schematic (Optional but Recommended)

```
HC-SR04 ECHO (5V) ---[1kΩ]---+---[2kΩ]--- GND
                               |
                            GPIO18 ESP32 (~3.3V)
```

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
   Water Tank Monitor
   ```

6. **Enter the bot username** — the username must end with the word `bot`.
   Example:
   ```
   WaterTankMonitor_bot
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

> ✅ The HC-SR04 sensor is controlled directly using Arduino's built-in `pulseIn()` function — **no additional library is needed**.

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
#define PIN_TRIG    5    // HC-SR04 TRIG pin
#define PIN_ECHO    18   // HC-SR04 ECHO pin
#define PIN_BUZZER  26   // Buzzer pin

// =============================================
// DISTANCE THRESHOLD CONFIGURATION (adjustable)
// =============================================
// If measured distance >= DISTANCE_THRESHOLD_CM,
// it means water is low → warning is triggered
#define DISTANCE_THRESHOLD_CM  15   // In centimeters

// =============================================
// VARIABLES
// =============================================
WiFiClientSecure client;
UniversalTelegramBot bot(BOT_TOKEN, client);

bool warningActive              = false;
unsigned long lastSend          = 0;
unsigned long lastMeasure       = 0;
const unsigned long SEND_INTERVAL    = 5000;  // Telegram send interval: 5 seconds
const unsigned long MEASURE_INTERVAL = 1000;  // Sensor read interval: 1 second

// =============================================
// HC-SR04 DISTANCE READ FUNCTION
// =============================================
float readDistance() {
  digitalWrite(PIN_TRIG, LOW);
  delayMicroseconds(2);
  digitalWrite(PIN_TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(PIN_TRIG, LOW);

  // Timeout 30ms (~5 meters)
  long duration = pulseIn(PIN_ECHO, HIGH, 30000);

  if (duration == 0) return -1; // Not detected / timeout
  return (duration / 2.0) / 29.1;
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
  String startMessage = "✅ Water Tank Monitor System ACTIVE.\n";
  startMessage += "Water volume monitoring has started.\n";
  startMessage += "Warning threshold: distance ≥ " + String(DISTANCE_THRESHOLD_CM) + " cm from sensor.";
  bot.sendMessage(CHAT_ID, startMessage, "");
}

// =============================================
// LOOP
// =============================================
void loop() {
  unsigned long now = millis();

  // Read sensor every 1 second
  if (now - lastMeasure >= MEASURE_INTERVAL) {
    lastMeasure = now;

    float distance = readDistance();

    if (distance < 0) {
      Serial.println("⚠️ Sensor not detected. Check connection.");
      return;
    }

    Serial.print("📏 Measured distance: ");
    Serial.print(distance);
    Serial.println(" cm");

    // Check if water is low (distance >= threshold)
    if (distance >= DISTANCE_THRESHOLD_CM) {

      // Activate buzzer
      digitalWrite(PIN_BUZZER, HIGH);

      // Send Telegram notification with 5-second interval
      if (!warningActive || (now - lastSend >= SEND_INTERVAL)) {
        Serial.println("🚨 Water is low! Sending notification...");

        String message = "🚨 *Warning! Water Level Low!*\n";
        message += "📏 Measured distance: *" + String(distance, 1) + " cm*\n";
        message += "⚠️ Warning threshold: " + String(DISTANCE_THRESHOLD_CM) + " cm\n";
        message += "Please refill the water tank!";

        bot.sendMessage(CHAT_ID, message, "Markdown");
        lastSend      = now;
        warningActive = true;
      }

    } else {
      // Water is sufficient — turn off buzzer
      if (warningActive) {
        digitalWrite(PIN_BUZZER, LOW);
        Serial.println("✅ Water volume back to normal.");

        String message = "✅ *Water Volume Normal*\n";
        message += "📏 Measured distance: *" + String(distance, 1) + " cm*\n";
        message += "The water tank has been sufficiently refilled.";

        bot.sendMessage(CHAT_ID, message, "Markdown");
        warningActive = false;
      }
    }
  }
}
```

> 📝 **Before uploading, make sure you have replaced:**
> - `YOUR_WIFI_NAME` → Your WiFi SSID
> - `YOUR_WIFI_PASSWORD` → Your WiFi password
> - `YOUR_TELEGRAM_BOT_TOKEN` → Token from BotFather
> - `YOUR_CHAT_ID` → Your Telegram Chat ID

> 🔧 **Changing the Distance Warning Threshold:**
> Simply change the value in the following line in the code:
> ```cpp
> #define DISTANCE_THRESHOLD_CM  15   // Change this number as needed
> ```
> Example: if your tank height is 50 cm and you want a warning when there are only 10 cm of water left from the bottom, set the value to `40` (50 cm - 10 cm).

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
[HC-SR04 Ultrasonic Sensor]
          |
          | Send TRIG pulse → receive ECHO reflection
          | Calculate distance (cm) to water surface
          ▼
[ESP32] — Distance >= 15 cm? (water is low)
          |
          |-- YES → Activate BUZZER (local alarm)
          |          Send Telegram notification every 5 seconds
          |
          |-- NO  → Buzzer stays off
          |          If there was a previous warning:
          |          Turn off buzzer + send "Water normal"
          ▼
[Telegram Server]
          |
          ▼
[Telegram Bot] ──► [Your Chat / Phone]
```

### How the HC-SR04 Sensor Works

```
[ HC-SR04 Sensor ] ← mounted at the top of the tank
        |
        |  ↕ Measured distance (cm)
        |
  ══════════════  ← Water surface
  |              |
  |  Water Tank  |
  |              |
  ══════════════
```

1. The **TRIG** pin sends an ultrasonic pulse for 10 microseconds.
2. The sound wave travels downward and reflects off the water surface.
3. The **ECHO** pin receives the reflected wave.
4. **Distance formula:** `Distance (cm) = (ECHO Duration / 2) / 29.1`
5. The **larger** the distance value → the **farther** the water surface from the sensor → the **less** water in the tank.

### Telegram Notification Flow

1. **ESP32 connects to WiFi** — uses a WiFi connection to access the internet.
2. **`UniversalTelegramBot` library** — sends HTTPS messages to the Telegram endpoint:
   ```
   https://api.telegram.org/bot<TOKEN>/sendMessage
   ```
3. **Message sending logic:**
   - When water is **first** detected as low → message + distance value is sent immediately
   - If the low water condition **continues** → message is resent every **5 seconds**
   - When water **returns to normal** → a confirmation message is sent and the buzzer is turned off
4. Every Telegram message includes the **actual distance value (cm)** for easy remote monitoring.

### Installation & Connection Tips

- Mount the HC-SR04 sensor **exactly at the center of the top** of the tank lid, facing downward.
- Ensure the sensor is positioned **horizontally / level** for accurate readings.
- Avoid excessive water turbulence during refilling as it may affect readings.
- Use a **2.4 GHz** WiFi network (ESP32 does not support 5 GHz).
- If using a mobile hotspot, make sure mobile data is active and stable.

---

## 📐 Distance Threshold Calibration Guide

Adjust the `DISTANCE_THRESHOLD_CM` value based on the dimensions of your water tank:

```
Example: Tank Height = 60 cm
Sensor mounted at the top (distance 0 cm = tank full)

Estimated Settings:
├── Water full       : distance ±  2 cm from sensor
├── Water at 75%     : distance ± 15 cm from sensor
├── Water at 50%     : distance ± 30 cm from sensor
├── Water at 25%     : distance ± 45 cm from sensor  ← Set DISTANCE_THRESHOLD_CM = 45
└── Water almost empty : distance ± 55 cm from sensor
```

> Measure your tank's dimensions yourself and determine at what distance the warning should be triggered. Then enter that value into `DISTANCE_THRESHOLD_CM` in the program code.

---

## 📁 Project Structure

```
Water-Tank-Monitor-IoT/
│
├── README.md                      ← This documentation
└── water_tank_monitor/
    └── water_tank_monitor.ino     ← Arduino program code
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

> 💡 **Created for an IoT-based water tank volume monitoring system using ESP32, HC-SR04 Ultrasonic Sensor, Buzzer, and Telegram Bot.**
