# Smart Blind Stick — Project Roadmap and Execution Method

Date: 2026-10-28
Status: planning baseline. This is a plan, not evidence of completed hardware validation.

## 1. Goal and scope

Build a low-cost prototype that supplements a conventional white cane by giving additional warnings about selected obstacles. The first version prioritizes predictable, local proximity alerts and accessible feedback. Camera-based object recognition is a later, optional context feature and is not part of the safety-critical path.

Out of scope for the first prototype:
- Claiming that the device replaces a white cane, orientation/mobility training, or a guide.
- Claiming detection of every obstacle, stairs, drop-offs, vehicles, or overhead hazards.
- Depending on cloud connectivity for basic alerts.
- Calling the prototype certified or clinically validated.

## 2. Current knowns and unknowns

Known from user:
- Two controller boards are available.
- Feedback options: vibration, buzzer and audio.
- No distance sensor is confirmed available.
- The selected concept now includes cellular data connectivity, an Android companion app, GPS/GNSS on the stick plus phone-location fallback, and manual SOS only (no periodic check-in in the first version).

Unknown:
- Exact controller models/revisions and logic voltage.
- Camera module, sensor and PSRAM.
- Whether vibration, buzzer and audio are separate modules; their voltage/current and driver circuits.
- Battery chemistry, cell count, regulator, charging/protection hardware.
- Exact cellular module/board and regional variant.
- SIM operator, local LTE coverage, SIM/data plan and operator APN.
- Whether the companion app can be hosted/served securely and how the phone fallback location will be shared.
- Existing code, IDE/build environment and actual board tests.

These unknowns block a final pin map, wiring diagram, BOM quantities, battery design and hardware-dependent firmware. Resolve them before committing to pin-level instructions.

## 3. Product requirements (initial draft)

R1. Core proximity warning operates locally without Wi-Fi, cloud or camera inference.
R2. A missing, disconnected, implausible or stale sensor reading is reported as a fault/unknown state, never as “clear”.
R3. Vision messages are advisory only; they cannot suppress or delay proximity alerts.
R4. Basic tactile warning remains available when audio is muted or environmental noise is high.
R5. Output loads are driven through appropriately rated hardware, not assumed safe for direct GPIO drive.
R6. Startup, reset, low power and communication loss have defined behavior.
R7. Limitations and test evidence are recorded in the repository.
R8. The device is always described as a supplement to a conventional white cane.
R9. SOS is user-triggered; the prototype does not send scheduled periodic check-ins in v1.
R10. An SOS event should include event ID, timestamp, last known coordinates, location age/quality, battery state and connectivity state when available.
R11. The companion service must acknowledge event receipt; the device must not report “sent/delivered” without the corresponding acknowledgement. Queue/retry behavior must be bounded and visible.
R12. If GNSS is unavailable, offer phone location only when the companion app has the required permission and a fresh location. Clearly label location source and age; never silently substitute stale coordinates.
R13. Cellular/internet/app failure must not disable local obstacle alerts. The UI must show SOS delivery as pending/failed when not acknowledged.
R14. Protect credentials and location data: use authenticated HTTPS/TLS, device-specific credentials, minimal retention, explicit contact consent, and no public unauthenticated endpoint. Do not embed service secrets in the Android app or firmware.

## 4. Proposed architecture

### Controller A — local sensing and alert state machine
Responsibilities:
- Acquire distance samples from the selected sensor.
- Validate range and sample age; filter noise and apply hysteresis.
- Produce distinct states such as STARTING, SENSOR_FAULT, CLEAR_MEASUREMENT, NEAR, URGENT and LOW_POWER.
- Control vibration/buzzer via verified drivers.
- Record diagnostic counters and handle resets without unsafe output behavior.

Important: CLEAR_MEASUREMENT means only that the selected sensor did not detect a target within its verified measurement envelope. It does not mean the path is safe.

### Controller B — optional camera/context processor
Responsibilities:
- Initialize the confirmed camera board and capture frames.
- Add object classification only after a baseline capture test and feasibility measurement.
- Send short, versioned and integrity-checked context messages to Controller A.
- Signal unavailable/stale status if capture or inference fails.

The second controller is not automatically assumed to be an ESP32-CAM until its markings are checked.

