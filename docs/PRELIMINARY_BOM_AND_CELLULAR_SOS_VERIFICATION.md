# Smart Blind Stick — Preliminary BOM and Compatibility Verification

Date: 2026-10-28
Branch: research/smart-blind-stick-architecture
Status: planning only. No parts purchased. No hardware tests performed. User budget ceiling: 10,000 DZD for parts not already owned. Prices below are public listing snapshots found on 2026-10-28 and must be reconfirmed with the seller before purchase.

## 1. Design decisions currently in scope

- Cellular data plus an Android companion app.
- GNSS/GPS on the stick, with phone-location fallback only when the app has permission and a fresh location.
- User-triggered SOS only; no periodic check-ins or automatic continuous tracking in v1.
- SOS event should carry a unique event ID, event time, location if available plus source/age, and battery/connectivity state where available.
- Device must show pending/failed if no backend acknowledgement is received.
- Local obstacle sensing and vibration warning must keep working without cellular coverage, app, backend, GNSS or camera.

## 2. Preliminary BOM

| Item | Qty. | Status | Minimum requirement / selection notes | Cost |
|---|---:|---|---|---|
| Controller board A | 1 | Already reported available; exact model unknown | Local obstacle sensor + tactile warning; exact board/revision and voltage needed before pin map | Already owned / TBD |
| Controller board B / camera board | 1 | Already reported two controllers; exact models unknown | Confirm whether either is ESP32-CAM; identify camera, PSRAM and exposed UART/pins | Already owned / TBD |
| Camera module | 1 if separate | Unknown | Exact sensor/module and connector; optional to SOS path | TBD |
| Distance sensor | 1 | Must select | Compare at least two options for outdoor behavior, field of view, range, logic voltage, interface and local price. No sensor is confirmed available | TBD |
| Cellular modem development board | 1 | Candidate; not selected | LTE data, GNSS or documented GNSS option, UART/USB, HTTPS/TLS capability, accessible documentation and a specific regional SKU with confirmed operator band compatibility | TBD |
| Cellular main antenna | 1 | Required with modem | Must match exact modem board connector and supported bands; avoid generic antenna assumptions | TBD |
| GNSS antenna | 1 | Required if modem GNSS used | Correct connector/type and mounting with sky visibility; verify active/passive antenna power support on carrier board | TBD |
| SIM card + data plan | 1 | Required | SIM registration/format, APN, LTE data access and coverage verified with the intended operator | SIM cost + monthly data TBD |
| Battery pack | 1 | Unknown / must size | Chemistry, cell count, capacity, discharge rating and protection chosen from measured worst-case load; do not select by mAh alone | TBD |
| Charger and battery protection | 1 set | Must verify | Compatible with chosen chemistry/cell count; protection against overcharge, overdischarge and short circuit | TBD |
| Regulator / power stage for modem | 1 | Likely required unless carrier board provides suitable supply | Follow exact carrier-board voltage range and transmit-burst current requirement; verify transient response and thermal margin | TBD |
| Regulator for controllers/sensor | As required | Must verify | Stable rails for each board; no assumed 3.3 V/5 V compatibility without datasheets | TBD |
| Decoupling capacitors / power wiring | As calculated | Required | Based on carrier-board design guide and measured brownout/transient tests | TBD |
| SOS push button | 1 | Required | Accessible, tactile, protected against accidental activation; define press/hold behavior and feedback | TBD |
| Vibration motor/actuator | 1 | Reported available; exact model unknown | Verify voltage/current; use suitable transistor/MOSFET/driver and protection as required | Existing / TBD |
| Buzzer/audio unit and driver | As needed | Reported available; exact models unknown | Identify whether passive/active buzzer, speaker/amplifier or other; never assume direct GPIO drive is safe | Existing / TBD |
| Driver components | As needed | Must verify | Sized from actual load current/voltage; include protection where inductive loads require it | TBD |
| Level shifter / interface components | As needed | Conditional | Only if confirmed UART/GPIO voltage levels require it; 5 V output must not be connected directly to a 3.3 V-only input | TBD |
| Enclosure, strain relief, mounts | 1 set | Must design | Secure mounting, accessible button, antenna placement, no sharp edges; evaluate splash protection and serviceability | TBD |
| Connectors, perfboard/PCB, wire | 1 set | Required | Strain relief and separation of RF/power/noise-sensitive sensor wiring; final quantities after design | TBD |
| Android companion app | 1 | Software work | Manual SOS, consent/contact setup, phone-location permission, event history/status and map link; no background tracking in v1 | Free if self-built; hosting TBD |
| Secure backend/API | 1 | Architecture choice pending | HTTPS/TLS, device authentication, event IDs/deduplication, explicit acknowledgement, access control and retention policy; disclose recurring cost | TBD |
| Optional microSD/storage | 0–1 | Optional | Only if local event buffering is required and the selected board supports it; do not add until needed | TBD |

