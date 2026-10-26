# Theoretical Stress Testing — 16 Scenarios

This document is theoretical analysis only, from the submitted design specification's Maker Journey account of 16-scenario theoretical stress testing. No physical test results are presented. Each scenario states the expected system behaviour in theory, and what physical test would be needed to confirm it. If theory and test disagree, the test wins.

Road: 15.5 m total, 2 x 7 m carriageways + 1.5 m median. Pole at centre. Speeds: 5–10 km/h initial, up to 20 km/h stretch. Latency target 250 ms.

## Scenario list

1. Single vehicle, correct carriageway, NHZ entry, slow speed.
   Theory: confirmed track, associated UWB session, dwell passed, horn suppression command sent, demo horn suppressed, override still available.
   Test needed: T3 + T7 + T8.

2. Single vehicle, correct carriageway, LBZ entry, slow speed.
   Theory: demo lamp switches high to low on confirmed command.
   Test needed: T3 + T7 + T8.

3. Single vehicle, wrong carriageway for the zone in question.
   Theory: track is on the other carriageway, no zone command for that zone. Mirror ambiguity must not flip this.
   Test needed: T3, both sides.

4. Vehicle on the median line, between carriageways.
   Theory: position is ambiguous, result UNKNOWN, no new restrictive command, lamp freezes, horn suppression lifts.
   Test needed: T3, median positions.

5. Two vehicles, same carriageway, one behind the other, both equipped.
   Theory: two separate tracks, two separate UWB sessions, two separate MQTT sequences, each vehicle gets only its own command.
   Test needed: T4 + T9.

6. Two vehicles, opposite carriageways, similar UWB ranges to the pole anchor.
   Theory: ranges alone cannot separate them. Radar tracks must separate them, or result is UNKNOWN for the ambiguous vehicle. No guessed identity.
   Test needed: T4, deliberate ambiguity.

7. Two vehicles crossing in front of the pole, tracks crossing.
   Theory: track IDs must not swap, UWB sessions must not swap, commands must not swap vehicles. If a swap is possible, result is UNKNOWN.
   Test needed: T4, crossing trajectories.

8. Vehicle enters zone, then Wi-Fi drops mid-zone.
   Theory: link loss detected, watchdog expires, lamp freezes, horn suppression lifts, no stale command is re-used on reconnect.
   Test needed: T6.

9. Vehicle enters zone, then MQTT broker restarts.
   Theory: same as scenario 8, plus no retained command is acted on after expiry, and sequence restarts safely.
   Test needed: T6.

10. Stale command arrives after expiry, for example delayed by network.
    Theory: ESP32 checks expiry and sequence, rejects the command, falls back to fail-open.
    Test needed: T6, delayed / replayed commands.

11. Duplicate command arrives twice.
    Theory: second copy is recognised by sequence number, ignored as new, demo output does not flicker.
    Test needed: T6.

12. Driver override pressed during horn suppression.
    Theory: override wins electrically, demo horn returns to driver control immediately, event is logged, software does not override the driver.
    Test needed: T6 + T7, override timing measured.

13. Driver override pressed during LBZ lamp low.
    Theory: same as scenario 12, for the lamp. Lamp returns to driver control, event logged.
    Test needed: T6 + T7.

14. Pole power dip or Pi reboot during a zone.
    Theory: gate watchdog expires, no unsafe command survives reboot, vehicle falls back to fail-open, session is re-established cleanly, not from stale state.
    Test needed: T6, power dip and reboot.

15. Radar loss, UWB still present, or UWB loss, radar still present.
    Theory: association cannot be confirmed on one layer alone, result is UNKNOWN, no new restrictive command. Single-layer operation is not presented as zone proof.
    Test needed: T6, single-layer loss.

16. Vehicle at 20 km/h, zone entry at the edge, latency at the limit.
    Theory: in 250 ms the vehicle travels 1.39 m, so the command lands about 1.4 m past the sensed position, before position error. Transition allowance must cover this, or the zone boundary must be set with that allowance in the map. Prediction, if used, must never override fail-open on a PREDICTED track.
    Test needed: T5, at 20 km/h, repeated entries, latency distribution.

## Summary, in theory

Scenarios 1 and 2 are the normal cases. Scenarios 3–7 test whether the system can tell vehicles and carriageways apart, or correctly say UNKNOWN. Scenarios 8–15 test whether failure always lands on the safe side. Scenario 16 tests whether the timing and the map allowance actually fit together at the highest stretch speed.

None of these 16 are passed until the corresponding physical test in Test-Protocols.md is run and logged. Theory passing is not a pass.
