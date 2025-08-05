# 🌱 IoT Soil Moisture Monitoring & Plant Watering Notification System

A smart plant monitoring system using **NodeMCU ESP8266** and a **soil moisture sensor**. This IoT project reads soil moisture levels and notifies the user through the **Blynk mobile app** when plants need watering. It also allows automated watering using a relay-controlled pump.

## 📸 Demo Video

▶️ [Watch Project on YouTube](https://www.youtube.com/watch?v=L8n1MecDGj4)

---

## 📦 Features

- 📊 Real-time soil moisture monitoring
- 📱 Blynk app integration for live data & control
- 🚨 Instant notifications when soil is too dry
- 💧 Optional auto-watering via relay and water pump
- ⚙️ User-adjustable moisture threshold via mobile app
- 🌡️ (Optional) Temperature & humidity monitoring with DHT11/22

---

## 🧰 Hardware Components

| Component              | Quantity |
|------------------------|----------|
| NodeMCU ESP8266        | 1        |
| Soil Moisture Sensor   | 1        |
| Relay Module (5V)      | 1        |
| Water Pump or Load     | 1        |
| Jumper Wires           | Several  |
| Breadboard             | 1        |
| Power Supply (5V/12V)  | 1        |
| (Optional) DHT11/22    | 1        |

---

## ⚙️ Software Requirements

- [Arduino IDE](https://www.arduino.cc/en/software)
- [Blynk App](https://blynk.io/)
- Arduino libraries:
  - `BlynkSimpleEsp8266.h`
  - `ESP8266WiFi.h`
  - `SimpleTimer.h`
  - (Optional) `DHT.h`

---

## 🧠 How It Works

1. The NodeMCU reads soil moisture levels from the sensor.
2. Moisture values are sent to the Blynk app via Wi-Fi.
3. If the moisture level is below a set threshold:
   - A notification is sent to the user's phone.
   - (Optional) A water pump is activated via a relay for a short time.
4. The user can monitor live readings and control settings from the app.

---

## 🔌 Circuit Diagram Overview

**Soil Moisture Sensor → NodeMCU:**
- VCC → 3.3V  
- GND → GND  
- A0 → A0

**Relay Module → NodeMCU:**
- VCC → VIN (or 5V)  
- GND → GND  
- IN → D1 (GPIO5)

**Water Pump:**
- Connected to relay NO/COM terminals  
- Powered via external 5V/12V supply

---

## 📲 Blynk App Setup

1. Create a new project in the Blynk app.
2. Select device: `NodeMCU`
3. Add the following widgets:
   - Gauge (Virtual Pin V1) → Soil Moisture
   - Notification widget
   - Slider (Virtual Pin V2) → Threshold %
4. Use the Auth Token from your Blynk email in the Arduino code.

---

## 💻 Arduino Code Snippet

```cpp
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <SimpleTimer.h>

char auth[] = "YourAuthToken";
char ssid[] = "YourWiFiSSID";
char pass[] = "YourWiFiPassword";

#define soilPin A0
#define relayPin D1

int threshold = 30;  // default threshold
SimpleTimer timer;

void setup() {
  Serial.begin(9600);
  Blynk.begin(auth, ssid, pass);
  pinMode(relayPin, OUTPUT);
  digitalWrite(relayPin, LOW);

  timer.setInterval(5000L, checkMoisture); // every 5 sec
}

BLYNK_WRITE(V2) {
  threshold = param.asInt(); // update from slider
}

void checkMoisture() {
  int sensorValue = analogRead(soilPin);
  int moisturePercent = map(sensorValue, 1023, 0, 0, 100);
  Blynk.virtualWrite(V1, moisturePercent);

  if (moisturePercent < threshold) {
    Blynk.notify("Soil is dry! Starting pump...");
    digitalWrite(relayPin, HIGH);
    delay(3000);
    digitalWrite(relayPin, LOW);
  }
}

void loop() {
  Blynk.run();
  timer.run();
}
