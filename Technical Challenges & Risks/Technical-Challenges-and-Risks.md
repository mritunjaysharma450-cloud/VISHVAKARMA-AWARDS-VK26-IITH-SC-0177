# Technical Challenges & Risks

The risks below were found by stress-testing the design on paper, before any bench test. Each risk is stated with its fallback.

1. Wi-Fi association + MQTT time-to-first-command at speed, inside a pole interval. This is the highest risk. The fallback is a static / walking-speed demonstration, never shortened dwell or predicted-state actuation.
2. Radar-UWB association at speed: range residual, time alignment, best-vs-second-best margin, track-ID switch, occlusion, crossing and following vehicles, and near-median false side. All numbers in the software designs are bench starting values, not measured limits.
3. Latency and boundary allowance at speed: 250 ms target budget, demonstration cap 10 km/h until measured. At 20 km/h with a 2.0 m allowance and 350 ms observation age, the arithmetic refutes the allowance (stacked age 600 ms = 3.33 m travel). That combination is deleted, not kept.
4. Gate independence: the gate shares Pi power, AP, and broker. Decision independence only applies, not power or transport independence. Gate firmware, key provisioning, ACL negative test, and fault injection are not supplied yet.
5. Hardware unknowns: MOCK pinouts, hub bus-powered vs separate supply, Pi USB currents and undervoltage, GPS breakout rail (3.3 V vs 5 V LDO), relay and switch ratings, lamp and horn inrush, deadman timeout over supply / temperature / capacitor tolerance, and thermal behaviour. All are listed as OPEN in the schematic READMEs.
6. Multi-vehicle: 4 software tracking slots are planned in the design specification, not proven by schematics. Multi-vehicle isolation remains to be tested with two vehicles, similar ranges, and crossed topics.
7. Regulatory and installation: radio authorisation, installation, thermal, and environmental requirements for outdoor use are not verified. Closed-course / indoor bench work only applies until verification.
8. Dwell-time confirmation: no dedicated paper was found. It remains team engineering practice, supported by analogy to buffer bands and track-confirmation tests, not by a dedicated dwell paper.

Full detail is given in Safety & Hazard Mitigation, and in the theoretical 16-scenario stress-testing file in that folder, with test protocols and blank results.
