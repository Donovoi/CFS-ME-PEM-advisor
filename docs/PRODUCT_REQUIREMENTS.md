# Product Requirements Specification

## 1. Document status

- Status: Draft v0.1
- Product: CFS/ME PEM Advisor
- Initial platform: Samsung Galaxy Watch + Android phone
- Initial development target: Galaxy Watch Ultra
- Product mode: self-management/wellness and research-support prototype

## 2. Problem statement

People with ME/CFS and related post-exertional symptom conditions can experience delayed worsening after physical, cognitive, emotional or orthostatic exertion. The delay makes it difficult to recognise when current activity exceeds sustainable capacity and difficult to know when physiology has returned toward a personal baseline.

Consumer wearables can measure signals associated with autonomic and exertional state, but no consumer watch metric is currently validated as a diagnostic test for PEM. The product therefore needs to detect **within-person physiological deviation**, combine it with user-reported symptoms and present cautious, explainable guidance without claiming diagnostic certainty.

## 3. Goals

### G-001 — Personal baseline
Build a stable, continuously updated model of the user's typical physiology under relatively stable conditions.

### G-002 — Early strain indication
Identify meaningful deviations in multiple physiological signals before or during periods the user later labels as a crash or PEM-like event.

### G-003 — Recovery visibility
Show whether key physiological signals have returned toward their personal baseline after a labelled event.

### G-004 — Pacing support
Provide optional, configurable reminders when physiological load is unusually high for that user, without implying that remaining below a threshold guarantees protection from PEM.

### G-005 — Explainability
Show which signals changed, by how much, and over what time window.

### G-006 — Privacy
Operate locally without requiring an account, cloud service or external analytics in the MVP.

### G-007 — Build a research-quality personal dataset
Collect sufficiently well-structured data, with consent and clear labels, to evaluate whether the signals are useful for this individual before attempting predictive modelling.

## 4. Non-goals for MVP

- Diagnose ME/CFS.
- Diagnose, confirm or exclude PEM.
- Tell a user they are safe to exercise or increase activity.
- Replace clinical assessment.
- Provide emergency monitoring.
- Detect arrhythmias, myocardial infarction, infection or other diseases.
- Use a fixed population HR/HRV threshold as a universal PEM threshold.
- Train a population machine-learning model before the data pipeline and labelling quality have been validated.
- Require a cloud account or subscription.

## 5. User profiles

### P-001 — Primary self-manager
A person with ME/CFS or a related post-exertional symptom condition who wants help understanding their own exertion/recovery patterns.

### P-002 — Research-oriented user
A technically comfortable user who wants detailed charts, exports and transparent feature values.

### P-003 — Future clinician/research collaborator
A professional reviewing exported data. This is a future mode and must not imply clinical validation of the app.

## 6. Core user stories

- As a user, I want the app to learn my normal HR/HRV rather than compare me with healthy population averages.
- As a user, I want to know when my physiology is unusually strained even if my absolute HR does not look extreme.
- As a user, I want to label the start and end of a crash so the app can compare it with earlier days.
- As a user, I want to see whether my resting HR, HRV and ordinary-activity response are returning toward baseline.
- As a user, I want alerts to explain why they fired.
- As a user, I want to mute/snooze alerts when sensory stimulation is unwelcome.
- As a user, I want all core features to work without internet access.
- As a user, I want to export my data in a documented format.
- As a user, I want to delete all of my data permanently.

## 7. Functional requirements

### Onboarding and consent

**FR-001** The app shall explain that it is not a diagnostic or emergency medical device.

**FR-002** The app shall explain that a normal-looking status does not mean it is safe to exceed known energy limits.

**FR-003** The app shall obtain explicit permission separately for each sensor/data category it uses.

**FR-004** The app shall permit operation with optional signals disabled, while clearly showing reduced confidence/coverage.

**FR-005** The app shall show a hardware/SDK compatibility check before beginning baseline collection.

### Watch sensor collection

**FR-010** The watch app shall collect continuous heart rate when monitoring is enabled.

**FR-011** The watch app shall collect IBI values where available and retain signal-quality metadata where the SDK provides it.

**FR-012** The watch app shall collect accelerometer-derived movement information sufficient to separate rest from ordinary movement.

**FR-013** The watch app shall support optional continuous skin temperature where supported.

**FR-014** Raw PPG shall be optional and disabled for continuous all-day collection by default.

**FR-015** The watch app shall continue the selected monitoring mode while the display is off, subject to Wear OS/Samsung background execution requirements.

**FR-016** The watch shall locally buffer measurements when the phone is unreachable.

