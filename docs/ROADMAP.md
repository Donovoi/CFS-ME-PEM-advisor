# Roadmap

## Guiding rule

Do not start by building a PEM classifier. Build a trustworthy sensor/data pipeline first, then baseline/feature logic, then alerts, then evaluate against labelled events.

## Phase 0 — Specification and project setup

**Status: in progress / documentation established**

Deliverables:

- product requirements;
- architecture;
- algorithm design;
- data model;
- UX/accessibility spec;
- validation plan;
- safety/regulatory notes;
- privacy/security plan;
- research bibliography;
- initial engineering decisions;
- issue backlog.

Exit criteria:

- requirements have stable IDs;
- major safety hazards documented;
- first build scope agreed;
- Samsung SDK development prerequisites understood.

## Phase 1 — Sensor feasibility spike

Goal: prove the Galaxy Watch Ultra can reliably provide the signals needed for the MVP.

Build:

- minimal Wear OS project;
- Samsung Health Sensor SDK integration;
- capability/permission diagnostics;
- continuous HR + IBI;
- accelerometer;
- foreground health service;
- screen-off collection;
- simple local log/buffer;
- developer diagnostics screen.

Tests:

- 2 h screen-off run;
- 8–12 h daytime run;
- overnight run if comfortable/useful;
- watch reboot;
- permission revoke;
- developer-mode/SDK policy behaviour;
- battery consumption.

Exit criteria:

- reliable HR/IBI collection with screen off;
- IBI quality characterised at rest and during movement;
- no silent collection failures;
- battery impact measured.

## Phase 2 — Reliable watch/phone data pipeline

Goal: no data loss during ordinary disconnection/reconnection.

Build:

- Android phone app shell;
- Room databases;
- versioned sync protocol;
- Wear OS Data Layer transport;
- stable IDs/sequences;
- de-duplication;
- sync acknowledgement;
- watch buffer retention/cleanup;
- data-coverage diagnostics.

Tests:

- phone disconnected 1/8/24 h;
- duplicate batch replay;
- interrupted transfer;
- watch/phone process kill;
- timezone change;
- low storage.

Exit criteria:

- 24 h deliberate disconnect can be recovered without duplicates/loss within defined tolerances;
- export can reconstruct the collection interval.

## Phase 3 — Morning check + HRV quality

Goal: establish a repeatable personal resting measurement.

Build:

- guided morning check;
- HR/IBI collection window;
- motion-quality check;
- artefact filtering;
- RMSSD/lnRMSSD;
- measurement-quality result;
- phone history chart.

Validation:

- repeat measurements under similar conditions;
- compare with reference device where available;
- test motion artefacts;
- tune minimum valid-beat/quality criteria.

Exit criteria:

- deterministic HRV calculation;
- poor-quality readings rejected rather than trusted;
- acceptable test/retest behaviour.

## Phase 4 — Personal baseline engine

Goal: compare the user with themselves.

Build:

- eligible-day logic;
- rolling median/MAD;
- robust deviation scores;
- provisional/established/stale baseline states;
- user exclusion tags;
- baseline snapshot/versioning;
- baseline explanation UI.

Exit criteria:

- synthetic/replayed baseline tests pass;
- one outlier does not materially redefine baseline;
- labelled crash periods can be excluded and recalculated reproducibly.

## Phase 5 — Activity and recovery features

Build:

- minute movement load;
- activity/rest segmentation;
- HR-to-movement relationship;
- HR-motion residual;
- post-activity HR recovery;
- data-coverage features;
- optional skin-temperature deviation.

Exit criteria:

- features work on replay data;
- movement contamination is visible;
- battery remains acceptable.

## Phase 6 — Symptom/event labelling

Build:

- sub-60-second symptom check-in;
- crash-like event onset/peak/end;
- retrospective editing;
- context tags;
- timeline alignment;
- compare-event view.

Exit criteria:

- event labels are easy enough to use during bad days;
- user-labelled data is clearly separated from app-derived inference.

## Phase 7 — Experimental strain/recovery engine

Build:

- transparent multi-signal score;
- state machine;
- explanation factors;
- `Insufficient data` gating;
- Recovery component view;
- replay harness and fixtures.

Initially no proactive multi-signal notifications; observe the state silently first.

Exit criteria:

- deterministic replay;
- no single normal outlier drives High strain;
- missing data never appears reassuring;
- several weeks of personal observational data available.

## Phase 8 — Pacing and strain alerts

Build:

- user-configured HR pacing reminder;
- sustained multi-signal alert rules;
- snooze/mute;
- low-stimulation mode;
- alert history;
- false-alert metrics.

Rollout:

1. shadow mode (calculate but do not notify);
2. review false positives/negatives;
3. enable alerts conservatively.

Exit criteria:

- acceptable false-alert burden;
- notifications are explainable and rate limited;
- no unsafe reassurance copy.

## Phase 9 — N-of-1 prospective evaluation

Freeze algorithm version.

Evaluate against future naturally occurring user-labelled events.

Track:

- sensitivity to labelled events;
- precision;
- false alerts/week;
- warning lead time;
- coverage;
- recovery tracking usefulness;
- battery/usability burden.

Exit criteria:

- enough prospective data to decide whether the approach is useful for the initial user;
- limitations documented honestly.

## Phase 10 — External pilot decision

Before involving other users:

- decide product vs research pathway;
- ethics/HREC assessment;
- regulatory classification review;
- privacy impact assessment;
- Samsung distribution partnership;
- release security review;
- support/incident process.

Only then create an external pilot build.

## Phase 11 — Personal prediction research (optional)

Only if labelled data supports it.

Experiment with interpretable personal models using chronological validation.

Candidate objective:

> probability of a user-labelled crash-like event within a defined future window

Do not expose this wording publicly without safety/regulatory review.

## Phase 12 — Public release / regulated pathway

Path depends on intended claims.

### Wellness/self-management route

- keep claims limited to physiological deviation/self-tracking;
- complete app-store, privacy, security and Samsung distribution requirements;
- formal regulatory assessment confirms intended positioning.

### Medical-device route

If the product claims to predict/monitor PEM or ME/CFS:

- formal TGA strategy;
- quality/risk lifecycle;
- clinical evidence;
- human factors;
- cybersecurity lifecycle;
- ARTG pathway as applicable;
- post-market monitoring.

## Initial engineering backlog order

1. Create Android multi-module skeleton.
2. Integrate Samsung SDK in Wear module.
3. Build sensor diagnostics screen.
4. Implement foreground HR/IBI collection.
5. Add accelerometer and quality metadata.
6. Add watch Room buffer.
7. Create phone app + Room store.
8. Implement versioned Data Layer sync.
9. Build export/replay fixture format.
10. Implement morning check.
11. Implement RMSSD + quality filter.
12. Implement baseline engine.
13. Implement symptom/event labelling.
14. Implement timeline.
15. Implement experimental state engine.
16. Run shadow-mode evaluation.

The first coding session should begin with items 1–4 only; adding more sensors before proving reliable continuous HR/IBI is intentionally deferred.
