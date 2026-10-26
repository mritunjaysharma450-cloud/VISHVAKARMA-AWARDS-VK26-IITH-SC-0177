V-ORRA | Application VK26-IITH-SC-0177 | ITM University Gwalior
Track: Sustainable Cities, Smart Urban Infrastructure and Asset Tracking
KiCad 10.0.7 editable schematic. Drawing title-block date: 24 October 2026 as requested.
This revised file set was generated and checked on 26 October 2026; file timestamps
and test timestamps are not represented by the title-block drawing date.

STATUS: engineering draft for a closed-course demo. ERC and netlist checks are
static design checks, not measured function, simulation, fabrication or bench proof.
MOCK means a functional interface whose physical terminal map/footprint remains
unverified. Do not build or solder from a MOCK symbol. Exact board revisions and
actual silkscreens must be matched before creating a wiring harness or PCB.

POLE UNIT ELECTRONIC SCHEMATIC
Files: Pole Unit Electronic Schematic.kicad_sch/.kicad_pro/.pdf/.net/.xml,
Gate and Wireless.kicad_sch, VORRA.kicad_sym, sym-lib-table, ERC Final.rpt,
ERC Final.json, Netlist Check.txt, Layout Check.txt and KiCad Test.

The official 27 W adapter powers the Pi 5 at USB-C only. Pi USB-A drives the
UGREEN hub upstream cable. P1 is the TI radar Micro-USB interface, P2 is the
Makerfabs MAUWBS3CA1 anchor USB-C interface, P3 is the ESP32-S3 gate USB
interface, and P4 is unused. The Pi fan header connects to the Active Cooler
through its supplied cable. RJ45 cable contacts 1–8 continue to the next pole.
USB cable returns, Pi fan return and gate return are the common GND net.
The gate 3.3 V circuit pulls IO6 high through 10 k; closed S1 pulls it low
for ARM request. Open/broken S1 means DENY, but a short to GND is not detected
by this single-contact circuit. IO5 drives the green LED through 330 ohm.

The dashed wireless links are logical only. A proposed gate contract uses the
single vehicle command topic vorra/v1/vehicles/<id>/commands: Pi assigns a
sequence and sends a candidate to the gate over USB; the gate preserves the
sequence and, when permitted, signs/publishes over Wi-Fi. The Pi application
must be denied command-topic write access by broker ACL. The vehicle must
verify the gate signature, session, owner epoch, sequence and expiry. Gate
firmware, cryptographic key provisioning, ACL negative test and reboot/fault
injection are NOT supplied. The gate shares Pi power, AP and broker; its
decision is separate, its transport and power are not independent.

KiCad 10.0.7 ERC: 0 errors, 0 warnings, 0 total violations.
Netlist: 32 nets; 4 intentionally unused terminals (hub P4 VBUS/D+/D-/GND).
No 5 V node is connected to the gate IO6/IO5 3.3 V circuit.

OPEN before bench or build: exact UGREEN model and bus/self-powered mode;
hub reverse blocking and backfeed; actual gate clone USB connector and pad
mapping; all USB and cooler steady/startup currents and Pi undervoltage;
anchor serial timing/firmware; gate switch short detection and debounce;
radar/anchor association accuracy, latency and multi-vehicle behaviour.
The reviewed 350 ms observation ceiling plus a proposed 250 ms processing
budget permits 600 ms total age: 1.67 m travel at 10 km/h, 3.33 m at
20 km/h. It cannot support a 2.0 m boundary allowance at 20 km/h.
The 250 ms budget also omits the proposed gate hop. Do not publish a safe
speed, boundary error or latency until observation age, gate hop and p95
end-to-end delay are measured and the margin is recalculated. Association
gates/dwell/track expiry, clock validity, lateral uncertainty, the defined
vehicle point, multi-vehicle slot behaviour and handover epoch leases are
open system-level work, not established by this circuit schematic.
Keep testing indoors or on a controlled closed course until radio, installation,
thermal and environmental requirements are verified.