**FR-017** The app shall expose monitoring state, permission failures and sensor/SDK errors to the user rather than silently producing incomplete scores.

### Controlled morning measurement

**FR-020** The app shall offer a short morning measurement under a repeatable resting protocol.

**FR-021** The morning check shall display instructions intended to reduce measurement noise (for example: remain still and avoid talking during the measurement).

**FR-022** The app shall compute resting HR and quality-controlled HRV features from the measurement.

**FR-023** The app shall reject or mark low-quality measurements when too few valid IBI values remain after artefact filtering.

### Baseline engine

**FR-030** The system shall maintain rolling personal baselines for eligible features.

**FR-031** Baselines shall use robust statistics that are not overly affected by isolated outliers.

**FR-032** The system shall distinguish baseline-building, baseline-available and baseline-stale states.

**FR-033** The system shall avoid automatically learning obvious user-labelled crash periods into the stable baseline.

**FR-034** The user shall be able to mark days as unsuitable for baseline learning (for example illness, unusual travel or sensor problems).

**FR-035** Every baseline-derived comparison shall record the baseline window/version used so historical scores are reproducible.

### Feature extraction

**FR-040** The system shall calculate resting HR deviation from baseline.

**FR-041** The system shall calculate HRV RMSSD from valid normal-to-normal IBI sequences where data quality is adequate.

**FR-042** The system shall calculate HRV deviation relative to the user's own baseline.

**FR-043** The system shall estimate activity/movement load from accelerometer data.

**FR-044** The system shall estimate HR response relative to recent movement/activity level.

**FR-045** The system shall estimate post-activity HR recovery where a suitable activity/rest transition is observed.

**FR-046** Optional experimental features shall be clearly marked and independently disableable.

### Strain/recovery engine

**FR-050** The MVP shall produce an explainable physiological status rather than a PEM diagnosis.

**FR-051** The status engine shall combine multiple independent features and data-quality information.

**FR-052** A single abnormal measurement shall not, by default, produce the highest-severity status.

**FR-053** The system shall support at least: Baseline, Elevated strain, High strain, Recovery, and Insufficient data.

**FR-054** Each status shall expose contributing factors in human-readable form.

**FR-055** The score/status shall decay or resolve only according to new data and defined recovery logic, not merely because time has passed.

**FR-056** The system shall never map Baseline status to `safe to exercise` or equivalent wording.

### Alerts

**FR-060** Alerts shall be configurable and disabled by default until sufficient personal baseline data exists.

**FR-061** The app shall support a configurable real-time high-HR pacing reminder independent of the multi-signal status engine.

**FR-062** The fixed HR reminder shall be described as a pacing reminder, not a PEM detector.

**FR-063** Multi-signal alerts shall require sustained or repeated evidence sufficient to limit nuisance notifications.

**FR-064** Alerts shall include the reason they fired and a timestamp.

**FR-065** Users shall be able to snooze, mute and configure haptic intensity.

**FR-066** The app shall support a low-stimulation mode with reduced animation, brightness demands and notification frequency.

### Symptom/event labelling

**FR-070** Users shall be able to record symptom check-ins in under one minute.

**FR-071** Users shall be able to label a crash/PEM-like event start, peak and recovery/end retrospectively.

**FR-072** Event terminology shall distinguish `user-labelled event` from `app-detected physiological deviation`.

**FR-073** The user shall be able to score fatigue, brain fog, dizziness/orthostatic symptoms, pain/flu-like symptoms, sleep quality and overall function using simple scales.

**FR-074** The user shall be able to add optional contextual tags without requiring free-text medical history.

### Timeline and explanation

**FR-080** The phone app shall show a timeline aligning physiological status, activity, alerts and user-labelled symptoms.

**FR-081** The user shall be able to inspect a day and see feature values versus baseline.

**FR-082** Graphs shall show missing data explicitly rather than interpolating it as normal.

**FR-083** The app shall distinguish observed sensor data, derived metrics and user-entered data visually and in exports.

### Synchronisation and storage

**FR-090** Watch/phone transfer shall tolerate temporary disconnection and resume without duplicate records.

**FR-091** The phone shall be the canonical long-term store in the MVP.

**FR-092** The watch shall maintain only the local history required for resilience and on-watch summaries.

**FR-093** Data Layer messages shall not be treated as the sole persistent copy of health data.

### Export and deletion

**FR-100** The user shall be able to export data locally in documented CSV and/or JSON form.

**FR-101** Exports shall include units, timestamps, timezone/offset information, data quality and algorithm/baseline version.

