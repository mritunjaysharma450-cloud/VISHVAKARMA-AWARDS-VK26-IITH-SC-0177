# Overall System Architecture — V-ORRA

V-ORRA Intelligent Vehicle Signalling and Connected Road Safety System.
Application ID: VK26-IITH-SC-0177. Track: Sustainable Cities, Smart Urban Infrastructure and Asset Tracking.

Status of this document: theory and evidence-based design, not a final validated product.
Final versions will be set only after bench testing and closed-course testing in Stage 3.
Iteration is expected, not a failure.

## 1. What the system does, in one paragraph

A roadside pole watches a surveyed road section. When an equipped vehicle is confirmed
to be inside a No Horn Zone or a Low Beam Zone, the pole sends a short, time-limited
command over local Wi-Fi to that vehicle only. The vehicle module then acts on its own
demo lamp and demo horn — hardware that sits on the demo unit itself, on its own battery.
It never touches the vehicle's own horn or headlamp wiring. If the pole is unsure, it sends
nothing new, the lamp freezes, and horn suppression lifts. The driver can always override,
and that override is logged.

## 2. Complete flow: mapping software to pole to vehicle

1. Mapping software holds the surveyed zone map. A zone is a polygon with a zone ID,
   a version, a survey date, and a pole ID. The map is made from surveyed road
   coordinates, not from a GPS point taken on the day.
2. The pole loads that map. It does not invent zones from live GPS alone.
3. The pole radar tracks moving objects in its field. The pole UWB anchor runs a
   ranging session with an equipped vehicle tag. Radar gives track and motion.
   UWB gives session identity and range. Neither alone decides a zone command.
4. The pole computer associates a radar track with a UWB session only when the
   evidence is strong enough: track motion, range, time alignment, and separation
   from other tracks. If two tracks could match, the result is UNKNOWN.
5. The pole checks the associated track position against the zone polygon, with a
   transition allowance for the distance travelled while a command is in flight.
6. Only on a confirmed, associated, non-predicted track, after dwell, does the pole
   publish a time-limited MQTT command to that vehicle.
7. The vehicle controller validates the command — vehicle ID, sequence, expiry —
   and only then drives the demo lamp or demo horn. On expiry, stale, duplicate,
   or link loss, it falls back: lamp freezes, horn suppression lifts.
8. The next pole, connected by Ethernet, receives the session handover so a vehicle
   leaving one pole section is not treated as a new unknown vehicle by the next pole,
   and is not given duplicate commands.

## 3. Pole unit, in brief

- TI AWRL6432BOOST, 57–64 GHz, 2 transmit / 3 receive, 120° azimuth field of view,
  80° elevation field of view, USB-powered. Source: TI evaluation board pages.
  Field of view is not coverage. Mounting direction and observation sector must be
  specified on site, and are to be tested.
- Makerfabs MAUWBS3CA1 pole anchor, S3-based board with USB-C, connected to
  the Raspberry Pi 5 via the UGREEN hub, hub port 2, 115200 baud, anchor firmware
  to be verified. This is NOT a bare module on the pole side. The vehicle tag
  is the bare MaUWB module by UART — pole and vehicle use different MaUWB
  forms, and jury files must not mix them up. Board revision and range reports
  on the Pi hub are ours to bench-verify before any claim that the link works.
- Raspberry Pi 5, 1 GB. GPIO is 3.3 V only, never 5 V. Official Pi 5 power table:
  recommended PSU 5.0 A, maximum total USB peripheral current 1.6 A with a 5 A
  supply, 600 mA with a 3 A supply, typical bare-board active current 800 mA.
- ESP32-S3 pole safety gate. It can withhold a pole command independently of the
  Pi application, on watchdog expiry, malformed command, or gate reset.
- UGREEN USB hub. Exact model and whether it is self-powered or bus-powered is
  not yet recorded in the submitted documents, and must be recorded before a
  power budget is claimed as final.

## 4. Vehicle module, in brief

- Makerfabs MaUWB vehicle tag, 3.3 V UART to the ESP32-S3, AT commands.
- ESP32-S3 N16R8 controller, 2.4 GHz Wi-Fi client to the pole Pi access point,
  MQTT client, command validation, demo actuation, override logging.
- NEO-6M GPS, 3.3 V UART, for broad context only. Its published accuracy is
  2.5 m CEP, up to 5 Hz. That is not 1 m lateral accuracy, and GPS alone never
  decides a zone command.
- Demo lamp and demo horn, on the demo unit's own battery, through MOSFET /
  relay drivers on an isolated demo rail. Vehicle horn and headlamp circuits
  are untouched. Closed-course demo only.
- Physical driver override switch, electrical priority over software control,
  logged when used.

## 5. Numbers that are targets, not results

All of these are proposed engineering targets from the submitted deck, not
experimentally achieved results:

- Road coverage: 15.5 m, that is 2 x 7 m carriageways + 1.5 m median.
- Detection: 10–20 m initial, 30–40 m experimental.
- Speed: 5–10 km/h initially, up to 20 km/h stretch.
- Positioning: ≤1 m lateral error, 95th percentile, stretch target.
- Latency: ≤250 ms in a controlled environment, target.
- Capacity: 1 equipped vehicle now, 4 software tracking slots planned.
- Power: pole 100 W / 156 Wh, vehicle 10,000 mAh, as submitted.

Why they need iteration, in brief: datasheets describe ideal or averaged
conditions. A real road adds reflections, noise, interference, component
tolerances, temperature, vibration, power fluctuation, and human variation.
First physical tests will move thresholds, ranges, and timings.

Latency, worked: distance = speed x time.
At 10 km/h: 2.78 m/s x 0.25 s = 0.69 m.
At 20 km/h: 5.56 m/s x 0.25 s = 1.39 m.
A 250 ms command therefore lands about 1.4 m past the sensed position at
20 km/h, before any position error is added. The zone logic needs a transition
allowance for this, and prediction must never override fail-open.

Mirror ambiguity, worked: a single UWB anchor measures slant range
r = sqrt(x^2 + y^2 + h^2). Positions at +y and -y give the same range.
One anchor alone cannot tell which carriageway a vehicle is on. That is why
radar track association and rejection of ambiguous cases are central to this
design, and why they must be tested with two vehicles at similar ranges.

## 6. What is not in this system

- No camera, no video, no number plates, no faces. Privacy is log-based only.
- No UWB data channel. Commands go by Wi-Fi MQTT only.
- No wiring into a vehicle's own electrical system.
- No forced immobilisation, no V-Gun actuation in this prototype.
- No CAD design in this file set yet. CAD and FEA are pending, and are marked
  as such, not claimed as done.
- No team or faculty photos in this file set yet. Those are to be supplied
  separately, with institutional ID badges in frame.

## 7. Sources for the verified parts

- TI AWRL6432BOOST evaluation board pages: band, TX/RX, field of view, USB power.
- Raspberry Pi official power table: Pi 5 PSU, USB current limits, bare-board current.
- Makerfabs MaUWB AT manual: 3.3 V power, 3.3 V TTL UART, damage above 3.3 V.
- Espressif ESP32-S3 datasheet: UARTs, Wi-Fi, 3.3 V supply range.
- u-blox NEO-6M product summary: 2.5 m CEP, up to 5 Hz.

Full references, with papers and manufacturer documents kept separate, are in
the references folder. Papers that were not used before submission are kept
separate and labelled as defensive backing only, not prior use.
