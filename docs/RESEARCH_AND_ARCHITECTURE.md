# Smart Blind Stick: Research and Architecture Plan

Date: 2026-10-28
Status: research baseline; hardware inventory and on-device measurements still required.

## 1. Project intent

Build an affordable ESP32-based assistive prototype that supplements (never replaces) a traditional white cane. The system should provide local obstacle alerts, optional camera-based object context, tactile/audio feedback, and user-triggered SOS with location. Cellular data and location are support features; the local obstacle warning must not depend on them.

## 2. User-confirmed inventory and selected product requirements

- Two controller boards are available, but exact models are unknown.
- Vibration, buzzer and audio feedback are reported available; module types and driver circuitry are unknown.
- No distance sensor is currently confirmed available.
- Camera board/model is unknown.
- Battery and regulator are unknown.
- Connectivity choice: cellular data plus an Android companion app (not SMS-only).
- Location choice: GNSS/GPS on the stick plus phone-location fallback where the app has permission and a fresh fix.
- SOS: user-triggered only; no scheduled check-in/periodic location reporting in v1.

The project cannot yet produce a final GPIO map, wiring diagram, battery sizing, or end-to-end firmware. First collect board/module markings and identify the power source.

## 3. Architecture recommendation

- Controller A: intended proximity/alert controller. Sample the eventual distance sensor, reject implausible/stale measurements, apply filtering and hysteresis, and control vibration/buzzer through verified driver circuits.
- Controller B: optional camera/context controller after its exact board and camera are identified. Vision output is advisory and must never clear, delay or downgrade a local proximity alert.
- Cellular/location subsystem: a modem board with LTE data, GNSS, documented UART/USB, TLS/HTTPS support and available carrier-board documentation is a candidate. Do not select a module by family name alone: exact SKU, region suffix, LTE band table, GNSS support, antenna connectors, supply voltage/current, SIM interface and firmware must be checked.
- A SIMCom SIM7600 family board is a research candidate because the family offers LTE and GNSS variants. This is not a selected part or proof of compatibility. A variant supporting LTE Band 3 (1800 MHz) is a useful starting requirement, based on ARPCE's published frequency information, but confirm the current assigned bands for the intended operator and test with a real SIM in the target area before purchase. Do not assume a single band guarantees coverage.
- Android companion app can supply fallback location only if it has location permission, obtains a fresh fix and sends it to the authenticated service. A server cannot magically obtain the phone's location if the phone/app is offline or has no permission.
- Suggested SOS path: physical SOS -> create unique event ID -> request current GNSS fix with a bounded wait -> send event using authenticated HTTPS/TLS -> server stores/accepts and returns explicit acknowledgement -> app displays event and location source/age. If GNSS fails, report “location unavailable” rather than pretending coordinates exist. If network fails, retain only a bounded pending event and indicate “not delivered”.
- If phone-location fallback is desired for a device-triggered SOS, explicitly design how the phone and device correlate the same event. For v1, a simpler separate “Send SOS with phone location” action in the app may be more reliable than assuming the phone will always be nearby and online.
- Inter-controller link: consider UART only after confirming both board models, exposed pins and logic levels. Use a versioned, bounded frame with message type, sequence number, integrity check and freshness handling. Ignore stale/invalid camera messages.
- Power: verify each board's supply requirements and peak current before finalizing battery/regulator. Cellular transmit bursts can cause brownouts. Do not drive a vibration motor or speaker directly from GPIO without verifying the load and suitable driver.

## 4. Safety, privacy and reliability constraints

