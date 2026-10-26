# Research Plan — V-ORRA, VK26-IITH-SC-0177

Research papers not included in the PPT — V-ORRA, VK26-IITH-SC-0177

## Purpose
This research plan lists the papers analyzed for V-ORRA, and how each group
supports an architecture decision. It is a design research record, not a
claim that the prototype has been tested. No paper in this file proves the
prototype's own speed, range, accuracy, or latency. Those numbers are listed
as theory or stretch targets elsewhere in this repository, and remain to be
measured in bench and closed-course tests.

## Architecture the research supports
Pole unit: Raspberry Pi 5 + TI AWRL6432 radar + Makerfabs MAUWBS3CA1 UWB
anchor + ESP32-S3 safety gate. Vehicle module: Makerfabs MaUWB tag +
ESP32-S3 controller + NEO-6M GPS (context only) + auxiliary demo horn and
lamp on a separate demo unit, never wired into the vehicle's own circuits.
Communication: Wi-Fi + MQTT, local edge processing, no cloud control.
No camera is used anywhere in this architecture.

## How the research was used
- Radar + UWB fusion, DS-TWR ranging, and anchor placement: why a radar
  track alone is not an identity, and why a UWB session is used to bind
  a track to an equipped vehicle.
- Tracking and association: why clustering, Kalman tracking, and
  Hungarian / JPDA / MHT association are used, and why a single frame
  is never a decision.
- Zone logic: why surveyed polygon zones with a buffer band are used,
  and why a protection bound that fails becomes UNKNOWN, not a forced
  carriageway decision.
- Edge + MQTT + fail-safe: why control stays local, why commands carry
  sequence, session, epoch, and expiry, and why link loss, stale data,
  or override must fail open for the horn and freeze the lamp.
- GPS background: why GPS alone, and RTK as a primary source, were not
  chosen — GPS is kept as broad context only.

## Table A — These are the research papers we analyzed and will use (38 papers, none on the PPT slide).

