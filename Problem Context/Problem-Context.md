# Problem Context

Context numbers, as cited in the submitted deck, not as V-ORRA measurements:

- WHO guidance: noise below 53 dB for health-related guidance, as cited.
- Indian silence-zone limits: 50 dB(A) day / 40 dB(A) night, as cited.
- India road fatalities 2024: 177,175, as cited in the deck.

Why GPS geofencing alone was not chosen: GPS position alone, at the
accuracy of a low-cost module, cannot reliably resolve which side of
a 1.5 m median a vehicle is on, and divider-only UWB ranging cannot
resolve carriageway by range alone, because +y and -y give the same
range (mirror ambiguity). That geometry result is why this design uses
radar tracking bound to a UWB session, with a surveyed transverse
anchor layout, and why a failed or ambiguous classification becomes
UNKNOWN, not a forced decision.

Why a camera was not used: by team decision, the architecture is
camera-free. Privacy is log-based only — no video, no number plates,
no faces are used anywhere in this design.
