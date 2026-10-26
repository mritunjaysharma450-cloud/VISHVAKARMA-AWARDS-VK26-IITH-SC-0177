# Pole Unit Software — Refined Design (v1.1)

This document describes a design only. It has not been bench-tested. This document refines the earlier pole design and the MQTT spec v1.0. Where v1.0 and the revised pole schematic disagreed, this version follows the schematic: single vehicle command topic, Pi owns sequence, gate preserves sequence and signs, vehicle verifies. That is a paper reconciliation — gate firmware, key provisioning, ACL negative test, and fault injection are not supplied yet, and remain to be tested.

Demo cap until measured: **≤10 km/h only.** 20 km/h is not used with the numbers below — at 20 km/h the stacked age (see §6) is 3.33 m, which refutes a 2.0 m transition allowance. Speed, lease, and allowance remain to be re-derived from measured p95 latency before any faster demo.

## 1. Hardware the software actually sees
- Raspberry Pi 5 (1 GB), host process, Python, Mosquitto broker (MQTTv5).
- AWRL6432BOOST radar by USB (USB-C via hub port 1 in the schematic). Input: radar point cloud / tracks, TI mmWave SDK framing as supplied by the board. No radar frame format is claimed here beyond what the board manual gives — frame rate, point format, and track IDs remain to be recorded at bench time.
- Makerfabs MAUWBS3CA1 pole anchor by USB-C (hub port 2), 115200 baud, anchor firmware as supplied. Input: range reports with anchor ID, tag ID, sequence, live-anchor mask, RSSI. **No MAUWBS3CA1 timestamp field is assumed.** Until the anchor manual proves a timestamp field, pole time is Pi receive time (monotonic clock), and USB/serial jitter is an unbudgeted error — it remains to be measured (T2/T4 in the test protocols).
- NEO-6M by USB-TTL, broad context only, never a tie-breaker and never sole zone input. GPS is context, not classification.
- ESP32-S3 pole safety gate by USB (hub port 3). Gate is a decision gate, not an independent system: it shares Pi power, Pi Wi-Fi AP, and the Pi broker. If Pi power, AP, or broker fails, the gate cannot authorise a new command on its own.
- Ethernet to the next pole, for handover / coordinator traffic only. No vehicle command is sent over Ethernet.

## 2. Processes and data flow (one pole)
```
radar_usb ──> RadarIngest ──┐
uwb_usb   ──> UwbIngest    ──┼──> TimeAlign ──> Transform ──> Association
gps_usb   ──> GpsContext ───┘                                     │
                                                                 ▼
                                            ZoneCheck ──> Candidate ──USB──> Gate
                                                                                  │
                                                                                  │ Wi-Fi / MQTT (gate-signed)
                                                                                  ▼
                                                                              Vehicle
Coordinator (on owner pole) ── Ethernet ──> other pole(s)
Logger ──> SQLite (local, pole) + serial log
Health ──> pole status topic, undervoltage / temperature / link flags
```
One process per ingest, one association process, one gate interface process. No ingest process publishes a vehicle command directly. All commands go through Association → ZoneCheck → Candidate → Gate → Vehicle, in that order.

## 3. Clocks and time alignment (starting values, to calibrate)
- All pole decisions use a monotonic clock. Wall-clock UTC is for logs only.
- Radar and UWB observations are stamped at Pi receive time (monotonic), because no proven sensor timestamp is assumed.
- Time-alignment gate, starting value: radar and UWB observations are within **100 ms** of each other to be associated. This is a bench starting value, not a measured jitter bound — USB/serial jitter remains to be measured and the gate is re-derived from it. If jitter exceeds 100 ms, the gate widens and the latency budget grows; the gate is not silently widened to force passes.
- Clock sync between poles, and between pole and vehicle, is by a sync-valid flag with a starting expiry of **60 s**. The old 20 ms clock bound is deleted — it was unmeasured and unrealistic without a sync mechanism. Until a sync mechanism is implemented and its drift is measured over 1 hour and after reboot, time-sync-valid is treated as unproven, and vehicles reject new commands when sync is invalid (see vehicle design).