## 3. Budget reality check (10,000 DZD ceiling)

Public listing snapshots checked on 2026-10-28:
- SIM7600-based TTGO ESP32 LTE + GNSS board listed at 14,500 DZD by Power Lab: https://powerlab.dz/product/ttgo-esp32-lte-gps-sim7600/
- SIM7600G-H PCIE listing at 14,048 DZD on Ubuy Algeria, excluding shipping and customs: https://www.ubuy.dz/ar/product/J99L9RFMS-original-sim7600g-h-pcie-simcom-4g-lte-cat-4-module-global-coverage-with-gnss-support-mini-pcie-connector-lte-tdd-lte-fdd-hspa-gsm-gprs-edge
- A SIM800L module is listed around 1,600 DZD by Digibox DZ: https://digibox-dz.com/en/products/DG0032/
- A separate NEO-6M GPS module is listed around 1,800 DZD by DZ Automation: https://dzautomation.net/categorie-produit/reseau/module-gsm-gprs-gps/
- A VL53L0X module is listed at 850 DZD and an HC-SR04 at 650 DZD by BinaryTech DZ: https://binarytech-dz.com/category/capteurs-ultrason-mouvements-et-gestes
- A VL53L1X ToF module is listed at 2,400 DZD by Power Lab: https://powerlab.dz/product/tof400c-vl53l1x-4m-precision-distance-measurements/

These are seller listings, not quotes or proof of stock, compatibility, quality or delivery cost. Prices may change. The LTE + GNSS candidate listings alone exceed the 10,000 DZD ceiling before adding a distance sensor, power hardware, antenna, wiring or enclosure. Therefore do not purchase the LTE candidate under the current budget without finding a verified lower-cost compatible option or increasing the budget.

SIM800L is a lower-cost 2G GSM/GPRS option, not LTE. It does not include GNSS, so pairing it with a separate GPS module increases parts and power complexity; more importantly, it only meets the chosen cellular-data requirement if the intended operator still provides usable 2G/GPRS service in the target areas and the data link can reach the secure service. Verify this first. Do not quietly switch the product requirement from cellular data to SMS-only. If no verified cellular-data solution fits the budget, defer cellular hardware and prototype local obstacle alerts plus GNSS separately, while keeping SOS/network integration modular.

### Budget rule

- Hard cap: 10,000 DZD for parts not already owned.
- No purchases are authorized by this document.
- Prioritize parts required for safe local obstacle alerts and reliable power.
- Treat SIM/data plan, backend hosting, delivery and import fees as separate costs and disclose them before commitment.
- Record supplier, date checked, exact SKU, stock status, delivery cost and return policy before choosing.
- Do not count a phone app as a substitute for an onboard cellular modem unless the intended SOS workflow explicitly relies on the phone being nearby and online.

## 4. Shortlist approach for cellular/GNSS

### Candidate family: SIMCom SIM7600

Official family pages:
- https://www.simcom.com/product/SIM7600.html
- https://www.simcom.com/product/SIM7600X-H.html

Why it is being researched:
- The family has LTE data and GNSS variants.
- Some variants provide multiple network protocols including TLS-related functions, and interfaces such as UART/USB.
- Exact band support differs by model/suffix; the family name alone is not a sufficient purchase specification.

