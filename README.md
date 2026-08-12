# Automated Water Tank

An IoT-based ultrasonic water level monitoring and automatic filling system using ESP32 and Blynk IoT.

The Automated Water Tank project continuously measures tank water level with a non-contact ultrasonic sensor, classifies it into Low / Medium / Full states, and reflects that status locally via an RGB LED and buzzer while streaming live data to a Blynk dashboard — giving both on-site and remote visibility into the tank without manual checking.

## Objective

- Measure tank water level accurately using a non-contact ultrasonic sensor
- Classify the level into Low, Medium, and Full states and reflect this locally via an RGB LED
- Publish live water level and tank status to the Blynk IoT cloud over Wi-Fi
- Visualise the system remotely through a Blynk dashboard (Gauge, Label, and LED widgets)
- Signal the Full condition audibly via a double-beep buzzer alert
- Represent motor run status on the dashboard, indicating when refilling is required

## Components

| Component | Quantity / Spec |
|---|---|
| ESP32 Dev Board (WROOM-32) | 1 |
| HC-SR04 Ultrasonic Sensor | 1 |
| RGB LED (Common Cathode) | 1 |
| Buzzer (Active, 3.3V/5V) | 1 |
| Resistors (220Ω) | 3 (RGB LED channels) |
| Breadboard | 1 |
| Jumper wires (M-M / M-F) | As required |
| USB-C cable | 1 |
| Wi-Fi network (2.4 GHz) | For ESP32 connectivity |
| Blynk account + Blynk IoT Console | Free tier is sufficient |
| Water container / tank (prototype: bottle) | 1 |

## Circuit connections

| Component Pin | ESP32 Pin |
|---|---|
| HC-SR04 VCC | 5V (VIN) |
| HC-SR04 GND | GND |
| HC-SR04 TRIG | GPIO 5 |
| HC-SR04 ECHO | GPIO 18 (via voltage divider — ECHO is 5V, ESP32 GPIO is 3.3V tolerant) |
| RGB LED — Red | GPIO 25 (via 220Ω resistor) |
| RGB LED — Green | GPIO 26 (via 220Ω resistor) |
| RGB LED — Blue | GPIO 27 (via 220Ω resistor) |
| RGB LED — Common Cathode | GND |
| Buzzer (+) | GPIO 14 |
| Buzzer (−) | GND |

> ⚠️ The HC-SR04 ECHO pin outputs 5V while ESP32 GPIOs are only 3.3V tolerant. Use a resistor voltage divider (e.g. 1kΩ / 2kΩ) on the ECHO line to protect the GPIO.

## How it works

The HC-SR04 is mounted above the tank, facing the water surface. It emits a 40 kHz ultrasonic pulse from `TRIG` and measures the echo return time on `ECHO`. That time converts to distance:

```
Distance (cm) = (duration × 0.0343) / 2
```

Using the known tank height, the ESP32 converts distance into a water level percentage:

```
Water Level (%) = ((Tank Height − Measured Distance) / Tank Height) × 100
```

The level is classified into three states, each mapped to a distinct RGB LED colour:

| Distance / Water Level | Tank Status | RGB LED | Buzzer | Motor Status (V2) |
|---|---|---|---|---|
| Distance > 90% of tank height (Level ≤ 30%) — Critical/Low | LOW | Red — solid ON | ON — 2 beeps | ON (1), steady |
| Level 31–70% | MEDIUM | Blue — solid ON | OFF | ON (1), steady |
| Level 71–94% | FULL | Green — solid ON | ON — 2 beeps | OFF (0) |

When the tank is nearly empty (distance > 90% of tank height), the red LED turns solidly on and the buzzer sounds the same double-beep alert used for the Full condition, so both extremes are audibly confirmed. Motor Status (V2) is held at a fixed value rather than blinked, since `Blynk.virtualWrite()` sets a steady state rather than a flashing one.

Level, status label, and motor-status flag are written every second to Blynk virtual pins `V0`, `V1`, and `V2`, driving the dashboard widgets below.

## Blynk dashboard setup

1. Create a Template on the Blynk Console named **"Automated Water Tank"** for the ESP32 (Wi-Fi).
2. Add datastream **V0** — Integer (0–100), "Water Level" — bind to a **Gauge** widget.
3. Add datastream **V1** — String, "Tank Status" — bind to a **Label** widget (displays LOW / MEDIUM / FULL).
4. Add datastream **V2** — Integer (0/1), "Motor Status" — bind to an **LED** widget (lit while refilling is needed).
5. Copy the Template ID, Template Name, and Auth Token from the Device Info tab into your sketch.

### Widgets used

| Widget | Virtual Pin | Purpose |
|---|---|---|
| Gauge | V0 | Live water level as a percentage (0–100) |
| Label | V1 | Tank status text: LOW / MEDIUM / FULL |
| LED | V2 | Motor status — lit while the tank requires refilling |

## Required libraries

- **Blynk** — search "Blynk" by Volodymyr Shymanskyy in the Arduino Library Manager
- **ESP32 board package** — install via Boards Manager: "esp32" by Espressif Systems
- **WiFi.h** — bundled with the ESP32 board package

## Getting started

1. Install the Arduino IDE and add the ESP32 board package via Boards Manager.
2. Install the Blynk library via Library Manager.
3. Create the Blynk template and datastreams as described above, and note your Template ID, Template Name, and Auth Token.
4. Wire the hardware per the circuit table, adding a voltage divider on the HC-SR04 ECHO line.
5. In your sketch, set your Wi-Fi SSID/password and Blynk credentials — **don't hardcode and commit real credentials**; use a separate `secrets.h` excluded via `.gitignore` instead.
6. Flash the sketch to the ESP32 and confirm Wi-Fi connects over Serial Monitor.
7. Open the Blynk dashboard and confirm the Gauge, Label, and LED widgets update as the water level changes.

## Advantages

- Non-contact sensing avoids corrosion and electrical risk from submerged probes
- Live remote monitoring via the Blynk dashboard from anywhere with Wi-Fi/internet
- Simple three-colour RGB status indication, intuitive at a glance
- Audible confirmation (double beep) at both the full and critical-low extremes
- Compact circuit — no display hardware required, keeping cost and wiring minimal

## Limitations

- Motor status is currently derived from level thresholds only; no physical motor/pump driver is wired in this build
- Requires a stable 2.4 GHz Wi-Fi connection — ESP32 does not support 5 GHz networks
- Ultrasonic accuracy can be affected by turbulence, foam, or vapour on the water surface
- Wi-Fi credentials and the Blynk auth token must be kept out of version control

## Applications

- Domestic overhead/underground water tank monitoring
- Remote farm or agricultural reservoir monitoring
- Industrial storage tank status tracking with cloud visibility
- Educational demonstration of ESP32 + Blynk IoT dashboard integration

## Future scope

- Add a motor/pump driver to close the loop into a full auto-fill system
- Add historical data logging in Blynk to analyze usage trends
- Move to deep-sleep duty cycling for battery-powered deployments

## Author

Sk imran — B.Tech, Electronics and Communication Engineering

