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
| DC Gear Motors | Drive wheels | 4 |
| L298N Motor Driver | Control motor direction/speed | 2 |
| Robot Chassis (4WD) | Base frame with 4 wheels | 1 |
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
| PA2 | Left motors FORWARD (L298N IN1) |
| PA3 | Left motors BACKWARD (L298N IN2) |
| PA4 | Right motors FORWARD (L298N IN3) |
| PA5 | Right motors BACKWARD (L298N IN4) |
| PB6 | PWM speed left side (L298N ENA) |
| PB7 | PWM speed right side (L298N ENB) |
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

### Step 2: Open in Code Composer Studio (CCS)
File → Import → CCS Projects
Browse → select the code/ folder
Click Finish

### Step 3: Build and Flash to TM4C123
Project → Build All  (Ctrl+B)
Run → Debug          (F11)
Run → Resume         (F8)
Open CCS Console → set baud 115200
to see live debug output from robot

---

## 🖥️ Development Environment

| Tool | Details |
|---|---|
| IDE | Code Composer Studio (CCS) |
| Compiler | TI ARM Compiler |
| Target MCU | TM4C123GH6PM (80MHz) |
| Debug Interface | ICDI (on-board TM4C LaunchPad) |
| Serial Monitor | CCS Console / PuTTY at 115200 baud |

---

## 📖 Detailed Documentation

| Topic | Link |
|---|---|
| How thermal detection works | [docs/thermal-detection.md](docs/) |
| State machine explained | [docs/state-machine.md](docs/) |
| Circuit wiring guide | [hardware/](hardware/) |
| Code walkthrough | [code/](code/) |

---

## 4WD Motor Wiring Logic
LEFT SIDE  (front-left + rear-left motors):
Both motor + terminals → L298N OUT1
Both motor - terminals → L298N OUT2
Controlled by PA2(fwd) PA3(bwd) PB6(PWM)
RIGHT SIDE (front-right + rear-right motors):
Both motor + terminals → L298N OUT3
Both motor - terminals → L298N OUT4
Controlled by PA4(fwd) PA5(bwd) PB7(PWM)

Both motors on same side are **wired in parallel** — they receive identical signals and move together as one unit.

---

## 👨‍💻 Author
**Jayanath Karney**
Course Project — Embedded Systems

---

## 📜 License
MIT License — free to use, modify, and share.
