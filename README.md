# Smart-IoT-Monitoring-and-Remote-Access

Real-Time Soil Moisture, Temperature & Humidity Monitoring Using ESP32, MQTT, Node-RED and Tailscale VPN
<img width="1536" height="1024" alt="IoT Monitoring System Architecture" src="https://github.com/user-attachments/assets/9bd91f55-7655-468b-89a6-a14faf5e36a9" />

# Ecoprosus IoT Monitoring System

### Real-Time Soil Moisture, Temperature & Humidity Monitoring Using ESP32, MQTT, Node-RED and Tailscale VPN

![Project](https://img.shields.io/badge/Project-Ecoprosus%20IoT-0A84FF)
![Platform](https://img.shields.io/badge/Platform-ESP32-E7352C)
![Protocol](https://img.shields.io/badge/Protocol-MQTT-6600CC)
![Dashboard](https://img.shields.io/badge/Dashboard-Node--RED-BE3030)
![Remote Access](https://img.shields.io/badge/Remote%20Access-Tailscale-00A4A6)
![Status](https://img.shields.io/badge/Status-Development-orange)

## 1. Project Overview

The **Ecoprosus IoT Monitoring System** is an IoT-based environmental monitoring platform designed to collect sensor data from ESP32 devices, transmit it over Wi-Fi using MQTT, process the incoming data on a Raspberry Pi, and display real-time measurements through a Node-RED Dashboard.

The system consists of two sensor nodes:

- **ESP32 Node 01:** Measures soil moisture using an FC-28 soil moisture sensor.
- **ESP32 Node 02:** Measures temperature and relative humidity using a DHT11 sensor.

A Raspberry Pi hosts the Mosquitto MQTT broker and Node-RED dashboard environment. Tailscale provides a secure private-network path for authorized remote access to the monitoring dashboard from a laptop or mobile device.

The architecture is designed to support centralized monitoring, modular sensor integration, remote accessibility, and future expansion to additional IoT devices.

## 2. System Architecture

![Ecoprosus IoT Monitoring System Architecture](docs/images/ecoprosus-iot-architecture.png)

**Architecture flow:**

1. ESP32 sensor nodes collect environmental measurements.
2. Sensor data is transmitted over Wi-Fi using MQTT.
3. The Mosquitto MQTT broker, hosted on the Raspberry Pi, receives and distributes messages.
4. Node-RED subscribes to the MQTT topics, processes the data, and updates the dashboard.
5. Tailscale provides secure remote network access to the authorized dashboard endpoint.
6. Users view the monitoring dashboard from a laptop or mobile browser.

> **Important:** Tailscale provides private network connectivity; it does not automatically publish or host the dashboard. The Raspberry Pi, Node-RED service, firewall rules, and Tailscale access permissions must be configured correctly.

## 3. Key Features

- Real-time soil moisture monitoring.
- Temperature and humidity monitoring.
- ESP32-based wireless sensor nodes.
- MQTT-based device-to-server communication.
- Raspberry Pi-based edge processing.
- Mosquitto MQTT broker integration.
- Node-RED Dashboard 2.0 visualization.
- Live sensor gauges, charts, and status indicators.
- OLED display support on sensor nodes.
- Red transmission (TX) LED and green acknowledgement (ACK) LED indicators.
- Secure remote access using Tailscale VPN.
- Modular architecture for future sensor expansion.
- Centralized monitoring from laptop and mobile devices.

## 4. Hardware Requirements

| Component | Quantity | Purpose |
|---|---:|---|
| ESP32 DevKit development board | 2 | Wireless sensor nodes |
| FC-28 soil moisture sensor/module | 1 | Soil moisture measurement |
| DHT11 sensor/module | 1 | Temperature and humidity measurement |
| SSD1306 OLED, 128 × 64, I²C | Up to 2 | Local sensor data display |
| Raspberry Pi 4 Model B | 1 | MQTT broker and dashboard host |
| microSD card | 1 | Raspberry Pi operating system and applications |
| Red LEDs | 2 | TX/status indication |
| Green LEDs | 2 | ACK indication |
| 220–330 Ω resistors | 4 | LED current limiting |
| Breadboard and jumper wires | As required | Circuit connections |
| Regulated power supplies | As required | Power for the nodes and Raspberry Pi |
| Wi-Fi router/access point | 1 | Local network connectivity |
| Laptop or mobile device | 1 or more | Dashboard access |

**Hardware notes:**

- Confirm the FC-28 module's supply-voltage requirements before connecting it.
- ESP32 GPIO inputs are not 5 V tolerant. Ensure sensor analog and digital outputs never exceed the ESP32's permitted input voltage.
- Check the OLED module's supply requirements and I²C voltage levels before connecting it.
- Use a suitable regulated power supply for each device.

## 5. Software and Technologies

| Technology | Role |
|---|---|
| Arduino IDE | ESP32 firmware development and uploading |
| ESP32 Arduino Core | Firmware platform |
| Wi-Fi | Wireless sensor-node connectivity |
| MQTT | Lightweight messaging protocol |
| Mosquitto | MQTT broker |
| Raspberry Pi OS | Server operating system |
| Node-RED | Data processing and automation |
| Node-RED Dashboard 2.0 | Sensor visualization |
| Tailscale | Secure private remote connectivity |
| WireGuard technology | Encrypted networking used by Tailscale |
| NTP | Network time synchronization |

The exact software versions should be recorded in the repository after installation and testing.

## 6. High-Level Data Flow

```text
FC-28 Soil Sensor
        |
        v
ESP32 Node 01
        |
        | MQTT over Wi-Fi
        v
Mosquitto MQTT Broker
(Raspberry Pi)
        |
        | MQTT Subscribe
        v
Node-RED Flow
        |
        v
Dashboard 2.0
        |
        v
Tailscale Private Network
        |
        v
Laptop / Mobile Browser


DHT11 Temperature & Humidity Sensor
        |
        v
ESP32 Node 02
        |
        | MQTT over Wi-Fi
        v
Mosquitto MQTT Broker
        |
        v
Node-RED Dashboard
```

Both sensor nodes communicate with the same MQTT broker. Node-RED consumes messages from the relevant topics and presents the available measurements through the dashboard.

## 7. ESP32 Node 01 — Soil Moisture Monitoring

### 7.1 Description

ESP32 Node 01 reads the analog output from an FC-28 soil moisture sensor and converts the ADC measurement into an estimated soil moisture percentage.

The node can publish measurements to MQTT, display information on an SSD1306 OLED, and use status LEDs to indicate transmission and acknowledgement events.

### 7.2 Pin Connections

| Component | Pin | ESP32 GPIO |
|---|---|---|
| FC-28 module | AO | GPIO 34 |
| FC-28 module | DO | GPIO 25 |
| FC-28 module | VCC | Compatible supply |
| FC-28 module | GND | GND |
| SSD1306 OLED | SDA | GPIO 21 |
| SSD1306 OLED | SCL | GPIO 22 |
| SSD1306 OLED | VCC | Compatible supply |
| SSD1306 OLED | GND | GND |
| Red TX LED | Anode through resistor | GPIO 18 |
| Green ACK LED | Anode through resistor | GPIO 23 |
| Both LEDs | Cathode | GND |

Use a separate current-limiting resistor for each LED. A typical starting value is 220–330 Ω.

**Electrical caution:** FC-28 AO and DO outputs must remain within the ESP32's input-voltage limits. The FC-28 module's digital output may use a higher voltage depending on its power supply and design; verify this before connecting DO to GPIO 25.

### 7.3 Soil Moisture Calibration

The firmware uses calibrated dry and wet ADC readings to estimate soil moisture.

Example calibration constants:

```cpp
#define WET_VALUE 1967
#define DRY_VALUE 4095
```

The percentage calculation is:

```cpp
moisturePercent =
    ((DRY_VALUE - adcValue) * 100.0) /
    (DRY_VALUE - WET_VALUE);
```

The result should be constrained to the range of 0–100%.

**Calibration procedure:**

1. Record the ADC value under the selected dry reference condition.
2. Record the ADC value under the selected wet reference condition.
3. Update the calibration constants in the firmware.
4. Verify the readings against the actual soil conditions.
5. Repeat the calibration if the sensor, soil type, installation depth, or supply conditions change.

The percentage is an estimate based on calibration, not a universal measurement of volumetric water content. Soil type and probe characteristics affect the results.

## 8. ESP32 Node 02 — Temperature and Humidity Monitoring

### 8.1 Description

ESP32 Node 02 reads temperature and relative humidity from a DHT11 sensor and publishes the measurements to the MQTT broker.

An optional SSD1306 OLED can display the local measurements and device status.

### 8.2 Pin Connections

The following table describes the intended connection for the current hardware configuration. Confirm the DHT11 GPIO assignment against the actual firmware before wiring.

| Component | Pin | ESP32 GPIO |
|---|---|---|
| DHT11 module | DATA | GPIO 19 |
| DHT11 module | VCC | Compatible supply |
| DHT11 module | GND | GND |
| SSD1306 OLED | SDA | GPIO 21 |
| SSD1306 OLED | SCL | GPIO 22 |
| SSD1306 OLED | VCC | Compatible supply |
| SSD1306 OLED | GND | GND |
| Red TX LED | Anode through resistor | GPIO 18 |
| Green ACK LED | Anode through resistor | GPIO 23 |
| Both LEDs | Cathode | GND |

If using a bare DHT11 sensor instead of a module, a suitable DATA pull-up resistor may be required. Follow the sensor's datasheet and the module's wiring instructions.

## 9. LED Status Indication

### Red TX LED — GPIO 18

The red LED indicates the firmware's transmission/status activity.

For the current Node 01 behavior, the double-blink sequence is:

1. LED ON for approximately 300 ms.
2. LED OFF for approximately 200 ms.
3. LED ON for approximately 300 ms.
4. LED OFF before the next sequence.

Confirm the exact timing and trigger conditions in the firmware before treating this as a definitive indication of successful MQTT publication.

### Green ACK LED — GPIO 23

The green LED indicates that the firmware has received a valid acknowledgement message.

For Node 01, the documented ACK topic is:

```text
ecoprosus/esp32-01/ack
```

When the firmware receives the expected `ACK` payload, it performs three LED blinks.

> A blink is a firmware-defined event, not proof that the dashboard has updated or that the sensor measurement is correct.

## 10. MQTT Communication

### 10.1 MQTT Broker

The Mosquitto MQTT broker runs on the Raspberry Pi.

Example development configuration:

```text
Broker host: 172.16.0.132
Broker port: 1883
Protocol: MQTT
```

The IP address is an example from the current development setup. Replace it with the Raspberry Pi's actual reachable address in your environment.

For production deployments, use authenticated MQTT connections and TLS where appropriate. Do not expose an anonymous MQTT listener directly to the public internet.

### 10.2 MQTT Topics

| Topic | Purpose |
|---|---|
| `ecoprosus/esp32-01/soil/moisture` | Node 01 soil moisture measurements |
| `ecoprosus/esp32-01/ack` | Node 01 acknowledgement messages |
| `ecoprosus/esp32-01/status` | Node 01 online/offline status |
| `ecoprosus/esp32-02/temperature/humidity` | Node 02 temperature and humidity measurements |

These topics reflect the current project naming convention. Verify them against the firmware and Node-RED flow before deployment.

### 10.3 Example Soil Moisture Payload

Node 01 uses a JSON message structure similar to:

```json
{
  "device_id": "ESP32_Node_01",
  "timestamp": "2026-10-22T10:00:00Z",
  "readings": {
    "soil_moisture_pct": 48,
    "soil_moisture_raw": 2700
  },
  "status": {
    "sensor": "connected",
    "reading_valid": true
  }
}
```

**Note:** This is an illustrative example of the payload structure, not a captured live message. The actual field values and timestamp format depend on the running firmware.

### 10.4 Publishing Interval

The current Node 01 configuration publishes sensor readings approximately every 30 seconds.

A production implementation should validate sensor readings before publishing and set `reading_valid` according to the actual validation result. Do not mark readings as valid unconditionally.

### 10.5 Online and Offline Status

The firmware can publish an `online` status and configure an MQTT Last Will message with an `offline` status.

The Last Will is published by the broker when the client disconnects unexpectedly, subject to MQTT session and connection behavior. It is not a guarantee of physical sensor health.

## 11. Raspberry Pi Setup

The Raspberry Pi hosts the Mosquitto broker and Node-RED dashboard.

### 11.1 Update the System

```bash
sudo apt update
sudo apt upgrade -y
```

### 11.2 Install Mosquitto

```bash
sudo apt install mosquitto mosquitto-clients -y
```

Enable and start the service:

```bash
sudo systemctl enable mosquitto
sudo systemctl start mosquitto
```

Check the service:

```bash
sudo systemctl status mosquitto
```

### 11.3 Test MQTT Locally

Open one terminal on the Raspberry Pi:

```bash
mosquitto_sub -h localhost -t "ecoprosus/test" -v
```

Open another terminal:

```bash
mosquitto_pub -h localhost -t "ecoprosus/test" -m "Hello Ecoprosus"
```

The subscriber should display the published test message.

If authentication is enabled, supply the appropriate credentials or client configuration instead of using anonymous access.

### 11.4 Configure Remote MQTT Access

The broker must listen on an address reachable by the ESP32 nodes.

Before enabling remote access:

- Restrict access to trusted local network devices.
- Configure authentication and appropriate topic permissions.
- Use TLS if the network or deployment requirements call for it.
- Avoid exposing port 1883 to the public internet.
- Verify firewall rules and broker listener configuration.

Do not assume that a working local MQTT test proves that remote ESP32 connections are correctly configured.

## 12. Node-RED Installation and Dashboard Setup

Install Node-RED using the official installation procedure appropriate for the Raspberry Pi operating system.

Official resources:

- Node-RED: https://nodered.org/
- Node-RED documentation: https://nodered.org/docs/
- Node-RED Dashboard 2.0: https://dashboard.flowfuse.com/

### 12.1 Install Dashboard 2.0

In the Node-RED user directory, install the Dashboard 2.0 package through the Node-RED palette manager or the documented package installation method.

Package name:

```text
@flowfuse/node-red-dashboard
```

Restart Node-RED if required after installation.

### 12.2 Create the Dashboard

Create a dashboard containing the relevant sensor pages and groups.

Suggested layout:

```text
Ecoprosus IoT Dashboard
|
+-- Overview
|
+-- Sensor Monitoring
|   |
|   +-- ESP32 Node 01 - Soil Moisture
|   |   +-- Soil Moisture Gauge
|   |   +-- Soil Moisture Chart
|   |   +-- Raw ADC Value
|   |   +-- Device Status
|   |
|   +-- ESP32 Node 02 - Environment
|       +-- Temperature Gauge
|       +-- Humidity Gauge
|       +-- Temperature Chart
|       +-- Humidity Chart
|
+-- Device Status
    +-- Last Update
    +-- MQTT Connectivity
```

The page names and structure above are recommendations; use the actual paths and groups defined in your Node-RED flow.

### 12.3 Configure MQTT Input Nodes

For each sensor, create an MQTT-in node and configure it with:

- The Raspberry Pi MQTT broker address.
- The appropriate broker port and authentication.
- The matching sensor topic.
- The required output format for the payload.

Use a JSON parsing node if the MQTT-in node returns the payload as a string.

### 12.4 Process Sensor Data

For Node 01, extract:

```text
msg.payload.device_id
msg.payload.timestamp
msg.payload.readings.soil_moisture_pct
msg.payload.readings.soil_moisture_raw
msg.payload.status.sensor
msg.payload.status.reading_valid
```

For Node 02, parse the actual temperature and humidity field names emitted by the firmware.

Ensure that the dashboard receives numeric values for gauges and charts. Validate missing, malformed, or stale messages before updating the display.

### 12.5 Configure Visualizations

Recommended dashboard widgets:

- Gauge for soil moisture percentage.
- Chart for historical soil moisture readings.
- Gauges for temperature and humidity.
- Charts for temperature and humidity trends.
- Table for recent readings.
- Text or status indicators for device connectivity.
- Last-update timestamp for each node.

Use meaningful ranges, units, labels, and alert thresholds appropriate to the actual sensors and monitoring requirements.

### 12.6 Deploy and Verify

1. Deploy the Node-RED flow.
2. Confirm the MQTT-in nodes are connected.
3. Publish a test message to each sensor topic.
4. Verify that the expected gauge or chart changes.
5. Verify that invalid or missing values do not corrupt the dashboard.
6. Confirm that the dashboard works on the local network before configuring remote access.

## 13. Tailscale Remote Access

Tailscale enables authorized devices to connect over a private network without requiring the dashboard to be exposed directly to the public internet.

Official documentation: https://tailscale.com/kb/

### Setup Outline

1. Install Tailscale on the Raspberry Pi using the official instructions.
2. Install Tailscale on the laptop or mobile device used for remote monitoring.
3. Sign in to the same authorized Tailscale network.
4. Verify that the Raspberry Pi appears as an available device.
5. Confirm that the dashboard service is running on the Raspberry Pi.
6. Access the dashboard through the Raspberry Pi's Tailscale IP address or MagicDNS name, using the configured Node-RED dashboard port and path.
7. Apply appropriate access controls and firewall rules.

Example URL pattern:

```text
http://<raspberry-pi-tailscale-ip>:1880/<dashboard-path>
```

Replace the placeholders with the actual Tailscale address, port, and dashboard path.

For example, a Node-RED Dashboard 2.0 application may use a path configured in the dashboard settings. Do not assume that every installation uses the same path.

**Security recommendations:**

- Restrict access to trusted users and devices.
- Protect Node-RED editor access with suitable authentication and network controls.
- Do not publish Tailscale credentials, authentication keys, or private device details in the repository.
- Use HTTPS or another suitable secure transport when required by the deployment.
- Keep the Raspberry Pi, Node-RED, and Tailscale software updated.

## 14. Repository Structure

A recommended GitHub repository layout is:

```text
ecoprosus-iot-monitoring/
|
+-- README.md
|
+-- firmware/
|   |
|   +-- esp32-node-01-soil-moisture/
|   |   +-- esp32-node-01-soil-moisture.ino
|   |
|   +-- esp32-node-02-temperature-humidity/
|       +-- esp32-node-02-temperature-humidity.ino
|
+-- node-red/
|   +-- flows.json
|   +-- README.md
|
+-- docs/
|   |
|   +-- images/
|   |   +-- ecoprosus-iot-architecture.png
|   |   +-- node-01-wiring.png
|   |   +-- node-02-wiring.png
|   |   +-- dashboard-overview.png
|   |
|   +-- hardware-connections.md
|   +-- mqtt-topics.md
|   +-- raspberry-pi-setup.md
|   +-- troubleshooting.md
|
+-- .gitignore
+-- LICENSE
```

Only include files that exist in your repository. Export Node-RED flows using the Node-RED editor and review them for credentials, tokens, internal addresses, and other sensitive configuration before committing them.

## 15. Installation and Commissioning Checklist

### Hardware

- [ ] ESP32 Node 01 powers on correctly.
- [ ] FC-28 analog and digital outputs are connected safely.
- [ ] ESP32 Node 02 powers on correctly.
- [ ] DHT11 provides plausible measurements.
- [ ] OLED displays operate correctly, if installed.
- [ ] TX and ACK LEDs operate according to the firmware.
- [ ] All required grounds and power connections are correct.

### Firmware

- [ ] Correct Wi-Fi credentials are configured locally.
- [ ] Correct MQTT broker address and port are configured.
- [ ] Device IDs and topic names match the Node-RED configuration.
- [ ] Soil moisture calibration is verified.
- [ ] JSON messages contain the expected fields.
- [ ] Timestamp synchronization is working.
- [ ] MQTT connection and reconnection behavior is tested.

### Raspberry Pi

- [ ] Mosquitto service is running.
- [ ] MQTT publish/subscribe test succeeds.
- [ ] Node-RED service is running.
- [ ] Dashboard 2.0 is installed.
- [ ] MQTT-in nodes receive live messages.
- [ ] Dashboard gauges and charts update correctly.

### Remote Monitoring

- [ ] Tailscale is connected on the Raspberry Pi.
- [ ] The remote device is authorized.
- [ ] Dashboard access works through the private network.
- [ ] Unauthorized access is restricted.
- [ ] No MQTT port or dashboard service is unnecessarily exposed to the public internet.

## 16. Testing and Validation

Use the following tests to verify the system end to end.

| Test | Procedure | Expected result |
|---|---|---|
| Sensor reading | Change soil conditions or inspect DHT11 readings | Sensor values respond plausibly |
| MQTT publishing | Subscribe to the relevant topic | Messages appear at the configured interval |
| JSON parsing | Inspect received payload | Expected fields are present |
| Dashboard update | Publish a test payload | Matching widget updates |
| ACK behavior | Publish the expected acknowledgement | ACK LED follows the firmware sequence |
| Device disconnect | Disconnect a node unexpectedly | Broker status behavior matches the configured Last Will |
| Wi-Fi recovery | Restore connectivity after interruption | Device reconnects according to firmware logic |
| Remote access | Open dashboard through Tailscale | Authorized remote device reaches the dashboard |

Do not treat the system as fully validated until each test has been performed and recorded on the actual hardware.

## 17. Troubleshooting

### ESP32 does not connect to Wi-Fi

- Confirm the SSID and password.
- Verify that the network is compatible with the ESP32's Wi-Fi requirements.
- Check serial output for connection errors.
- Confirm signal strength and router availability.

### MQTT connection fails

- Verify the broker IP address and port.
- Check Mosquitto service status.
- Confirm that the broker listener is reachable from the ESP32.
- Verify credentials, topic permissions, and firewall rules.
- Subscribe to the expected topic to confirm whether messages arrive.

### Sensor values are incorrect

- Check sensor power, ground, and signal connections.
- Verify the ESP32 GPIO assignments.
- For FC-28, check AO voltage and ADC readings.
- Repeat soil moisture calibration.
- For DHT11, verify the DATA pin and sensor library configuration.
- Reject invalid readings rather than displaying misleading values.

### OLED does not display data

- Verify SDA and SCL wiring.
- Confirm the I²C address, commonly `0x3C` for the configured OLED.
- Confirm the supply voltage.
- Run an I²C scanner if required.
- Check the display initialization and library configuration.

### Node-RED dashboard does not update

- Verify that the MQTT-in node is connected.
- Confirm the topic matches the ESP32 firmware.
- Check whether the MQTT payload is a string or parsed JSON object.
- Verify the JSON field names and numeric conversions.
- Inspect the Node-RED debug sidebar for errors.

### Tailscale remote access fails

- Confirm that both devices are connected to the authorized Tailscale network.
- Verify the Raspberry Pi's Tailscale IP or MagicDNS name.
- Confirm that Node-RED is listening on a reachable interface.
- Check the dashboard path, port, and firewall rules.
- Review Tailscale access-control policies.

## 18. Security Considerations

Security is an essential part of an IoT deployment.

- Never commit Wi-Fi passwords, MQTT credentials, API keys, or Tailscale authentication keys to GitHub.
- Keep secrets in local configuration files excluded by `.gitignore`, or use an appropriate secrets-management approach.
- If credentials have already been committed, rotate them; deleting them from the latest version alone may not remove them from Git history.
- Use MQTT authentication and topic-level permissions.
- Restrict broker and dashboard access to authorized devices.
- Do not expose an unauthenticated MQTT broker to the public internet.
- Protect Node-RED editor access.
- Keep operating systems and packages updated.
- Validate incoming MQTT data before using it in dashboards or automation.
- Use appropriate TLS and access-control measures for the deployment environment.

Example `.gitignore` entries:

```gitignore
# Local credentials and secrets
secrets.h
credentials.json
.env

# Local Arduino configuration
arduino_secrets.h

# Temporary and generated files
*.log
.DS_Store
```

Adapt the exclusions to the actual project files. Do not commit real credentials in example configuration files.

## 19. Known Limitations

- FC-28 readings depend on soil conditions, probe construction, calibration, and sensor placement.
- DHT11 measurements have limited accuracy and response performance compared with more advanced environmental sensors.
- Wi-Fi availability affects sensor connectivity.
- MQTT delivery depends on broker availability and the configured client/session behavior.
- Dashboard freshness depends on the arrival and validation of new messages.
- A device's online status does not necessarily prove that its sensor is functioning correctly.
- Remote monitoring depends on the Raspberry Pi, Node-RED service, and Tailscale connectivity being operational.
- The example configuration values in this README must be verified against the actual firmware and deployed services.

## 20. Future Enhancements

Potential improvements include:

- Add more ESP32 sensor nodes.
- Integrate additional environmental sensors.
- Store measurements in a time-series database.
- Add historical reporting and downloadable CSV data.
- Implement configurable high/low threshold alerts.
- Add sensor-health monitoring and stale-data detection.
- Improve MQTT security with TLS and per-device credentials.
- Add firmware-over-the-air updates.
- Add device registration and centralized configuration.
- Add automated backups for Node-RED flows and Raspberry Pi configuration.
- Build a deployment and recovery procedure for production environments.

## 21. Contributing

Contributions and improvements are welcome.

1. Fork the repository.
2. Create a feature branch.
3. Make and test your changes.
4. Update the relevant documentation.
5. Submit a pull request describing the changes.

Please avoid committing private network credentials, secrets, or personally identifying information.

## 22. License

Choose a suitable open-source license before publishing the repository. If you intend to use the MIT License, add a `LICENSE` file containing the standard MIT License text and your copyright details.

Do not claim that the repository is MIT-licensed until that license has actually been added.

## 23. Author and Project Information

**Project:** Ecoprosus IoT Monitoring System  
**Category:** Internet of Things (IoT), Embedded Systems, Environmental Monitoring  
**Hardware:** ESP32, FC-28 Soil Moisture Sensor, DHT11, Raspberry Pi  
**Communication:** Wi-Fi and MQTT  
**Visualization:** Node-RED Dashboard 2.0  
**Remote Connectivity:** Tailscale VPN

---

*Ecoprosus IoT Monitoring System — connecting sensor nodes, edge computing, real-time visualization, and secure remote monitoring in one modular architecture.*