**FR-102** The user shall be able to export a concise report summarising recent deviations and labelled events.

**FR-103** The user shall be able to delete individual events and all stored data.

**FR-104** Deletion shall propagate to local watch/phone copies where technically possible.

### Settings and transparency

**FR-110** The app shall expose algorithm version and active experimental features.

**FR-111** The user shall be able to inspect how the current score is composed.

**FR-112** Any future machine-learning model shall expose model version, training scope and confidence/coverage limitations.

## 8. Non-functional requirements

### Reliability

**NFR-001** Sensor collection shall fail visibly rather than silently.

**NFR-002** Data sync shall be idempotent.

**NFR-003** All records shall use stable IDs and monotonic sequence information where useful for de-duplication.

**NFR-004** Algorithm calculations shall be deterministic for a fixed input dataset and algorithm version.

### Battery

**NFR-010** The MVP shall prioritise HR/IBI + low-cost movement sensing over continuous raw PPG.

**NFR-011** Monitoring settings shall show their expected battery impact qualitatively.

**NFR-012** Battery benchmarks shall be collected on the Galaxy Watch Ultra before enabling all-day modes by default.

### Performance

**NFR-020** Watch UI actions should respond in under 250 ms for local interactions under normal load.

**NFR-021** Sensor callbacks shall perform minimal work; expensive feature extraction shall run off the UI thread.

**NFR-022** The watch shall batch/persist data to reduce unnecessary wakeups where compatible with monitoring goals.

### Privacy and security

**NFR-030** No account shall be required for MVP use.

**NFR-031** No third-party analytics or advertising SDK shall be included in the MVP.

**NFR-032** Health data shall not be transmitted to a server by default.

**NFR-033** Release logs shall not contain raw health measurements or symptom text.

**NFR-034** Sensitive exports shall be user-initiated and clearly identify their destination.

**NFR-035** The project shall maintain a documented threat model and vulnerability reporting process.

### Accessibility and ME/CFS-specific usability

**NFR-040** Core workflows shall be usable with minimal typing.

**NFR-041** The UI shall support large text and Android/Wear accessibility settings.

**NFR-042** Important information shall not rely on colour alone.

**NFR-043** Animation and haptic intensity shall be reducible.

**NFR-044** Daily check-ins should take less than 60 seconds for the common path.

### Compatibility

**NFR-050** First-class support shall target the Galaxy Watch Ultra during prototype development.

**NFR-051** Hardware capability detection shall gate unsupported sensor features.

**NFR-052** The software shall handle Samsung Health Sensor SDK/service version mismatch as an explicit compatibility error.

## 9. Data-quality requirements

- HRV is only calculated from windows meeting minimum IBI count and artefact-quality rules.
- Motion-contaminated resting windows are rejected or down-weighted.
- Missing data reduces confidence and can force `Insufficient data`.
- Sensor state and permission state are stored alongside measurements.
- Time changes and timezone changes must not corrupt ordering.
- The app must not silently substitute population defaults for missing personal baseline data.

## 10. Success criteria for the first usable prototype

The prototype is ready for personal testing when it can:

1. Collect HR/IBI and movement reliably with the watch screen off.
2. Buffer and synchronise at least 24 hours of data without loss in a deliberate phone-disconnect test.
3. Complete a controlled morning measurement and compute quality-controlled RMSSD.
4. Build a rolling personal baseline from stable days.
5. Show explainable HR/HRV deviation from baseline.
6. Record symptom/crash labels and align them on a timeline.
7. Produce a deterministic experimental strain state from replayed data.
8. Export a complete local dataset.
9. Delete all stored data.
10. Demonstrate acceptable battery impact in a full-day Galaxy Watch Ultra test.

## 11. Deferred requirements

- Population model / federated learning.
- Cloud backup/sync.
- Clinician portal.
- Automated treatment recommendations.
- Integration with electronic medical records.
- Formal PEM prediction claim.
- iOS support.
- Non-Samsung watch support.

## 12. Evidence assumptions

- PEM can be delayed and therefore immediate HR alone cannot establish whether PEM will occur.
- Within-person HR and HRV deviations are promising signals but are not specific to PEM.
- Sleep loss, infection, dehydration, heat, medications, caffeine, stress and many other factors can alter HR/HRV.
- A user-labelled event is the MVP's ground-truth label for personal pattern analysis; it is not an objective diagnosis.
- Evidence from Long COVID may inform feature selection but must not be treated as automatically equivalent to ME/CFS evidence.

See `RESEARCH.md` and `SAFETY_REGULATORY.md` for sources and limitations.
