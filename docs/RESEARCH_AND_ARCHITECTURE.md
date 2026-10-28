# SmartBlindStick: Research and Architecture Plan

Date: 2026-10-28
Status: research baseline; hardware inventory and on-device measurements still required.

## 1. Project intent

Build an affordable ESP32-CAM-based assistive prototype that supplements (never replaces) a traditional white cane. The system should provide local obstacle alerts, optional camera-based object context, and tactile/audio feedback. The core proximity warning must remain operational without Wi-Fi, cloud services, or camera inference.

## 2. Architecture recommendation

- Main ESP32 controller: time-critical sensor sampling, filtering, alert state machine, watchdog/timeouts, battery monitoring where supported.
- ESP32-CAM AI Thinker: camera capture and best-effort visual classification. Treat vision output as advisory until measured on the exact board and firmware.
- Distance sensing: begin with the existing ultrasonic sensor if already owned; evaluate a ToF sensor such as VL53L1X only after comparing actual cost, field of view, range, ambient-light behavior, and mounting angle.
- Feedback: distinct vibration patterns for near/urgent hazards; short audio prompts for optional object labels. Do not use audio that masks traffic or other environmental sounds.
- Inter-controller link: choose UART or a simple framed protocol only after the board inventory is confirmed. Include message type, sequence number, checksum or validation, and freshness timeout. Stale camera messages must be ignored.
- Power: separate/regulated rails as needed; confirm the camera board's peak current and regulator limits before finalizing battery and wiring.

## 3. Safety constraints

1. The electronic prototype supplements rather than replaces a conventional white cane, mobility training, or a human guide.
2. Sensor and vision failures must not silently suppress the basic warning path.
3. Never infer that an area is safe because no object was detected.
4. Explicitly test overhead obstacles, narrow/small objects, glass/reflective surfaces, low obstacles, stairs and drop-offs, wet surfaces, sunlight, rain, and crowded/noisy environments.
5. Keep high-priority alerts concise and non-blocking; vibration should remain available when audio is noisy or inappropriate.
6. Do not claim medical-device status, certification, or real-world safety until independent validation supports those claims.

## 4. Research findings

### On-device vision feasibility
Published work describes a dual-microcontroller smart cane using ESP32 and ESP32-CAM with on-device TinyML, and a separate 2025 TinyML study reports a 78.1% F1 score with 1.368 s inference on its tested hardware. These are feasibility signals, not performance guarantees for our board or dataset. We must benchmark image capture, inference, end-to-end alert latency, memory, heat, and battery runtime on the exact ESP32-CAM module.

Sources:
- https://doi.org/10.1109/siot68426.2025.11368785
- https://doi.org/10.1109/icvee66651.2025.11281396
- https://doi.org/10.1109/icac54203.2021.9671191

### Sensor fusion and user feedback
Reviews of technology-assisted white canes report that sensors have different strengths and blind spots; ultrasonic sensing is common but has limited angular resolution, while infrared/laser performance can degrade in bright sunlight. Vibration can be less affected by ambient noise and may improve privacy; audio should not block environmental hearing. Multiple sensing modes can help but do not automatically make a system safe.

Sources:
- https://pmc.ncbi.nlm.nih.gov/articles/PMC6292384/
- https://pmc.ncbi.nlm.nih.gov/articles/PMC6453076/
- https://pmc.ncbi.nlm.nih.gov/articles/PMC10007677/

### ToF option
ST's VL53L1X product page lists ranging up to 4 m and up to 50 Hz under specified conditions, with a typical 27-degree full field of view. Actual range varies with target reflectance, ambient light, mode, and setup; it must not be treated as a guaranteed 4 m detection envelope.

Source:
- https://www.st.com/en/imaging-and-photonics-solutions/vl53l1x.html
- https://www.st.com/resource/en/datasheet/vl53l1x.pdf

### NVIDIA skills fit
NVIDIA's skills catalog includes Physical AI, robotics, computer vision and Jetson workflows. These may help future simulation, dataset, or Jetson-based inference work, but Jetson-specific skills do not directly accelerate firmware running on an ESP32-CAM. Do not add NVIDIA hardware or paid/cloud dependencies unless a measured requirement justifies them.

Sources:
- https://github.com/NVIDIA/skills
- https://docs.nvidia.com/skills

## 5. Implementation phases

### Phase A — inventory and baseline
- Record exact board model, camera sensor, PSRAM presence, sensor part numbers, vibration driver, audio module, battery and regulator.
- Capture current firmware, pin map, wiring photo, and known working test.
- Create a bill of materials with locally available parts and prices in DZD.

### Phase B — deterministic obstacle alert
- Build and test sensor reading, filtering, hysteresis, and near/urgent states.
- Test each sensor independently and measure false-positive/false-negative cases.
- Verify boot, sensor disconnect, stuck readings, low battery, and reset behavior.

### Phase C — camera and inter-controller protocol
- Verify camera capture and JPEG frame rate on the exact ESP32-CAM.
- Define compact, versioned messages with a freshness timeout.
- Measure camera-to-alert latency and memory/PSRAM use; if inference is too slow, use the camera as an optional context channel rather than the safety-critical path.

### Phase D — human interface and power
- Define vibration patterns with user feedback; keep audio optional and brief.
- Measure battery runtime under worst-case camera, Wi-Fi and vibration load.
- Make charging, enclosure, strain relief and water protection explicit design items.

### Phase E — validation and release gate
- Build a repeatable test matrix for object size, material, distance, angle, lighting, motion, stairs/drop-offs and environmental noise.
- Test with non-hazardous stationary obstacles first; do not test by exposing a blind person to uncontrolled hazards.
- Document limitations and known failure modes. Only claim verified results from logged measurements.

## 6. Exit criteria before any real-world trial

- No unhandled crash or lock-up in an extended bench test.
- Basic alert continues when the camera, Wi-Fi or secondary controller is unavailable.
- Measured alert latency and battery runtime are documented.
- Sensor failure, stale messages, and low power produce a defined warning state.
- Known blind spots are disclosed; controlled user evaluation is supervised and follows appropriate ethics and accessibility practices.

## 7. Open questions that block a final wiring diagram

1. Is the camera board the AI Thinker ESP32-CAM with OV2640 and PSRAM?
2. Which exact distance sensor(s) are already available?
3. Is there a second ESP32 controller, or should we keep the first prototype on one board?
4. Which outputs are available: vibration motor, buzzer, speaker/audio module, or headphones?
5. What battery and regulator are available?

Do not assign final GPIO pins, battery sizing, or voltage wiring until these are answered.