### Connectivity and location subsystem — proposed, not selected
- Candidate: a cellular modem/development board supporting LTE data, GNSS, UART/USB and HTTPS/TLS. SIMCom SIM7600 family is a research candidate, not a final part number; regional suffix/band table must be verified before purchase. Some variants also support SMS/voice, but v1 requirements do not depend on SMS or calling.
- A separate GNSS receiver is an alternative if the chosen modem variant lacks GNSS or its antenna/layout is unsuitable.
- Android companion app can provide fallback location only when permission is granted and a fresh location is available. A phone-only location is not available when the phone is disconnected, off, or the app cannot access location.
- Suggested flow: physical SOS button -> locally record event -> obtain current GNSS fix within a bounded wait -> send event over LTE data to authenticated backend -> backend acknowledges -> companion app receives/presents event. If GNSS fails, send an explicit “location unavailable” event; if network fails, retain a bounded pending event and show failure locally.
- The phone fallback needs a separate, explicit design: for example, the companion app can attach its fresh location to an SOS request it initiates. Do not assume a remote server can retrieve a phone's location without app permission and a phone-to-service message.
- Keep the device-side interface modular. Cellular modem integration should not consume pins needed for the camera or interfere with the local sensor/alert path.

### Link between the two controllers
UART is a candidate, not yet a final choice. Confirm board pinouts, voltage levels, available UARTs and conflicts first. Use bounded framing, protocol version, sequence number, CRC, parser recovery and freshness timeout. Never allow a stale or invalid message to clear a local warning.

### Feedback
Vibration is the primary quiet, tactile channel. Buzzer/audio are secondary and configurable. Define distinct patterns only after testing that they are distinguishable and do not overwhelm environmental hearing. Confirm what “audio” hardware actually means before designing its driver.

### Power
Select power architecture only after the boards, modem board and output loads are identified. Verify peak current and transient response, regulator headroom, battery protection/charging and brownout behavior. Cellular transmit bursts can reset an undersized supply; use the exact carrier-board documentation and measured current profile. Do not choose a battery based on nominal capacity alone. Use separate/isolated supply branches if measurements and regulator design require it, with a common reference only as specified by the board interface.

## 5. Development phases and deliverables

### Phase 0 — requirements and inventory (current)
Deliverables:
- This roadmap and open-question list.
- Exact board/module inventory, photos and datasheet links.
- Costed BOM in DZD with “already owned”, “must buy”, and optional items.
Exit gate: every part used in the first prototype has a known model or clearly marked uncertainty.

### Phase 1 — choose sensor and system boundaries
Activities:
- Compare at least two feasible distance-sensor options based on local price/availability, interface/voltage, outdoor behavior, field of view, mounting and limitations.
- Choose which controller owns the sensor and outputs.
- Write requirements and measurable test cases.
Exit gate: sensor part number and safe electrical interface selected.

### Phase 2 — cellular/location/app feasibility spike
Activities:
- Confirm exact modem SKU and hardware revision, GNSS support, LTE bands, SIM voltage/format, antenna connectors and vendor carrier-board supply requirements.
- Confirm a local operator SIM/data plan works at the intended use locations; test attach, APN, TLS/HTTPS, reconnect, low signal and data usage.
- Test GNSS time-to-first-fix and accuracy outdoors/indoors, with location age recorded.
- Define the minimal secure backend contract and Android SOS screen; no periodic tracking or check-in in v1.
- Verify the cellular board's peak-current demand and power-supply transient response before connecting it to the project battery.
Exit gate: documented compatibility evidence for the exact SKU and operator; a successful test request and explicit backend acknowledgement on bench; no claims about production reliability.

### Phase 3 — electrical design
Activities:
- Produce pin map from exact board documentation.
- Check GPIO conflicts with camera, flash/SD, UART, modem control and boot-strapping functions.
- Calculate output driver and regulator requirements from datasheets and measured loads.
- Draw wiring diagram and power tree; independently review voltage/current compatibility.
Exit gate: no unresolved voltage-level or GPIO conflict; power budget documented.

### Phase 4 — minimal firmware, one function at a time
Order:
1. Build/flash “heartbeat” firmware on each board and record board/toolchain versions.
2. Read sensor only and log raw samples; no alert claims yet.
3. Add validation/filtering/hysteresis and deterministic state machine.
4. Add one output at a time using verified drivers.
5. Add fault handling and serial diagnostics.
6. Add inter-controller protocol and its host/unit tests.
7. Add camera capture baseline.
8. Test modem AT commands, SIM/network attach, GNSS fix and TLS request as a separate isolated spike.
9. Add SOS event serialization, authenticated upload, server acknowledgement, bounded retry and user-visible failure.
10. Only then test simple on-device vision; keep it advisory.

Exit gate: reproducible build, recorded test output, no unresolved critical faults.

