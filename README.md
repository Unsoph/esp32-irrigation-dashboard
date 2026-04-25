# 💧 ESP32 Smart Irrigation System — MQTT Dashboard

A Wi-Fi connected smart irrigation controller built on the ESP32.  
Monitors tank water level and soil moisture in real time, controls two pumps  
via an automated state machine, and exposes a live dashboard over MQTT +  
GitHub Pages as well as a local web UI served directly from the ESP32.

---

## 🌐 Live Dashboard

**[View Dashboard →](https://YOUR_USERNAME.github.io/esp32-irrigation-dashboard/)**

> Connects to the ESP32 via MQTT over WebSocket (HiveMQ public broker).  
> No app or installation needed — works in any modern browser.

---

## ✨ Features

- 📡 **Real-time sensor data** — water level (%) + raw distance (cm), soil moisture (%)
- 🤖 **Automatic state machine** — IDLE → FILLING → READY → IRRIGATING → FAULT
- 🔧 **Manual override** — control Pump 1 & Pump 2 independently from the dashboard
- ⚙️ **Configurable thresholds** — tank low/full %, soil dry/wet % adjustable at runtime
- 🔒 **Safety lockout** — pumps auto-cut after 2 minutes of continuous run
- 💾 **NVS persistence** — thresholds survive reboots (stored in ESP32 flash)
- 🌍 **Dual dashboard** — GitHub Pages (remote) + local web UI at `http://<ESP32-IP>/`
- 📊 **Live water level chart** — last 60 readings plotted in the browser

---

## 🔧 Hardware

| Component | Details |
|---|---|
| Microcontroller | ESP32 (any variant with ADC on GPIO 34) |
| Water level sensor | HC-SR04 ultrasonic (Trigger: GPIO 18, Echo: GPIO 19) |
| Soil moisture sensor | Resistive/capacitive ADC sensor (GPIO 34) |
| Relay 1 — Fill pump | Active-LOW relay on GPIO 26 |
| Relay 2 — Irrigation pump | Active-LOW relay on GPIO 27 |
| Tank height | 30 cm (configurable via `TANK_HEIGHT_CM`) |

---

## 📦 Dependencies (Arduino IDE)

Install all via **Tools → Manage Libraries**:

| Library | Recommended Version |
|---|---|
| `AsyncTCP` (mathieucarbou fork) | ≥ 3.3.8 |
| `ESP Async WebServer` (mathieucarbou fork) | ≥ 3.7.4 |
| `PubSubClient` | ≥ 2.8 |

> ⚠️ Use the **mathieucarbou** forks, not the original `me-no-dev` versions,  
> to avoid const-qualifier compilation errors.

---

## ⚡ Quick Start

### 1. Configure Wi-Fi & MQTT

Open `code.ino` and edit the top section:

```cpp
#define WIFI_SSID     "YOUR_SSID"
#define WIFI_PASSWORD "YOUR_PASSWORD"

#define MQTT_BROKER   "broker.hivemq.com"   // free public broker
#define MQTT_PORT     1883
```

### 2. Flash to ESP32

Select your board in Arduino IDE and upload. Open Serial Monitor at **115200 baud**  
to confirm Wi-Fi connection and MQTT topics.

### 3. Open the Dashboard

- **Remote:** Visit your GitHub Pages URL (see above)  
- **Local:** Navigate to `http://<ESP32-IP>/` on your LAN

---

## 📨 MQTT Topics

| Topic | Direction | Description |
|---|---|---|
| `ritwik/data` | ESP32 → Dashboard | JSON sensor + state payload, every 2 s |
| `ritwik/control` | Dashboard → ESP32 | Commands to control pumps and settings |

### Payload — `ritwik/data` (JSON)

```json
{
  "waterLevel": 75.3,
  "tankDistCm": 7.4,
  "soilMoisture": 42.1,
  "pump1": false,
  "pump2": true,
  "state": "IRRIGATING",
  "autoMode": true,
  "tankLowPct": 20,
  "tankFullPct": 90,
  "soilDryPct": 30,
  "soilWetPct": 70,
  "irrMaxMs": 30000,
  "faultReason": ""
}
```

### Commands — `ritwik/control`

| Command payload | Action |
|---|---|
| `{"cmd":"pump1_on"}` | Turn Pump 1 ON (manual mode only) |
| `{"cmd":"pump1_off"}` | Turn Pump 1 OFF |
| `{"cmd":"pump2_on"}` | Turn Pump 2 ON (manual mode only) |
| `{"cmd":"pump2_off"}` | Turn Pump 2 OFF |
| `{"cmd":"mode_auto"}` | Switch to AUTO mode |
| `{"cmd":"mode_manual"}` | Switch to MANUAL mode |
| `{"cmd":"clear_fault"}` | Clear fault state, return to IDLE |
| `{"cmd":"threshold","tankLow":25,"tankFull":85,"soilDry":30,"soilWet":70}` | Update thresholds |

---

## 🔄 State Machine
water < tankLow%
IDLE ──────────────────────► FILLING
│
water >= tankFull%
│
▼
READY ◄──────────────────┐
│ │
soil < soilDry% && │
water >= tankFull% │
│ │
▼ │
IRRIGATING │
│ │
soil wet / timer / tank low ────────────┘
│
(any pump > 2 min)
│
▼
FAULT

---

## 🛡️ Safety Features

- **Pump safety lockout** — any pump running for >2 minutes triggers a FAULT automatically
- **Fault clear** requires explicit `clear_fault` MQTT command or browser control
- **Manual mode** stops all pumps immediately on activation
- Pumps default to **OFF** on boot (active-LOW relays held HIGH)
Eashan Jain
Shreyas Sharma
---

## 📁 File Structure