| Sr. No. | Topic | Source / Year | Reference Link |
|---|---|---|---|
| 1 | Robust Urban INS/GNSS Positioning Under Degraded GNSS Conditions Using a Dual-Adaptive Cubature Kalman Filter | Electronics, 2026 | [https://www.mdpi.com/2079-9292/15/10/2064](https://www.mdpi.com/2079-9292/15/10/2064) |
| 2 | Evaluating PPP-RTK and Network RTK for Vehicle-Based Kinematic Positioning in Urban and Suburban Environments | Geomatics, 2026 | [https://www.mdpi.com/2673-7418/6/3/50](https://www.mdpi.com/2673-7418/6/3/50) |
| 3 | Satellite Positioning Accuracy Improvement in Urban Canyons Through a New Weight Model Utilizing GPS Signal Strength Variability | Sensors, 2025 | [https://www.mdpi.com/1424-8220/25/15/4678](https://www.mdpi.com/1424-8220/25/15/4678) |
| 4 | Adaptive Kalman Filter-Based UWB Location Tracking with Optimized DS-TWR in Workshop Non-Line-of-Sight Environments | Sensors, 2025 | [https://www.mdpi.com/1424-8220/25/24/7682](https://www.mdpi.com/1424-8220/25/24/7682) |
| 5 | The Influence of Ultra-Wideband Anchor Placement on Localization Accuracy | Sensors, 2025 | [https://www.mdpi.com/1424-8220/25/16/5115](https://www.mdpi.com/1424-8220/25/16/5115) |
| 6 | Comprehensive outdoor UWB dataset: Static and dynamic measurements in LOS/NLOS environments | Scientific Data, 2025 | [https://www.nature.com/articles/s41597-025-05887-9](https://www.nature.com/articles/s41597-025-05887-9) |
| 7 | Single Anchor Localization by Combining UWB Angle-of-Arrival and Two-Way-Ranging: an Experimental Evaluation of the DW3000 | International Conference on Localization and GNSS (ICL-GNSS), 2024 | [https://ieeexplore.ieee.org/document/10578498/](https://ieeexplore.ieee.org/document/10578498/) |
| 8 | A Fusion Strategy for Vehicle Positioning at Intersections Utilizing UWB and Onboard Sensors | Sensors, 2024 | [https://www.mdpi.com/1424-8220/24/2/476](https://www.mdpi.com/1424-8220/24/2/476) |
| 9 | UWB-Based Real-Time Indoor Positioning Systems: A Comprehensive Review | Applied Sciences, 2024 | [https://www.mdpi.com/2076-3417/14/23/11005](https://www.mdpi.com/2076-3417/14/23/11005) |
| 10 | Reduction of Multipath Effect in GNSS Positioning by Applying Pseudorange Acceleration as Weight | Sensors, 2024 | [https://www.mdpi.com/1424-8220/24/21/6880](https://www.mdpi.com/1424-8220/24/21/6880) |
| 11 | Trustworthiness for an Ultra-Wideband Localization Service | Sensors, 2024 | [https://www.mdpi.com/1424-8220/24/16/5268](https://www.mdpi.com/1424-8220/24/16/5268) |
| 12 | Untethered Ultra-Wideband-Based Real-Time Locating System for Road-Worker Safety | Sensors, 2024 | [https://www.mdpi.com/1424-8220/24/8/2391](https://www.mdpi.com/1424-8220/24/8/2391) |
| 13 | Ultra-Wideband Positioning Technology: Exploring TWR, TDoA, and PDoA Methods for Precise Ranging and Positioning | IntechOpen book chapter, 2024 | [https://www.intechopen.com/chapters/1200549](https://www.intechopen.com/chapters/1200549) |
| 14 | Ultra-Wideband (UWB) Positioning System Based on ESP32 and DWM3000 Modules | arXiv preprint, 2024 | [https://arxiv.org/abs/2403.10194](https://arxiv.org/abs/2403.10194) |
| 15 | Fusion of GNSS Pseudoranges with UWB Ranges Based on Clustering and Weighted Least Squares | Sensors, 2023 | [https://www.mdpi.com/1424-8220/23/6/3303](https://www.mdpi.com/1424-8220/23/6/3303) |
| 16 | Nonlinear Regression-Based GNSS Multipath Dynamic Map Construction and Its Application in Deep Urban Areas | IEEE Transactions on Intelligent Transportation Systems, 2023 | [https://ieeexplore.ieee.org/document/10056407/](https://ieeexplore.ieee.org/document/10056407/) |
| 17 | Ranging and Positioning with UWB | IntechOpen book chapter, 2023 | [https://www.intechopen.com/chapters/85753](https://www.intechopen.com/chapters/85753) |
| 18 | Angle of Arrival and Centimeter Distance Estimation on a Smart UWB Sensor Node | IEEE Transactions on Instrumentation and Measurement, 2023 | [https://ieeexplore.ieee.org/document/10143293/](https://ieeexplore.ieee.org/document/10143293/) |
| 19 | Indoor Positioning System with UWB Based on a Digital Twin | Sensors, 2022 | [https://www.mdpi.com/1424-8220/22/16/5936](https://www.mdpi.com/1424-8220/22/16/5936) |
| 20 | Node Calibration in UWB-Based RTLSs Using Multiple Simultaneous Ranging | Sensors, 2022 | [https://www.mdpi.com/1424-8220/22/3/864](https://www.mdpi.com/1424-8220/22/3/864) |
| 21 | Performance Evaluation of a UWB Positioning System Applied to Static and Mobile Use Cases in Industrial Scenarios | Electronics, 2022 | [https://www.mdpi.com/2079-9292/11/20/3294](https://www.mdpi.com/2079-9292/11/20/3294) |
| 22 | A Low-Cost Foot-Placed UWB and IMU Fusion-Based Indoor Pedestrian Tracking System for IoT Applications | Sensors, 2022 | [https://www.mdpi.com/1424-8220/22/21/8160](https://www.mdpi.com/1424-8220/22/21/8160) |
| 23 | Localization in Structured Environments with UWB Devices without Acceleration Measurements, and Velocity Estimation Using a Kalman–Bucy Filter | Sensors, 2022 | [https://www.mdpi.com/1424-8220/22/16/6308](https://www.mdpi.com/1424-8220/22/16/6308) |
| 24 | Experimental Evaluation of IEEE 802.15.4z UWB Ranging Performance under Interference | Sensors, 2022 | [https://www.mdpi.com/1424-8220/22/4/1643](https://www.mdpi.com/1424-8220/22/4/1643) |
| 25 | Improving the Accuracy of Decawave's UWB MDEK1001 Location System by Gaining Access to Multiple Ranges | Sensors, 2021 | [https://www.mdpi.com/1424-8220/21/5/1787](https://www.mdpi.com/1424-8220/21/5/1787) |
| 26 | Robust GNSS Shadow Matching for Smartphones in Urban Canyons | IEEE Sensors Journal, 2021 | [https://ieeexplore.ieee.org/document/9440901/](https://ieeexplore.ieee.org/document/9440901/) |
| 27 | Precise Positioning of Autonomous Vehicles Combining UWB Ranging Estimations with On-Board Sensors | Electronics, 2020 | [https://www.mdpi.com/2079-9292/9/8/1238](https://www.mdpi.com/2079-9292/9/8/1238) |
| 28 | Bias Analysis of Parameter Estimator Based on Gauss-Newton Method Applied to Ultra-Wideband Positioning | Applied Sciences, 2020 | [https://www.mdpi.com/2076-3417/10/1/273](https://www.mdpi.com/2076-3417/10/1/273) |
| 29 | Optimization of Time Synchronization and Algorithms with TDOA Based Indoor Positioning Technique for Internet of Things | Sensors, 2020 | [https://www.mdpi.com/1424-8220/20/22/6513](https://www.mdpi.com/1424-8220/20/22/6513) |
| 30 | Identification of NLOS and Multi-Path Conditions in UWB Localization Using Machine Learning Methods | Applied Sciences, 2020 | [https://www.mdpi.com/2076-3417/10/11/3980](https://www.mdpi.com/2076-3417/10/11/3980) |
| 31 | SnapLoc: An Ultra-Fast UWB-Based Indoor Localization System for an Unlimited Number of Tags | Proceedings of IPSN '19, 2019 | [https://dl.acm.org/doi/10.1145/3302506.3310389](https://dl.acm.org/doi/10.1145/3302506.3310389) |
| 32 | Analysis of the Scalability of UWB Indoor Localization Solutions for High User Densities | Sensors, 2018 | [https://www.mdpi.com/1424-8220/18/6/1875](https://www.mdpi.com/1424-8220/18/6/1875) |
| 33 | Improving UWB-Based Localization in IoT Scenarios with Statistical Models of Distance Error | Sensors, 2018 | [https://www.mdpi.com/1424-8220/18/5/1592](https://www.mdpi.com/1424-8220/18/5/1592) |
| 34 | Research on Extended Kalman Filter and Particle Filter Combinational Algorithm in UWB and Foot-Mounted IMU Fusion Positioning | Mobile Information Systems, 2018 | [https://www.hindawi.com/journals/misy/2018/1587253/](https://www.hindawi.com/journals/misy/2018/1587253/) |
| 35 | Enhanced Map-Matching Algorithm with a Hidden Markov Model for Mobile Phone Positioning | ISPRS International Journal of Geo-Information, 2017 | [https://www.mdpi.com/2220-9964/6/11/327](https://www.mdpi.com/2220-9964/6/11/327) |
| 36 | Loose and Tight GNSS/INS Integrations: Comparison of Performance Assessed in Real Urban Scenarios | Sensors, 2017 | [https://www.mdpi.com/1424-8220/17/2/255](https://www.mdpi.com/1424-8220/17/2/255) |
| 37 | A Practical, Robust and Fast Method for Location Localization in Range-Based Systems | Sensors, 2017 | [https://www.mdpi.com/1424-8220/17/12/2869](https://www.mdpi.com/1424-8220/17/12/2869) |
| 38 | Survey on Ranging Sensors and Cooperative Techniques for Relative Positioning of Vehicles | Sensors, 2017 | [https://www.mdpi.com/1424-8220/17/2/271](https://www.mdpi.com/1424-8220/17/2/271) |

## Table B — Additional supporting literature found 2026-10-26 during submission preparation, NOT prior use (14 papers, none on the PPT slide).

| No. | Paper | Source / Year | Link | Backs | Does NOT prove |
|---|---|---|---|---|---|
| B1 | MmWave Radar Point Cloud Segmentation using GMM in Multimodal Traffic Monitoring — Jin, Sengupta, Cao & Wu | Proc. IEEE International Radar Conference (RADAR) 2020, pp. 732–737 | [https://ieeexplore.ieee.org/document/9114662/](https://ieeexplore.ieee.org/document/9114662/) | Single mmWave FMCW radar physically detects and separates vehicles from pedestrians and clutter from its point cloud; FMCW basis, up to 4 GHz bandwidth, up to ~5 cm range resolution | 60 GHz specifically (hardware is 76–81 GHz AWR1843); roadside divider-pole geometry; traffic-speed tracking; carriageway classification. Segmentation scores modest (IoU 0.56 cars) — supports detection, not precise localisation |
| B2 | Tracking of Multiple Static and Dynamic Targets for 4D Automotive Millimeter-Wave Radar Point Cloud in Urban Environments — Tan, Ma, Zhu, Li, Zheng, Huang & Bai | Remote Sensing 15(11), 2923, 2023 | [https://www.mdpi.com/2072-4292/15/11/2923](https://www.mdpi.com/2072-4292/15/11/2923) | Density-based clustering of a multi-point vehicle return, Kalman tracking, dynamic/static separation, track birth/retention/deletion on real data | Roadside/static-pole sensing (radar moves with ego vehicle); 60 GHz operation (band not stated); UWB association; carriageway classification; any transferable accuracy |
| B3 | Automotive Radar-Based Vehicle Tracking Using Data-Region Association — Cao, Lan, Li & Liu (full first names were not retrieved — initials only appear in all records consulted) | IEEE Transactions on Intelligent Transportation Systems 23(7), 8997–9010, 2022 | [https://ieeexplore.ieee.org/document/9525315/](https://ieeexplore.ieee.org/document/9525315/) | Vehicle-as-extended-target tracking from radar alone; association must handle a spread of reflection points per vehicle — why clustering comes before tracking | Roadside pole geometry; 60 GHz operation; carriageway classification; UWB fusion; any transferable accuracy. Real-vs-simulated details were not verified; no real-road result is presented |
| B4 | The Hungarian method for the assignment problem — Kuhn, Harold W. | Naval Research Logistics Quarterly 2(1–2), 83–97, 1955 | [https://onlinelibrary.wiley.com/doi/10.1002/nav.3800020109](https://onlinelibrary.wiley.com/doi/10.1002/nav.3800020109) | Track-to-measurement/session assignment as an optimal 1-to-1 assignment with a cost matrix, not ad-hoc greedy matching — mathematical basis for Hungarian assignment | Anything about sensors, radar, vehicles, tracking, or UWB. Does not prove Hungarian works at roadside speeds or with range-residual costs; the cost matrix is the team's own design |
| B5 | An algorithm for tracking multiple targets — Reid, Donald B. | IEEE Transactions on Automatic Control 24(6), 843–854, 1979 | [https://ieeexplore.ieee.org/document/1102177/](https://ieeexplore.ieee.org/document/1102177/) | MHT: tracks as hypotheses, measurement as existing track / new target / false report, hypothesis grooming — supports track initiation, false-report handling, multi-observation association | Not radar, vehicle, roadside, UWB, or zone logic. General 1979 algorithm; does not validate radar–UWB session binding; MHT tree growth is a real Pi 5 compute cost to bound |
| B6 | Automotive radar system for multiple-vehicle detection and tracking in urban environments — Eltrass & Khalil | IET Intelligent Transport Systems 12(8), 783–792, 2018 | [https://ietresearch.onlinelibrary.wiley.com/doi/10.1049/iet-its.2017.0370](https://ietresearch.onlinelibrary.wiley.com/doi/10.1049/iet-its.2017.0370) | UKF-CTRA, JPDA for association in clutter, composite M/N tests for track management, on single and multi-target scenarios | Ego-vehicle radar, not roadside pole; no UWB, no tag session, no zone classification, no NHZ/LBZ actuation; does not prove AWRL6432 behaviour or 50 km/h pole-interval performance |
| B7 | Protection level optimization method of ARAIM algorithm for urban road safety — Deng, Sun, Zhang et al. (full author list beyond first three not fully confirmed; article in Chinese, abstract in English) | Journal of Beijing University of Aeronautics and Astronautics 51(1), 222–234, 2025 (online 2023-05-11) | [https://www.chndoi.org/Resolution/Handler?doi=10.13700/j.bh.1001-5965.2022.1020](https://www.chndoi.org/Resolution/Handler?doi=10.13700/j.bh.1001-5965.2022.1020) | Computed protection/confidence bound compared against an alert limit, integrity risk traded against continuity risk — direct analogue of a σ cap → UNKNOWN rule | GNSS ARAIM, not UWB/radar roadside positioning. Protection levels at a 10 m alert-limit scale — far coarser than a 1.5 m divider. Backs the bound-vs-limit concept only, not any numeric bound |
| B8 | Integrity Performance for Precise Positioning in Automotive — Norman, Infante & de Groot | Proc. ION GNSS+ 2019, Miami, pp. 1653–1663 | [https://www.ion.org/publications/abstract.cfm?articleID=16996](https://www.ion.org/publications/abstract.cfm?articleID=16996) | Protection levels computed on automotive hardware, assessed against actual error and alert limits, continuing to bound error under injected faults | GNSS/PPP+INS automotive positioning, not pole-side UWB/radar. 1–2 m protection levels are open-sky figures, not urban-canyon or divider-scale figures. Does not prove any V-ORRA numeric bound |
| B9 | Design and Experiment of Autonomous Flight UAS Geofence Algorithm — Fu, Liang, Zhang, Qi & Zhang (proceedings TOC pp. 132–136; one citation index lists 137–143) | Proc. 2nd International Conference on Computer Science and Software Engineering (CSSE 2019), ACM | [https://dl.acm.org/doi/10.1145/3339363.3339381](https://dl.acm.org/doi/10.1145/3339363.3339381) | Spatial hysteresis concept: hard boundary alone misjudges crossings, so a pre-control / buffer layer is needed before the hard line | UAS, not road vehicles; buffer is a spatial pre-control layer, not a dwell-time confirmation rule — no dwell-time value, no consecutive-fix confirmation, no time-based state machine studied |
| B10 | Geofencing Motion Planning for Unmanned Aerial Vehicles Using an Anticipatory Range Control Algorithm — Thomas & Sarhadi | Machines (MDPI) 12(1), 36, 2024 | [https://www.mdpi.com/2075-1702/12/1/36](https://www.mdpi.com/2075-1702/12/1/36) | Polygon geozone definition and point-in-polygon inclusion testing, including concave zones and PIP edge cases; anticipatory soft-zone/buffer principle engaging before the hard border | Fixed-wing UAV, high-fidelity 6-DoF simulation only — no hardware flight test of the anticipatory controller; control-override paper, not advisory zone-state paper; no dwell-time rule; no road vehicles, roadside sensing, radar/UWB, or multi-vehicle association |
| B11 | Performance evaluation of MQTT and CoAP via a common middleware — Thangavel, Ma, Valera, Tan & Tan | IEEE ISSNIP 2014, Singapore | [https://ieeexplore.ieee.org/document/6827678/](https://ieeexplore.ieee.org/document/6827678/) | MQTT as a suitable lightweight layer for resource-constrained edge devices | 2014, pre-MQTT-5.0; WSN middleware testbed, no vehicle, no roadside, no 2.88 s pole-interval budget, no QoS 2 latency guarantee. Latency still needs bench measurement |
| B12 | Security model design and formal verification of MQTT protocol — Gao, Xi & Zhang | Discover Applied Sciences 7, 1227, 2025 | [https://link.springer.com/10.1007/s42452-025-07749-w](https://link.springer.com/10.1007/s42452-025-07749-w) | Plain MQTT security gaps as the reason for an application-level gate; MQTT-SA adds identity authentication, key generation and encrypted data, formally verified with Scyther; nonces and timestamps as freshness ingredients | MQTT-SA is a proposed protocol with a client–broker–server triple, NOT plain MQTT as run on local Mosquitto; model verification ≠ deployment result; no vehicle context; the forged-but-well-formed-command gap remains open |
| B13 | MARAS: Mutual Authentication and Role-Based Authorization Scheme for Lightweight Internet of Things Applications — Şeker, Dalkılıç & Çabuk | Sensors 23(12), 5674, 2023 | [https://www.mdpi.com/1424-8220/23/12/5674](https://www.mdpi.com/1424-8220/23/12/5674) | MQTT natively offers little beyond username/password and lacks mutual authentication — reason for a gate; mutual auth plus role/topic authorisation on constrained devices, HMAC one-time passwords making intercepted credentials unusable on replay | MARAS depends on a trusted OAuth 2.0 server alongside MQTT, which this control loop does not have — only principles transfer, not the scheme; lab proof-of-concept, not a vehicle deployment; HOTP ≠ sequence numbers |
| B14 | Functional Safety Networks and Protocols in the Industrial Internet of Things Era — Peserico, Morato, Tramarin & Vitturi | Sensors 21(18), 6073, 2021 | [https://www.mdpi.com/1424-8220/21/18/6073](https://www.mdpi.com/1424-8220/21/18/6073) | Black-channel principle: treat the channel as untrusted, put a safety layer in application software; CRC, message numbering and timestamping against corruption, loss and delay; watchdog timeout → safe state | Industrial automation (FSoE/ProfiSAFE), not vehicles; certified industrial profiles — this design implements only the pattern in application code and holds no such certification; does not validate MQTT payload handling |

## What this plan does not claim
- No divided-road, traffic-speed carriageway precision/recall experiment
  exists in these papers for this exact pole + tag system.
- No 60 GHz roadside vehicle experiment was found in this set — radar
  papers are 76–81 GHz or band-unstated, and the 60 GHz case rests on
  shared FMCW physics plus manufacturer documentation, not on a 60 GHz
  vehicle experiment found here.
- Dwell-time confirmation has no dedicated paper in this set — it remains
  team engineering practice, supported by analogy to buffer bands and
  track-confirmation tests.
- Wi-Fi association + MQTT latency at speed, the sequence gate itself,
  and LBZ held-maximum are unmeasured and undefined in this plan, and
  are listed as work to test, not as proven results.
- Manufacturer documents (TI, Makerfabs, Espressif, u-blox, Raspberry Pi)
  and the MQTT specification are not counted in the 52 papers above.
  They prove component interfaces, not system performance.
