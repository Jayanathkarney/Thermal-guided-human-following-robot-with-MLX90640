# 🌡️ Thermal Detection — How It Works

---

## What is the MLX90640?

The MLX90640 is an **infrared thermal camera** — it measures
heat emitted by objects without touching them.

```
Normal camera  → detects visible light (what eyes see)
Thermal camera → detects infrared radiation (heat)
```

Every object above absolute zero (-273°C) emits
infrared radiation. Hotter objects emit more.

```
Human skin  → ~33-36°C  → bright in thermal image
Cold wall   → ~20-24°C  → dark in thermal image
Hot lamp    → ~40-60°C  → very bright (false positive risk)
```

---

## Sensor Specifications

| Property | Value |
|---|---|
| Resolution | 32 × 24 pixels = 768 pixels total |
| Field of View | 55° × 35° |
| Temperature Range | -40°C to 300°C |
| Accuracy | ±1.5°C |
| Interface | I2C (address 0x33) |
| Refresh Rate | 0.5Hz to 64Hz |
| Supply Voltage | 3.3V only |

---

## How a Thermal Frame Looks

```
Each cell = one pixel = one temperature reading

Col:  0    1    2  ...  31
Row 0: [22.1][22.3][22.0]...[22.2]   ← cool background
Row 1: [22.0][22.1][35.2]...[22.1]   ← 35.2°C = hot pixel!
Row 2: [22.2][35.8][36.1]...[22.3]   ← human torso here
...
Row 23:[22.0][22.1][22.0]...[22.1]   ← floor (cool)

frame[0]   = Row0, Col0  = 22.1°C
frame[32]  = Row1, Col0  = 22.0°C
frame[767] = Row23, Col31 = 22.1°C
```

---

## Step 1: Calibration — Finding Ambient Temperature

Before moving, robot learns the room temperature
using a **histogram + percentile method.**

### Why not just average all pixels?
```
Simple average is WRONG:
Room at 24°C but one hot lamp at 55°C
Average = pulled upward → threshold too high
→ robot misses humans

Histogram 20th percentile is CORRECT:
Bottom 20% of pixels = coldest = background
Not affected by hot objects at all
```

### How histogram works:
```
Temperature range: -10°C to 70°C
Split into 160 bins of 0.5°C each

Bin 0  = -10.0 to -9.5°C
Bin 1  =  -9.5 to -9.0°C
...
Bin 68 =  24.0 to 24.5°C  ← most room pixels land here
...
Bin 160=  69.5 to 70.0°C

Collect 10 frames × 768 pixels = 7680 readings
Sort all into bins
Find bin where cumulative count = 20% of 7680 = 1536
That bin center = ambient_temp
```

### Example:
```
7680 total pixels
Bin 68 (24.0-24.5°C) contains 2000 pixels → cumulative hits 1536
ambient_temp = 24.25°C
threshold    = 24.25 + 1.5 = 25.75°C

Any pixel above 25.75°C → HOT PIXEL
```

---

## Step 2: Hot Pixel Counting

```c
int count_hot_pixels(void)
{
    float threshold = ambient_temp + 1.5f;
    int count = 0;
    for(i = 0; i < 768; i++)
        if(frame[i] > threshold)
            count++;
    return count;
}
```

### What different hot pixel counts mean:
```
0  - 10  pixels hot → empty room, no humans
10 - 40  pixels hot → small warm object (mug, laptop)
40 - 69  pixels hot → below threshold, ignored
70 - 150 pixels hot → HUMAN DETECTED ✓
150-300  pixels hot → multiple humans or very close human
300+     pixels hot → sensor too close to heat source
```

### Why threshold = 70 pixels?
```
Human torso at 1-2 meters fills ~80-150 pixels
Hot wall/lamp typically affects 5-20 pixels
70 is the safe middle ground:
  → rejects false positives from warm objects
  → catches humans reliably at 0.5m to 2m range

Tuning guide:
  Getting false positives → raise to 90-100
  Missing humans far away → lower to 50-60
```

---

## Step 3: Directional Scanning

Servo rotates camera to 3 positions:

```
Position 0° (RIGHT):
  Camera points right
  Wait 300ms to settle
  Capture frame → count hot pixels → cR

Position 90° (FORWARD):
  Camera points straight ahead
  Wait 300ms
  Capture frame → count → cFwd

Position 180° (LEFT):
  Camera points left
  Wait 300ms
  Capture frame → count → cL
```

### Direction Decision Logic:
```
Step 1: Any position ≥ 70 pixels?
        NO  → human not found → keep searching
        YES → continue to step 2

Step 2: Find maximum
        best = max(cR, cFwd, cL)
        chosen_dir = direction of best

Step 3: Forward preference
        If chosen_dir is LEFT or RIGHT
        AND cFwd ≥ 70
        AND (best - cFwd) ≤ 20 pixels
        THEN → choose FORWARD anyway

        Reason: avoid unnecessary turns
                20 pixel margin = ~2.6% of frame
```

### Example scenarios:
```
Scenario 1: Human directly ahead
cR=20  cFwd=120  cL=15
→ cFwd wins → go STRAIGHT ✓

Scenario 2: Human clearly to the left
cR=15  cFwd=30   cL=140
→ cL wins, cFwd below 70 → turn LEFT ✓

Scenario 3: Human slightly left
cR=15  cFwd=95   cL=110
→ cL wins by 15px, cFwd≥70, diff=15≤20
→ forward preference → go STRAIGHT ✓
  (saves an unnecessary turn)

Scenario 4: Human clearly to the right
cR=130  cFwd=25  cL=10
→ cR wins, cFwd below 70 → turn RIGHT ✓
```

---

## Common Issues & Fixes

| Problem | Cause | Fix |
|---|---|---|
| Robot never detects human | Threshold too high | Lower HUMAN_PIXEL_THRESHOLD to 50 |
| False detection from walls | Warm walls in sunlight | Raise AMBIENT_MARGIN_C to 2.0 |
| Misses human far away | Human fills few pixels | Lower threshold to 50, check FOV angle |
| Camera not responding | I2C wiring wrong | Check pull-ups, check 3.3V power |
| Ambient calibration wrong | Hot object in view during calibration | Remove heat sources, re-power robot |
