# Vehicle Module Software — Refined Design (v1.1)

Status: design only, not bench-tested. Firmware for the ESP32-S3 DevKit
controller in the revised vehicle schematic (SN74HC74 lamp latch,
CD74HC423 horn deadman, 4PDT override S2, SPDT manual S4, relay K1).
This document pairs with the Pole Unit Software Design Refined v1.1 and
the MQTT spec v1.1. Gate firmware, key provisioning, and bench measurements
are NOT supplied yet.

Demo cap until measured: **≤10 km/h only**, same as pole. Vehicle logic
does not change the speed cap — it enforces it by failing open when
timing, sync, or authority is not proven.

## 1. What the vehicle controller does — and does not do
- DOES: Wi-Fi client to the pole AP, MQTT client, command validation,
  horn/lamp state machines, deadman pulses, latch strobe, override sense,
  GPS context parsing, logging, status/ack/override publishing.
- DOES NOT: parse UWB ranges, run association, decide zones, or invent
  commands. The bare MaUWB vehicle tag reports ranges directly to the pole
  anchor over UWB — the ESP32 does not receive UWB ranges and must not
  claim to. Zone decisions arrive only as signed MQTT commands from the
  pole gate.
- DOES NOT: wire into, read from, or drive the real vehicle's horn,
  headlamp, or battery. All actuation is the auxiliary demo unit on its
  own 12 V pack, on its own returns, as drawn in the schematic.
  Closed-course demo only.

## 2. Firmware modules (ESP32-S3, ESP-IDF, one task per module)
```
wifi_task ──> mqtt_task ──> validator ──> control_task ──┐
gps_task ──> gps_context (broad only)                     ├─> horn_task ──> GPIO8 (coil, via Q2)
override_task ──> GPIO9 sense                             ├─> deadman_task ─> GPIO10 pulses (via U7)
                                                              └─> lamp_task ──> GPIO7 mode + GPIO11 strobe (via U5)
logger_task ──> flash ring buffer + UART log
health_task ──> status topic, Wi-Fi RSSI, heap, uptime
```
No module may drive a GPIO directly except horn_task, deadman_task, and
lamp_task, and only through control_task state. Validation failure,
override, fault, or timeout must stop all three drivers, in hardware
where the schematic provides it (deadman, latch, relay NC contact).

## 3. Pins (from the revised schematic, not guessed)
- UART0, GPIO43 TX / GPIO44 RX: programming / debug only, no vehicle logic.
- UART1, GPIO17 TX / GPIO18 RX: bare MaUWB tag — **tag is not wired to
  the ESP32 in the pole-vehicle link design**; the tag's UART is for
  configuration / bench only. Vehicle firmware must not depend on tag UART
  data for actuation. (If a future revision wires tag UART to the ESP32,
  this document must be revised and the schematic re-checked.)
- UART2, GPIO4 TX / GPIO5 RX: NEO-6M GPS, 9600 baud NMEA, PPS unused.
  GPS is broad context only, never sole zone input, never a tie-breaker.
  Datasheet values (2.5 m CEP, up to 5 Hz) are datasheet values, not
  measured vehicle performance.
- GPIO7: LAMP_MODE (mode data to SN74HC74 U5, pin 2, via R10).
- GPIO11: LAMP_STROBE (update strobe to U5, pin 3, via R11).
- GPIO8: HORN_SUPPRESS (coil command to Q2, via R3).
- GPIO10: HORN_WDI (deadman edges to CD74HC423 U7, pin 2).
- GPIO9: OVERRIDE_N sense (S2D, via R5, debounced in firmware).
- GPIO0, 3, 45, 46 (strapping), 35–37 (memory): kept free, as drawn.
  No ADC2 analogue use (ADC2 stops with Wi-Fi).
- MaUWB RESET / WAKEUP and exact pad mapping: OPEN, do not guess.
- GPS breakout rail: chip rail must remain 2.7–3.6 V — the 3.3 V VCC
  into a breakout expecting 5 V is a known trap, resolved only by
  checking the actual breakout (own LDO vs direct chip rail) before soldering.

## 4. Wi-Fi and MQTT client behaviour
- Wi-Fi client to the pole Pi AP, 2.4 GHz, as drawn. Association and
  MQTT time-to-first-command at speed are the highest-risk path in the
  whole system (pole interval 2.88 s at 50 km/h in earlier work; at
  ≤10 km/h the interval is longer, but still must be measured).
  Honest fallback if association + MQTT first-command fails at speed:
  static / walking-speed demo, never shortened dwell or predicted-state
  actuation.
