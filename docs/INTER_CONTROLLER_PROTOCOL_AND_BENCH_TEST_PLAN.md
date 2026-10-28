# Smart Blind Stick — Inter-Controller Protocol and Bench Test Plan

Date: 2026-10-28
Status: design proposal; not hardware-validated. Do not treat this document as a wiring guide or safety certification.

## 1. Confirmed inventory vs unknowns

User-reported:
- Two controllers are available.
- Feedback options available: vibration, buzzer, audio.
- Distance sensor: none confirmed.
- Camera board/model: unknown.
- Battery and regulator: unknown.

Do not infer that both controllers are ESP32 variants, that the camera is AI-Thinker/OV2640, or that any feedback module can be driven directly from a GPIO. Record board silkscreen/model, camera module, sensor part number, power source and driver circuitry before pin assignment.

## 2. Functional separation

**Controller A — proximity and alert controller (intended safety path)**
- Reads the eventual distance sensor.
- Applies plausibility checks, filtering and hysteresis.
- Drives vibration and buzzer through verified driver circuits.
- Keeps basic alerts independent of camera inference, Wi-Fi and Controller B.
- Reports sensor fault/stale data distinctly from “no obstacle detected”.

**Controller B — camera/context controller (optional advisory path)**
- Captures images only after its exact board/camera is identified.
- May send a bounded object/context label to Controller A.
- Must never clear, delay, or downgrade a proximity alert.
- If it reboots, disconnects or sends stale/invalid data, Controller A continues its own local alert logic.

If the physical boards do not match this role split, revise it before implementation.

## 3. Link protocol proposal (not yet implemented)

Start with UART only if both boards expose compatible UART pins and voltage levels. Use 3.3 V logic only after verifying the exact boards; connect grounds together and avoid connecting two outputs together. UART pins must be chosen after board/pinout inspection, not from a generic ESP32 pin list.

Suggested frame:
- Start marker: 2 bytes A5 5A
- Protocol version: 1 byte
- Message type: 1 byte
- Sequence number: 1 byte
- Payload length: 1 byte (bounded; initial maximum 32 bytes)
- Payload: 0–32 bytes
- CRC-16/CCITT-FALSE: 2 bytes over version through payload

Suggested message types:
- 0x01 STATUS: protocol version, uptime/health flags.
- 0x10 VISION_HINT: class ID, confidence quantized 0–100, capture age; advisory only.
- 0x11 VISION_UNAVAILABLE: camera/inference unavailable.
- 0x20 DIAGNOSTIC: bounded error code for bench logs.

Parsing requirements:
- Never trust payload length until it passes the maximum bound.
- Resynchronize on the start marker after malformed frames.
- Reject unsupported versions, invalid message types, bad CRC and stale sequence/freshness.
- Never accept a vision hint as evidence that the path is clear.
- Log malformed frames and timeouts during bench tests; rate-limit logs in normal operation.

Freshness timeout and UART baud rate are deliberately not fixed here: choose them after measuring the actual message rate, scheduling and latency on both boards. Record the selected values and measured worst-case latency before relying on them.

## 4. Distance sensor selection gate

No distance sensor is currently confirmed, so obstacle-alert firmware cannot yet be validated end to end.

Candidate to evaluate:
- **VL53L1X ToF module**: ST specifies up to 4 m and up to 50 Hz under stated conditions, with a typical 27° field of view. These are not guaranteed real-world cane detection values. Validate outdoor sunlight, target materials, angle, rain/dirt and mounting geometry. Primary source: https://www.st.com/en/imaging-and-photonics-solutions/vl53l1x.html and https://www.st.com/resource/en/datasheet/vl53l1x.pdf
- **Ultrasonic module**: may be lower cost and easier to source, but the exact module must be identified. Some common modules use a 5 V ECHO output; do not connect an unverified 5 V signal directly to an ESP32 GPIO. Check its datasheet and use a suitable level shifter/divider if required.

