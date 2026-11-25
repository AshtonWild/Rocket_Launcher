# How it works

A circuit-by-circuit description of both boards. Part numbers refer to the schematics in [schematics/](schematics). Pinouts and ESP32 pin tables are on the board pages: [Rocket V1](rocket-v1.md) and [Ground Station V1](ground-station-v1.md).

Both boards are built around a socketed 30-pin ESP32 DevKit V1. The DevKit provides the processor, WiFi, USB port and a 3.3 V regulator. Everything else on the boards is power handling, protection and connectors.

---

## Rocket V1 flight computer

### Battery input and protection

```
J1 (XT30) ─ F1 ─ Q1 ─ J2 (power switch) ─ VBAT
```

- **J1** takes a 2S LiPo (6.0–8.4 V). Pin 2 is positive.
- **F1** is a 3 A fast fuse. It is sized below the roughly 3 A rating of the J2 header, which carries all the board's current.
- **Q1 (IRF4905)** blocks a reversed battery without wasting voltage the way a diode would. It is a P-channel MOSFET with its gate tied to ground through R15 (1k). With the battery the right way round, current first flows through the transistor's internal diode, the source rises to battery voltage, the gate sits a full battery voltage below it, and the transistor switches on fully. With the battery reversed, the gate is never below the source, so the transistor stays off and its internal diode blocks.
- **J2** is the power switch, in series with everything downstream.

### Power rails

| Rail | Source | Feeds |
|---|---|---|
| VBAT | Battery, after the switch | Both BEC inputs, battery voltage divider, TP1 |
| Servo 5 V | Servo BEC output (J3 pin 3) | The four servo connectors only |
| +5 V | Logic BEC output (J4 pin 3) through D1 | ESP32 VIN, TP2 |
| +3.3 V | ESP32 DevKit's on-board regulator | IMU, microSD, pull-ups, TP3 |

- **Two BECs** keep servo current out of the logic supply. A stalled servo can pull the servo rail down without resetting the ESP32.
- **C5 (1000 µF) and C6 (100 nF)** sit on the servo rail to supply the short current spikes servos draw when they start moving.
- **D1 (1N5819)** stops the USB port from back-feeding the logic BEC when the DevKit is plugged into a computer. It costs about 0.3–0.4 V, which is why the logic BEC should be set to 5.2–5.5 V.
- **C1 (100 µF) and C2 (100 nF)** smooth the 5 V rail at the ESP32.
- **C3 (100 nF)** decouples 3.3 V at the ESP32. **C7 (47 µF) and C8 (100 nF)** hold up the 3.3 V supply to the microSD header, because SD cards draw sharp current pulses when writing.

### Battery voltage measurement

R1 (100k) and R2 (47k) divide the battery voltage to 32% of its value so it fits the ESP32's 0–3.3 V input range. C4 (100 nF) filters noise. The result goes to IO35.

| Battery | At IO35 |
|---|---|
| 8.4 V (full) | 2.69 V |
| 7.4 V (nominal) | 2.37 V |
| 6.0 V (empty) | 1.92 V |

Battery voltage = pin voltage × 3.13.

### Servo outputs

Each of the four servo signals follows the same pattern:

```
ESP32 pin ─┬─ 330 Ω ─ servo signal pin
           └─ 10k to ground
```

- The **10k pull-down** holds the signal low while the ESP32 is booting or in reset, so servos don't twitch on undefined signals.
- The **330 Ω series resistor** limits current if a servo lead is shorted or plugged in backwards, and damps ringing on the cable.

| Servo | ESP32 pin | Series | Pull-down |
|---|---|---|---|
| LEFT (J5) | IO26 | R3 | R7 |
| RIGHT (J6) | IO25 | R4 | R8 |
| UP (J7) | IO27 | R5 | R9 |
| DOWN (J8) | IO32 | R6 | R10 |

The 3.3 V signal from the ESP32 is enough for standard hobby servos running on 5 V.

### IMU (J9)

A GY-521 module (MPU6050 accelerometer and gyroscope) plugs into J9 and talks over I²C on IO21 (SDA) and IO22 (SCL). The module carries its own pull-up resistors. Its AD0 pin is grounded, which sets its address to 0x68. Its interrupt output goes to IO34.

### microSD (J10)

A 3.3 V microSD module plugs into J10 and uses the ESP32's standard SPI pins: IO18 clock, IO19 data out of the card, IO23 data into the card, IO4 chip select. R11 (10k) pulls chip select high so the card stays deselected during boot.

### Umbilical (J11)

A 3-wire serial link to the Ground Station on the ESP32's second serial port: IO16 receives, IO17 transmits. R12 and R13 (330 Ω) protect the pins against shorts and static on the cable. R14 (10k) pulls the receive line high so it reads as idle when the cable is unplugged.

### Status LED (D2)

IO2 drives D2 through R16 (330 Ω), giving about 4 mA. IO2 also drives the DevKit's on-board blue LED, so both light together.

### Ground

Both sides of the board are filled with ground copper, joined by about 70 small vias. This gives every signal a short return path and keeps servo noise out of the sensors.

---

## Ground Station V1 launch controller

### Logic power

5 V enters on J1 and passes through **D1 (1N5819)**, which protects against a reversed supply and stops USB from back-feeding it. **C1 (100 µF) and C2 (100 nF)** smooth the rail. The ESP32 DevKit's regulator makes 3.3 V for the sensor modules and pull-ups.

### Igniter supply chain