Purchase gate:
1. Get a photo/link of the exact development board and its modem marking.
2. Download the matching hardware design/datasheet.
3. Confirm GNSS is present and supported on that exact SKU.
4. Compare its LTE band list with the current operator's assigned spectrum and coverage in the intended area. ARPCE's frequency report is a starting reference, not a guarantee of current coverage: https://www.arpce.dz/fr/file/w5i7z4
5. Confirm the carrier board exposes usable UART/USB, antenna connectors and documented power input.
6. Verify whether local equipment approval requirements apply to the actual product/design. ARPCE reference: https://www.arpce.dz/fr/service/agr
7. Test with the real SIM before accepting compatibility.

Do not buy a module based only on an online listing saying “4G global”. Do not assume SIM7600G-H, SIM7600E-H or any other suffix has identical bands or GNSS features.

## 5. Algeria network verification checklist

For each candidate operator (Mobilis, Djezzy, Ooredoo):
- [ ] Identify intended operator and exact plan.
- [ ] Ask the operator/support for current LTE bands available in the target city/areas and whether the SIM permits device data use.
- [ ] Compare those bands with the exact modem SKU datasheet.
- [ ] Confirm APN and data-plan conditions.
- [ ] Test SIM registration and LTE attach in the intended area, indoors and outdoors.
- [ ] Send a small HTTPS request to the test endpoint and verify server acknowledgement.
- [ ] Test weak/no coverage, reconnect, data exhaustion and invalid APN.
- [ ] Estimate data usage from measured SOS payload size and realistic number of events; avoid periodic upload in v1.
- [ ] Confirm the app/backend works over the operator's network without relying on inbound connections to the SIM. Prefer device-initiated outbound HTTPS.
- [ ] Check applicable ARPCE equipment approval before any wider commercialization.

Initial frequency reference:
- ARPCE publishes mobile 4G and frequency-management information: https://www.arpce.dz/ar/service/4g and https://www.arpce.dz/fr/file/w5i7z4
- Band 3 (1800 MHz) is a useful band to investigate first, but do not treat one band as proof of compatibility; verify exact current assignments, operator, geography and exact modem variant.

## 6. Power budget and safety verification

Do not finalize battery or regulator from the modem's nominal/idle current alone.

Record for every module:
- Datasheet operating voltage range.
- Typical and peak/transient current.
- Startup and radio-transmit current profile.
- Logic input/output voltage levels.
- Permitted temperature range.
- Connector and antenna requirements.

Bench plan:
1. Test the modem carrier board with its documented supply requirements on a separate suitable bench supply.
2. Observe voltage droop/current peaks during network registration and data transmission.
3. Repeat with GNSS active, weak signal, camera active, vibration motor and buzzer.
4. Verify no ESP32 resets, brownouts, overheating or unstable sensor readings.
5. Calculate a conservative power budget with measured duty cycles and margin.
6. Estimate battery runtime using measured average load and usable battery energy, not label capacity alone.
7. Verify charger/protection compatibility with the exact battery chemistry and cell count.
8. Confirm the SOS path still gives local feedback if the modem power rail fails.

Safety notes:
- Never connect a modem board to an ESP32 3.3 V rail unless the board documentation explicitly permits it and the current budget is adequate.
- Do not drive a vibration motor or speaker directly from a GPIO without verifying current and using a suitable driver.
- Do not connect a 5 V UART/GPIO output directly to a 3.3 V-only ESP32 input.
- Final pin map and wiring require exact board revisions and datasheets.

## 7. SOS message and app minimum

Proposed event fields:
- event_id (unique; duplicate-safe)
- device_id (non-secret public identifier)
- event_timestamp_utc (if time is available)
- location: latitude/longitude or explicit unavailable state
- location_source: stick_gnss or phone_gnss
- location_age_seconds and accuracy estimate if available
- battery_state/voltage only if measured and calibrated
- connectivity state and firmware version

