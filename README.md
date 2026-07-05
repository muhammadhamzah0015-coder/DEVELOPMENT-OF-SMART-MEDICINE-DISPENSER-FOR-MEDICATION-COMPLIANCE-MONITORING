Here is a clean, professional README file formatted in Markdown. You can save this text as `README.md` and include it in your project repository alongside your ESP32 code.

---

# Smart Medicine Dispenser - Main Controller (ESP32-S3)

## Overview

This repository contains the main controller code for an IoT-based Smart Medicine Dispenser. Designed as a general Electrical Engineering system, it assists patients with cognitive impairments (such as dementia) in adhering to their daily medication schedules. The system utilizes an ESP32-S3 microcontroller to orchestrate physical dispensing, real-time AI voice interaction, patient presence detection, and cloud-based caretaker alerts.

## Key Features

* **AI Voice Assistant:** Integrates OpenAI (ChatGPT) and ByteDance ASR via I2S microphone/speaker for conversational medication reminders and verbal confirmation.
* **Mechanical Dispensing:** Precision tray rotation using a 28BYJ-48 stepper motor driven by a ULN2003 motor driver.
* **Cloud Notifications & Escalation:** Automated patient reminders via the Signal app and critical caretaker escalation alerts via the Telegram Bot API.
* **ESP-NOW Communication:** Low-latency wireless triggering of a secondary ESP32-CAM module to perform visual confirmation.
* **Internet Time Synchronization:** Accurate daily scheduling using Google NTP servers (no external hardware RTC required).
* **Non-Volatile Memory:** Tracks daily dose progress using the ESP32 `Preferences` library to recover state after power loss.
* **Local UI:** Real-time system status and AI dialogue transcriptions displayed on an ST7735 TFT screen.

## Hardware Requirements

* ESP32-S3 Microcontroller
* Secondary ESP32-CAM module (for machine vision)
* ST7735 SPI TFT Display
* 28BYJ-48 Stepper Motor + ULN2003 Motor Driver
* INMP441 I2S Microphone
* MAX98357 I2S Audio Amplifier + Speaker
* HC-SR04 Ultrasonic Sensor
* Touch Button Sensor (Capacitive or physical)

## Pin Mapping

Ensure your schematic matches the following GPIO assignments defined in the code:

| Component | Pin Function | ESP32-S3 GPIO |
| --- | --- | --- |
| **TFT Display** | CS, DC, MOSI, SCLK, RST | 21, 38, 14, 13, 8 |
| **Stepper Motor** | IN1, IN2, IN3, IN4 | 9, 10, 16 (Updated), 12 |
| **Microphone** | SCK, WS/LRC, SD | 5, 4, 6 |
| **Audio Amp** | DOUT, BCLK, LRC | 2, 42, 41 |
| **Ultrasonic** | TRIG, ECHO | 7, 15 |
| **Touch Button** | Signal | 48 |

## Software Dependencies

Install the following libraries via the Arduino IDE Library Manager or ZIP file:

* `ArduinoASRChat` (ByteDance ASR Integration)
* `ArduinoGPTChat` (OpenAI Integration)
* `ESP32-audioI2S` (Audio playback)
* `Adafruit_GFX` & `Adafruit_ST7735` (TFT Display)
* `AccelStepper` (Motor control)
* Standard ESP32 libraries: `WiFi`, `esp_now`, `HTTPClient`, `WiFiClientSecure`, `Preferences`

## Setup & Configuration

Before flashing the code to your ESP32-S3, update the following variables in the script:

1. **Wi-Fi Credentials:**
```cpp
const char* ssid     = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";

```


2. **API Keys:**
Replace the placeholder strings for `asr_api_key`, `openai_apiKey`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`, `PATIENT_API_KEY`, and `PATIENT_PHONE`.
3. **ESP-NOW Address:**
Update `camAddress[]` with the specific MAC address of your ESP32-CAM module.
4. **Medication Schedule:**
Adjust the `prototypeSchedule[4]` array to set the required dispensing time windows (in 24-hour format).

## System Workflow

1. **Phase 1 (Wake & Remind):** Checks NTP time against the schedule. Sends a Signal message and plays an audio tone. Ultrasonic sensor scans for patient presence.
2. **Phase 2 (Vision & AI):** Upon detection, wakes the ESP32-CAM via ESP-NOW. AI voice assistant greets the patient and requests button press. Implements a 3-second callback and 15-second timeout safety loop.
3. **Phase 3 (Dispense & Confirm):** Stepper motor rotates tray. Camera attempts visual confirmation of consumption. If visual fails, AI falls back to verbal confirmation. Final status is sent to the caretaker via Telegram.
