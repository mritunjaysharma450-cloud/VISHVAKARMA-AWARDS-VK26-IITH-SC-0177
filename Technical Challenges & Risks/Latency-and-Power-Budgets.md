# Latency and Power Budgets — V-ORRA Pole and Vehicle

Status: theory and evidence-based calculation, not measured results.
Every budget below must be measured on the bench before it is claimed as a result.

## 1. Latency budget — proposed allocation, 250 ms target

This is a design allocation, not a datasheet figure and not a measurement.

- Radar frame acquisition, processing, tracking, UART delivery: 95 ms — to be measured
- Pi track parsing, UWB association, zone decision: 40 ms — to be measured
- MQTT serialisation, publish, local broker handling: 10 ms — to be measured
- Wi-Fi delivery to vehicle ESP32-S3: 50 ms — to be measured
- ESP32 command validation and demo output actuation: 15 ms — to be measured
- Contingency: 40 ms — design allowance
- Total: 250 ms

Plausibility note: this is plausible only for one already-connected vehicle,
local MQTT, light track processing. It is not a worst-case guarantee. Radar
frame period and tracker confirmation, UWB ranging wait, Wi-Fi retries,
power-saving, interference, and MQTT reconnection can all push it over.
Measurement must be from sensing event to physical demo output transition,
not from MQTT publish alone, and must report median, 95th percentile, and
maximum, at 5, 10, and 20 km/h.

Travel in 250 ms:
- 5 km/h: 1.39 m/s x 0.25 s = 0.35 m
- 10 km/h: 2.78 m/s x 0.25 s = 0.69 m
- 20 km/h: 5.56 m/s x 0.25 s = 1.39 m

## 2. Power budget — what is known, what is not

Pole, known from official sources:
- Pi 5 bare board, typical active: 800 mA at 5 V = about 4 W, bare board only.
- Pi 5 USB peripheral limit: 1.6 A total with a 5 A supply, 600 mA with a 3 A supply.
  A hub, radar, and UWB bridge sharing that limit must be checked against the
  actual supply used, not assumed.
- ESP32-S3 gate, chip level: about 0.29 W in Wi-Fi RX, about 1.12 W peak in Wi-Fi TX.
  That is chip level, not full board draw.

Pole, unknown until measured:
- AWRL6432BOOST actual draw. TI notes USB power, and a 5 V / 2 A adapter as an
  option for higher-power cases. A 10 W-rated supply does not mean 10 W continuous.
- MaUWB anchor current. A 34 mA figure is reported for a tag configuration.
  It does not validate the pole anchor current, peaks, or supply arrangement.
- UGREEN hub model, idle draw, self-powered vs bus-powered.
- Conversion and wiring losses, pole supply topology.

Submitted pole figures, clarified:
- 100 W is a power station output capability. 156 Wh is stored energy.
  They are not the same thing, and neither is average system power.
- Ideal arithmetic at 100 W continuous: 156 Wh / 100 W = 1.56 h.
  That is not a predicted runtime. Actual runtime needs measured draw,
  converter losses, allowable discharge, and the exact source specification.

Vehicle, known and unknown:
- 10,000 mAh label, at an assumed nominal 3.7 V internal battery: about 37 Wh
  nominal, before conversion losses. It is not automatically 50 Wh at 5 V output.
- NEO-6M continuous operation is published at 111 mW at 3.0 V, chip level.
- ESP32-S3, UWB tag, regulators, demo lamp, and demo horn draws are unmeasured
  in the submitted documents. Horn inrush and lamp current must be measured
  separately, before fuse and runtime are claimed.

Measurement plan for power, in order:
1. Input voltage, average current, peak current, and voltage dips per rail,
   while sensing, transmitting, actuating, and rebooting.
2. Pi undervoltage behaviour under full USB load.
3. Hub self-powered vs bus-powered check.
4. Usable energy and cutoff behaviour of the pole source and vehicle bank.
5. Demo horn inrush and demo lamp draw, separately.

## 3. What to measure first

1. MaUWB bridge link: AT response and range reports, Pi and ESP32, at 3.3 V.
2. Radar TLV stream on the Pi: frame rate, loss, CPU and memory on the 1 GB Pi.
3. Lateral classification at 10 m and 20 m, then 30–40 m experimentally,
   against a surveyed reference, both carriageways and the median.
4. Association with two vehicles at similar ranges, and crossing tracks.
5. Fail-open injection: Wi-Fi loss, MQTT disconnect, stale and duplicate
   commands, Pi reboot, gate reboot, radar loss, UWB loss.
6. Latency distribution, as above.
7. Power per rail, as above.
