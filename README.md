# 🌐 ESP32 IoT Project Collection

A collection of three independent IoT projects built on the **ESP32** microcontroller. Each project features real-time sensor monitoring, a local web dashboard accessible from any device on the same network, and event logging — all programmed using the Arduino IDE.

---

## 📦 Projects Overview

| # | Project | Sensor | Output | Dashboard |
|---|---------|--------|--------|-----------|
| 1 | [Smart Canopy System](#1-smart-canopy-system) | FC-37 Rain Sensor | SG-90 Servo | Web Dashboard (auto-refresh 5s) |
| 2 | [Smart Water Tank Monitor](#2-smart-water-tank-monitor) | HC-SR04 Ultrasonic | Buzzer | Web Dashboard (auto-refresh 1.2s) |
| 3 | [Vibration Detection System](#3-vibration-detection-system) | SW-420 Vibration Sensor | Buzzer | Web Interface (polling 800ms) |

---

## 1. Smart Canopy System

> 📁 `Smart-Canopy-IoT/` · Full documentation: [`Smart_Canopy_Documentation.md`](Smart_Canopy_Documentation.md)

### Description

An automatic canopy system that uses an **FC-37 rain sensor** to detect rainfall and an **SG-90 servo motor** to open or close the canopy accordingly. When rain is detected (DO pin goes LOW), the ESP32 drives the servo to the closed position. When rain stops, the servo returns to the open position. A local web dashboard allows real-time monitoring and manual override — without re-uploading the code.

### Components

| Component | Role |
|-----------|------|
| ESP32 | Main microcontroller + WiFi |
| FC-37 Rain Sensor | Detects presence of rainwater (active LOW) |
| SG-90 Servo | Opens / closes the canopy mechanically |

### Key Pin Connections

| Component | Pin | ESP32 Pin |
|-----------|-----|-----------|
| FC-37 | DO | GPIO34 |
| SG-90 | Signal | GPIO13 |
| SG-90 | VCC | 5V (VIN) |

### Dashboard Features

- Live canopy state (OPEN / CLOSED)
- Rain sensor status (CLEAR / RAIN DETECTED)
- Control mode indicator (AUTOMATIC / MANUAL)
- Manual override — open or close the canopy at will
- Configurable servo angles (0–180°) without re-uploading
- Event log (last 10 events)
- Auto-refresh every 5 seconds

### System Flow

```
[FC-37 Rain Sensor] -- LOW signal when rain detected
        ↓
[ESP32] → Servo moves to CLOSED position → Event logged
[Web Dashboard] ← accessible via browser on same network

When rain stops:
[FC-37] HIGH → [ESP32] → Servo OPEN + event logged
```

### Required Libraries

- `ESP32Servo` — install via Arduino Library Manager
- `WiFi`, `WebServer` — bundled with ESP32 board package

---

## 2. Smart Water Tank Monitor

> 📁 `Water-Tank-Monitor-IoT/` · Full documentation: [`Smart_Water_Tank_Documentation.md`](Smart_Water_Tank_Documentation.md)

### Description

A water level monitoring system that uses an **HC-SR04 ultrasonic sensor** mounted at the top of a tank, pointing downward. The ESP32 measures the distance to the water surface and classifies the level into three states: **OVERFULL**, **NORMAL**, or **LOW**. A **buzzer** activates as a local alarm when the level is too high or too low. All readings and thresholds are visible and configurable in real time through a web dashboard.

### Components

| Component | Role |
|-----------|------|
| ESP32 | Main microcontroller + WiFi |
| HC-SR04 | Ultrasonic distance sensor (5V) |
| Buzzer | Audible alarm for OVERFULL and LOW states |

### Key Pin Connections

| Component | Pin | ESP32 Pin |
|-----------|-----|-----------|
| HC-SR04 | TRIG | GPIO5 |
| HC-SR04 | ECHO | GPIO18 |
| HC-SR04 | VCC | 5V (VIN) |
| Buzzer | (+) | GPIO26 |

> ⚠️ The HC-SR04 ECHO pin outputs 5V; the ESP32 is 3.3V tolerant. A voltage divider (1kΩ + 2kΩ) on the ECHO line is recommended for long-term safety.

### State Classification

```
Sensor (top of tank)
        │
  ══════════  ← OVERFULL threshold (default: 5 cm)   → Buzzer ON
  │          │
  │  NORMAL  │                                        → Buzzer OFF
  │          │
  ══════════  ← LOW threshold (default: 25 cm)        → Buzzer ON
  │          │
  ══════════  ← tank bottom
```

**Fill % formula:** `((LOW_CM - distance) / (LOW_CM - FULL_CM)) × 100`

### Dashboard Features

- Animated SVG tank visualization with color-coded fill level
- Live distance reading (cm) and fill percentage
- State badge (NORMAL / OVERFULL / LOW / UNKNOWN)
- Configurable FULL and LOW thresholds — applied live without re-upload
- Event log (last 12 state-change events)
- Auto-refresh every 1.2 seconds

### Required Libraries

- `WiFi`, `WebServer` — bundled with ESP32 board package
- HC-SR04 controlled via built-in `pulseIn()` — no extra library needed

---

## 3. Vibration Detection System

> 📁 `Vibration-Detection-System/` · Full documentation: [`Vibration_Detection_Documentation.md`](Vibration_Detection_Documentation.md)

### Description

A vibration monitoring system that uses an **SW-420 vibration sensor** to detect tremors or physical disturbances. When a vibration is detected (DO pin goes HIGH), the ESP32 activates a **buzzer** as a local alarm and logs the event. A local web interface updates in real time, showing the current vibration status, system uptime, and a full event log of detected and cleared vibrations.

### Components

| Component | Role |
|-----------|------|
| ESP32 | Main microcontroller + WiFi |
| SW-420 Vibration Sensor | Detects vibration / tremors (active HIGH) |
| Buzzer | Audible alarm on vibration detection |

### Key Pin Connections

| Component | Pin | ESP32 Pin |
|-----------|-----|-----------|
| SW-420 | DO | GPIO34 |
| SW-420 | VCC | 3.3V |
| Buzzer | (+) | GPIO26 |

### System Flow

```
[SW-420 Sensor] -- HIGH signal when vibration occurs
        ↓
[ESP32] → Buzzer ON → Event logged → Web interface updated

When vibration stops:
[SW-420] LOW → [ESP32] → Buzzer OFF → Stop event logged
```

### Dashboard Features

- Real-time vibration status (active / standby) with visual feedback
- Ambient background pulse animation during active vibration
- Live system clock and uptime counter
- Time since last vibration event
- Event log (last 20 events with timestamps)
- Polling every 800 ms — no manual page refresh needed

### Required Libraries

- `WiFi`, `WebServer` — bundled with ESP32 board package

---

## 🛠️ Common Setup (All Projects)

All three projects share the same development environment and upload workflow.

### Software Requirements

| Tool | Link |
|------|------|
| Arduino IDE | https://www.arduino.cc/en/software/ |
| Arduino IDE Install Tutorial | https://youtu.be/lTKvZRfJRgw |
| CH340 USB Driver | https://youtu.be/ctuudlz-d0Q |
| ESP32 Board Install | https://drive.google.com/file/d/1pb8Wt_b7EgaFoNqN1fqX-U0KogYZ6rVh/view?usp=sharing |

### Upload Steps

1. Connect the ESP32 to your computer via USB.
2. In Arduino IDE: **Tools → Board → esp32 → ESP32 Dev Module**
3. **Tools → Port** → select the correct COM port.
4. Replace `YOUR_WIFI_NAME` and `YOUR_WIFI_PASSWORD` in the code.
5. Press **Ctrl+U** to upload.

> If the upload fails, hold the **BOOT** button on the ESP32 while pressing Ctrl+U.

### Accessing the Web Dashboard

1. After upload, open **Serial Monitor** (Ctrl+Shift+M) at baud rate **115200**.
2. Wait for the line: `IP Address: 192.168.x.x`
3. On any device connected to the **same 2.4 GHz WiFi network**, open a browser and navigate to that IP address.

> ⚠️ The ESP32 does **not** support 5 GHz WiFi. Always use a 2.4 GHz network or hotspot.

---

## 📁 Repository Structure

```
ESP32-IoT-Projects/
│
├── README.md                          ← This file
│
├── Smart-Canopy-IoT/
│   ├── Smart_Canopy_Documentation.md
│   ├── img/
│   │   ├── Image_2.jpg
│   │   └── web_canopy.png
│   └── smart_canopy/
│       └── smart_canopy.ino
│
├── Water-Tank-Monitor-IoT/
│   ├── Smart_Water_Tank_Documentation.md
│   ├── web_water_tank.png
│   └── water_tank_monitor/
│       └── water_tank_monitor.ino
│
└── Vibration-Detection-System/
    ├── Vibration_Detection_Documentation.md
    ├── img/
    │   ├── Image_3.jpg
    │   └── web_vibration.png
    └── vibration_detection/
        └── vibration_detection.ino
```

---

> All three projects are built with ESP32 + Arduino IDE and serve a local web interface over WiFi. No cloud service or external server is required — everything runs on the local network.