- MQTT client: MQTTv5, QoS 1, Clean Start, SessionExpiryInterval = 0.
  Subscribe: `vorra/v1/vehicles/<vehicle_id>/commands` only.
  Publish: acks, status (retain Yes, Last Will OFFLINE), override.
- Reconnect: exponential back-off, starting 1 s, max 30 s, to calibrate.
  On Wi-Fi loss, broker loss, or MQTT disconnect, the vehicle does NOT
  hold suppression — it fails open by its own timers (§7), even if the
  last command said suppress. Link loss is a fail-open event, not a hold.
- Time sync: vehicle keeps a monotonic clock for all expiry decisions.
  Pole wall-clock expires_at is converted with the last sync offset,
  only while sync-valid (starting expiry 60 s, same as pole). If sync is
  invalid or expired, new commands are rejected (REJECTED_AUTHORITY /
  REJECTED_UNCERTAIN), and existing suppression still ends at its own
  t_end (§7). Vehicle must never act on a command whose absolute expiry
  it cannot verify.

## 5. Command validation (vehicle side, in order)
Every command on the command topic is checked, in order, before any
state change:
1. Signature verifies against the provisioned gate public key. No
   signature, bad signature, or unprovisioned key → REJECTED_AUTHORITY,
   log, no state change, no lease extension.
2. Schema version = 1.0 (or the version in the spec — mismatch → reject).
3. vehicle_session_id matches the current registered session.
4. owner_epoch matches the current epoch.
5. Sequence is monotonic per session (one counter, shared NHZ+LBZ),
   strictly greater than the last accepted sequence. Duplicate or
   out-of-order → DUPLICATE, no state change, no lease extension.
6. zone_map_version is known.
7. zone_type matches command (NHZ ↔ horn, LBZ ↔ lamp; mismatch → reject).
8. Observation age, as stated in the command, ≤250 ms at receipt.
9. Time-sync-valid flag true, and absolute expiry verifiable.
10. Lease_ms and expires_at consistent (expires_at − issued_at ≈ lease_ms,
    within a starting tolerance of 50 ms, to calibrate).
11. Rate/flood limit not exceeded (starting: 20/s per vehicle).
12. Override not active (firmware override check, in addition to the
    electrical S2 bypass — either one wins).
Any failure → ack with the matching REJECTED_* result, log, no actuation,
no deadman pulses, no latch strobe, no lease extension. An invalid,
expired, or unauthenticated message must never extend suppression.

## 6. State machines (vehicle)
CONTROL: IDLE / ARMED / OVERRIDE / FAULT.
HORN: AVAILABLE / SUPPRESSED. Normally closed relay K1 makes AVAILABLE
the de-energised state — horn available without coil power.
LAMP: HIGH / LOW / FREEZE. FREEZE is the latch holding its last mode
while the 3.3 V logic rail is alive, with no new strobe.
NHZ and LBZ timers are independent. A horn command never changes lamp
state, and a lamp command never changes horn state.

Transitions:
- Valid HORN_SUPPRESS + deadman valid → HORN SUPPRESSED (coil energised
  through Q2 AND Q4, §8).
- Valid RELEASE, expiry, stale, override, fault, or link loss → HORN
  AVAILABLE (coil de-energised, relay NC contact returns horn).
- Valid LAMP_LOW / LAMP_HIGH → lamp_task writes mode data to GPIO7,
  then strobes GPIO11 once, and only once per mode change. Refresh
  commands with the same mode do NOT re-strobe — re-strobing on every
  refresh would jitter the latch and hide a stale-command fault.
- Stale / UNKNOWN / PREDICTED / link loss → no strobe, latch retains
  last mode = FREEZE. FREEZE is not a firmware choice — it is the
  absence of a strobe while the logic rail is alive.
- Override (electrical S2 or firmware GPIO9) → CONTROL OVERRIDE, horn
  and lamp go to manual control (S4 selects HIGH/LOW, S2B/S2C bypass
  relay and coil), firmware stops all automatic pulses and strobes,
  override is logged and published on the override topic.
- Fault (Wi-Fi loss, MQTT loss, sync invalid, validation failure
  streak, brownout, watchdog) → CONTROL FAULT, horn AVAILABLE,
  lamp FREEZE, no new automatic commands accepted until re-registration
  and a fresh valid command.

## 7. Lease, refresh, and expiry (vehicle, monotonic)
- t_end = min(absolute expiry, last valid refresh + 300 ms).
- Lease absolute: 500 ms from issue, as in the pole design.
- Refresh target: 100 ms. A valid refresh updates "last valid refresh"
  but does NOT extend absolute expiry.
