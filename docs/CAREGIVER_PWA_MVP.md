# SmartBlindStick Caregiver PWA — MVP scope

## Product decision
- Audience: caregiver only.
- Languages: Arabic (default, RTL), French, English.
- Visual direction: modern safety/map-inspired UI with clear status, large controls, high contrast, and mobile-first layout.
- Delivery approach: static PWA-style prototype first; no backend, device integration, map provider, or push notification service is connected yet.
- Working branch: `feature/caregiver-pwa-mvp`.
- No merge or deployment is part of this work.

## Prototype
The first screen is in `app/index.html`. It demonstrates:
- Device connection state, last confirmed contact, and battery reporting status.
- SOS state and a local-only demo event preview.
- Last known location area with source/age/accuracy fields reserved for real data.
- Recent event history placeholder.
- Arabic, French, and English language switching.
- Responsive layout and keyboard focus styles.

All values are explicitly demo/unknown. The preview button only adds a local UI event. It does not send an SOS, contact a device, request phone location, or notify a caregiver.

## Proposed MVP data contract
An SOS event should include:
- `event_id`: unique identifier for deduplication.
- `device_id`: authenticated device identity (not a secret embedded in a public page).
- `event_type`: manual SOS.
- `occurred_at`: device event time, if trustworthy.
- `received_at`: server receipt time.
- `delivery_state`: pending / server_received / app_notified / failed, only as evidenced.
- `location`: nullable object with latitude, longitude, source (stick_gnss or phone), measured_at, accuracy_m, and freshness.
- Explicit `location_unavailable` state if no valid fix exists.

A server acknowledgement means the server received the event; it does not prove that the caregiver's phone displayed a notification. The UI must not show "sent/delivered" without evidence from the relevant layer.

## Safety, privacy, and reliability requirements
1. Local obstacle alerts on the stick must not depend on LTE, GNSS, the caregiver app, or the backend.
2. SOS is manual-only in v1. No continuous tracking or periodic check-ins.
3. Never label an old coordinate as live; show source and age, plus accuracy where available.
4. Use authenticated device-to-server requests, TLS certificate validation, rate limiting, event-ID deduplication, and server-side secrets.
5. Minimize retained location data; define caregiver authorization, revocation, retention, and deletion before real-user trials.
6. If network delivery cannot be confirmed, show pending/not confirmed and a retry state rather than success.
7. The stick complements a white cane and must not be represented as a replacement or a guaranteed obstacle detector.

## Next implementation stages
1. Review the static prototype on mobile and correct layout/accessibility issues.
2. Add automated checks for language switching, RTL/LTR, keyboard focus, and demo-only actions.
3. Define and implement a small authenticated backend API and health endpoint.
4. Add persistent event history and acknowledgement states against a test service.
5. Implement push notifications only after choosing and testing a supported delivery channel on Android.
6. Integrate actual device telemetry after controller/module models and message protocol are confirmed.
7. Run failure-mode tests: no GNSS fix, stale location, offline device, duplicate SOS, server timeout, app notification denied, and invalid credentials.

## Added in the current branch update — camera, voice, and health-status UI

The caregiver page now includes:
- A phone-camera panel using the browser `getUserMedia` API, with explicit open/close controls and a local snapshot preview. The page does not upload images. Camera access requires browser permission and a secure context (normally HTTPS); browser/device support varies.
- Voice readout using the browser speech-synthesis API and optional speech recognition when supported. Voice commands are best-effort and have button alternatives.
- A health-status panel for battery/voltage, sensors/controller, and connectivity. These values deliberately remain unknown because no real device telemetry protocol or hardware has been connected. The read-aloud status also states this limitation.
- Arabic, French, and English labels for the new UI.

## Important limits and next steps
- The phone-camera control opens the phone's camera, not a camera module mounted on the stick. Stick-camera streaming requires confirming the exact camera/controller models and implementing a separate authenticated transport.
- The new health panel is a UI placeholder, not real hardware monitoring. Next, confirm both controller models, camera module, sensor inventory, battery measurement circuit, and communication protocol before defining telemetry fields or wiring.
- Speech recognition is not consistently available across mobile browsers and may depend on browser-managed services/network access. Do not treat it as an emergency channel; SOS must remain a deliberate manual action with explicit delivery acknowledgements.
- Static hosting (for example, GitHub Pages) can serve the prototype at no hosting cost, but it cannot by itself provide authenticated device ingestion, reliable background push, or secure server-side secrets. Keep the camera local and do not add image uploads by default.
- Camera, microphone, language, accessibility, and real-device integration still require browser and Android-device testing. No such runtime or hardware tests are claimed by this change.

## Acceptance gates
- Demo actions never claim a real SOS was sent.
- Unknown device and location values remain visibly unknown.
- Language switch updates the complete visible interface and direction.
- No secrets or production endpoints are hard-coded in the static page.
- No merge/deploy until checks and integration evidence are reviewed separately.
