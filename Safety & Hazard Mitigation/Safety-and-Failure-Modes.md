# Safety and Failure Modes — V-ORRA

This document is a theory-based failure analysis. It is not a test report.
The fail-safe behaviour described below remains to be proven by fault injection before any moving demonstration.

The prototype safety boundary, as stated in the submitted design specification, is independent 5 V demonstration hardware only, with no direct modification of actual vehicle electrical systems, validation in Stage 3, and closed-course demonstration only.

## 1. Core safety rules

- On UNKNOWN, PREDICTED, stale, or link loss: the lamp FREEZES and horn suppression LIFTS.
- Physical driver override always wins, electrically, and is logged.
- A command is time-limited. An expired command is not acted on.
- A duplicate, retained, or replayed command is not acted on as new.
- Prediction never overrides fail-open.

## 2. Failure modes

### Localisation error
Cause: radar angle error, UWB ambiguity, pole tilt, map error, or GPS context error.
Effect: wrong carriageway, wrong zone, or a command to the wrong vehicle.
Mitigation in design: radar + UWB association, UNKNOWN on ambiguity, surveyed zone map, transition allowance, and dwell before actuation.
Test required: known positions across both carriageways and the median, at 10 m and 20 m, with lateral error, bias, 95th percentile, missed detections, and false-side assignments reported separately. This test has not yet been performed.

### Wi-Fi / MQTT loss
Cause: interference, AP handover, broker crash, or ESP32 reboot.
Effect: no new command, or a stale command lingering.
Mitigation: command expiry, sequence numbers, vehicle-side watchdog, fail-open on link loss, and no retained command acted on after expiry.
Test required: Wi-Fi disconnected mid-zone, broker stopped, Pi rebooted, and ESP32 rebooted, with horn suppression lifting and the lamp freezing in every case.

### Power failure or reset
Cause: pole supply dip, vehicle bank cutoff, or USB current limit on the Pi.
Effect: Pi reboot, gate reboot, lost track, or lost command.
Mitigation: gate watchdog, Pi undervoltage check, hub power arrangement recorded, and demo rail separate from logic rail.
Test required: pole power cut, vehicle power cut, brown-out, and reboot during a zone, with override still working and no unsafe command surviving reboot.

### Relay / actuator malfunction
Cause: relay contact sticking, MOSFET failure, horn inrush, or lamp overload.
Effect: demo horn stays suppressed, or demo lamp stays high, when it should not.
Mitigation: demo hardware only, isolated demo rail, fuse sized from measured draw (not yet measured), and override electrical priority, not software priority.
Test required: forced relay fault, forced MOSFET fault, measured inrush and lamp draw, with override still releasing the demo output.

### Incorrect zone configuration
Cause: wrong polygon, wrong version, wrong pole ID, or map not surveyed.
Effect: a vehicle is commanded in the wrong place, or not commanded where it should be.
Mitigation: zone map version, survey date, pole ID in every map file, version check on load, and no live GPS-only zones.
Test required: a wrong-version map loaded, and a map for the wrong pole loaded, with the pole refusing it or logging it, and not acting on it.

### Pole instability or environment
Cause: wind, vibration, rain, heat, enclosure blocking radar, or cable length.
Effect: radar tilt, false tracks, UWB range change, or power loss.
Mitigation: pole of approximately 2 m, PVC/GI under consideration, weighted base, and enclosures to be designed after component and PCB measurements. FEA has not been performed.
Test required: physical stability evaluation in Stage 3, enclosure radar transparency check, heat and weather check, and grounding and DC distribution check.

### Identity collision or association error
Cause: two vehicles at similar UWB ranges, crossing tracks, or intermittent tag.
Effect: command to the wrong vehicle, or a correct vehicle treated as unknown.
Mitigation: association only on strong evidence, UNKNOWN on ambiguity, vehicle-specific MQTT topics, and sequence per vehicle.
Test required: two tags at similar ranges, crossing trajectories, and intermittent observations, with wrong-identity rate and correct UNKNOWN rate measured.

### Outdoor radio use
Cause: radar and UWB transmission outside a controlled demonstration.
Effect: operation without authorisation.
Mitigation: lab and closed-course only until DoT / WPC authorisation and closed-site permission are verified. Manufacturer documents alone do not establish authorisation for this installation. Authorisation is to be verified, not assumed.

## 3. Standards, as submitted, with status

- IS/IEC 60529, IS/IEC 62368-1:2023, IEC 61508: listed in the submitted design specification as applicable standards to be checked, not as certified compliance.
- DoT / WPC RF authorisation: to be verified before outdoor transmission.
- No claim of certification is made in this document. Certification, if pursued, is Stage 3 work.