## 4. Coordinate transform and classified point
- Zone map is surveyed LOCAL_CARTESIAN: s longitudinal, y lateral from median centre y = 0. Cross-section for the 15.5 m example: Carriageway A y −7.75 to −0.75, median −0.75 to +0.75, Carriageway B +0.75 to +7.75. The sample map accompanying this design is synthetic 40 m straight geometry, map_id ROAD-DEMO-001, survey_status PENDING_SURVEY — never a surveyed site.
- **Classified point = UWB antenna position on the vehicle**, with a surveyed offset from vehicle centre recorded at installation. Radar track centroid is transformed to that antenna point before comparison. The radar-centroid-to-antenna offset (0.5–2 m class of error in the stress test) is an error term, not zero — it remains to be measured and included in the lateral uncertainty band.
- Radar alone cannot resolve mirrored sides: at a 29° face-value resolution (unconfirmed as a TI datasheet figure — it is not cited as datasheet), a vehicle at s = 20 m, y = ±3 m gives 17° separation, below resolution. Mirror resolution therefore comes from the UWB association, not from radar angle alone, and a single-anchor geometry is advisory only.

## 5. Association — hypothesis test, not identity proof
Output per vehicle session: UNIQUE, UNKNOWN, or PREDICTED. Only UNIQUE goes to ZoneCheck. PREDICTED never becomes CONFIRMED_INSIDE.
Association steps, in order:
1. Time-align radar track and UWB range (gate §3).
2. Transform radar track to s/y/z in the surveyed frame.
3. Slant-range residual gate, starting value: **|radar range − UWB range| ≤ 1.5 m**. Starting value only — it is re-derived from measured radar σ and UWB σ at bench, per distance, not one global constant.
4. Motion consistency gate, starting value: range-rate residual ≤ 0.5 m/s. Starting value only, to calibrate.
5. Best-vs-second-best margin gate, starting value: best candidate cost beats second-best by **≥ 0.5 m in range-residual cost**. If margin is not met, output is UNKNOWN, not a guess.
6. Track continuity: a track ID switch, occlusion gap, or crossing of two similar-range vehicles forces UNKNOWN until a fresh UNIQUE association is re-established. No track is carried across an ID switch by assumption.

Costs, weights, and thresholds in steps 3–5 are determined experimentally. They are not claimed as measured, and no bench result is filled in here.

## 6. Zone check, dwell, and timing — refined, with the failure stated
- Zone check runs only after UNIQUE. Zone check output: NHZ, LBZ, or outside, plus a lateral uncertainty band.
- **Lateral uncertainty band, starting value: ±1.0 m around the median boundary.** For the 15.5 m example, that means confirmed Carriageway A only when y < −1.75 m, confirmed B only when y > +1.75 m, and UNKNOWN in between. This band is conservative and will reduce the confirmed rate — that is the cost of an unmeasured lateral error.
- **Dwell, starting definition: 2 consecutive fresh UNIQUE associations, max interval 150 ms between them, both in the same zone.** Dwell time is therefore at least one radar/UWB frame interval, and it is added to the latency budget (§7). If dwell + processing exceeds the budget, the answer is to lower speed or widen the transition allowance from measured data — never to shorten dwell or actuate on a predicted state.
- **Transition allowance, starting value: 2.0 m entry/exit, marked AGAINST at 20 km/h and usable only as a bench starting point at ≤10 km/h.** Formula: B ≥ v·T + E_longitudinal + M_timing. At 20 km/h, v·T alone for T = 250 ms is 1.389 m, leaving 0.611 m before any lateral error — not sufficient. At 10 km/h, v·T = 0.694 m, leaving more margin, but the allowance is still unproven until E and M are measured.
- **Observation age ceiling: 250 ms at actuation, not 350 ms.** The old 350 ms ceiling exceeded the 250 ms processing budget and stacked to 600 ms total age (350 + 250) = 1.67 m at 10 km/h, 3.33 m at 20 km/h. New rule: a command is not acted on if the newest observation it rests on is older than 250 ms at the vehicle, and the gate hop time is inside the 250 ms processing budget, not on top of it.

