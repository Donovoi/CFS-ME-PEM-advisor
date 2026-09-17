# Initial Product and Architecture Decisions

This file records important decisions so future code changes can be judged against the original rationale. Each item can later be promoted into a formal ADR if needed.

## D-001 — Personal baseline first

**Decision:** Primary interpretation compares the user with their own recent stable baseline.

**Why:** ME/CFS limits are highly individual and fluctuate. Current wearable research also supports within-person HR/HRV changes rather than a single universal value.

**Consequence:** Baseline quality/versioning is a core subsystem, not a cosmetic chart feature.

## D-002 — No `PEM detected` state in MVP

**Decision:** User-facing states are Baseline, Elevated strain, High strain, Recovery and Insufficient data.

**Why:** No validated wrist-wearable biomarker currently confirms/excludes PEM; HR/HRV are non-specific. Disease prediction/monitoring claims also change safety/regulatory obligations.

**Consequence:** We can evaluate correspondence with user-labelled events without overclaiming what the app knows.

## D-003 — Pacing HR reminder is separate from strain engine

**Decision:** A user-configured HR threshold may produce an immediate pacing reminder, but it does not feed a binary PEM judgment.

**Why:** HR can be a practical pacing cue, but neither crossing nor staying below a single threshold proves what will happen later.

## D-004 — HR + IBI + accelerometer are the primary sensor stack

**Decision:** Prove reliable continuous HR/IBI and movement first.

**Why:** Samsung exposes processed HR+IBI at 1 Hz and accelerometer at 25 Hz; together they support resting HR, RMSSD, movement context and HR-for-movement features at manageable data volume.

**Deferred:** continuous raw PPG as a default.

## D-005 — Raw PPG is research/diagnostic-of-the-sensor-pipeline only initially

**Decision:** Do not collect raw 25 Hz PPG all day by default.

**Why:** It substantially increases data/battery/storage burden and no MVP algorithm currently justifies that cost.

## D-006 — Phone is canonical long-term store

**Decision:** Watch buffers data; phone owns long-term history and most analysis.

**Why:** Better storage/compute/visualisation and easier export. Wear OS Data Layer is transport, not persistent storage.

## D-007 — Local-first MVP

**Decision:** No account/backend/cloud dependency.

**Why:** Minimises privacy/security scope, supports poor connectivity and makes the first validation about physiology/algorithms rather than infrastructure.

## D-008 — Explainable deterministic algorithm before machine learning

**Decision:** Start with robust personal baselines and transparent multi-signal rules.

**Why:** The first task is to learn whether the signals are useful. A black-box model trained on a tiny personal dataset risks leakage and false confidence.

**Future trigger:** adequate labelled prospective data.

## D-009 — User-labelled events are the MVP outcome label

**Decision:** Store crash/PEM-like events as explicitly user-labelled.

**Why:** There is no simple objective consumer gold standard. The app must distinguish self-report from inferred physiology.

## D-010 — Do not provoke PEM for app testing

**Decision:** Initial validation observes naturally occurring events and uses ordinary controlled sensor tests.

**Why:** Deliberately worsening illness is not justified for an engineering prototype.

## D-011 — Missing data is a state, not zero strain

**Decision:** Inadequate data forces `Insufficient data` or lowers confidence visibly.

**Why:** Sensor failure must never look reassuring.

## D-012 — Recovery is component-based

**Decision:** Show which signals have returned near baseline and which have not; avoid a single immediate `recovered` declaration.

**Why:** PEM can be delayed and physiology/symptoms may normalise at different rates.

## D-013 — Baseline updates slowly and excludes labelled unstable periods

**Decision:** Use rolling robust statistics with explicit eligibility/exclusion.

**Why:** Automatically learning a crash as `normal` could suppress useful deviation signals.

## D-014 — Algorithm and baseline provenance are immutable/versioned

**Decision:** Every derived status records feature version, algorithm version and baseline snapshot.

**Why:** Required for reproducibility, regression testing and any serious future research/regulatory work.

## D-015 — Galaxy Watch Ultra is the first hardware test target

**Decision:** Optimise and validate on one known device before claiming broad support.

**Compatibility objective:** later expand to supported Galaxy Watch4+ features with capability detection; skin temperature requires supported Watch5+ hardware.

## D-016 — Kotlin/Compose, pure-Kotlin algorithm core

**Decision:** Android/Wear apps use Kotlin and Compose; algorithms/models should avoid Android dependencies where possible.

**Why:** Enables fast deterministic JVM replay/unit testing from exported data.

## D-017 — Accessibility and low-stimulation mode are core requirements

**Decision:** Reduced haptics/animation/alert frequency and minimal-input workflows are first-class features.

**Why:** The target users may be cognitively fatigued or sensory-sensitive precisely when the app is most needed.

## D-018 — Regulatory review is a release gate, not an afterthought

**Decision:** Reassess intended purpose before external/public distribution and before any claim that the app predicts/monitors PEM.

**Why:** TGA classification depends heavily on intended purpose/claims; Samsung separately restricts sensor SDK use to fitness/wellness information and requires partner registration for distribution.

## D-019 — No third-party analytics in MVP

**Decision:** Product analytics/advertising SDKs are excluded initially.

**Why:** Health data sensitivity and the lack of a need compelling enough to justify expanded data flows.

## D-020 — Build order prioritises trustworthy data over UI polish

**Decision:** sensor reliability -> sync -> controlled HRV -> baseline -> labels/timeline -> score -> alerts.

**Why:** Attractive dashboards built on incomplete or low-quality data would create false confidence and make later debugging much harder.
