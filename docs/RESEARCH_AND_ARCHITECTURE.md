# Smart Blind Stick: Research and Architecture Plan

Date: 2026-10-28
Status: research baseline; hardware inventory and on-device measurements still required.

## 1. Project intent

Build an affordable ESP32-based assistive prototype that supplements (never replaces) a traditional white cane. The system should provide local obstacle alerts, optional camera-based object context, and tactile/audio feedback. The core proximity warning must remain operational without Wi-Fi, cloud services, camera inference, or the second controller.

## 2. User-confirmed inventory

- Two controller boards are available, but exact models are unknown.
- Vibration, buzzer and audio feedback are reported available; module types and driver circuitry are unknown.
- No distance sensor is currently confirmed available.
- Camera board/model is unknown.
- Battery and regulator are unknown.

The project cannot yet produce a final GPIO map, wiring diagram, battery sizing, or end-to-end obstacle detection firmware. First collect board/module markings and identify the power source.

## 3. Architecture recommendation

- Controller A: intended proximity/alert controller. Sample the eventual distance sensor, reject implausible/stale measurements, apply filtering and hysteresis, and control vibration/buzzer through verified driver circuits.
- Controller B: optional camera/context controller after its exact board and camera are identified. Vision output is advisory and must never clear, delay or downgrade a local proximity alert.
- Distance sensing: compare locally available modules before purchase. The ST VL53L1X is one ToF candidate; its published maximum range and rate are conditional, not guaranteed cane performance. An ultrasonic option may be cheaper but its exact model and output voltage must be checked.
- Inter-controller link: consider UART only after confirming both board models, exposed pins and logic levels. Use a versioned, bounded frame with message type, sequence number, integrity check and freshness handling. Ignore stale/invalid camera messages.
- Power: verify each board's supply requirements and peak current before finalizing battery/regulator. Do not drive a vibration motor or speaker directly from a GPIO without verifying the load and suitable driver.

See [INTER_CONTROLLER_PROTOCOL_AND_BENCH_TEST_PLAN.md](INTER_CONTROLLER_PROTOCOL_AND_BENCH_TEST_PLAN.md) for the protocol proposal and bench-test matrix.

## 4. Safety constraints

1. The electronic prototype supplements rather than replaces a conventional white cane, mobility training, or a human guide.
2. Sensor, camera or link failures must not silently suppress the basic warning path.
3. Never infer that an area is safe because no object was detected.
4. Explicitly test overhead obstacles, narrow/small objects, glass/reflective surfaces, low obstacles, stairs and drop-offs, wet surfaces, sunlight, rain, and crowded/noisy environments.
5. Keep high-priority alerts concise and non-blocking; vibration should remain available when audio is noisy or inappropriate.
6. Do not claim medical-device status, certification, or real-world safety until independent validation supports those claims.

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
- Build a bill of materials with local prices in DZD.

### Phase B — deterministic obstacle alert
- Implement and test sensor reading, plausibility checks, filtering, hysteresis and distinct near/urgent states.
- Test each sensor independently and measure false-positive/false-negative cases.
- Verify boot, sensor disconnect, stuck readings, low battery, and reset behavior.
- Keep “sensor fault/unknown” distinct from “no obstacle detected”.

### Phase C — camera and inter-controller protocol
- Verify camera capture and JPEG frame rate on the exact camera board.
- Implement the versioned, bounded protocol in the companion test-plan document only after confirming compatible UART pins and logic levels.
- Measure message loss, stale-message rejection, camera-to-hint latency and memory/PSRAM use. If inference is too slow, retain camera only as optional context.

### Phase D — human interface and power
- Define vibration patterns with user feedback; keep audio optional and brief.
- Measure battery runtime under worst-case camera, Wi-Fi and vibration load.
- Make charging, enclosure, strain relief and water protection explicit design items.

### Phase E — validation and release gate
- Build a repeatable test matrix for object size, material, distance, angle, lighting, motion, stairs/drop-offs and environmental noise.
- Test with non-hazardous stationary obstacles first; do not test by exposing a blind person to uncontrolled hazards.
- Document limitations and known failure modes. Only claim verified results from logged measurements.

## 7. Exit criteria before any real-world trial

- No unhandled crash or lock-up in an extended bench test.
- Basic alert continues when the camera, Wi-Fi or secondary controller is unavailable.
- Measured alert latency and battery runtime are documented.
- Sensor failure, stale messages, and low power produce a defined warning state.
- Known blind spots are disclosed; controlled user evaluation is supervised and follows appropriate ethics and accessibility practices.

## 8. Open questions blocking final wiring

1. What are the exact models/revisions of both controller boards?
2. What is the exact camera board and camera sensor?
3. Which output modules are available, and does each have a driver/amplifier?
4. What battery and regulator are available?
5. Which distance sensor should be acquired after comparing local availability and price?

Do not assign final GPIO pins, battery sizing, or voltage wiring until these are answered.
