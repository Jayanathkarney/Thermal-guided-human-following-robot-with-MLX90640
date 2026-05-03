# 💻 Source Code

Complete Code Composer Studio (CCS) project for the
Thermal-Guided Human Following Robot.

---

## 📦 Download

👉 **[Download ZIP: thermal-robot-tm4c123-code.zip](thermal_guided_robot_trial_4.zip)**

---

## 📁 What's Inside the ZIP

```
thermal-robot-tm4c123-code/
│
├── main.c                  ← main state machine + motor control
├── mlx90640.c              ← thermal camera frame capture
├── mlx90640.h              ← thermal camera header
├── MLX90640_I2C_Driver.c   ← I2C communication driver
├── MLX90640_I2C_Driver.h   ← I2C driver header
├── ultrasonic.c            ← HC-SR04 distance measurement
├── ultrasonic.h            ← ultrasonic header
└── tm4c123gh6pm.h          ← TM4C register definitions
```

---

## 🚀 How to Import into CCS

### Step 1: Extract ZIP
```
Right click ZIP → Extract All
Remember the extracted folder location
```

### Step 2: Import into CCS
```
Open Code Composer Studio
File → Import → Code Composer Studio → CCS Projects
Click Browse → navigate to extracted folder
Select the project → Click Finish
```

### Step 3: Verify Target
```
Right click project → Properties
General → Device → verify:
  Device: TM4C123GH6PM
  Connection: Stellaris In-Circuit Debug Interface
```

### Step 4: Build
```
Project → Build All   (Ctrl + B)
Check Console window — should say:
"Build Finished, 0 errors"
```

### Step 5: Flash and Run
```
Connect TM4C123 LaunchPad via USB
Run → Debug   (F11)
Run → Resume  (F8)
```

### Step 6: Monitor Serial Output
```
Open terminal (PuTTY or CCS Console)
Port   → your COM port (check Device Manager)
Baud   → 115200
Config → 8N1 (8 data, no parity, 1 stop)

You should see:
Robot Start
MLX ready
Ultrasonic ready
Calibrating
F1 ... F10
Amb=24.5 Thr=26.0
Moving 2s
...
```

---

## 🔧 Key Parameters to Tune

Open `main.c` → top section has all tunable parameters:

| Parameter | Line | Default | Change if... |
|---|---|---|---|
| HUMAN_PIXEL_THRESHOLD | ~30 | 70 | Missing humans → lower to 50 |
| AMBIENT_MARGIN_C | ~40 | 1.5°C | False positives → raise to 2.0 |
| OBSTACLE_STOP_DIST_CM | ~45 | 35cm | Stopping too early/late |
| FORWARD_STEPS | ~20 | 40 | Change forward distance |
| TURN_TIME_LEFT_US | ~25 | 300000 | Adjust turn angle |
| UTURN_SPIN_US | ~28 | 600000 | Adjust U-turn angle |

---

## 📋 File Descriptions

### `main.c`
Core file containing:
- System clock setup (80MHz PLL)
- State machine (CALIBRATE→SCAN→TURN→MOVE)
- Motor control functions
- PWM setup for speed control
- Servo control (bit-banged)
- UART debug output
- Timer1 microsecond delay
- Calibration histogram logic

### `mlx90640.c` + `.h`
- Initializes MLX90640 via I2C
- Reads raw 2-subframe data
- Applies factory calibration math
- Returns 768 float temperature array

### `MLX90640_I2C_Driver.c` + `.h`
- Low level I2C read/write functions
- Handles I2C start/stop/ack conditions
- Runs at 400kHz fast mode

### `ultrasonic.c` + `.h`
- Initializes Timer4 as free-running counter
- Generates 10µs TRIG pulse on PE0
- Measures ECHO pulse width on PE1
- Converts timer ticks → centimeters
- Returns distance in cm

---

## ⚠️ Common CCS Issues

| Issue | Fix |
|---|---|
| Build error: missing tm4c123gh6pm.h | Add TivaWare to include path |
| Flash fails: no connection | Check USB, install Stellaris drivers |
| Camera not found on I2C | Check 4.7kΩ pull-ups on SDA/SCL |
| Motors not moving | Check L298N jumper removed, check PWM wiring |
| Serial output garbled | Verify baud rate = 115200 exactly |
