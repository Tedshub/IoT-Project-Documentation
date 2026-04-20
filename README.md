# 🌐 3 IoT Simple Projects

A collection of three beginner-friendly IoT projects built with **ESP32**, various sensors, and **Telegram Bot** notifications. Each project is self-contained with full wiring diagrams, program code, and setup instructions.

---

## 📁 Projects Overview

### 1. 💧 Smart Water Tank Monitor
**File:** [Smart_Water_Tank_Documentation.md](Smart_Water_Tank_Documentation.md)

Monitors water volume inside a tank using an **HC-SR04 ultrasonic sensor**. When the water level drops below a defined threshold, a **buzzer** alarm triggers and a **Telegram notification** is sent automatically.

**Key Features:**
- Continuous water level monitoring (every 1 second)
- Local buzzer alarm when water is low
- Telegram alert with actual distance reading (cm)
- Repeat notification every 5 seconds if low water persists
- "Water normal" confirmation sent when tank is refilled

**Components:** ESP32 · HC-SR04 Ultrasonic Sensor · Buzzer

---

### 2. ☂️ Smart Canopy System
**File:** [Smart_Canopy_Documentation.md](Smart_Canopy_Documentation.md)

Detects rainfall using an **FC-37 rain sensor** and automatically opens or closes a canopy via an **SG-90 servo motor**. Sends real-time status updates to Telegram whenever rain starts or stops.

**Key Features:**
- Automatic canopy closing when rain is detected
- Automatic canopy opening when rain stops
- Telegram notification on every state change
- Repeat notification every 5 seconds if rain continues
- Servo angle fully adjustable to match canopy mechanics

**Components:** ESP32 · FC-37 Rain Sensor · SG-90 Servo Motor

---

### 3. 🔔 Vibration Detection System
**File:** [Vibration_Detection_Documentation.md](Vibration_Detection_Documentation.md)

Detects physical vibrations or activity using an **SW-420 vibration sensor**. Triggers a **buzzer** alarm locally and sends a **Telegram notification** as soon as vibration is detected.

**Key Features:**
- Instant vibration detection with local buzzer alert
- Immediate Telegram notification on first detection
- Repeat notification every 5 seconds while vibration continues
- "Vibration stopped" confirmation sent when activity ceases

**Components:** ESP32 · SW-420 Vibration Sensor · Buzzer

---

## 🧰 Common Hardware Requirements

All three projects share the following base requirements:

| Item | Details |
|------|---------|
| Microcontroller | ESP32 (with built-in WiFi) |
| Power Supply | 5V via USB adapter or power bank |
| IDE | Arduino IDE |
| Connectivity | 2.4 GHz WiFi network |
| Notification | Telegram Bot (via BotFather) |

---

## 💻 Common Software Requirements

| Software / Library | Purpose |
|--------------------|---------|
| Arduino IDE | Code editor and uploader |
| CP2102 USB Driver | ESP32 serial communication |
| `UniversalTelegramBot` | Sending Telegram messages |
| `ArduinoJson` | Parsing Telegram API responses |
| `WiFi` *(built-in)* | WiFi connectivity for ESP32 |
| `ESP32Servo` *(Smart Canopy only)* | Servo motor control |

---

## 📂 Repository Structure

```
3-IOT-SIMPLE-PROJECT/
│
├── README.md                             ← You are here
│
├── img/
│   ├── Image_1.jpg                       ← Smart Water Tank wiring diagram
│   ├── Image_2.jpg                       ← Smart Canopy wiring diagram
│   └── Image_3.jpg                       ← Vibration Detection wiring diagram
│
├── Smart_Water_Tank_Documentation.md     ← Project 1 documentation
├── Smart_Canopy_Documentation.md         ← Project 2 documentation
└── Vibration_Detection_Documentation.md  ← Project 3 documentation
```

---

## 🚀 Getting Started

1. **Set up Arduino IDE** — download and install from [arduino.cc](https://www.arduino.cc/en/software/)
2. **Install the CP2102 driver** for ESP32 USB communication
3. **Add the ESP32 board** to Arduino IDE (see individual project docs for tutorial links)
4. **Install required libraries** via Arduino IDE Library Manager
5. **Create a Telegram Bot** using [@BotFather](https://t.me/BotFather) and note your **Bot Token** and **Chat ID**
6. **Open the desired project**, fill in your WiFi credentials and Telegram details, then upload to ESP32

> Full step-by-step instructions are provided in each project's documentation file.

---

## 📡 How Telegram Notifications Work

All three projects share the same notification architecture:

```
[Sensor] ──► [ESP32] ──► [WiFi] ──► [Telegram API] ──► [Your Phone]
```

- ESP32 connects to your WiFi network on startup
- Sensor events trigger HTTPS requests to the Telegram Bot API
- Messages are delivered instantly to your Telegram chat
- The system sends a confirmation message on startup so you know it's online

---

> 💡 **Built for learning IoT fundamentals** — sensor integration, WiFi connectivity, and real-time remote notifications using ESP32 and Telegram Bot.