## 7. Latency budget (targets, not measured)
Total target ≤250 ms, including the gate hop:
radar 95 + Pi processing 40 + gate candidate→authorised hop (target ≤20, to measure) + MQTT 10 + Wi-Fi 50 + ESP32 15 + contingency 20 = 250 ms.
Travel in 250 ms: 0.347 m at 5 km/h, 0.694 m at 10 km/h, 1.389 m at 20 km/h.
Dwell time is additional — see §6. If measured p95 exceeds 250 ms, the demo speed cap falls, the transition allowance rises, or both. No measured latency is claimed here.

## 8. Gate interface — candidate, authorisation, signing (paper design)
- Pi → gate over USB, candidate message: vehicle_session_id, zone_type, command (HORN_SUPPRESS / LAMP_LOW / LAMP_HIGH / RELEASE), sequence, owner_epoch, zone_map_version, lease_ms, expires_at_utc, issued_at.
- Gate checks, in order: (1) gate switch state — closed S1 = ARM request, open/broken = DENY, (2) gate heartbeat fresh, (3) candidate fields valid, (4) sequence and owner_epoch match the current authorised session, (5) no override active on the gate side. If any check fails, gate sends no authorised command. Gate does not invent a command, change zone_type, or extend a lease.
- Gate preserves Pi sequence verbatim and, when permitted, signs the full command payload and publishes it over Wi-Fi to the single topic `vorra/v1/vehicles/<vehicle_id>/commands` (QoS 1, retain No).
- Signing: gate private key, vehicle holds gate public key, provisioned offline before the demo (USB / local file, never over MQTT). Key provisioning, rotation, and revocation are not designed here — they are a required pre-demo step, and until signing is implemented, broker ACL is transport control only, not command authentication. ACL is not presented as authentication.
- Gate failure modes: gate USB unplug, gate reboot, gate hang, gate switch short to GND (single-contact circuit does not detect a short — stated in the schematic README), gate power loss. In every gate failure, no new authorised command is issued, and vehicles fail open by their own lease/timeout (see vehicle design). Gate boots INHIBITED, with heartbeat + reauthorisation required before any command.
- Gate switch short detection and debounce are OPEN — the schematic circuit alone does not provide them.

## 9. MQTT topics and validation (v1.1, supersedes v1.0 conflict note)
Broker: MQTTv5, QoS 1 at-least-once (NOT exactly-once), Clean Start, SessionExpiryInterval = 0, Last Will OFFLINE on status topics.
- `vorra/v1/vehicles/<vehicle_id>/commands` — gate → vehicle, QoS 1, retain No. Single command topic. No candidate/authorized split topics on the broker.
- `vorra/v1/vehicles/<vehicle_id>/acks` — vehicle → pole, QoS 1, No. Ack is an application result (APPLIED, REJECTED_EXPIRED, REJECTED_STALE, REJECTED_AUTHORITY, REJECTED_OVERRIDE, DUPLICATE, REJECTED_UNCERTAIN), not delivery proof and not actuation proof.
- `vorra/v1/poles/<pole_id>/status` — pole → others, QoS 1, retain Yes.
- `vorra/v1/vehicles/<vehicle_id>/override` — vehicle → pole, QoS 1, No.
- `vorra/v1/vehicles/<vehicle_id>/status` — vehicle → pole, QoS 1, Yes.
- `vorra/v1/vehicles/<vehicle_id>/registration` — vehicle → pole on boot, new session_id, device identity as provisioned. Session registration handshake is required before any command for that session is accepted. Registration alone does not authorise actuation.

