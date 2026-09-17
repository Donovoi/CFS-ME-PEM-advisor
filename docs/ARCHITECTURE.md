# System Architecture

## 1. Architecture goals

The architecture must prioritise reliable sensing, offline operation, explainability, privacy and reproducibility over early predictive sophistication.

The first build is intentionally split into a **Wear OS sensor client** and an **Android phone companion**. The watch collects and buffers data; the phone performs most long-term storage, visualisation, baseline calculation and heavier analysis.

## 2. High-level design

```text
+-------------------------+              +------------------------------+
| Galaxy Watch            |              | Android phone                |
|                         |              |                              |
| Samsung Health Sensor   |              | Data Layer receiver          |
| SDK                     |              |                              |
|  - HR + IBI             |              | Canonical local database     |
|  - accelerometer        |   Data       |                              |
|  - skin temperature*    |<------------>| Feature/baseline engine      |
|  - PPG*                 |   Layer      |                              |
|                         |              | Strain/recovery state engine |
| Foreground health       |              |                              |
| service                 |              | Timeline + explanations      |
|                         |              |                              |
| Local resilient buffer  |              | Symptom/event labels         |
|                         |              |                              |
| On-watch status/alerts  |              | Export/delete                |
+-------------------------+              +------------------------------+

* optional / capability dependent
```

No backend is required for the MVP.

## 3. Watch responsibilities

### 3.1 Sensor service

A foreground health service owns sensor subscriptions while continuous monitoring is enabled.

Primary trackers:

- `HEART_RATE_CONTINUOUS`
- `ACCELEROMETER_CONTINUOUS`

Optional trackers:

- `SKIN_TEMPERATURE_CONTINUOUS`
- short-duration `PPG_CONTINUOUS` research windows

Samsung documents that screen-off continuous HR collection requires a foreground service and appropriate background/sensor permissions; a wake lock may be needed to prevent deep sleep from suspending collection depending on implementation.

### 3.2 Sensor ingest pipeline

Sensor callbacks should do as little work as possible:

1. timestamp sample;
2. attach source/device/quality metadata;
3. append to a bounded in-memory queue;
4. persist in batches;
5. update only lightweight on-watch state.

No expensive model inference should run inside sensor callbacks.

### 3.3 Local watch persistence

The watch keeps a durable local queue so that losing Bluetooth/Wi-Fi connectivity does not lose measurements.

Recommended logical tables:

- raw/near-raw HR+IBI samples;
- minute feature windows;
- pending sync batches;
- watch configuration;
- last-known status summary.

The buffer should be capacity-controlled. When storage pressure occurs, successfully synchronised raw samples are deleted before unsynchronised data.

### 3.4 On-watch feature processing

Only features needed for immediate pacing alerts should be calculated on-watch, for example:

- current HR;
- sustained time above user-configured pacing reminder threshold;
- recent movement level;
- simple data-quality status.

The phone remains authoritative for baseline/status calculations in the MVP.

### 3.5 On-watch UI

Primary watch surfaces:

- current physiological state summary;
- current HR and optional pacing reminder;
- `Log symptoms` shortcut;
- `Start morning check`;
- monitoring/battery/sensor status;
- low-stimulation notification settings.

## 4. Phone responsibilities

### 4.1 Canonical database

The phone is the long-term source of truth for:

- measurements/features;
- symptom/event labels;
- baseline snapshots;
- algorithm outputs;
- alerts;
- settings;
- provenance/version information.

Data Layer is a transport, not a database.

### 4.2 Ingest and de-duplication

Every watch-originated record includes:

- stable UUID;
- watch device ID (app-local pseudonymous identifier);
- event timestamp;
- receive timestamp;
- sequence number where useful;
- schema version.

Phone ingest is idempotent. Resending a batch after interruption must not create duplicates.

### 4.3 Feature engine

The phone computes repeatable time-window features from sensor samples, including:

- resting HR;
- RMSSD and related HRV quality statistics;
- movement load;
- HR-versus-movement residual;
- activity/rest transitions;
- post-activity HR recovery;
- optional temperature deviation;
- data-coverage metrics.

Features are versioned. Changing a formula creates a new feature version rather than silently altering historical meaning.

### 4.4 Baseline engine

Maintains per-feature rolling robust baselines using only eligible periods. User-labelled crashes, invalid data and explicitly excluded days are not incorporated into stable baseline learning.

### 4.5 State engine

Maps features + baseline deviation + data quality + recent labelled-event state into an explainable state such as:

- Baseline
- Elevated strain
- High strain
- Recovery
- Insufficient data

The engine must return both state and explanation factors.

### 4.6 Phone UI

