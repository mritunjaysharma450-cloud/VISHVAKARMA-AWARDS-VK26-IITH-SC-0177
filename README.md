# V-ORRA Intelligent Vehicle Signalling & Connected Road Safety System

Application: VK26-IITH-SC-0177, Vishwakarma Awards 2026-27,
Sustainable Cities / Smart Urban Infrastructure & Asset Tracking.
Team: ITM University Gwalior. Faculty mentor: Ashish Singh Chandel,
Professor of Practice, SOET, ITM University Gwalior.

This repository contains the technical content behind the Stage 2 Design
Specification presentation. The overall architecture is presented first,
followed by each product independently, then research, software, safety,
and testing.

## Contents
1. Overall System Architecture — complete flow: mapping software →
   pole unit → vehicle unit, and how each part works in unison.
2. Pole Unit Electronic Schematic — pole KiCad design, Rev A.
3. Vehicle Module Electronic Schematic — vehicle KiCad design, Rev A.
4. Pole Unit Software and Vehicle Module Software — refined software
   designs for pole and vehicle, with the single MQTT gate contract.
5. Research Plan — papers analyzed, with direct links to the original
   papers, and what each group supports.
6. Safety and Testing — failure modes, latency and power budgets,
   test protocols (results blank), and 16-scenario theoretical stress
   testing.
7. Team Lab Photos — lab work photos and captions.

## Scope and limits
This is design, theory, and draft hardware work, not a finished, tested
product. Nothing in this repository is presented as final, and nothing
is presented as a measured road result.

- Schematics: made in KiCad on 24 October 2026, revised 26 October 2026
  due to component changes (lamp latch, supervisor, horn deadman, manual
  selection, gate protocol). ERC 0 errors, 0 warnings for both schematics,
  tested 26 October 2026. ERC and netlist are static design checks only,
  not bench proof, simulation, fabrication, or soldering proof. Symbols
  marked MOCK have unverified physical terminal maps, and a MOCK symbol
  is not a basis for building or soldering. Exact board revisions,
  currents, ratings, and timeouts are listed as OPEN in each schematic
  README, and those values remain to be measured before any build.
- Software: design only. This design has not been bench-tested. The demo
  speed cap is 10 km/h until latency, association, and gate timing are
  measured. Association numbers, dwell, and lateral bands in the software
  designs are initial bench values, not measured limits.
- Zone map sample: synthetic 40 m straight geometry, survey pending.
  The sample is not a surveyed site.
- Test results: protocols are provided, results are blank. Theory
  passing a scenario is not a test pass.
- CAD: not included in this version. FEA: not performed, and no FEA plot
  is included in this repository.
- Team / faculty photos: lab photos are described in Team Lab Photos.
  A joint mentor photo is not included in this version.
- Dates: design work in this repository is from October 2026. Files
  revised or prepared on 26 October 2026 are labelled as such, and are
  not described as 24 October work alone. No dates have been changed.

## Why iteration will still be needed
Physical testing is required for a final version. Real component
tolerances differ from datasheet ideal conditions; timing, power, and
UWB/radar accuracy are unmeasured on the actual boards; calibration,
multi-vehicle, and environmental effects (weather, occlusion,
reflections, temperature) cannot be proven by theory alone; and surveyed
site geometry will change zone boundaries. That is why every theory-only
file in this repository says so, and why test protocols are included
with blank results.

## Safety boundary
All actuation in this design is auxiliary demo hardware on a demo unit's
own battery, on a closed course only. Nothing is wired into a vehicle's
own horn, headlamp, or battery circuits. Horn suppression fails open,
lamp mode freezes on stale or unknown data, and a physical driver
override always wins and is logged. No camera, video, number plate, or
face data is used anywhere in this architecture — privacy is log-based only.

## Folders in this repository
- System Architecture & Specs/
- CAD Design & Mechanical Specs/
- Electronics Schematics & BOM/
- Maker Journey/
- Safety & Hazard Mitigation/
- Technical Challenges & Risks/
- References & Sources/
- Image Sources/

## Contents not included in this version
Cleaned MQTT / zone-map specification and pole design documents,
reconciled bill of materials, CAD files, surveyed zone map, bench and
closed-course test results, and the joint mentor photo. Those items are
not in this version.