Validation, in order, at pole and again at vehicle (16 checks):
schema_version, vehicle_session_id known and registered, owner_epoch current, sequence monotonic per session (one counter, shared NHZ+LBZ), zone_map_version known, zone_type matches command, dwell/consecutive confirmation satisfied, observation age ≤250 ms, monotonic-clock + time-sync-valid flag, lease_ms and expires_at consistent, rate/flood limit not exceeded (starting: 20 commands/s per vehicle, to calibrate — normal refresh is 10/s), physical feedback distinguished from ack, override not active, command not expired, command not a replay.
Any failure → reject, log, no actuation, no lease extension. An invalid, expired, or unauthenticated message never extends suppression.

Timings (bench starting points, ≤10 km/h only): lease 500 ms absolute, refresh target 100 ms, no-refresh timeout 300 ms, max observation age 250 ms (revised, §6), effective expiry = min(absolute expiry, last valid refresh + 300 ms). Refresh does not extend absolute lease unless a new valid command is issued. Release notification for UNKNOWN may be allowed without CONFIRMED, but it is never treated as a positive command.

## 10. Coordinator and handover
- Coordinator is a process on the current owner pole. Identity, host, and election are OPEN — the coordinator is named before any multi-pole demo, and a single-pole demo states "coordinator = this pole, no handover" explicitly.
- Owner epoch fencing: epoch increments on coordinator change (e.g. 42 → 43). A command with an old epoch is rejected by gate and vehicle.
- Handover trigger, starting definition: new pole reports a fresh UNIQUE association for the same session in the overlap zone, coordinator transfers ownership, old pole stops issuing, new pole starts only after its own fresh UNIQUE + dwell. Overlap width and handover gap time are to measure, not assume.
- Ethernet partition: if coordinator heartbeat is lost (starting: 500 ms heartbeat, 2 s lease — to calibrate), both poles stop positive commands and vehicles fail open. No split-brain positive commands, no quorum claim without a quorum design.
- Pole reboot: all associations cleared, pole state returns to IDLE, no CONFIRMED_INSIDE restored from disk or memory. Re-authorisation requires a fresh registration, association, and dwell.

## 11. Pole states
IDLE / TRACKING / ASSOCIATED / CONFIRMED_INSIDE / UNKNOWN / HANDOVER.
PREDICTED is an association output, never a pole state that can issue a command, and PREDICTED never transitions directly to CONFIRMED_INSIDE.
UNKNOWN, PREDICTED, stale, and link-loss states all fail open at the vehicle (lamp freezes, horn suppression lifts — vehicle design).

## 12. Logging and health (pole)
Log every: radar/UWB observation (with monotonic time, no personal data), association result and margin, zone decision, candidate, gate decision, MQTT publish, ack, override, handover, epoch change, error, undervoltage flag, CPU temperature, USB disconnect, broker disconnect.
Local SQLite, one row per decision, with fields: time_monotonic, time_utc, vehicle_session_id, track_id, association, margin, zone, command, sequence, owner_epoch, gate_result, ack_result. No log row is a test pass — test results live in the test log, separate from operation logs.

## 13. Items remaining to be tested (pole)
MAUWBS3CA1 ↔ bare MaUWB interoperability (role, network, channel, firmware, valid range reports — P0); radar/UWB Δt and jitter; association σ, gates, and margins per distance; lateral error and classified-point offset; dwell and latency p95 at 5/10 km/h; gate hop time; gate USB unplug / reboot / hang injection; coordinator election, epoch lease, Ethernet partition; flood threshold; broker/Pi reboot release and re-authorisation times; multi-vehicle similar-range, crossing, following, occlusion, and track-ID-switch wrong-identity rate; near-median false-side rate; Pi USB currents, undervoltage, hub self- vs bus-powered mode, thermal.
