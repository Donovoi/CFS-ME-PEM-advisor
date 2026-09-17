# CFS/ME PEM Advisor

A privacy-first Galaxy Watch + Android companion app for people living with ME/CFS and related post-exertional symptom conditions.

> **Project status:** planning/specification. No production app exists yet.

## Purpose

The project aims to help a user understand when their physiology is behaving unusually relative to **their own baseline**, especially around exertion, symptom flares and recovery.

The app will combine wearable signals such as heart rate (HR), inter-beat interval (IBI), heart-rate variability (HRV), movement/activity and optional skin temperature with short symptom check-ins. It will present explainable **physiological strain** and **recovery** signals rather than claiming to diagnose post-exertional malaise (PEM).

The central idea is:

1. Learn the user's stable baseline.
2. Detect within-person deviations from that baseline.
3. Combine multiple signals rather than relying on a single heart-rate threshold.
4. Ask the user to label symptoms/crashes so the system can learn their individual patterns.
5. Warn conservatively when several signals suggest unusual strain or incomplete recovery.
6. Never interpret a normal score as permission to exceed the user's known energy limits.

## Why this approach

PEM is delayed, heterogeneous and currently has no validated consumer wearable biomarker. However, current evidence supports investigating within-person changes in resting HR, HRV and recovery patterns. A 2026 study of 4,244 users of the Visible chronic-illness app found that higher-than-usual morning HR and lower-than-usual HRV were associated with worse same-day symptoms/crashes. Wearable HRV research in Long COVID also reports delayed autonomic recovery after exertion. These findings are promising but do **not** establish a diagnostic test for PEM.

ME/CFS guidance from NICE and the CDC supports personalised energy management/pacing and acknowledges that activity/heart-rate trackers can be useful self-monitoring tools.

## Product principles

- **Personalised, not population-normal:** compare the user primarily with their own rolling stable baseline.
- **Multi-signal:** no single vital sign is treated as PEM.
- **Explainable:** every score must show which measurements contributed.
- **Conservative:** false reassurance is more dangerous than a missed convenience alert.
- **Local-first:** the MVP requires no cloud account and keeps health data on the user's devices.
- **Offline-first:** the watch remains useful when the phone or internet is unavailable.
- **Low cognitive load:** interfaces are designed for fatigue, brain fog and sensory sensitivity.
- **Research-aware:** distinguish established evidence from experimental features.
- **No forced progression:** the app will never tell a user to increase activity because a score looks good.

## Initial hardware target

Primary development target:

- Samsung Galaxy Watch Ultra (Wear OS)
- Samsung Android phone

Compatibility objective:

- Galaxy Watch4 series and later for supported Samsung Health Sensor SDK features
- Skin-temperature features only where the hardware supports them

Samsung's Health Sensor SDK currently exposes continuous heart rate including IBI, accelerometer, PPG and skin temperature. Public distribution of apps using the SDK requires Samsung partner registration; developer mode can be used during development.

## MVP signals

**Primary**

- Continuous HR
- IBI-derived HRV (especially RMSSD from quality-controlled resting windows)
- Accelerometer / movement load
- Resting HR
- HR recovery after ordinary activity
- HR relative to movement level
- Morning controlled measurement
- User symptom/crash/recovery labels

**Secondary / optional**

- Skin temperature
- Sleep/recovery data where available with permission
- Respiratory rate where available
- User-entered context tags such as infection, poor sleep, medication change, caffeine, travel or unusual stress
- Experimental orthostatic-response estimates

Raw PPG will initially be reserved for short research/quality-assessment windows because continuous raw sampling can increase battery and storage cost.

## User-facing states

The MVP should use language such as:

- **Baseline** — measurements are near the user's recent stable range.
- **Elevated strain** — one or more signals are meaningfully different from baseline.
- **High strain** — multiple independent signals are abnormal or the deviation is sustained.
- **Recovery** — measurements are improving after a labelled symptom event but have not yet stabilised.
- **Insufficient data** — the system cannot form a reliable interpretation.

