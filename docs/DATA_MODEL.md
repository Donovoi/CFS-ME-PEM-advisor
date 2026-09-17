# Data Model

## 1. Goals

The data model must support:

- reliable watch-to-phone synchronisation;
- reproducible feature calculation;
- personal baseline versioning;
- user-labelled crash/recovery events;
- explainable status history;
- export and deletion;
- future algorithm replay without depending on undocumented app state.

The phone is the canonical long-term store. The watch keeps a resilient short-term copy of unsynchronised data plus enough summary state for on-watch operation.

## 2. Time conventions

Every time-series record should store:

- `timestamp_utc` — canonical instant;
- `utc_offset_minutes` — offset at capture time where useful for human reconstruction;
- `local_date` — derived/stored for day-oriented views;
- `source_clock`/sequence metadata where relevant.

Do not rely on local wall-clock time alone. Timezone changes must not reorder physiological data.

## 3. Core entities

### 3.1 Device

```text
Device
- id: UUID
- role: WATCH | PHONE
- manufacturer
- model
- os_version
- app_version
- sensor_sdk_version?
- pseudonymous_install_id
- first_seen_at
- last_seen_at
```

Do not store hardware serial numbers unless a future requirement makes them necessary.

### 3.2 SensorSample

Near-raw watch observation.

```text
SensorSample
- id: UUID
- device_id: UUID
- sensor_type: HR | IBI | ACCEL | SKIN_TEMP | PPG | OTHER
- timestamp_utc
- sequence_no?
- value(s)
- unit
- sdk_quality_code?
- app_quality_flags
- schema_version
- synced_at?
```

For high-rate data such as accelerometer/PPG, implementation may use compact binary/chunk storage rather than one SQL row per sample, provided export/replay semantics remain documented.

### 3.3 FeatureWindow

Derived summary over a defined interval.

```text
FeatureWindow
- id: UUID
- feature_name
- start_at
- end_at
- value
- unit
- feature_version
- source_coverage_pct
- quality_score
- quality_flags[]
- source_device_id
```

Examples:

- `hr_mean_1m`
- `hr_resting_window`
- `ln_rmssd_morning`
- `movement_load_1m`
- `hr_motion_residual_5m`
- `hr_recovery_1m`
- `skin_temp_deviation`

### 3.4 MorningCheck

```text
MorningCheck
- id: UUID
- started_at
- ended_at
- settling_seconds
- accepted_ibi_count
- rejected_ibi_count
- motion_quality
- resting_hr
- rmssd_ms?
- ln_rmssd?
- overall_quality
- rejection_reason?
- feature_version
```

Store enough quality metadata to explain why a check was accepted or rejected.

### 3.5 SymptomCheckIn

```text
SymptomCheckIn
- id: UUID
- timestamp_utc
- fatigue: ordinal?
- brain_fog: ordinal?
- dizziness_orthostatic: ordinal?
- pain_flu_like: ordinal?
- sleep_quality: ordinal?
- overall_function: ordinal?
- overall_worse_than_usual: boolean?
- optional_note?
```

Scales must be stable and documented; do not silently change scale meaning across app versions.

### 3.6 UserEvent

Represents a user-labelled crash/PEM-like episode or important context event.

```text
UserEvent
- id: UUID
- type: CRASH_LIKE | ACUTE_ILLNESS | POOR_SLEEP | MED_CHANGE |
        CAFFEINE | HEAT | TRAVEL | STRESS | CUSTOM
- onset_at?
- logged_at
- peak_at?
- ended_at?
- severity?
- note?
- exclude_from_baseline: boolean
```

`CRASH_LIKE` means user-labelled symptom worsening; it is not an app diagnosis.

### 3.7 BaselineSnapshot

```text
BaselineSnapshot
- id: UUID
- feature_name
- generated_at
- eligible_start_date
- eligible_end_date
- eligible_sample_count
- median
- mad
- optional_quantiles
- baseline_algorithm_version
- eligibility_rule_version
- confidence: PROVISIONAL | ESTABLISHED | STALE
```

Historical snapshots are immutable.

### 3.8 PhysiologicalState