Select only after checking local availability and price, actual module voltage/interface, outdoor performance, mounting direction, and testability. One forward-facing sensor cannot guarantee detection of overhead hazards, drop-offs, stairs, narrow objects or every surface.

## 5. Power and output safety

- Do not finalize battery capacity, regulator or wiring until exact boards and modules are identified.
- The ESP32-CAM and camera/Wi-Fi activity can create current peaks; verify the board vendor's requirements and measure brownouts under load.
- Do not drive a vibration motor directly from a GPIO. Select a transistor/MOSFET driver based on motor current; add flyback protection when required by the actual load topology.
- Identify whether “audio” means a buzzer, amplified speaker, I2S module or headphones. Do not assume a speaker can be driven directly from a GPIO.
- Keep high-priority vibration distinct and brief; audio must not mask traffic or other environmental sounds.
- Disconnect power before rewiring. Never test the prototype as the sole mobility aid.

## 6. Bench test matrix

Run and record each test on the actual assembled hardware; no results are claimed by this plan.

| Test | Setup | Pass condition |
|---|---|---|
| Boot baseline | Power-cycle each board repeatedly | Predictable startup; no unintended motor activation |
| Sensor absent | Start Controller A without a sensor attached | Explicit sensor-fault state; never report “clear” |
| Sensor unplugged | Disconnect sensor during a bench run | Fault detected within a documented timeout; fault is distinguishable from valid range |
| Stuck reading | Simulate repeated identical/out-of-range samples | Implausible/stale data rejected and diagnostic recorded |
| Near/urgent transition | Move a safe, stationary target through calibrated distances | Alerts transition at documented thresholds without excessive chatter |
| Camera offline | Power off or reset Controller B | Controller A's local alert path remains functional |
| Bad UART frame | Inject truncated frame, bad CRC, invalid length and unknown version | Invalid frames ignored; parser recovers without lockup |
| Stale vision hint | Deliver an old or delayed vision message | Hint ignored; no safety alert suppressed |
| Output driver | Test vibration/buzzer separately under measured supply | No GPIO overload, brownout or unintended reboot |
| Low battery | Test only with a safe, monitored supply/battery setup | Defined low-power behavior; no misleading “all clear” indication |
| Extended run | Repeat sensor and communications tests for a documented duration | No lockups; reset count, latency and failures logged |

Initial bench distances and timing thresholds must be calibrated from the selected sensor datasheet and measured behavior. Do not use an arbitrary timeout as a safety guarantee.

## 7. Release gate

Before any supervised user evaluation:
- Exact hardware inventory and wiring are documented.
- Sensor calibration, false positives/negatives and end-to-end alert latency are measured.
- Controller B failure cannot suppress Controller A's local warning path.
- Battery runtime and brownout behavior are measured under worst-case load.
- Known blind spots and limitations are written down.
- Controlled testing starts with stationary obstacles and a sighted supervisor; the prototype supplements, never replaces, a conventional white cane and mobility training.

## 8. Primary references

- Espressif camera driver and board pin maps: https://github.com/espressif/esp32-camera
- ESP-IDF UART driver documentation: https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-reference/peripherals/uart.html
- ST VL53L1X product page: https://www.st.com/en/imaging-and-photonics-solutions/vl53l1x.html
- ST VL53L1X datasheet: https://www.st.com/resource/en/datasheet/vl53l1x.pdf
- ESP32-CAM pin-use notes (secondary hardware reference; verify against the exact board): https://github.com/raphaelbs/esp32-cam-ai-thinker/blob/master/docs/esp32cam-pin-notes.md

## 9. Next required user-side evidence

When convenient, provide photos or exact text from:
1. Both controller boards (front/back and readable model markings).
2. Camera module and ribbon/board markings.
3. Vibration, buzzer and audio modules, including any driver board.
4. Battery and regulator labels.
5. Any sensor acquired later.

Until these are known, no final GPIO map, pin-level wiring, or firmware claims are authorized.
