# 🌿 Verdant — ESP32 EnviroMonitor

A real-time environmental monitoring dashboard for ESP32-based sensor setups. Tracks oxygen, CO₂, soil moisture, and light levels from your device, with a live MJPEG camera feed from an ESP32-CAM module — all in a single self-contained HTML file.

---

## Features

- **O₂ monitoring** — animated breathing ring with colour-coded alerts (normal / elevated / low)
- **CO₂ monitoring** — live ppm readout with Normal / Elevated / Danger thresholds
- **Soil moisture** — percentage gauge with descriptive status labels
- **Light level** — lux gauge with auto-scaling (lx / k lx)
- **30-reading sparkline history** for all four sensors
- **ESP32-CAM live feed** — MJPEG stream with snapshot and fullscreen support
- **Event log** — timestamped connection and sensor activity log
- **Demo mode** — generates realistic data automatically when no device is connected
- **AI-powered feedback loop** — in-app feedback form with Claude-generated responses
- **Zero dependencies** — single `.html` file, no build step, no framework

---

## Screenshots

> _Connect your ESP32 and open `esp32-dashboard.html` in any modern browser._

---

## Getting Started

### 1. Open the dashboard

Just open `esp32-dashboard.html` in your browser — no server required.

### 2. Flash your ESP32

Your ESP32 firmware needs to expose a JSON endpoint. A minimal Arduino sketch example:

```cpp
#include <WiFi.h>
#include <WebServer.h>

const char* ssid     = "YOUR_SSID";
const char* password = "YOUR_PASSWORD";

WebServer server(80);

void handleSensors() {
  // Replace these with your real sensor reads
  float o2       = 20.9;
  int   co2      = 412;
  int   moisture = 45;
  int   light    = 18500;

  String json = "{";
  json += "\"o2\":"       + String(o2, 1)  + ",";
  json += "\"co2\":"      + String(co2)    + ",";
  json += "\"moisture\":" + String(moisture) + ",";
  json += "\"light\":"    + String(light);
  json += "}";

  server.sendHeader("Access-Control-Allow-Origin", "*");
  server.send(200, "application/json", json);
}

void setup() {
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) delay(500);
  server.on("/sensors", handleSensors);
  server.begin();
}

void loop() {
  server.handleClient();
}
```

### 3. Connect the dashboard

1. Enter your ESP32's local IP address in the **ESP32 IP** field (e.g. `192.168.1.100`)
2. Press **Connect** — the dashboard will start polling every 5 seconds
3. Adjust the poll interval or endpoint path in the **Config** card if needed

---

## Sensor JSON Format

Your ESP32 endpoint (default: `GET /sensors`) must return:

```json
{
  "o2":       20.9,
  "co2":      412,
  "moisture": 45,
  "light":    18500
}
```

| Field      | Unit       | Description                        |
|------------|------------|------------------------------------|
| `o2`       | % vol      | Oxygen concentration               |
| `co2`      | ppm        | Carbon dioxide concentration       |
| `moisture` | %          | Soil moisture (0 = dry, 100 = sat) |
| `light`    | lux        | Ambient light level                |

---

## ESP32-CAM Setup

The camera panel streams MJPEG video directly from your ESP32-CAM module.

1. Flash the **CameraWebServer** example sketch (included in Arduino ESP32 board support)
2. Note your module's IP address from the serial monitor
3. In the dashboard, paste the stream URL into the CAM field:
   ```
   http://<camera-ip>:81/stream
   ```
4. Press **Load ▶**

> **Tip:** The sensor ESP32 and camera ESP32-CAM can be separate devices on the same network.

---

## Recommended Sensors

| Sensor    | Measures  | Module          |
|-----------|-----------|-----------------|
| MH-Z19B   | CO₂       | UART / PWM      |
| Grove SEN0193 | Soil moisture | Analog     |
| BH1750    | Light (lux) | I²C           |
| MQ-2 / custom | O₂   | Analog / I²C   |

---

## Configuration

All config lives in the **Config** card inside the dashboard — no code edits needed.

| Setting         | Default     | Description                          |
|-----------------|-------------|--------------------------------------|
| ESP32 IP        | 192.168.1.100 | IP address of your ESP32           |
| Poll interval   | 5 s         | How often to fetch sensor data       |
| Endpoint path   | `/sensors`  | HTTP path on your ESP32 web server   |
| CAM stream URL  | _(empty)_   | Full URL to MJPEG stream             |

---

## Feedback

The dashboard includes a built-in feedback form (click **Feedback** in the header). Submissions are processed by Claude (claude-sonnet-4-6) which generates a personalised response in real time.

Feedback categories: Sensor accuracy · Camera feed · UI/design · Charts & history · Connection issues · Feature request · Bug report

---

## Browser Support

Works in any modern browser with `fetch` and `<canvas>` support:

- Chrome / Edge 88+
- Firefox 85+
- Safari 14+

> **CORS:** Your ESP32 web server must include the `Access-Control-Allow-Origin: *` header so the browser can fetch sensor data from a different origin. See the example sketch above.

---

## Project Structure

```
esp32-dashboard.html   ← entire app (HTML + CSS + JS, single file)
README.md              ← this file
```

---

## License

MIT — free to use, modify, and deploy.