```text
PhysiologicalState
- id: UUID
- timestamp_utc
- state: ESTABLISHING_BASELINE | BASELINE | ELEVATED_STRAIN |
         HIGH_STRAIN | RECOVERY | INSUFFICIENT_DATA
- experimental_score?
- coverage_confidence
- algorithm_version
- baseline_snapshot_ids[]
- explanation_factors[]
```

`explanation_factors` must contain machine-readable factor IDs plus user-facing values, for example:

```text
- factor: MORNING_HR_HIGH
  current: 72 bpm
  baseline: 62 bpm
  deviation: +1.8 robust-z
```

### 3.9 Alert

```text
Alert
- id: UUID
- type: PACING_HR | STRAIN | DATA_FAILURE | SENSOR_PERMISSION
- triggered_at
- delivered_at?
- acknowledged_at?
- snoozed_until?
- source_state_id?
- rule_version
- reason_factors[]
```

### 3.10 BaselineExclusion

Explicit record of why an interval is not eligible for baseline learning.

```text
BaselineExclusion
- id: UUID
- start_at
- end_at
- reason
- source: USER | SYSTEM
- source_event_id?
- created_at
```

### 3.11 AppSettings

Settings should be versioned and split by concern:

- enabled sensors;
- pacing threshold/reminder duration;
- low-stimulation mode;
- morning-check reminder;
- data retention;
- experimental feature flags;
- export preferences.

Avoid putting unbounded opaque JSON blobs in the database for core settings.

## 4. Sync envelope

Watch-to-phone batches should use a versioned envelope:

```text
SyncEnvelope
- protocol_version
- source_device_id
- batch_id: UUID
- created_at
- first_sequence_no
- last_sequence_no
- records[]
- checksum
```

Phone responses may acknowledge ranges/batches. A batch can be resent safely because record IDs are stable and ingest is idempotent.

## 5. Data provenance

Every derived value must be traceable to:

- source time window;
- feature algorithm version;
- baseline version if applicable;
- source coverage/data quality;
- app version.

This makes it possible to compare algorithm revisions without rewriting history.

## 6. Retention proposal

MVP defaults, subject to battery/storage testing and user feedback:

| Data | Watch | Phone |
|---|---:|---:|
| Unsynchronised HR/IBI | until synced + safety margin | 30 days raw |
| Minute HR/movement features | 7–14 days | retained until user deletes |
| Raw accelerometer | short rolling buffer / aggregated quickly | normally not retained raw |
| Raw PPG | research windows only | 7 days by default |
| Skin temperature | until synced | 30 days raw + long-term features |
| Morning checks | until synced | retained |
| Symptom/events | until synced | retained |
| Baseline snapshots | current + recent | retained |
| State/alerts | recent | retained |

Users should be able to opt into longer raw-data retention for research/export.

## 7. Export schema

At minimum provide:

```text
manifest.json
sensor_hr_ibi.csv
features.csv
morning_checks.csv
symptoms.csv
events.csv
baselines.csv
states.csv
alerts.csv
```

`manifest.json` should record:

- export format version;
- app/algorithm/feature versions;
- units;
- timezone handling;
- device models without unique hardware identifiers;
- feature definitions/reference to schema documentation.

## 8. Deletion model

### Delete event

Removing a user event must also invalidate/recompute baseline/state outputs whose eligibility depended on it. Historical derived results should either be recomputed or marked obsolete; they must not silently continue to claim the old provenance.

### Delete all

`Delete all data` must remove:

- phone database;
- watch app database/buffer;
- cached exports inside app-controlled storage;
- derived baselines and states.

User-created copies of exported files outside app-controlled storage cannot be remotely deleted; the UI must say this clearly.

## 9. Migration strategy

- Every table has explicit schema migrations.
- Feature formula changes are feature-version changes, not only DB migrations.
- Baseline/state calculation changes increment their own algorithm version.
- Destructive production migrations are not allowed without an explicit user-facing export/reset path.

## 10. Research-readiness requirements

The schema should enable questions such as:

- What changed 24–48 hours before a user-labelled crash?
- Which features normalised first during recovery?
- Does elevated HR for a given movement load precede symptom worsening?
- How often did the algorithm warn without a subsequent labelled event?
- How often was an event preceded by no measurable warning?

This is why provenance, missing-data markers and stable user labels are mandatory rather than optional metadata.
