# ✋🤖 Hand Gesture Controlled IoT System

**ESP32 + Python + OpenCV + MediaPipe + MQTT (Docker on GCP)**

This final-year project demonstrates a **smart, contactless interface** that allows users to control multiple LEDs using hand gestures. By combining **Computer Vision (MediaPipe + OpenCV)** with **IoT (ESP32 + MQTT)** and deploying the broker on the **cloud (GCP)** via Docker, it delivers a modern, scalable, and accessible solution for smart environments.

---

## 🚀 Key Features

* 🖐️ Real-time hand gesture recognition with **MediaPipe**
* 💡 Individual control of 5 LEDs (Thumb, Index, Middle, Ring, Pinky)
* 🔄 Universal gestures to turn **all LEDs ON/OFF**
* 🌐 **MQTT communication** between Python client and ESP32
* ☁️ **Cloud-hosted MQTT broker** on **GCP VM using Docker**
* 🧱 Modular, scalable architecture for smart home and assistive tech

---

## 🧠 System Overview

1. A **webcam** captures the live video feed
2. **MediaPipe** processes hand landmarks in real time
3. A **Python script** analyzes the gestures and publishes MQTT messages
4. **ESP32** subscribes to MQTT topics and toggles corresponding LEDs
5. **Mosquitto broker**, hosted in a **Docker container on GCP**, manages communication

---

## 🛠️ Tech Stack

* **Python 3.11+**
* **MediaPipe** & **OpenCV** for gesture detection
* **ESP32** microcontroller programmed with **Arduino IDE**
* **MQTT** protocol with **Mosquitto broker**
* **Docker** for containerization
* **Google Cloud Platform (GCP)** for hosting MQTT broker
* **Fritzing** for circuit design

---

## 🖥️ Getting Started

### 1. Python Client (Computer Side)

Install dependencies:

```bash
pip install opencv-python mediapipe paho-mqtt
```

Edit the IP address in `gesture_control_mqtt.py`:

```python
MQTT_BROKER = "your_gcp_vm_ip"
```

Run the client script:

```bash
python gesture_control_mqtt.py
```

---

### 2. MQTT Broker (Cloud via GCP + Docker)

1. Launch a GCP Virtual Machine (Ubuntu)
2. Install Docker:

```bash
sudo apt update
sudo apt install docker.io
```

3. Run Mosquitto broker in Docker:

```bash
docker run -it -p 1883:1883 eclipse-mosquitto
```

Your broker is now running and accessible globally via the VM’s IP address on port `1883`.

---

## ✋ Supported Hand Gestures

| Gesture          | Action                 |
| ---------------- | ---------------------- |
| All fingers up   | Turn **ALL LEDs ON**   |
| All fingers down | Turn **ALL LEDs OFF**  |
| Thumb only up    | Turn ON **Yellow LED** |
| Index only up    | Turn ON **Red LED**    |
| Middle only up   | Turn ON **Green LED**  |
| Ring only up     | Turn ON **Blue LED**   |
| Pinky only up    | Turn ON **White LED**  |

---

## 📈 Future Enhancements

* 🎙️ Add **voice command** support
* ✋ Enable **multi-hand gesture** recognition
* 📱 Develop a **mobile app** for remote access
* 🏠 Integrate with platforms like **Home Assistant**, **Google Home**, etc.
* 📶 Add Wi-Fi setup UI for ESP32 (captive portal)

---

## 🔒 Ethical & Social Impact

* 📷 All video processing is done **locally** — ensuring **user privacy**
* ♿ Supports users with **mobility or speech impairments**
* ⚡ Uses **energy-efficient ESP32**, suitable for low-power environments
* 🌍 Designed to be **inclusive, affordable, and globally adaptable**

---

## 🧾 License

This project is licensed under the [MIT License](LICENSE).

---

## 🤝 Contributing

Pull requests are welcome!
You’re encouraged to contribute improvements in gesture recognition, cloud infrastructure, hardware expansion, or UI enhancements.

---

## 👥 Author

* **Abduvahhobov Javohir** – Full-stack Development
  *(Python, OpenCV, MediaPipe, MQTT, ESP32 firmware, GCP VM setup, Dockerized Mosquitto broker)*

> Final Year Project — IoT Internship
> Tashkent Turin Polytechnic University


