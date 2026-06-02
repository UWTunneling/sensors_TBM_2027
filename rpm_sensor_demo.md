# RPM Sensor Demo

Notes for the TBM RPM-sensor demo on the ESP32-S3.

## Quick start

```text
Cmd+Shift+P  ->  "ESP-IDF: Open ESP-IDF Terminal"
cd sensors_TBM_2027
idf.py build flash monitor      # Ctrl+] to exit the monitor
```

Hardware note: the test fan starts spinning around **3.75 V** and stalls again around **9 V**.

## The sensor: Monarch Instrument ROS-P

A **remote optical sensor** — it shines a red LED and looks for the reflection
off a **retroreflective** mark passing once per revolution.

| Property | Value |
|---|---|
| Detection | Reflective tape (Monarch T-5 / 3M Scotchlite), **NOT magnetic** |
| Output | **0–5 V TTL pulse**, push-pull, on the leading edge of the tape |
| Power | 5 Vdc @ 30 mA |
| Speed range | 1–250,000 RPM |
| Operating distance | 1 in – 36 in, up to ~45° off, recommended ~15° off perpendicular |

**Pinout (1/8" phone plug / wires):**

| Wire | Function |
|---|---|
| Brown | +5 V excitation |
| Blue | Common / GND |
| Black | Signal (TTL) |
| Shield | Ground |

### ⚠️ Wiring gotchas

- **5 V output into a 3.3 V ESP32-S3 pin is over-voltage.** Put a logic level
  shifter, resistor divider, or 3.3 V Schmitt buffer (74HC14) on the signal line.
  Do **not** wire the black signal wire straight to the GPIO.
- Power the brown wire from **5 V** (not 3.3 V) or the red emitter is too weak.
- The output is push-pull, so **no pull-up is needed** on the GPIO.

### Green "On-Target" LED

- Lights when the sensor sees a reflection.
- **Blinks once per revolution** when correctly aimed at the tape (good state).
- **Steady on with nothing spinning** = false trigger: too close (<1"), aimed at
  something shiny, or picking up ambient/fluorescent light.
- **Won't light at all** if pointed at magnetic/plain tape, unpowered, or misaimed.

### Reflective tape matters

- Use genuine **retroreflective** tape (bounces light straight back over a wide
  angle). Plain mirror/silver/foil tape only reflects at an exact perpendicular
  angle and gives a weak, fussy return.
- Flashlight test: retroreflective tape lights up brilliantly like a road sign;
  plain tape stays dull.

## How the firmware measures RPM

Period method: time between successive falling edges, then
`RPM = 60e6 / (period_us * PULSES_PER_REV)`.

Tunable config lives in `main/rpm.h` (`RPM_CONFIG_DEFAULT`):

- `min_pulse_us` — glitch/debounce reject. Caps max RPM at `60e6 / min_pulse_us`.
  Raise it to swallow noise bursts; lower it for higher RPM on a clean signal.
- `timeout_ms` — no edge for this long => reports 0 (stopped).
- `window` — number of recent periods median-filtered (smooths jitter, rejects a
  single bad period).

### Debugging a noisy signal

- Set `RPM_DEBUG_EDGES 1` in `main/rpm.c` to print every raw edge — use it to
  confirm edges are actually reaching the GPIO. Set back to `0` for normal runs.
- Wildly high RPM (hundreds of thousands) = the sensor is firing many edges per
  real pass (chatter / reflections / wrong target). Fix the signal, not the math.
- A slow hand-wipe over the tape is the **worst** test (the mark lingers in the
  beam and oscillates). Test with the real spinning target and one tape mark.
- **RPM stuck at 0?** The signal/power wires may be loose — check that the
  **signal wire is connected securely** (and that power/ground are seated too).
  A loose connection means no edges reach the GPIO, so RPM reads 0.

## Build / flash issues hit during setup (2026-06-01) and fixes

### 1) CMake: "Include directory .../main/include is not a directory"

- **Cause:** `main/CMakeLists.txt` declared `INCLUDE_DIRS "include"` (no such dir),
  and `SRCS` was empty so `main.c` / `rpm.c` were never compiled.
- **Fix:** set `SRCS "main.c" "rpm.c"` and `INCLUDE_DIRS "."` (`rpm.h` lives in
  `main/`, not `main/include`).

### 2) Compile: "fatal error: driver/gpio.h: No such file or directory"

- **Cause:** ESP-IDF v6 split the monolithic `driver` component into
  per-peripheral components; `driver/gpio.h` now comes from `esp_driver_gpio`.
- **Fix:** in `REQUIRES`, replaced `driver` with `esp_driver_gpio`
  (kept `esp_timer`, `esp_common`).
- **Note:** other peripherals need their own split components
  (e.g. `esp_driver_i2c`, `esp_driver_uart`, `esp_adc`).

### 3) Flash: "This chip is ESP32-S3, not ESP32. Wrong chip argument?"

- **Cause:** project configured for plain ESP32 but the board is an ESP32-S3.
- **Fix:** `idf.py set-target esp32s3`.

### 4) set-target won't take: "Target 'esp32s3' ... not consistent with target 'esp32' in the environment"

- **Cause:** an `IDF_TARGET=esp32` environment variable in the shell overrides the
  command-line target.
- **Fix:**

  ```bash
  unset IDF_TARGET
  idf.py set-target esp32s3
  idf.py build flash monitor
  ```

- Verify: `grep CONFIG_IDF_TARGET= sdkconfig` → expect `"esp32s3"`.
- If `IDF_TARGET` reappears in new terminals, remove the export from
  `~/.zshrc` / `~/.zprofile` / `~/.bashrc`.

### Clean-rebuild sequence if config gets stale

```bash
unset IDF_TARGET
idf.py fullclean
rm -f sdkconfig
idf.py set-target esp32s3
idf.py build flash monitor
```
