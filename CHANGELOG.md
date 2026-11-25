# Changelog

## V1 (2026-11-24)

First release of the Rocket Launcher hardware, designed by Ashton W.

### Rocket V1 flight computer

- ESP32 DevKit driving four canard servos, with an MPU6050 IMU and microSD logging.
- XT30 battery connector, 3 A fuse and reverse-polarity protection.
- Separate servo and logic BECs.
- Capacitors on the 3.3 V supply at the microSD header to absorb the card's write-current pulses.
- Test points for battery, 5 V, 3.3 V and ground.
- Status LED on IO2.

### Ground Station V1 launch controller

- ESP32 DevKit with a two-transistor igniter firing circuit behind an ARM plug and key switch.
- Hardware ARMED LED driven directly by the armed 12 V line.
- Igniter continuity check and igniter battery voltage measurement.
- Headers for GPS, compass, barometer and IMU modules, a launch button and a buzzer.
- Test points for 12 V, 5 V, 3.3 V and ground.
- Fire transistor drive sized so both switches turn on fully from a sagging battery and a 3.3 V signal.