- If no valid refresh for 300 ms, or absolute expiry passes, whichever
  is first: horn suppression lifts, lamp freezes, state goes to
  UNKNOWN/FAULT as applicable. No grace period beyond t_end.
- Release notification for UNKNOWN may be accepted without CONFIRMED,
  but it must never be treated as a positive command and must never
  extend a lease.

## 8. Horn deadman and lamp latch — firmware duties
- Deadman: while suppression is valid, deadman_task pulses GPIO10
  every **50 ms** (starting value, to calibrate against the measured
  CD74HC423 timeout). Pulses are edges only — a stuck HIGH or stuck LOW
  GPIO10 produces no new edge, and coil current ceases after the hardware
  pulse expires. Firmware must NOT pulse deadman when suppression is
  expired, overridden, faulted, or unauthenticated.
- CD74HC423 timeout is set by R8 = 100k and C3 = 4.7 µF in the schematic
  — a preliminary value. The timeout over supply, temperature, and
  capacitor tolerance must be measured before any timeout guarantee is
  claimed. Do not quote a deadman timeout in milliseconds here until
  measured.
- Lamp latch: mode data GPIO7 must be stable before strobe GPIO11.
  Strobe pulse starting width: **1 ms**, to calibrate against the
  SN74HC74 datasheet and bench. TPS3839G33 holds latch clear during
  logic-rail start-up; initial mode is LOW, as drawn.
- If logic power fails, latch retention is lost — neither lamp can stay
  on without demo actuator power. That limit is stated on the schematic
  and must be stated in any jury description. ESP reset with the 3.3 V
  rail alive does NOT clear U5 — that is the FREEZE guarantee, and it
  must be bench-tested (reset injection, §10).

## 9. Override — electrical wins, firmware logs
- S2 is one mechanically ganged break-before-make 4PDT functional MOCK
  in the schematic — exact purchased model, terminal map, and contact
  ratings are OPEN. Firmware override sense (GPIO9, debounced 50 ms
  starting value) is a log and a firmware stop, not the override itself.
  The electrical bypass (S2B cuts coil supply, S2C grounds horn return
  around K1 including a welded-open contact, S2A opens MOSFET source
  returns and grounds S4 common) is what actually wins, even if firmware
  is hung.
- Manual selection: S4 SPDT selects HIGH or LOW return in override.
  S4 terminal map and rating are OPEN (MOCK). Firmware must not assume
  S4 position — it reads lamp state only from its own stored mode, and
  in override it must not strobe the latch.
- Override is always logged (GPIO9 transition, monotonic time, session,
  horn/lamp state at override) and published on the override topic when
  link allows. If link is down, override is logged locally and published
  on reconnect. Override log is not a substitute for the electrical bypass.

## 10. GPS context, logging, and health
- GPS: parse NMEA at 9600 baud, record position, speed, heading as
  context only. GPS must never override a pole command, never create a
  zone decision, and never be the sole input for any actuation.
  GPS dropout, 2.5 m CEP, and up to 5 Hz are context limits, not
  classification performance.
- Log (flash ring buffer + UART): monotonic time, UTC if sync-valid,
  session, sequence, command, validation result, horn/lamp state,
  deadman pulse state, override transitions, Wi-Fi RSSI, MQTT
  disconnects, faults, GPS fix status. No personal data, no video,
  no plates, no faces — log-based privacy only, as per the camera-free
  decision.
- Health status: periodic status publish with control state, horn/lamp
  state, Wi-Fi RSSI, uptime, fault flags. Status is not an ack and is
  not actuation proof.

## 11. What is still ours to test (vehicle, do not claim as done)
Wi-Fi association + MQTT time-to-first-command at 5/10 km/h; gate
candidate→authorised hop time; signature verification time and key
provisioning; sequence/epoch replay injection; flood injection; clock
sync drift over 1 hour and after reboot; deadman timeout over supply /
temperature / capacitor tolerance; latch strobe width and ESP-reset
FREEZE injection; relay weld injection and S2 bypass injection; lamp
HIGH/LOW steady and inrush vs AO3400A thermal; horn steady and inrush
vs K1/S2 contact ratings; buck/LDO currents and temperature at Wi-Fi
TX peaks; battery and fuse selection after load measurement; GPS
breakout rail check (3.3 V vs 5 V LDO); override debounce and logging
under link loss; multi-vehicle command isolation (one vehicle's command
must never actuate another vehicle — test with two vehicles, similar
session IDs, and crossed topics).
