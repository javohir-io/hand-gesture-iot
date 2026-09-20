# ✋ Hand Gesture Controlled IoT System

**Contactless smart-device control using Computer Vision, IoT, and Cloud-hosted MQTT**

[![Python](https://img.shields.io/badge/Python-3.11%2B-blue)](https://www.python.org/)
[![OpenCV](https://img.shields.io/badge/OpenCV-CV-green)](https://opencv.org/)
[![MediaPipe](https://img.shields.io/badge/MediaPipe-Hand%20Tracking-orange)](https://developers.google.com/mediapipe)
[![ESP32](https://img.shields.io/badge/ESP32-Microcontroller-red)](https://www.espressif.com/)
[![MQTT](https://img.shields.io/badge/MQTT-Mosquitto-purple)](https://mosquitto.org/)
[![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED)](https://www.docker.com/)
[![GCP](https://img.shields.io/badge/Cloud-GCP-4285F4)](https://cloud.google.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 📖 Overview

This project implements a **contactless hand-gesture interface** for controlling physical devices in real time. A webcam feed is processed locally using **MediaPipe** hand-landmark detection; recognized gestures are translated into **MQTT** messages and published to a **cloud-hosted Mosquitto broker**, deployed via **Docker** on a **Google Cloud Platform (GCP)** virtual machine. An **ESP32** microcontroller subscribes to the relevant topics and drives five independently addressable LEDs accordingly.

The result is a low-latency, privacy-respecting, and scalable proof of concept for gesture-driven smart environments and assistive technology.

---

## 🧭 Table of Contents

- [Key Features](#-key-features)
- [System Architecture](#-system-architecture)
- [Tech Stack](#-tech-stack)
- [Hardware Requirements](#-hardware-requirements)
- [Project Structure](#-project-structure)
- [Getting Started](#-getting-started)
  - [1. MQTT Broker (GCP + Docker)](#1-mqtt-broker-gcp--docker)
  - [2. ESP32 Firmware](#2-esp32-firmware)
  - [3. Python Gesture Client](#3-python-gesture-client)
- [Supported Gestures](#-supported-gestures)
- [Configuration](#-configuration)
- [Troubleshooting](#-troubleshooting)
- [Roadmap](#-roadmap)
- [Ethical & Social Impact](#-ethical--social-impact)
- [Contributing](#-contributing)
- [License](#-license)
- [Author](#-author)

---

## 🚀 Key Features

- 🖐️ **Real-time gesture recognition** powered by MediaPipe's hand-landmark model
- 💡 **Independent control** of 5 LEDs (Thumb, Index, Middle, Ring, Pinky)
- 🔄 **Global gestures** to switch all LEDs ON or OFF simultaneously
- 🌐 **MQTT-based messaging** between the Python client and ESP32
- ☁️ **Cloud-hosted broker** — Mosquitto running in a Docker container on a GCP VM, reachable from anywhere
- 🧱 **Modular architecture** — easy to extend with new gestures, devices, or topics
- 🔒 **Local video processing** — no camera frames ever leave the client machine

---

## 🏗️ System Architecture

```
┌─────────────┐     ┌────────────────┐     ┌──────────────────┐     ┌─────────────┐
│   Webcam    │────▶│  MediaPipe +   │────▶│   MQTT Client     │────▶│   Mosquitto  │
│ (video feed)│     │  OpenCV (CV)   │     │ (paho-mqtt, pub)  │     │ Broker (GCP  │
└─────────────┘     └────────────────┘     └──────────────────┘     │  + Docker)   │
                                                                       └──────┬──────┘
                                                                              │ subscribe
                                                                              ▼
                                                                      ┌───────────────┐
                                                                      │     ESP32      │
                                                                      │ (WiFi + GPIO)  │
                                                                      └───────┬────────┘
                                                                              │
                                                                              ▼
                                                                     💡 5x LEDs (GPIO)
```

**Flow:**
1. The webcam captures a live video stream.
2. MediaPipe extracts 21 hand landmarks per detected hand, in real time.
3. A Python script interprets finger positions into discrete gestures.
4. The client publishes an MQTT message per gesture to the corresponding topic.
5. The ESP32, subscribed to those topics, toggles the matching GPIO pin / LED.
6. All communication is brokered by Mosquitto, running in Docker on a GCP VM, making the system accessible from any network.

---

## 🛠️ Tech Stack

| Layer            | Technology                                  |
| ----------------- | -------------------------------------------- |
| Computer Vision  | Python 3.11+, OpenCV, MediaPipe             |
| Messaging        | MQTT (paho-mqtt), Eclipse Mosquitto         |
| Firmware         | ESP32 (Arduino IDE / C++)                    |
| Infrastructure   | Docker, Google Cloud Platform (Compute Engine) |
| Circuit Design   | Fritzing                                     |

---

## 🔩 Hardware Requirements

- ESP32 development board
- 5x LEDs (Yellow, Red, Green, Blue, White) + current-limiting resistors
- Breadboard and jumper wires
- USB webcam (or built-in laptop camera)
- A computer with Python 3.11+ installed
- Stable Wi-Fi network shared by the ESP32 and the host running the Python client

---

## 📁 Project Structure

```
hand-gesture-iot/
├── gesture_control_mqtt.py     # Python client: CV + gesture logic + MQTT publisher
├── firmware/
│   └── esp32_led_control.ino   # ESP32 sketch: MQTT subscriber + GPIO control
├── docker/
│   └── docker-compose.yml      # Mosquitto broker deployment
├── docs/
│   └── circuit_diagram.fzz     # Fritzing circuit design
├── requirements.txt
├── LICENSE
└── README.md
```

---

## 🖥️ Getting Started

### 1. MQTT Broker (GCP + Docker)

**a. Provision a GCP VM**
Create an Ubuntu-based Compute Engine instance and open inbound TCP port `1883` in the firewall rules (and `9001` if using MQTT over WebSockets).

**b. Install Docker**

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable --now docker
```

**c. Run the Mosquitto broker**

```bash
docker run -d --name mosquitto \
  -p 1883:1883 \
  -v mosquitto_data:/mosquitto/data \
  -v mosquitto_log:/mosquitto/log \
  eclipse-mosquitto
```

> 💡 For production use, mount a custom `mosquitto.conf` with authentication (username/password or TLS certificates) enabled rather than running an open broker.

Your broker is now reachable at `<GCP_VM_EXTERNAL_IP>:1883`.

---

### 2. ESP32 Firmware

1. Open `firmware/esp32_led_control.ino` in the Arduino IDE.
2. Install the required libraries: `WiFi.h`, `PubSubClient.h`.
3. Update your Wi-Fi credentials and broker IP:

```cpp
const char* ssid       = "YOUR_WIFI_SSID";
const char* password   = "YOUR_WIFI_PASSWORD";
const char* mqtt_server = "your_gcp_vm_ip";
```

4. Wire the 5 LEDs to their designated GPIO pins (see `docs/circuit_diagram.fzz`).
5. Flash the sketch to the ESP32 and open the Serial Monitor to confirm a successful Wi-Fi and MQTT connection.

---

### 3. Python Gesture Client

**a. Install dependencies**

```bash
pip install -r requirements.txt
```

or manually:

```bash
pip install opencv-python mediapipe paho-mqtt
```

**b. Configure the broker address**

Edit `gesture_control_mqtt.py`:

```python
MQTT_BROKER = "your_gcp_vm_ip"
MQTT_PORT = 1883
```

**c. Run the client**

```bash
python gesture_control_mqtt.py
```

A window will open showing the live camera feed with hand landmarks overlaid. Detected gestures are published to MQTT in real time and reflected on the ESP32's LEDs.

---

## ✋ Supported Gestures

| Gesture            | MQTT Topic (example)  | Action                  |
| ------------------ | ---------------------- | ------------------------ |
| All fingers up     | `leds/all`              | Turn **ALL LEDs ON**     |
| All fingers down   | `leds/all`              | Turn **ALL LEDs OFF**    |
| Thumb only up      | `leds/thumb`            | Toggle **Yellow LED**    |
| Index only up      | `leds/index`            | Toggle **Red LED**       |
| Middle only up     | `leds/middle`           | Toggle **Green LED**     |
| Ring only up       | `leds/ring`             | Toggle **Blue LED**      |
| Pinky only up      | `leds/pinky`            | Toggle **White LED**     |

---

## ⚙️ Configuration

| Variable       | Location                    | Description                          |
| -------------- | ---------------------------- | -------------------------------------- |
| `MQTT_BROKER`  | `gesture_control_mqtt.py`    | Public IP of the GCP broker VM        |
| `MQTT_PORT`    | `gesture_control_mqtt.py`    | Default `1883`                        |
| `ssid` / `password` | ESP32 sketch            | Wi-Fi credentials for the ESP32       |
| `mqtt_server`  | ESP32 sketch                 | Same as `MQTT_BROKER` above           |

---

## 🐞 Troubleshooting

- **ESP32 won't connect to MQTT** — verify the GCP firewall allows inbound traffic on port `1883` and that the broker container is running (`docker ps`).
- **No hand detected** — ensure adequate lighting and that the hand is fully within frame; MediaPipe confidence thresholds can be tuned in the Python script.
- **High latency** — consider hosting the broker in a GCP region closer to your location, or reduce webcam resolution/frame rate.
- **LEDs flicker or don't respond** — check GPIO wiring against `docs/circuit_diagram.fzz` and confirm correct pin mappings in the ESP32 sketch.

---

## 📈 Roadmap

- [ ] 🎙️ Voice command support as an alternative input modality
- [ ] ✋ Multi-hand gesture recognition
- [ ] 📱 Companion mobile app for remote monitoring/control
- [ ] 🏠 Integration with Home Assistant / Google Home
- [ ] 📶 Wi-Fi provisioning UI for ESP32 (captive portal)
- [ ] 🔐 MQTT authentication and TLS by default

---

## 🔒 Ethical & Social Impact

- 📷 **Privacy-first**: all video processing happens locally — no frames are transmitted or stored
- ♿ **Accessibility**: designed to support users with mobility or speech impairments
- ⚡ **Energy-efficient**: built on the low-power ESP32 platform, suitable for always-on smart environments
- 🌍 **Inclusive by design**: affordable, open-source components make the system globally adaptable

---

## 🤝 Contributing

Contributions are welcome! To propose a change:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes with clear messages
4. Open a pull request describing the improvement

Areas especially open to contribution: gesture recognition accuracy, cloud infrastructure hardening, additional hardware support, and UI/UX improvements.

---

## 🧾 License

This project is licensed under the [MIT License](LICENSE).

---

## 👤 Author

**Abduvahhobov Javohir**
Full-Stack Development — Python, OpenCV, MediaPipe, MQTT, ESP32 Firmware, GCP VM Setup, Dockerized Mosquitto Broker

*Final Year Project — IoT Internship*
*Tashkent Turin Polytechnic University*
