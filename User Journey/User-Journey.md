# User Journey

This is the intended journey for an equipped vehicle on a surveyed,
closed course. It is a design journey, not a road test result.

1. Before the journey: zone polygons are surveyed and stored in a
   zone map. The sample map in this repo is synthetic 40 m straight
   geometry, survey pending — never present it as a surveyed site.
2. Vehicle enters pole coverage: pole radar tracks the vehicle, pole
   UWB anchor ranges the vehicle tag, and the pole associates the
   radar track with the UWB session. Only a UNIQUE association may
   proceed; UNKNOWN or PREDICTED associations do not actuate.
3. Zone decision: after dwell (2 consecutive fresh UNIQUE associations
   in the same zone, bench starting value), the pole issues a command
   through the safety gate to the vehicle, by MQTT over Wi-Fi.
4. In a No Horn Zone: the vehicle's auxiliary demo horn is suppressed,
   on the demo unit only, never on the vehicle's own horn circuit.
5. In a Low Beam Zone: the vehicle's auxiliary demo lamp switches
   high to low, on the demo unit only, never on the vehicle's own
   headlamp circuit.
6. On unknown, predicted, stale, or link-loss data: demo lamp freezes
   at its last mode, horn suppression lifts (fail-open).
7. At all times: a physical driver override on the demo unit always
   wins, electrically, and is logged. The driver is never locked out
   of horn or lamp control by software alone.
8. After the journey: pole and vehicle logs record decisions, acks,
   overrides, and faults, with no video or plate data.
