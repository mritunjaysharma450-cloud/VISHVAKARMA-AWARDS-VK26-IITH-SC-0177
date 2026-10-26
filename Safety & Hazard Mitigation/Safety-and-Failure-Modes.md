# Safety and Failure Modes — V-ORRA

Status: theory-based failure analysis, not a test report.
Fail-safe behaviour below must be proven by fault injection before any moving demo.

Prototype safety boundary, from the submitted deck: independent 5 V demonstration
hardware only, no direct modification of actual vehicle electrical systems,
validation in Stage 3, closed-course demo only.

## 1. Core safety rules, no exceptions

- On UNKNOWN, PREDICTED, stale, or link loss: lamp FREEZES, horn suppression LIFTS.
- Physical driver override always wins, electrically, and is logged.
- A command is time-limited. An expired command is not acted on.
- A duplicate, retained, or replayed command is not acted on as new.
- Prediction never overrides fail-open.

## 2. Failure modes

### Localisation error
Cause: radar angle error, UWB ambiguity, pole tilt, map error, GPS context error.
Effect: wrong carriageway, wrong zone, or a command to the wrong vehicle.
Mitigation in design: radar + UWB association, UNKNOWN on ambiguity, surveyed
zone map, transition allowance, dwell before actuation.
Test: known positions across both carriageways and the median, at 10 m and 20 m,
report lateral error, bias, 95th percentile, missed detections, and false-side
assignments separately. Ours to test.

### Wi-Fi / MQTT loss
Cause: interference, AP handover, broker crash, ESP32 reboot.
Effect: no new command, or a stale command lingering.
Mitigation: command expiry, sequence numbers, vehicle-side watchdog, fail-open
on link loss, no retained command acted on after expiry.
Test: disconnect Wi-Fi mid-zone, kill broker, reboot Pi, reboot ESP32, check
that horn suppression lifts and lamp freezes, every time.

### Power failure or reset
Cause: pole supply dip, vehicle bank cutoff, USB current limit on the Pi.
Effect: Pi reboot, gate reboot, lost track, lost command.
Mitigation: gate watchdog, Pi undervoltage check, hub power arrangement
recorded, demo rail separate from logic rail.
Test: cut pole power, cut vehicle power, brown-out, reboot during a zone,
check override still works and no unsafe command survives reboot.

### Relay / actuator malfunction
Cause: relay contact sticking, MOSFET failure, horn inrush, lamp overload.
Effect: demo horn stays suppressed, or demo lamp stays high, when it should not.
Mitigation: demo hardware only, isolated demo rail, fuse sized from measured
draw (not yet measured), override electrical priority, not software priority.
Test: force relay fault, force MOSFET fault, measure inrush and lamp draw,
check override still releases the demo output.

### Incorrect zone configuration
Cause: wrong polygon, wrong version, wrong pole ID, map not surveyed.
Effect: a vehicle is commanded in the wrong place, or not commanded where it
should be.
Mitigation: zone map version, survey date, pole ID in every map file,
version check on load, no live GPS-only zones.
Test: load a wrong-version map, load a map for the wrong pole, check that
the pole refuses it or logs it, and does not act on it.

### Pole instability or environment
Cause: wind, vibration, rain, heat, enclosure blocking radar, cable length.
Effect: radar tilt, false tracks, UWB range change, power loss.
Mitigation: pole ~2 m, PVC/GI under consideration, weighted base, enclosures
to be designed after component and PCB measurements. FEA not performed.
This is honestly pending, not claimed as done.
Test: physical stability evaluation in Stage 3, enclosure radar transparency
check, heat and weather check, grounding and DC distribution check.

### Identity collision or association error
Cause: two vehicles at similar UWB ranges, crossing tracks, intermittent tag.
Effect: command to the wrong vehicle, or a correct vehicle treated as unknown.
Mitigation: association only on strong evidence, UNKNOWN on ambiguity,
vehicle-specific MQTT topics, sequence per vehicle.
Test: two tags at similar ranges, crossing trajectories, intermittent
observations, measure wrong-identity rate and correct UNKNOWN rate.

### Outdoor radio use
Cause: radar and UWB transmission outside a controlled demo.
Effect: operation without authorisation.
Mitigation: lab and closed-course only until DoT / WPC authorisation and
closed-site permission are verified. Manufacturer documents alone do not
establish authorisation for this installation. Ours to verify, not assumed.

## 3. Standards, as submitted, with status

- IS/IEC 60529, IS/IEC 62368-1:2023, IEC 61508: listed in the submitted deck
  as applicable standards to be checked, not as certified compliance.
- DoT / WPC RF authorisation: to be verified before outdoor transmission.
- No claim of certification is made in this repo. Certification, if pursued,
  is Stage 3 work.