```
J2 + ─ D5 ─ F1 ─┬─ J3 ARM plug ─ J4 key switch ─ "armed 12 V"
                └─ TP1, battery voltage divider
```

- **D5 (1N5822)** blocks a reversed igniter battery.
- **F1** is a 3 A fast fuse. It protects the board and wiring if the igniter leads short.
- **J3 (ARM plug)** and **J4 (key switch, pole A)** are two independent mechanical breaks in the positive supply. With either open, no part of the firing circuit has power.
- Everything after the key switch is the **armed 12 V** line.

### Firing switches

The igniter sits between two electronic switches, one on each side:

```
armed 12 V ─ Q2 (high side) ─ J7 + ··· igniter ··· J7 − ─ Q1 (low side) ─ igniter battery −
```

Both must be on for current to flow. A single failed transistor or a single stuck ESP32 pin cannot fire the igniter.

**High side, Q2 (IRF4905, P-channel).** R5 (100k) ties its gate to the armed 12 V line, which holds it off. To turn it on, the ESP32 raises FIRE_HIGH (IO25). That switches on the small transistor Q3 (2N7000), which pulls Q2's gate down through R6 (10k). With a 12 V battery the gate ends up about 10.9 V below the source, switching Q2 on fully. R7 (1k) and R8 (10k) are Q3's gate resistor and pull-down.

**Low side, Q1 (IRF3708, N-channel).** The ESP32 raises FIRE_LOW (IO33), which drives the gate through R9 (1k). R10 (10k) pulls the gate low the rest of the time. The IRF3708 was chosen because it is rated to switch on fully from under 3 V.

Both ESP32 pins have pull-downs and are not driven during boot or reset, so both switches default to off.

### Igniter terminals

- **J7** is the igniter (e-match) terminal block.
- **J6 (SAFE shunt)** is wired directly across J7. A jumper here shorts the igniter leads together so stray current cannot heat the igniter. If the board fires with the shunt fitted, the current goes through the shunt and the fuse blows.
- **D2 (P6KE18CA)** is a surge suppressor from the igniter's positive side to igniter ground. Long igniter leads act as an inductor and produce a voltage spike when the current stops; D2 clamps it to protect Q2.

### Continuity check

```
armed 12 V ─ R11 470k ─ J7 + ··· igniter ··· J7 − ─ R12 330k ─┬─ IO35
                                                              └─ R13 100k ─ ground
```

A tiny current, about 13 µA, flows through the igniter and into a voltage divider. That is roughly ten thousand times too small to fire an igniter. C6 (100 nF) filters the reading.

| Situation | At IO35 (12 V battery) |
|---|---|
| Not armed | 0 V |
| Armed, no igniter | 0 V |
| Armed, igniter connected | about 1.3 V |
| Armed, SAFE shunt fitted | about 1.3 V, with or without an igniter |
| Armed, Q2 failed short | about 2.8 V |

The check only works when the board is armed, because the test current comes from the armed 12 V line.

### Igniter battery measurement

R3 (100k) and R4 (27k) divide the igniter battery voltage to 21.3% and feed IO39. C5 (100 nF) filters it. The divider is connected before the ARM plug, so it reads whenever the battery is connected.

| Battery | At IO39 |
|---|---|
| 12.0 V | 2.55 V |
| 14.4 V | 3.06 V |

Battery voltage = pin voltage × 4.70. Do not use a battery above about 15 V.

### ARMED LED (D6)

D6 (red) and R23 (2.2k) run from the armed 12 V line to igniter battery negative, drawing about 4.5 mA. The LED is purely hardware: it lights whenever the ARM plug and key switch are both closed and the battery is connected, even with the ESP32 unplugged.

### Key switch sense

The key switch's second pole (J5) connects 3.3 V to IO32 through R1 (1k). R2 (10k) pulls the pin low when the key is off and C4 (100 nF) filters it. This lets the firmware know the key position without touching the 12 V side.

### Launch button (J8)

R14 (10k) pulls the signal up to 3.3 V. Pressing the button connects it to ground, so IO4 reads low when pressed. R15 (1k) and C7 (100 nF) protect the pin and filter contact bounce and noise picked up on a long button lead.

### Buzzer and status LED

- **Buzzer (BZ1).** A 5 V active buzzer switched by Q4 (PN2222A). IO19 drives Q4's base through R17 (1k); R18 (10k) holds it off otherwise. D3 (1N5819) across the buzzer absorbs its switch-off spike.
- **Status LED (D4).** IO18 drives it through R16 (330 Ω).

### Sensors

- **I²C bus (IO21 SDA, IO22 SCL).** Shared by J9 (GY-521 IMU), J10 (QMC5883L compass) and J11 (BMP180 barometer). The modules carry their own pull-ups.
- **GPS (J12).** Receive-only: the GPS module's transmit pin goes to IO34. R19 (10k) pulls the line high so it reads as idle with no module fitted.

### Umbilical (J13)

The mirror image of the rocket's umbilical connector: IO17 transmits on pin 1 and IO16 receives on pin 2, each through 330 Ω (R20, R21), with R22 (10k) pulling the receive line high. A straight cable connects the two boards.

### Grounding

The igniter circuit has its own ground, separate from the logic ground copper. They are joined at exactly one point, the link **NT1**. Firing current flows from the battery through the igniter and back without passing through the ground under the ESP32 and sensors. NT1 must be fitted (a wire link or 0 Ω resistor) or the firing switches have no reference and will not work.

The four mounting holes are plated and connected to logic ground.
