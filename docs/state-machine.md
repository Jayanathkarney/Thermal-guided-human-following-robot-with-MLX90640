
# 🧠 State Machine — Robot Decision Logic

---

## What is a State Machine?

A state machine is a system that is always in
**one state at a time** and switches between
states based on **events or conditions.**

```
Robot is always in exactly ONE of these states:
  CALIBRATE, MOVE, SCAN, TURN,
  OBSTACLE_CHECK, WAIT_HUMAN, UTURN
```

---

## Full State Diagram

```
                    ┌─────────────┐
         ┌─────────→│  CALIBRATE  │ (once at startup)
         │          └──────┬──────┘
         │                 │ done
         │          ┌──────▼──────┐
         │    ┌────→│    SCAN     │←──────────────────┐
         │    │     └──────┬──────┘                   │
         │    │            │ human found               │
         │    │     ┌──────▼──────┐                   │
         │    │     │    TURN     │                   │
         │    │     └──────┬──────┘                   │
         │    │            │ always                    │
         │    │     ┌──────▼──────┐   obstacle        │
         │    │     │    MOVE     │──────────────┐    │
         │    │     └─────────────┘              │    │
         │    │      (2 seconds,                 │    │
         │    │       no obstacle)               │    │
         │    │            │ clean run           │    │
         │    └────────────┘                     │    │
         │                               ┌───────▼──────┐
         │                               │OBSTACLE_CHECK│
         │                               └───┬──────┬───┘
         │                          human    │      │ wall
         │                     ┌─────────────┘      │
         │              ┌──────▼──────┐      ┌──────▼──────┐
         │              │ WAIT_HUMAN  │      │    UTURN    │
         │              └──────┬──────┘      └──────┬──────┘
         │           cleared   │ or timeout          │
         └─────────────────────┘                     │
                                                     └──→SCAN
```

---

## Each State Explained

### 🔵 STATE: CALIBRATE
```
WHEN:    Once at startup only
DOES:    Takes 10 thermal frames
         Builds histogram of all pixel temperatures
         Finds 20th percentile = ambient_temp
         Sets threshold = ambient_temp + 1.5°C
GOES TO: SCAN (immediately after)
```

---

### 🟢 STATE: SCAN
```
WHEN:    After CALIBRATE, after MOVE (2s clean run),
         after WAIT_HUMAN, after UTURN
DOES:    Rotate servo RIGHT → capture frame → count pixels
         Rotate servo FORWARD → capture → count
         Rotate servo LEFT → capture → count
         Compare counts → decide direction

IF human found:
         chosen_dir = best direction
         GOES TO: TURN

IF human NOT found:
         Rotate robot slightly (search behavior)
         search_count++ 
         After 4 small rotations → switch search side
         STAYS IN: SCAN (keeps searching)
```

---

### 🔄 STATE: TURN
```
WHEN:    After SCAN finds human
DOES:    chosen_dir = 0 → turn RIGHT 300ms
         chosen_dir = 1 → no turn (already forward)
         chosen_dir = 2 → turn LEFT  300ms
GOES TO: MOVE (always)
```

---

### 🟡 STATE: MOVE
```
WHEN:    After TURN
DOES:    Set PWM duty = 20000 (80% speed)
         Drive forward
         Every 50ms → check ultrasonic distance
         Total time = 40 steps × 50ms = 2 seconds

IF obstacle detected (dist < 35cm):
         STOP immediately
         GOES TO: OBSTACLE_CHECK

IF 2 seconds complete with no obstacle:
         STOP
         GOES TO: SCAN
```

---

### 🔴 STATE: OBSTACLE_CHECK
```
WHEN:    Obstacle detected during MOVE
DOES:    Wait 200ms (let robot fully stop)
         Capture one thermal frame
         Count hot pixels

IF hot pixels ≥ 70:
         → obstacle is a HUMAN
         GOES TO: WAIT_HUMAN

IF hot pixels < 70:
         → obstacle is a WALL or furniture
         GOES TO: UTURN
```

---

### 🟠 STATE: WAIT_HUMAN
```
WHEN:    Human is blocking path
DOES:    Every 200ms → check ultrasonic distance
         wait_cycles counter increments each check

IF dist ≥ 40cm:
         Human moved away → clear path
         GOES TO: SCAN

IF wait_cycles ≥ 25 (5 seconds timeout):
         Human not moving → give up waiting
         GOES TO: SCAN

ELSE:    Keep waiting, increment counter
         STAYS IN: WAIT_HUMAN
```

---

### ⚫ STATE: UTURN
```
WHEN:    Wall or dead-end detected
DOES:    Reverse 300ms        (get clearance from wall)
         Spin RIGHT 600ms     (180° turn)
         Move forward 250ms   (clear the obstacle)
GOES TO: SCAN
```

---

## Timing Summary

| Action | Duration |
|---|---|
| Calibration | ~10 × 200ms = 2 seconds |
| Servo settle time | 300ms per position |
| Full scan (3 positions) | ~900ms + frame capture time |
| Forward movement | 2 seconds (40 × 50ms) |
| One turn (90°) | 300ms |
| U-turn (180°) | 600ms spin + 300ms reverse |
| Human wait timeout | 25 × 200ms = 5 seconds |

---

## Key Parameters You Can Tune

| Parameter | Default | Effect |
|---|---|---|
| HUMAN_PIXEL_THRESHOLD | 70 | Lower = more sensitive |
| AMBIENT_MARGIN_C | 1.5°C | Lower = more sensitive |
| OBSTACLE_STOP_DIST_CM | 35cm | When to stop for obstacle |
| HUMAN_WAIT_DIST_CM | 40cm | When human is considered clear |
| WAIT_TIMEOUT_CYCLES | 25 | How long to wait for human |
| FORWARD_STEPS | 40 | How long robot moves forward |
| UTURN_SPIN_US | 600000 | 180° spin duration |
