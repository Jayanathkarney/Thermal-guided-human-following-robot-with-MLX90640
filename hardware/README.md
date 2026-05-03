
# 🔌 Hardware Guide

Complete wiring, connections, and hardware setup for the
Thermal-Guided Human Following Robot.

---

## 🧰 Full Component List

| Component | Specification | Purpose |
|---|---|---|
| TM4C123GH6PM | 80MHz ARM Cortex-M4 | Main brain |
| MLX90640 | 32×24 IR array, I2C, 3.3V | Thermal camera |
| HC-SR04 | 2cm-400cm, 5V | Distance measurement |
| SG90 Servo | 180°, PWM, 5V | Rotate camera |
| DC Gear Motor ×4 | 3-6V, ~200RPM | Drive wheels |
| L298N ×2 | Dual H-bridge, 5-35V | Motor driver |
| 4WD Robot Chassis | Metal/plastic frame | Physical base |
| Li-Po 7.4V 2S | 1000-2200mAh | Main power |
| 3.3V Regulator | AMS1117-3.3 | Power MLX90640 |
| Jumper wires | Male-Male, Male-Female | Connections |

---

## ⚡ Power Architecture

```
Li-Po 7.4V Battery
       │
       ├──→ L298N #1 (12V pin)  → motors left side
       │         │
       │    L298N 5V out ──→ Servo SG90
       │
       ├──→ L298N #2 (12V pin)  → motors right side
       │
       └──→ TM4C123 (VIN pin)
                  │
             TM4C 3.3V out ──→ MLX90640 VDD
                  │
             TM4C 5V  out ──→ HC-SR04 VCC
```

**NEVER connect MLX90640 directly to 5V — it runs on 3.3V only.
Doing so will permanently damage the sensor.**

---

## 🔌 Complete Pin Connection Table

### TM4C123 → L298N #1 (Left Side Motors)

| TM4C123 Pin | L298N #1 Pin | Purpose |
|---|---|---|
| PA2 | IN1 | Left motors FORWARD |
| PA3 | IN2 | Left motors BACKWARD |
| PB6 | ENA | Left motors PWM speed |
| GND | GND | Common ground |

### TM4C123 → L298N #2 (Right Side Motors)

| TM4C123 Pin | L298N #2 Pin | Purpose |
|---|---|---|
| PA4 | IN3 | Right motors FORWARD |
| PA5 | IN4 | Right motors BACKWARD |
| PB7 | ENB | Right motors PWM speed |
| GND | GND | Common ground |

### Motor Wiring (4 DC Motors → 2 L298N)

| Motor | L298N Terminal |
|---|---|
| Front-Left motor + | L298N #1 OUT1 |
| Front-Left motor - | L298N #1 OUT2 |
| Rear-Left  motor + | L298N #1 OUT1 |
| Rear-Left  motor - | L298N #1 OUT2 |
| Front-Right motor + | L298N #2 OUT3 |
| Front-Right motor - | L298N #2 OUT4 |
| Rear-Right  motor + | L298N #2 OUT3 |
| Rear-Right  motor - | L298N #2 OUT4 |

**Left pair wired in parallel → both spin together**
**Right pair wired in parallel → both spin together**

---

### TM4C123 → HC-SR04 (Ultrasonic)

| TM4C123 Pin | HC-SR04 Pin | Purpose |
|---|---|---|
| PE0 | TRIG | Send trigger pulse |
| PE1 | ECHO | Receive echo pulse |
| 5V  | VCC  | Power |
| GND | GND  | Ground |

⚠️ **ECHO pin outputs 5V — TM4C123 is 3.3V tolerant only.**
Use a voltage divider on ECHO line:

```
ECHO(5V) ──┤10kΩ├──┬──→ PE1 (3.3V safe)
                   │
                 22kΩ
                   │
                  GND

Voltage at PE1 = 5V × 22/(10+22) = 3.4V ≈ safe
```

---

### TM4C123 → SG90 Servo

| TM4C123 Pin | Servo Wire | Purpose |
|---|---|---|
| PC4 | Signal (orange) | PWM position control |
| 5V  | Power (red) | Supply voltage |
| GND | Ground (brown) | Ground |

---

### TM4C123 → MLX90640 (Thermal Camera)

| TM4C123 Pin | MLX90640 Pin | Purpose |
|---|---|---|
| PB2 | SCL | I2C clock |
| PB3 | SDA | I2C data |
| 3.3V | VDD | Power (3.3V ONLY) |
| GND | GND | Ground |

**Add 4.7kΩ pull-up resistors:**
```
3.3V ──┤4.7kΩ├──→ SCL (PB2)
3.3V ──┤4.7kΩ├──→ SDA (PB3)
```
I2C lines need pull-ups to work reliably.
Without them — camera will not communicate.

---

## 🤖 Robot Assembly Order

```
Step 1: Assemble 4WD chassis
        → mount 4 motors into chassis brackets
        → attach wheels

Step 2: Mount L298N drivers
        → L298N #1 on left side of chassis
        → L298N #2 on right side of chassis

Step 3: Wire motors to L298N
        → left pair in parallel → OUT1, OUT2
        → right pair in parallel → OUT3, OUT4

Step 4: Mount servo on front
        → facing forward
        → centered at 90°

Step 5: Mount MLX90640 on servo horn
        → camera faces same direction as servo
        → secure with hot glue or small screws

Step 6: Mount HC-SR04 on front
        → same height as expected human torso
        → roughly 20-30cm from ground

Step 7: Mount TM4C123 LaunchPad
        → center of chassis
        → USB port accessible for flashing

Step 8: Wire everything per connection table above

Step 9: Connect battery last
        → double check all connections before powering
```

---

## ⚠️ Common Wiring Mistakes

| Mistake | Consequence | Fix |
|---|---|---|
| MLX90640 on 5V | Sensor permanently damaged | Use 3.3V only |
| No pull-ups on I2C | Camera not detected | Add 4.7kΩ resistors |
| ECHO direct to MCU | MCU pin damage over time | Use voltage divider |
| Motors share GND with MCU | Noise causes resets | Separate grounds, join at battery |
| ENA/ENB jumper left on L298N | No PWM speed control | Remove jumper, wire PB6/PB7 |

---

## 📐 L298N ENA/ENB Jumper — Important!

L298N comes with **jumper caps** on ENA and ENB:

```
Default (jumper ON):  motor always full speed
                      PWM from MCU has NO effect

Required (jumper OFF): remove jumper cap
                       wire ENA → PB6
                       wire ENB → PB7
                       now PWM controls speed ✓
```

**Remove both jumper caps before wiring PWM pins.**