Minimum app behavior:
- User-defined emergency contacts and explicit consent.
- Manual SOS action and clear confirmation/cancel pattern to reduce accidental activation.
- Location permission requested only when needed; explain why.
- Display event time, location source/age, map link, and server receipt status.
- Show “not delivered”/“pending” rather than implying a contact has seen the alert.
- No automatic continuous tracking or periodic check-in in v1.
- Keep location history minimal, access-controlled and deletable.

Security:
- Device authenticates to the backend using a device-specific credential.
- TLS certificate validation is mandatory.
- Do not put backend admin/API secrets in the mobile app or firmware.
- Server validates payloads, rate limits requests, deduplicates event_id and returns explicit acknowledgement.
- Define credential revocation and data deletion before user trials.

## 8. Verification sequence and gates

| Gate | Test | Evidence required | Stop condition |
|---|---|---|---|
| G0 Inventory | Identify boards, camera, output loads and battery | Photos/markings + datasheets | Unknown voltage/model affecting safety |
| G1 Modem SKU | Verify exact SKU, bands, GNSS and interfaces | Official datasheet + carrier-board guide | Missing band/GNSS/interface requirement |
| G2 Operator | SIM attach, APN, HTTPS request, server acknowledgement | Timestamped logs and operator/area notes | Cannot attach/send reliably |
| G3 GNSS | Fix, accuracy/source/age outdoors and indoor fail case | Recorded NMEA/fix metadata and coordinates checked independently | Stale/misleading location shown as current |
| G4 Power | Transmit burst, camera, alert load and brownout tests | Supply voltage/current traces or logged measurements | Reset, unsafe heating or voltage outside spec |
| G5 App/API | Authenticated SOS and acknowledgement; retry/failure cases | App screenshots/logs plus server test logs | False “delivered”, exposed secret or unauthenticated endpoint |
| G6 Integration | Local obstacle alert under modem/network failure | Reproducible bench test log | Connectivity suppresses/delays local warning |
| G7 Usability | Controlled, supervised SOS and feedback test | Consent and observations | Accidental triggering, inaccessible controls or unclear failure state |

A document update is not proof that any gate passed. No hardware, network, app or safety tests have been run as part of this planning update.

## 9. Budget template

Fill prices only after checking local suppliers; avoid fabricated estimates.

| Cost category | Local price (DZD) | Source/date | Needed for v1? |
|---|---:|---|---|
| Cellular modem development board | TBD | TBD | Yes for device cellular data |
| LTE antenna | TBD | TBD | Yes |
| GNSS antenna (if separate) | TBD | TBD | Conditional |
| SIM/data plan | TBD | Operator/plan date | Yes |
| Battery + protection/charger | TBD | TBD | Yes, if no suitable existing power system |
| Regulator/power stage | TBD | Datasheet + supplier | Yes if existing rail is insufficient |
| Distance sensor | TBD | TBD | Yes for obstacle alert |
| SOS button + driver/output components | TBD | TBD | Yes |
| Enclosure, wire, connectors | TBD | TBD | Yes |
| Optional hosting/backend | TBD | Provider/date | Depends on architecture |
| Total one-time | TBD | — | — |
| Recurring monthly | TBD | — | Must be disclosed |

No purchases are authorized by this document. First produce a local quote comparison and a total-cost estimate, including recurring data/hosting charges.

## 10. Open decisions

1. Exact controller and camera board models.
2. Exact battery/regulator and available output drivers.
3. Local distance sensor choices and prices.
4. Exact cellular carrier-board SKU/revision and antenna connectors.
5. Intended operator and usage locations for coverage verification.
6. User-confirmed parts budget ceiling: 10,000 DZD; acceptable monthly data/hosting cost remains unknown.
7. Whether a minimal backend can be hosted without recurring cost; verify instead of assuming “free forever”.
8. Whether phone-location fallback should be a separate SOS action in the app for v1.

## 11. Change control

- Keep work on research/smart-blind-stick-architecture.
- Do not purchase parts, merge, publish or claim test success without evidence and explicit approval.