Phone surfaces provide richer detail:

- today dashboard;
- feature explanation;
- symptom check-in;
- crash/recovery labelling;
- longitudinal timeline;
- event comparison;
- export/delete;
- experiment settings;
- data coverage and battery diagnostics.

## 5. Watch-to-phone communication

Use the Wear OS Data Layer API for direct watch/Android phone communication.

Recommended channels:

- **DataItem:** small durable configuration/state snapshots.
- **MessageClient:** immediate commands where delivery only makes sense when connected.
- **DataItem/Asset or chunked application protocol:** measurement batches, depending final payload design.

Important constraints:

- Data Layer payloads should remain small and versioned.
- Non-urgent synchronisation may be delayed; urgent delivery is reserved for genuinely time-sensitive status/config changes.
- The app retains its own persistent copies on both sides because Data Layer is not a storage layer.

## 6. Proposed modules

```text
root
├── app-phone/
│   ├── ui/
│   ├── data/
│   ├── sync/
│   ├── features/
│   ├── baseline/
│   ├── scoring/
│   └── export/
├── app-wear/
│   ├── ui/
│   ├── sensors/
│   ├── service/
│   ├── data/
│   ├── sync/
│   └── alerts/
├── core-model/
├── core-algorithm/
├── core-protocol/
└── test-fixtures/
```

`core-algorithm` should be pure Kotlin wherever possible so recorded datasets can be replayed in JVM unit tests without Android hardware.

## 7. Data-flow detail

### Continuous path

```text
Samsung sensor -> Watch callback -> Watch buffer
                                  -> 1-minute aggregation
                                  -> sync queue
                                  -> Data Layer
                                  -> Phone ingest
                                  -> feature windows
                                  -> baseline comparison
                                  -> state engine
                                  -> UI / optional alert
```

### Morning measurement path

```text
User starts morning check
 -> enforce stillness/quality checks
 -> collect HR + IBI for configured window
 -> artefact filtering
 -> resting HR + RMSSD + quality
 -> compare with personal morning baseline
 -> update daily state
```

### Crash-label path

```text
User labels event
 -> Event stored immediately
 -> baseline learner excludes affected interval
 -> state history annotated
 -> later analysis compares pre-event, event and recovery windows
```

## 8. Background execution

### Watch

Continuous sensing should use a foreground service with `foregroundServiceType="health"` and only the permissions required for enabled sensors. Permission handling must account for Android target-version differences documented by Samsung.

### Phone

Long-running periodic work should use WorkManager where appropriate. Live watch transfers are handled by Data Layer listeners rather than a continuously running phone service.

## 9. Storage strategy

### Watch

- Room/SQLite local buffer.
- Short retention after confirmed sync.
- Minute summaries retained longer than raw samples.

### Phone

- Room/SQLite canonical database.
- Application-private storage.
- Device storage encryption is assumed; additional application-level protection for exported/particularly sensitive data should be evaluated during implementation.

### Retention defaults (proposal)

- HR/IBI raw/near-raw: 30 days on phone unless user opts into research retention.
- Minute features: indefinite until user deletes.
- User labels/events: indefinite until user deletes.
- Raw PPG research windows: 7 days by default unless explicitly retained.

These defaults should be configurable before public release.

## 10. Algorithm isolation and replayability

Algorithm input/output types should contain no Android framework dependencies.

Every generated state records:

- algorithm version;
- feature version;
- baseline version/window;
- input coverage;
- timestamp;
- explanation factors.

A CLI/JVM replay harness should eventually be able to load an exported dataset and reproduce the same states.

## 11. Error states

The architecture must explicitly handle:

- sensor permission denied;
- Samsung SDK policy/partner error;
- Health Sensor Service incompatible/outdated;
- sensor unavailable on hardware;
- phone disconnected;
- insufficient IBI quality;
- clock/timezone change;
- low storage;
- low battery;
- watch reboot;
- phone app reinstall;
- schema migration failure.

No error should be converted into a reassuring `Baseline` state.

## 12. Future architecture options

Deferred until the local prototype is validated:

- opt-in encrypted cloud backup;
- research cohort data donation;
- clinician portal;
- federated/personalised machine learning;
- Health Connect import/export;
- cross-platform watch support.

## 13. External platform facts informing this design

- Samsung Health Sensor SDK currently supports continuous HR including IBI, accelerometer, PPG and skin temperature on supported Galaxy Watches.
- Samsung requires partner registration for public distribution of apps using the Health Sensor SDK; developer mode can be used locally during development.
- Wear OS Data Layer is intended for watch/phone communication and should not be used as the app's persistent datastore.

References are maintained in `RESEARCH.md`.