### Phase 5 — bench validation
Run the test matrix in INTER_CONTROLLER_PROTOCOL_AND_BENCH_TEST_PLAN.md. Add:
- Calibrated target distance and angle tests.
- Multiple target materials/sizes and sunlight tests.
- Latency measurement from target change to tactile alert.
- False-positive and false-negative logs.
- Power/brownout, battery runtime and thermal checks under modem transmit bursts, camera activity and alerts.
- Disconnect, corrupted message, stale message, reboot and low-power tests.
- SIM absent, incorrect APN, no coverage, weak signal, data exhausted, GNSS unavailable, stale location, backend timeout, TLS/auth failure, duplicate SOS, retry exhaustion and server acknowledgement tests.
- Verify the app shows event time, source and age of location, battery if reported, and delivery status.
Exit gate: measured results and known failure modes documented. Passing a bench test does not prove real-world mobility safety.

### Phase 6 — usability and controlled evaluation
- Review alert patterns with potential users and orientation/mobility specialists where available.
- Use a sighted supervisor and controlled, stationary hazards; never make the prototype the sole safety aid.
- Obtain appropriate consent and follow relevant ethics/privacy practices before collecting user data.
- Test the SOS interaction with deliberate user input, accessible controls and accidental-trigger prevention.
Exit gate: feedback and limitations recorded; do not claim broad safety without appropriate evidence.

### Phase 7 — release candidate
- Freeze hardware revision and firmware version.
- Provide BOM, pin map, wiring, build instructions, tests/results, limitations, troubleshooting and recovery instructions.
- Review privacy/security if camera images, location, LTE or remote services are enabled.
- Tag a release only after agreed gates pass; merge/deploy only with explicit user approval.

## 6. Verification strategy

Every test record should include:
- Test ID, date, board revision, firmware commit, sensor/module revision.
- Setup, target/material/distance/angle and environment.
- Expected behavior and actual behavior.
- Latency and measurement values where applicable.
- Pass/fail/inconclusive, failure notes and reproducibility steps.

Never report “passed” based solely on source-code review or a successful file write. Distinguish static review, compilation, unit tests, hardware bench tests and supervised user evaluation.

## 7. Candidate research sources

- Espressif ESP32 camera driver and supported sensors: https://github.com/espressif/esp32-camera
- Espressif official ESP-IDF framework: https://www.espressif.com/products/sdks/esp-idf
- Espressif UART documentation: https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/peripherals/uart.html
- ST VL53L1X product page: https://www.st.com/en/imaging-and-photonics-solutions/vl53l1x.html
- ST VL53L1X datasheet: https://www.st.com/resource/en/datasheet/vl53l1x.pdf
- SIMCom SIM7600 family overview and regional variant tables: https://www.simcom.com/product/SIM7600.html
- SIMCom SIM7600-H family overview: https://www.simcom.com/product/SIM7600X-H.html
- ARPCE mobile 4G information: https://www.arpce.dz/ar/service/4g
- ARPCE frequency-management report (use to begin verification, not as a substitute for confirming the exact current operator/band and location): https://www.arpce.dz/fr/file/w5i7z4
- ARPCE equipment approval information: https://www.arpce.dz/fr/service/agr
- WHO assistive technology overview: https://www.who.int/news-room/fact-sheets/detail/assistive-technology
- UNHCR 2024 guidance on assistive technology and white canes: https://www.unhcr.org/sites/default/files/2024-08/facilitating-access-to-assistive-technology-and-rehabilitation-2024.pdf
- Related research and reviews are catalogued in RESEARCH_AND_ARCHITECTURE.md; reported results must not be treated as expected performance for this prototype.

## 8. Decisions to make next

1. Identify both controller boards from clear photos/markings.
2. Identify camera and its sensor.
3. Identify vibration, buzzer and audio modules and any driver boards.
4. Identify battery and regulator labels.
5. Compare distance sensor options and local DZD prices before purchase.
6. Decide whether to test a cellular board candidate with B3 (1800 MHz) support and GNSS after checking the current ARPCE/operator requirements, intended area, exact SKU and carrier-board power design.
7. Decide the app/backend approach only after the data flow, security and recurring-cost review.
8. Choose the build environment after board identification: Arduino IDE/PlatformIO may be simpler for a small prototype; ESP-IDF offers official lower-level tooling. Keep the choice consistent and record versions.

## 9. Change control

- Work on research/smart-blind-stick-architecture.
- Keep changes small and reviewable.
- Do not merge, publish a release or claim successful hardware tests without evidence and explicit user approval.
