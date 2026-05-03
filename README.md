# 🤖 Thermal-Guided Human Following Robot with MLX90640

A fully autonomous robot that detects and follows humans 
using an infrared thermal camera — built on the 
**TM4C123GH6PM** microcontroller.

---

## 📌 What Does This Robot Do?

- Scans surroundings using **MLX90640 thermal camera** 
  mounted on a servo
- Detects humans by counting **hot pixels** in the 
  thermal frame
- Navigates autonomously toward the detected human
- Avoids obstacles using **HC-SR04 ultrasonic sensor**
- Makes smart decisions: follow, wait, or U-turn

---

## ⚙️ How It Works (Simple)
CALIBRATE → learn room temperature
↓
SCAN → rotate camera, count hot pixels
↓
TURN → face the human
↓
MOVE → drive toward human (2 seconds)
↓
OBSTACLE? → wall → U-TURN
→ human blocking → WAIT
↓
Repeat forever

---

## 🧰 Hardware Required

| Component | Purpose | Quantity |
|---|---|---|
| TM4C123GH6PM LaunchPad | Main microcontroller | 1 |
| MLX90640 (32x24) | Thermal camera | 1 |
| HC-SR04 | Ultrasonic distance sensor | 1 |
| SG90 Servo Motor | Rotate camera left/right | 1 |
| DC Gear Motors | Drive wheels | 2 |
| L298N Motor Driver | Control motor direction/speed | 1 |
| Robot Chassis | Base frame with wheels | 1 |
| Li-Po Battery (7.4V) | Power supply | 1 |
| Jumper Wires | Connections | many |

---

## 📁 Repository Structure
├── code/          → All C source files for TM4C123
├── docs/          → Detailed documentation & explanations
├── hardware/      → Pin mapping, circuit diagrams, wiring
├── images/        → Photos, diagrams, screenshots
├── videos/        → Demo video links

---

## 🔌 Pin Map (Quick Reference)

| TM4C123 Pin | Connected To |
|---|---|
| PA2 | Left motor FORWARD |
| PA3 | Left motor BACKWARD |
| PA4 | Right motor FORWARD |
| PA5 | Right motor BACKWARD |
| PB6 | PWM Motor speed (L298N ENA) |
| PB7 | PWM Motor speed (L298N ENB) |
| PC4 | Servo signal |
| PE0 | Ultrasonic TRIG |
| PE1 | Ultrasonic ECHO |
| PB2 | I2C SCL (MLX90640) |
| PB3 | I2C SDA (MLX90640) |

---

## 🚀 Getting Started

### Step 1: Clone this repository
git clone https://github.com/Jayanathkarney/
Thermal-guided-human-following-robot-with-MLX90640

### Step 2: Open in Keil uVision
File → Open Project → select code/robot.uvprojx

### Step 3: Flash to TM4C123
Build → Flash → Run
Open serial monitor at 115200 baud to see debug output

---

## 📖 Detailed Documentation

| Topic | Link |
|---|---|
| How the thermal detection works | [docs/thermal-detection.md](docs/) |
| State machine explained | [docs/state-machine.md](docs/) |
| Circuit wiring guide | [hardware/](hardware/) |
| Code walkthrough | [code/](code/) |

---

## 👨‍💻 Author
**Jayanath Karney**
Course Project — Embedded Systems

---

## 📜 License
MIT License — free to use, modify, and share.