The MVP should **not** display `PEM detected`, `PEM excluded`, or `safe to exercise`.

## Repository documentation

- [`docs/PRODUCT_REQUIREMENTS.md`](docs/PRODUCT_REQUIREMENTS.md) — functional and non-functional requirements.
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — proposed watch/phone architecture and data flow.
- [`docs/ALGORITHM.md`](docs/ALGORITHM.md) — baseline, features, scoring and alert logic.
- [`docs/DATA_MODEL.md`](docs/DATA_MODEL.md) — planned entities, retention and synchronisation.
- [`docs/UX.md`](docs/UX.md) — watch/phone interaction model and accessibility requirements.
- [`docs/VALIDATION.md`](docs/VALIDATION.md) — technical, algorithmic and eventual clinical validation plan.
- [`docs/SAFETY_REGULATORY.md`](docs/SAFETY_REGULATORY.md) — safety boundaries and Australian regulatory considerations.
- [`docs/PRIVACY_SECURITY.md`](docs/PRIVACY_SECURITY.md) — privacy, threat model and security requirements.
- [`docs/ROADMAP.md`](docs/ROADMAP.md) — staged implementation plan.
- [`docs/RESEARCH.md`](docs/RESEARCH.md) — evidence base and source notes.
- [`docs/DECISIONS.md`](docs/DECISIONS.md) — initial architecture/product decisions.

## Proposed technology stack

- **Language:** Kotlin
- **Watch UI:** Jetpack Compose for Wear OS
- **Phone UI:** Jetpack Compose
- **Sensors:** Samsung Health Sensor SDK
- **Watch/phone communication:** Wear OS Data Layer API
- **Local persistence:** Room/SQLite with application-level protection for sensitive data where practical
- **Background work:** foreground health service on the watch for continuous collection; WorkManager on phone for deferred processing/export tasks
- **Architecture:** modular, unidirectional state flow; phone is the canonical long-term store, watch maintains a resilient local buffer
- **Testing:** JUnit, Android instrumentation tests, sensor-replay tests and deterministic algorithm fixtures

## Safety and regulatory boundary

This project begins as a **self-management/wellness and research-support tool**, not a diagnostic or treatment device. Samsung states that Health Sensor SDK measurements are intended for fitness and wellness information, not diagnosis or treatment.

In Australia, intended purpose matters. TGA guidance states that software intended to diagnose, prevent, monitor, predict or treat disease can meet the definition of a medical device. If this project's claims evolve from showing personalised physiological deviations to claiming it can predict or monitor PEM/ME/CFS as a medical condition, a formal regulatory assessment will be required before public supply.

See [`docs/SAFETY_REGULATORY.md`](docs/SAFETY_REGULATORY.md).

## Development rule

The first implementation milestone is **not** a PEM classifier. It is a trustworthy sensor pipeline and personalised-baseline engine whose output can be validated against user-labelled events.

## References

- NICE NG206, ME/CFS diagnosis and management: https://www.nice.org.uk/guidance/ng206/chapter/recommendations
- CDC, ME/CFS management and PEM: https://www.cdc.gov/me-cfs/management/index.html
- Nelson et al. (2026), *Digital physiological biomarkers predict within-person symptom changes in complex chronic illness*, npj Digital Medicine: https://www.nature.com/articles/s41746-026-02543-3
- Ruijgt et al. (2026), wearable HRV and PEM in Long COVID: https://pubmed.ncbi.nlm.nih.gov/42501245/
- Samsung Health Sensor SDK: https://developer.samsung.com/health/sensor/overview.html
- Wear OS Data Layer API: https://developer.android.com/training/wearables/data/overview
- TGA, software-based medical devices: https://www.tga.gov.au/resources/guidance/understanding-how-we-regulate-software-based-medical-devices
