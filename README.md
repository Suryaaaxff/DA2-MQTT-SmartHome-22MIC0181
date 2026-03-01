# DA-2: Publish–Subscribe System with MQTT Message Broker

**Name:** Suriya Kumari
**Student ID:** 22MIC0181
**Lab:** DA-2 — Node-RED MQTT Pub-Sub System
**Date:** 01 March 2026
**VideoMd:** https://drive.google.com/file/d/1jQZwNkQVy-lcMNRPrH1HcOIvmwshq0o0/view?usp=drivesdk

---

## 📌 Overview

This project implements a **Smart Home Monitoring System** using Node-RED and an MQTT message broker. It demonstrates the Publish-Subscribe (Pub-Sub) design pattern where publishers and subscribers communicate through a broker without direct knowledge of each other.

---

## 🏗️ System Architecture

```
[Temperature Publisher] ---> [MQTT Broker] ---> [Wildcard Subscriber home_Suriya/#]
[Motion Publisher]      --->               --->         |
[Status Publisher]      --->               --->    [Parse & Route]
                                                        |
                                          ┌─────────────┼─────────────┐
                                     [Gauge]       [Chart]     [Motion Status]
                                          └─────────────┴─────────────┘
                                                   [Dashboard]
```

---

## 📡 Publishers

### Publisher 1 — Temperature Sensor
- **Topic:** `home_Suriya/livingroom/temperature`
- **Interval:** Every 5 seconds
- **QoS:** 1 (At least once delivery)
- **Payload:**
```json
{
  "sensor": "temp_01",
  "temperature": 28.9,
  "humidity": 49.4,
  "unit": "celsius",
  "timestamp": "2026-03-01T18:47:43.303Z"
}
```

### Publisher 2 — Motion Sensor
- **Topic:** `home_Suriya/hallway/motion`
- **Interval:** Every 7 seconds
- **QoS:** 2 (Exactly once delivery)
- **Payload:**
```json
{
  "sensor": "motion_01",
  "status": "detected",
  "room": "hallway",
  "timestamp": "2026-03-01T18:10:07.052Z"
}
```

### Publisher 3 — Status Publisher
- **Topic:** `home_Suriya/status`
- **Trigger:** On system start
- **Purpose:** Announces system online status
- **Retain:** true

---

## 📥 Subscriber

### Wildcard Subscriber
- **Topic:** `home_Suriya/#`
- **Description:** The `#` wildcard matches ALL topics under `home_Suriya/` — temperature, motion, and status — with a single subscription. This is the core of the pub-sub pattern: the subscriber does not need to know which publishers exist.

---

## 🔀 Parse & Route Function

The Parse & Route function node processes all incoming wildcard messages and routes them to the correct dashboard widget:

```javascript
var data;
try { data = JSON.parse(msg.payload); } catch (e) { data = msg.payload; }

var out1 = null, out2 = null, out3 = null;

if (msg.topic.includes("temperature")) {
    out1 = { payload: data.temperature || 0, topic: msg.topic };
} else if (msg.topic.includes("motion")) {
    out2 = { payload: data.status === "detected" ? 1 : 0, topic: msg.topic };
}

if (msg.topic.startsWith("home_Suriya/")) {
    out3 = {
        payload: "[" + msg.topic + "] " + JSON.stringify(data),
        topic: "home_Suriya/status"
    };
}

return [out1, out2, out3];
```

---

## 📊 Dashboard

The Node-RED dashboard (`localhost:1880/ui`) visualizes live data with:

| Widget | Data Source | Description |
|--------|-------------|-------------|
| Temperature Gauge | `home_Suriya/livingroom/temperature` | Live °C reading (0–50 range) |
| Temperature Chart | `home_Suriya/livingroom/temperature` | Historical trend over time |
| Motion Status | `home_Suriya/hallway/motion` | DETECTED / CLEAR indicator |
| Message Log | All topics | Raw message log with topic labels |

---

## 📋 Message Log Sample

```
[home_Suriya/livingroom/temperature]
{"sensor":"temp_01","temperature":28.9,"humidity":49.4,"unit":"celsius","timestamp":"2026-03-01T18:47:43.303Z"}

[home_Suriya/hallway/motion]
{"sensor":"motion_01","status":"detected","room":"hallway","timestamp":"2026-03-01T18:10:07.052Z"}

[home_Suriya/status]
{"topic_received":"home_Suriya/livingroom/temperature","status":"OK","broker_time":"2026-03-01T18:47:43.400Z"}
```

---

## ⚙️ QoS vs Retained Messages

| Feature | Value | Meaning |
|---------|-------|---------|
| Temperature QoS | **1** | Broker delivers message **at least once** — suitable for sensor readings where occasional duplicates are acceptable |
| Motion QoS | **2** | Broker delivers message **exactly once** — important for motion events where duplicates could trigger false alarms |
| Status Retain | **true** | Broker stores the last status message so any **new subscriber gets it immediately** on connect, without waiting for the next publish cycle |
| QoS 0 (not used) | 0 | Fire and forget — no delivery guarantee |

---

## 🔧 How to Run

### Prerequisites
- Node.js installed
- Node-RED installed (`npm install -g node-red`)
- Mosquitto MQTT broker installed and running on `localhost:1883`
- node-red-dashboard palette installed

### Steps
```bash
# 1. Start Mosquitto broker
mosquitto -v

# 2. Start Node-RED
node-red

# 3. Open browser
# Flow editor: http://127.0.0.1:1880
# Dashboard:   http://localhost:1880/ui

# 4. Import flow.json via Menu > Import > Select flows.json

# 5. Click Deploy
```

---

## 📁 Repository Structure

```
├── flows.json              # Node-RED flow export
├── README.md               # This file
├── screenshots/
│   ├── flow-screenshot.png     # Node-RED canvas
│   ├── dashboard-screenshot.png # Live dashboard
│   └── message-log.png         # Debug panel messages
└── video/
    └── demo-link.txt           # Link to video demonstration
```

---

## 🎯 Assignment Checklist

- [x] At least two publishers
- [x] One wildcard subscriber (`home_Suriya/#`)
- [x] Dashboard with live data visualization
- [x] QoS demonstrated (QoS 1 and QoS 2)
- [x] Retained messages demonstrated
- [x] Status topic included
- [x] Message log from at least two topics
- [x] Video demonstration

---

*Submitted for DA-2 | VIT | 2026*
