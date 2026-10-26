# Test Protocols — V-ORRA, closed course only

Status: protocols only. No results are claimed. Results log stays blank until
tests are actually run. Do not fill results in advance.

## General rules for every test
- Closed course, surveyed site, no public road, no vehicle electrical modification.
- Demo lamp and demo horn only, on demo battery. Driver override present and tested first.
- Record: date, site, pole ID, zone map version, vehicle ID, speed, weather,
  observer, and every raw log file name. A test without a log is not a result.
- Pass criteria are written before the test, not after seeing the data.

## T1 — MaUWB link, bench, no vehicle moving
Setup: bare MaUWB anchor via USB-to-UART bridge to Pi, bare MaUWB tag via UART
to ESP32-S3, both at 3.3 V, common ground, 115200 baud.
Check: AT response, range reports, range vs tape-measured distance at 1, 2, 5, 10 m.
Pass: AT response consistent, range error recorded, no 5 V on any MaUWB pin.
Fail: no AT response, range jumping without movement, or any voltage above 3.3 V.

## T2 — Radar on Pi, bench and static
Setup: AWRL6432BOOST by USB to Pi, TI SDK demo, 1 GB Pi.
Check: enumeration, TLV stream, frame rate, packet loss, Pi CPU and memory.
Pass: continuous TLV stream for 10 minutes, frame rate recorded, loss rate recorded.
Then static targets at 5, 10, 20 m, on both sides of a centre line.

## T3 — Lateral classification, static then walking
Setup: surveyed positions across both 7 m carriageways and the 1.5 m median,
pole at centre, radar mounting direction recorded.
Check: estimated position vs surveyed position, per position.
Record separately: absolute lateral error, bias, 95th percentile, missed
detections, false-side assignments. Repeat at 10 m, then 20 m, then 30–40 m
experimentally. Do not claim 30–40 m until T3 passes at 20 m.

## T4 — Association, deliberate ambiguity
Setup: one stationary tag, one moving tag, then two moving tags.
Check: correct track-to-tag association, wrong-identity rate, and correct
UNKNOWN rate when ranges are similar or tracks cross.
Pass criteria, to be set before running: wrong-identity rate below the
threshold set in the test plan, and UNKNOWN used instead of a guessed identity.

## T5 — Latency distribution
Setup: timestamps at radar observation, UART receipt, Pi decision, MQTT
publish, ESP32 receipt, demo output transition. Use monotonic timestamps,
logic analyser or GPIO capture where feasible.
Check: median, 95th percentile, maximum, at 5, 10, and 20 km/h, repeated
zone entries, plus Wi-Fi disconnect / reconnect cases.
Pass for the deck target: 95th percentile below 250 ms in a controlled
environment, with sample count recorded. A single fast run is not a pass.

## T6 — Fail-open injection
Inject, one at a time: Wi-Fi loss, MQTT disconnect, stale command, duplicate
command, retained / replayed command, Pi reboot, gate reboot, radar loss,
UWB loss, power dip.
Check, every time: horn suppression lifts, lamp freezes, override works,
no unsafe command survives reboot. Any exception is a fail, not a note.

## T7 — Demo actuation, static
Setup: demo lamp and demo horn on demo battery, MOSFET / relay drivers.
Check: lamp high to low in LBZ, horn suppression in NHZ, only on confirmed
associated non-predicted track after dwell. Measure lamp current and horn
inrush separately, check fuse sizing from measured draw, check override
electrical priority, not software priority.

## T8 — Campus road, bike-mounted vehicle module
Setup: vehicle module mounted on a bike, closed campus road, surveyed zones,
speeds 5–10 km/h first, up to 20 km/h only if T3–T6 pass.
Check: zone entry and exit commands, latency, position error, override use,
and log completeness. No public road, no car electrical modification.

## T9 — Multi-vehicle, mock modules
Setup: 2–3 mock vehicle modules, then real modules if available.
Check: 4 software tracking slots, separate sequence per vehicle, no duplicate
commands, no identity collision, pole-to-pole handover over Ethernet.
Pass: each vehicle gets only its own commands, handover does not duplicate.

## T10 — Power, measured
Setup: per-rail voltage, average current, peak current, voltage dips,
while sensing, transmitting, actuating, and rebooting.
Check: Pi undervoltage, hub power arrangement, usable energy and cutoff
of pole source and vehicle bank, horn inrush and lamp draw separately.
Runtime is calculated from measured draw, not from 100 W / 156 Wh labels.

## Results log — blank, to be filled only after testing
Test | Date | Site | Result | Log file | Notes
--- | --- | --- | --- | --- | ---
T1 | | | | |
T2 | | | | |
T3 | | | | |
T4 | | | | |
T5 | | | | |
T6 | | | | |
T7 | | | | |
T8 | | | | |
T9 | | | | |
T10 | | | | |