1. The electronic prototype supplements rather than replaces a conventional white cane, mobility training, or a human guide.
2. Sensor, camera, network or controller-link failures must not silently suppress the basic warning path.
3. Never infer that an area is safe because no object was detected.
4. Explicitly test overhead obstacles, narrow/small objects, glass/reflective surfaces, low obstacles, stairs and drop-offs, wet surfaces, sunlight, rain, and crowded/noisy environments.
5. Keep high-priority alerts concise and non-blocking; vibration should remain available when audio is noisy or inappropriate.
6. SOS data should be minimized: event time, unique ID, location plus source/age/quality, and battery/connectivity state only as needed. Define contact consent, retention/deletion, access control and device revocation.
7. Use TLS with certificate validation and authenticated device requests; no secret API keys in firmware or the app. The backend must deduplicate event IDs and return explicit acknowledgements.
8. An SOS delivery failure must be visible locally. A submitted request is not proof the companion received or acted on it.
9. No automatic periodic tracking in v1. Obtain explicit consent before any future background location tracking.
10. Do not claim medical-device status, certification, or real-world safety until independent validation supports those claims.

## 5. Research findings

### On-device vision feasibility

Published work describes a dual-microcontroller smart cane using ESP32 and ESP32-CAM with on-device TinyML, and a separate 2025 TinyML study reports a 78.1% F1 score with 1.368 s inference on its tested hardware. These are feasibility signals, not performance guarantees for our board or dataset. Benchmark image capture, inference, end-to-end alert latency, memory, heat, and battery runtime on the exact hardware before deciding that vision is useful.

Sources:
- https://doi.org/10.1109/siot68426.2025.11368785
- https://doi.org/10.1109/icvee66651.2025.11281396
- https://doi.org/10.1109/icac54203.2021.9671191

### Sensor fusion and user feedback

Reviews of technology-assisted white canes report that sensors have different strengths and blind spots; ultrasonic sensing is common but has limited angular resolution, while optical/IR performance can degrade in bright sunlight or vary with target/surface. Vibration can be less affected by ambient noise and may improve privacy; audio should not block environmental hearing. Multiple sensing modes can help but do not automatically make a system safe.

Sources:
- https://pmc.ncbi.nlm.nih.gov/articles/PMC6292384/
- https://pmc.ncbi.nlm.nih.gov/articles/PMC6453076/
- https://pmc.ncbi.nlm.nih.gov/articles/PMC10007677/

### ToF option

ST's VL53L1X product page lists ranging up to 4 m and up to 50 Hz under specified conditions, with a typical 27-degree full field of view. Actual range varies with target reflectance, ambient light, mode, and setup; it must not be treated as a guaranteed 4 m detection envelope.

Sources:
- https://www.st.com/en/imaging-and-photonics-solutions/vl53l1x.html
- https://www.st.com/resource/en/datasheet/vl53l1x.pdf

### Cellular modem and Algeria compatibility

SIMCom documents LTE/GNSS capabilities across the SIM7600 family, but band support varies by exact model and suffix. The SIM7600 family page lists different frequency combinations across variants; therefore “SIM7600” alone is not a sufficient purchasing specification. Begin by checking for LTE Band 3 support against the ARPCE frequency information, then verify the current operator-specific allocation and coverage in the intended area with a real SIM. Confirm the exact development-board revision, GNSS support, antennas, APN, supply requirements and local equipment approval status before purchase.

Sources:
- SIMCom SIM7600 family: https://www.simcom.com/product/SIM7600.html
- SIMCom SIM7600-H family: https://www.simcom.com/product/SIM7600X-H.html
- ARPCE 4G information: https://www.arpce.dz/ar/service/4g
- ARPCE frequency-management report: https://www.arpce.dz/fr/file/w5i7z4
- ARPCE equipment approval: https://www.arpce.dz/fr/service/agr

### NVIDIA skills fit

NVIDIA's skills catalog includes Physical AI, robotics, computer vision and Jetson workflows. These may help future simulation, dataset, or Jetson-based inference work, but Jetson-specific skills do not directly accelerate firmware running on an ESP32-class microcontroller. Do not add NVIDIA hardware or paid/cloud dependencies unless a measured requirement justifies them.

Sources:
- https://github.com/NVIDIA/skills
- https://docs.nvidia.com/skills

## 6. Implementation phases

### Phase A — inventory and baseline
- Record exact model/revision for both boards, camera sensor, PSRAM presence, output modules/driver, battery and regulator.
- Identify and select a distance sensor based on local price/availability, voltage/interface, outdoor behavior, mounting and testability.
- Capture current firmware, pin map, wiring photo, and known working test.
- Build a BOM with local prices in DZD.
- No purchases until exact module SKU, local price and budget impact are reviewed.

