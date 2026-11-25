# User manual

How to build, test and use the Rocket V1 flight computer and the Ground Station V1 launch controller.

> **Firmware is not part of this project.** This manual covers everything the hardware defines. What the status LEDs, buzzer and launch button actually do depends on the firmware you load onto each ESP32. Steps that depend on firmware are marked **(firmware)**.

> **These boards have not been manufactured or tested yet.** Treat the first build as a prototype and work through the bench tests in order.

## Contents

1. [What you need](#1-what-you-need)
2. [Assembling the boards](#2-assembling-the-boards)
3. [Bench testing the Rocket V1](#3-bench-testing-the-rocket-v1)
4. [Bench testing the Ground Station V1](#4-bench-testing-the-ground-station-v1)
5. [Installing in the rocket](#5-installing-in-the-rocket)
6. [Launch-day procedure](#6-launch-day-procedure)
7. [Troubleshooting](#7-troubleshooting)

---

## 1. What you need

**Boards and parts.** Order the boards with the files in each project's `JLCPCB` folder (see [manufacturing.md](manufacturing.md)). The parts for each board are listed in its `_BOM.csv`.

**Off-board items for the Rocket V1**

| Item | Notes |
|---|---|
| ESP32 DevKit V1, 30-pin | Check the pin row spacing matches the socket before soldering |
| 2S LiPo battery with XT30 connector | 6.0–8.4 V |
| Two 5 V BEC modules | One for servos, one for logic. Set the logic BEC to 5.2–5.5 V if adjustable |
| Four hobby servos | Standard 3-pin leads: signal, +5 V, ground |
| GY-521 (MPU6050) module | 8-pin |
| microSD module, 3.3 V type | 6-pin. Do not use a 5 V-only module |
| Power switch | Rated 3 A or more, on a 2-pin lead |

**Off-board items for the Ground Station V1**

| Item | Notes |
|---|---|
| ESP32 DevKit V1, 30-pin | |
| 5 V supply | 5.2–5.5 V preferred |
| 12 V igniter battery | Not above 15 V |
| Key switch, 2-pole | One pole switches the 12 V line, the other tells the ESP32 the key is on |
| Two jumper plugs | One ARM plug (J3), one SAFE shunt (J6). Make them easy to tell apart |
| Launch button | Momentary, normally open, on a 2-wire lead |
| GPS, QMC5883L compass, BMP180 barometer, GY-521 modules | Optional, as your firmware needs |
| 3-wire umbilical cable | Wired pin 1 to pin 1, straight through |

**Tools.** Soldering iron, solder, side cutters, a multimeter, and a low-power 12 V test load such as a 12 V indicator bulb for testing the firing circuit.

---

## 2. Assembling the boards

Solder the shortest parts first so the board lies flat:

1. Resistors and diodes. Match each diode's stripe to the stripe on the silkscreen.
2. Small capacitors, transistors and LEDs. The long LED leg is positive.
3. The ESP32 socket strips. Plug them onto the DevKit first so they sit square.
4. Pin headers, fuse holders, test points and the buzzer.
5. Terminal blocks, the XT30 and the tall capacitors last. Match the stripe on each electrolytic capacitor to the marked negative side.

**Rocket V1 specifics**

- Q1 (IRF4905) lies flat on the board.
- J1 (XT30): the flat side of the connector is positive and is marked "+" on the board. **Check this against your battery lead before connecting a battery.**
- After testing, put a dab of hot glue or silicone under C5 and the other tall capacitors. Launch vibration can snap their leads.

**Ground Station V1 specifics**

- **NT1 must be fitted.** Solder a wire link or a 0 Ω resistor in its position. Without it the firing circuit cannot work.
- Q1 is an IRF3708 and Q4 is a PN2222A. Check each part's pin order against its datasheet before soldering.
- Use a red LED for D6 (ARMED) so it cannot be confused with the status LED D4.

**Before applying power to either board**

- Inspect every joint and look for solder bridges.
- With a multimeter on resistance, measure between each power test point and the GND test point. A reading near 0 Ω means a short: find it before going further.

---

## 3. Bench testing the Rocket V1

Do these in order. Stop and investigate if any step fails.

### Step 1: Power rails, with the ESP32, modules and servos unplugged

1. Fit the fuse and the power switch. Plug in both BECs.
2. Connect the battery and switch on.
3. Measure against the GND test point:

| Test point | Expect |
|---|---|
| VBAT (TP1) | Battery voltage, 6.0–8.4 V |
| 5V (TP2) | Logic BEC output minus about 0.3–0.4 V |
| Servo connector pin 2 (any of J5–J8) | Servo BEC output, about 5 V |

4. Switch off and disconnect the battery.

### Step 2: With the ESP32 fitted

1. Plug in the ESP32, USB port facing the way the silkscreen outline shows.
2. Power up and measure **3V3 (TP3): expect 3.3 V**.
3. The DevKit's power LED should light.

### Step 3: Modules **(firmware)**

1. Power off. Plug in the GY-521 and microSD modules, checking pin 1 of each against the header.
2. Insert a microSD card.
3. Load your firmware over USB and confirm it finds the IMU and can write to the card.

### Step 4: Servos **(firmware)**

1. Plug in one servo at a time, signal wire to pin 1.
2. Confirm each responds on the right channel: LEFT, RIGHT, UP, DOWN.
3. Move all four at once and watch that the ESP32 does not reset. If it does, check the servo BEC and C5.

---

## 4. Bench testing the Ground Station V1

**Never connect a real igniter on the bench.** Use a 12 V indicator bulb or similar low-power load on J7 for every test in this section.

### Step 1: Logic power, ESP32 unplugged, no igniter battery

1. Connect 5 V to J1 (pin 1 positive).
2. Measure **5V (TP2)**: expect the supply voltage minus about 0.3–0.4 V.
3. Disconnect power, fit the ESP32, reconnect and measure **3V3 (TP3): expect 3.3 V**.

### Step 2: Igniter supply and arming chain

Start with the ARM plug out, key off and SAFE shunt out.

1. Connect the 12 V battery to J2 (pin 1 positive).
2. Measure **12V (TP1)**: expect battery voltage minus about 0.5 V.
3. **ARMED LED must be off.**
4. Fit the ARM plug only: ARMED LED stays off.
5. Remove the ARM plug and turn the key on only: ARMED LED stays off.
6. Fit the ARM plug and turn the key on: **ARMED LED lights.**
7. Remove either one: the LED goes out.

If the ARMED LED lights with only one of the two fitted, stop. The arming chain is miswired.

### Step 3: Continuity check

With the board armed and a multimeter on the ESP32 socket pin IO35 (measured against GND):

| Setup | Expect |
|---|---|
| Nothing on J7 | 0 V |
| Test bulb on J7 | About 1.3 V with a 12 V battery |
| SAFE shunt fitted, nothing on J7 | About 1.3 V |

The test bulb must not glow at any point in this step.

### Step 4: Firing switches, without firmware

This proves both switches work using a jumper wire in place of the ESP32.

1. Disarm (ARM plug out, key off). Remove the ESP32 from its socket.
2. Connect the test bulb to J7. Leave the SAFE shunt out.
3. Power the board from 5 V. With the ESP32 removed there is no 3.3 V on the board, so take 3.3 V from a bench supply or a separate DevKit's 3V3 pin, sharing ground with the board.
4. Arm the board. The ARMED LED lights; the bulb stays off.
5. Touch 3.3 V to socket pin **IO25** only: bulb stays off.
6. Touch 3.3 V to socket pin **IO33** only: bulb stays off.
7. Touch 3.3 V to **both** pins: **the bulb lights.**
8. Remove either: the bulb goes out.
9. Disarm.

If the bulb lights with only one pin driven, one of the firing transistors is shorted or fitted wrongly. Do not use the board until it is fixed.

### Step 5: Inputs and outputs **(firmware)**

1. Refit the ESP32 and load your firmware.
2. Confirm the firmware reads the key position (IO32), the launch button (IO4, low when pressed), battery voltage (IO39) and continuity (IO35).
3. Confirm the buzzer (IO19) and status LED (IO18) work.
4. Run a full firing sequence into the test bulb.
5. Reset the ESP32 while armed, several times. **The bulb must never flash during boot.**

### Step 6: Umbilical **(firmware)**

Connect J13 on the Ground Station to J11 on the Rocket with the 3-wire cable, pin 1 to pin 1, and confirm the two boards exchange data.

---

## 5. Installing in the rocket

- Mount the Rocket V1 on its four M3 holes with standoffs. Nothing metal should touch the underside.
- Secure the ESP32 in its socket with a cable tie or tape, or solder it in for flight.
- Mount the board rigidly and note its orientation. The IMU measures the board's motion, so your firmware needs to know which way it faces.
- Tie down the battery and every lead. A loose battery will shift the rocket's balance and can pull connectors apart.
- Route the umbilical so it pulls free cleanly at lift-off without snagging.
- Check the rocket's balance point with everything installed.

---

## 6. Launch-day procedure

Follow your national model-rocketry safety code and your club's rules. Where this manual and the code differ, the code wins. For reference, the US National Association of Rocketry code requires at least 5 m (15 ft) from the pad for D motors and smaller and 9 m (30 ft) for larger motors.

This hardware puts the ARM plug, key switch and SAFE shunt on the Ground Station board, which sits at the pad. Your safe distance therefore comes from a long launch-button lead on J8 or a wireless command **(firmware)**. Decide which before launch day and test it on the bench.

**The three safety devices**

| Device | Safe state | What it does |
|---|---|---|
| SAFE shunt (J6) | Fitted | Shorts the igniter leads together |
| ARM plug (J3) | Removed | Breaks the 12 V supply |
| Key switch (J4) | Off, key in your pocket | Breaks the 12 V supply a second time |

### Before leaving for the field

- Charge both batteries. Pack spare 3 A fuses for both boards.
- Run the bench firing test into the bulb once more.
- Pack the ARM plug, SAFE shunt and key together.

### Setting up at the pad

1. Confirm: **SAFE shunt fitted, ARM plug removed, key off and in your pocket.**
2. Place the Ground Station and connect its 5 V supply.
3. Connect the 12 V igniter battery. The ARMED LED must be off.
4. Put the rocket on the pad. Switch on the Rocket V1 and connect the umbilical.
5. Wait for both boards to report ready **(firmware)**.

### Connecting the igniter

6. Check again that the ARMED LED is off and the SAFE shunt is fitted.
7. Install the igniter in the motor and connect its leads to J7.
8. Make sure the igniter clips do not touch each other or the launch rod.

### Arming

9. Clear everyone else from the pad area.
10. Fit the ARM plug.
11. Turn the key on. **The ARMED LED lights.** From this moment, treat the igniter as live.
12. Check the ARMED LED is still lit. If it went out when you armed, the fuse has blown, which means the firing switches are conducting when they should not be. Disarm and do not launch.
13. Remove the SAFE shunt last.
14. Walk directly to the launch position. Do not linger at the pad.

### Launching

15. Confirm continuity and battery voltage **(firmware)**.
16. Check the sky and the range are clear. Give a loud countdown.
17. Press the launch button **(firmware)**.

### After launch

18. Return to the pad. Turn the key off and remove it, pull the ARM plug, and fit the SAFE shunt.
19. Check the Ground Station's fuse F1. A low-resistance igniter can blow it during firing; replace it if so.
20. Disconnect the igniter battery.

### If the motor does not ignite

1. **Do not approach.** Wait at least 60 seconds.
2. Approach from the side. First turn the key off and remove it, then pull the ARM plug, then fit the SAFE shunt.
3. Confirm the ARMED LED is off before touching the igniter.
4. Replace the igniter and start again from "Connecting the igniter".

---

## 7. Troubleshooting

| Symptom | Likely cause | Check |
|---|---|---|
| **Rocket V1** | | |
| Nothing powers up | Fuse blown, switch open, battery reversed | Fuse F1, switch on J2, XT30 polarity |
| VBAT present but no 5 V | Logic BEC not connected or D1 reversed | J4 pin 3 voltage, D1 stripe |
| ESP32 resets when servos move | Servo and logic supplies sharing a BEC, or weak servo BEC | Both BECs fitted, C5 polarity |
| SD card errors or resets while logging | 5 V-only SD module, or low 3.3 V | Module type, voltage at TP3 while writing |
| A servo doesn't respond | Lead reversed, or wrong channel | Signal wire on pin 1 |
| Battery reading is wrong | Divider resistors swapped | R1 is 100k, R2 is 47k |
| **Ground Station V1** | | |
| ARMED LED never lights | Fuse blown, D5 reversed, key switch on wrong header, LED reversed | TP1 voltage, then voltage across J4 |
| ARMED LED lights with only plug or only key | Arming chain miswired | Stop. Recheck J3 and J4 wiring |
| Continuity always reads 0 V | Board not armed, igniter open, NT1 missing | Arm first; NT1 fitted |
| Continuity reads about 2.8 V | High-side switch Q2 failed short | Replace Q2 before any further use |
| Fires into bulb with one pin driven | Q1 or Q2 shorted or reversed | Replace before any further use |
| Won't fire, continuity good | Fuse blown, SAFE shunt still fitted, weak battery, NT1 missing | F1, J6, battery voltage under load |
| Fuse blows on firing | SAFE shunt left in, or igniter leads shorted | Remove shunt, separate the clips |
| Buzzer silent | Q4 fitted backwards, buzzer reversed | Buzzer "+" to the marked pad |
| No GPS data | Module pin order differs from J12 | See known issues on the Ground Station page |
| ESP32 brown-outs on WiFi | 5 V supply too low after D1 | Supply 5.2–5.5 V |
