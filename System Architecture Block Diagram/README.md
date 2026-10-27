# System Architecture Block Diagram

V-ORRA Intelligent Vehicle Signalling & Connected Road Safety System.
Application: VK26-IITH-SC-0177, Vishwakarma Awards 2026-27, Sustainable
Cities / Smart Urban Infrastructure & Asset Tracking. Team: ITM University
Gwalior. Faculty mentor: Ashish Singh Chandel, Professor of Practice, SOET,
ITM University Gwalior.

## Reading order

1. System Architecture Block Diagram — the complete system.
2. Pole Unit Electronics and Data Flow — the roadside pole unit, in its
   own subfolder.
3. Smart Vehicle Electronics and Actuation — the vehicle unit, in its own
   subfolder.

Each diagram is supplied in two forms: a PDF for viewing, and an editable
drawing file for diagrams.net. The PDF and the drawing file contain the
same diagram.

## Complete flow

Zone Mapping Software defines surveyed No Horn Zone and Low Beam Zone
boundaries. The roadside pole unit senses a vehicle, associates the radar
track with the UWB tag range, and decides whether the vehicle is inside,
outside, or unknown relative to a zone. A separate pole safety gate
authorizes a vehicle-specific command. The command travels through the
pole-hosted MQTT broker to the vehicle unit. The vehicle unit validates
gate authorization, session, sequence, replay, and expiry independently
before acting, and returns applied-state acknowledgement and fault status.

Only two wireless links cross between pole and vehicle: UWB two-way
ranging for identity and range, and 2.4 GHz Wi-Fi / MQTT for commands.
No cable connects pole and vehicle. Each unit has its own independent
power supply and local ground.

## Pole unit

The pole unit contains a 60 GHz radar, a UWB anchor, a powered USB hub,
a Raspberry Pi 5 edge processing unit, and a separate ESP32-S3 safety
gate. Radar and anchor reports enter the Pi through the hub. The Pi
acquires, calibrates, tracks, associates, and geofences. The gate
approves a proposal only when all checks pass: gate armed, proposal
valid, association acceptable, confidence sufficient, session valid,
command fresh, and watchdog and authorization state healthy. Otherwise
the gate denies a new restrictive command.

## Vehicle unit

The vehicle unit contains a bare UWB tag, an ESP32-S3 vehicle controller,
an auxiliary horn suppression circuit, an auxiliary lamp circuit, and a
physical override. GPS is diagnostic context only and is never a zone
authority. The vehicle unit validates every command before acting.

Horn suppression uses a normally-closed relay and a deadman timer.
Suppression requires fresh deadman pulses and a coil command. Suppression
lifts on expiry, fault, or link loss, and the normally-closed relay
returns the auxiliary horn to available when coil power is lost.

Lamp control uses a lamp mode latch. The latch holds the HIGH or LOW
mode on stale or unknown data. Stale or unknown lighting data freezes
the lamp mode. No automatic HIGH-beam command is issued on uncertainty.

The physical override cuts the automatic actuation rail and bypasses
the relay. The override wins even if firmware hangs. Manual HIGH/LOW
lamp selection and horn bypass remain available under override.

## Safety boundary

All actuation is auxiliary demonstration hardware on a demo unit's own
power supply, on a closed course only. Nothing is wired into a vehicle's
own horn, headlamp, or battery circuits. No camera, video, number plate,
or face data is used anywhere in this architecture. Privacy is log-based
only.

## What the diagrams are, and what they are not

These block diagrams show functions and interfaces. They do not show
pin numbers, component part numbers, or final circuit wiring. Detailed
wiring, part numbers, and voltages belong to the electronic schematics
in Electronics Schematics & BOM. Where a block diagram and a schematic
differ in level of detail, the schematic defines the circuit.

These diagrams were prepared on 27 October 2026. They replace the
AI-generated block diagram image used in the Stage 2 presentation. They
are not Stage 2 submission files, and they are not presented as Stage 2
work.

This architecture is design and theory, based on available evidence and
resources. It has not been bench-tested as a complete system, and it is
not presented as final. Physical testing remains required for a final
version, because real component tolerances differ from datasheet ideal
conditions, timing, power, and UWB and radar accuracy are unmeasured on
the actual boards, calibration and multi-vehicle and environmental
effects cannot be proven by theory alone, and surveyed site geometry
will change zone boundaries.