### Phase B — deterministic obstacle alert
- Implement and test sensor reading, plausibility checks, filtering, hysteresis and distinct near/urgent states.
- Test each sensor independently and measure false-positive/false-negative cases.
- Verify boot, sensor disconnect, stuck readings, low battery, and reset behavior.
- Keep “sensor fault/unknown” distinct from “no obstacle detected”.

### Phase C — camera and inter-controller protocol
- Verify camera capture and JPEG frame rate on the exact camera board.
- Implement the versioned, bounded protocol in the companion test-plan document only after confirming compatible UART pins and logic levels.
- Measure message loss, stale-message rejection, camera-to-hint latency and memory/PSRAM use. If inference is too slow, retain camera only as optional context.

### Phase D — connectivity, location and SOS
- First test modem/GNSS on a separate bench power supply suitable for the carrier board; record current and brownout behavior.
- Verify SIM registration, APN, LTE data, HTTPS/TLS, weak-signal reconnect and data usage using a local operator SIM.
- Verify GNSS fix and record location accuracy/source/age in open sky and difficult environments.
- Define authenticated backend endpoints, device identity, event ID/deduplication, acknowledgement, bounded retries and pending-event behavior.
- Build the minimum Android app flow: contacts/consent, manual SOS, location permission, event status and map link. Keep background tracking and periodic reporting out of v1.
- Test that device GPS and phone location are never silently confused.

### Phase E — human interface and power
- Define vibration patterns with user feedback; keep audio optional and brief.
- Measure battery runtime under worst-case camera, modem transmit, sensor and vibration load.
- Make charging, enclosure, strain relief and water protection explicit design items.

### Phase F — validation and release gate
- Build a repeatable test matrix for object size, material, distance, angle, lighting, motion, stairs/drop-offs and environmental noise.
- Test cellular failures: no SIM, no coverage, weak signal, wrong APN, data exhausted, backend timeout, TLS/auth failure, stale/missing GNSS, duplicate SOS, retry exhaustion and acknowledged delivery.
- Test with non-hazardous stationary obstacles first; do not test by exposing a blind person to uncontrolled hazards.
- Document limitations and known failure modes. Only claim verified results from logged measurements.

## 7. Exit criteria before any real-world trial

- No unhandled crash or lock-up in an extended bench test.
- Basic alert continues when the camera, modem, network or secondary controller is unavailable.
- Measured alert latency and battery runtime are documented.
- Sensor failure, stale messages, low power and SOS delivery failure produce defined visible states.
- A test SOS reaches the authenticated service and companion app with explicit receipt evidence; location source and age are visible.
- Known blind spots are disclosed; controlled user evaluation is supervised and follows appropriate ethics and accessibility practices.

## 8. Open questions blocking final wiring and purchase

1. What are the exact models/revisions of both controller boards?
2. What is the exact camera board and camera sensor?
3. Which output modules are available, and does each have a driver/amplifier?
4. What battery and regulator are available?
5. Which distance sensor should be acquired after comparing local availability and price?
6. Which cellular modem development board is locally available, what exact suffix/revision is it, and does it support the required LTE bands and GNSS?
7. Which operator SIM is intended, and what are the data plan, APN and coverage at likely usage locations?
8. What is the maximum acceptable total cost in DZD for modem, antennas, power circuitry, sensor, enclosure and any hosting?
9. Should v1 use a minimal hosted backend, or is a local phone-to-device design acceptable? Any backend must be secured and its recurring costs disclosed.
10. Is the phone expected to be near the stick during SOS? If not, phone-location fallback cannot be assumed.

Do not assign final GPIO pins, battery sizing, or voltage wiring until these are answered.

## 9. Change control

- Work on research/smart-blind-stick-architecture.
- Keep changes small and reviewable.
- Do not merge, publish a release or claim successful hardware tests without evidence and explicit user approval.
