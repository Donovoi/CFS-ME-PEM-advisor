# Algorithm Design

## 1. Status

This document defines the **MVP experimental algorithm**, not a validated medical algorithm.

The output is a personalised physiological **strain/recovery state**. It must not be presented as a diagnosis, confirmation or exclusion of PEM.

## 2. Design principles

1. Compare the user primarily with themselves.
2. Prefer robust statistics over population cut-offs.
3. Require multiple signals for high-severity status.
4. Carry data quality into every derived result.
5. Never convert missing data into reassurance.
6. Separate real-time pacing reminders from slower multi-signal recovery/strain inference.
7. Make every result reproducible and explainable.
8. Collect user labels before attempting sophisticated prediction.

## 3. Signal hierarchy

### Tier 1 — primary MVP features

- Morning/resting heart rate
- HRV RMSSD from quality-controlled IBI
- Movement/activity load
- HR response relative to movement
- Post-activity HR recovery
- User-labelled symptom/crash state

### Tier 2 — useful context

- Sleep duration/quality where available
- Skin-temperature deviation
- Respiratory rate where available
- Resting time / overnight features
- User contextual tags

### Tier 3 — experimental

- Orthostatic-transition response inferred from watch motion + HR
- Raw PPG morphology features
- EDA where supported
- Personal predictive models

Tier 3 features cannot materially drive MVP user-facing alerts without explicit experimental opt-in.

## 4. Morning measurement

A controlled morning measurement provides the cleanest repeated comparison.

Proposed protocol:

1. User starts a 90–120 second check while resting and still.
2. First 20–30 seconds are treated as settling time.
3. HR + IBI are collected continuously.
4. Accelerometer verifies low movement.
5. Implausible/artifactual IBI values are removed.
6. If data quality is insufficient, the reading is rejected and the user may retry.
7. Resting HR, RMSSD and quality statistics are stored.

The exact duration should be validated against watch signal quality and user burden. The 2026 Visible study used a 60-second morning PPG measurement; the app may use a slightly longer window if it materially improves reliable IBI-derived HRV without creating excessive burden.

## 5. HRV calculation

Primary HRV feature: **RMSSD**.

For valid normal-to-normal intervals `NN_i` in milliseconds:

```text
RMSSD = sqrt(mean((NN_i+1 - NN_i)^2))
```

Implementation requirements:

- Calculate only from valid IBI sequences.
- Reject windows with excessive motion or insufficient accepted beats.
- Store accepted/total beat count and artefact percentage.
- Preserve the raw calculation window boundaries.
- Consider log-transforming RMSSD (`lnRMSSD`) for statistical modelling because its distribution is commonly skewed.

Artefact filtering thresholds must be validated empirically and versioned rather than hidden as magic constants.

## 6. Baseline construction

### 6.1 Warm-up period

Proposal:

- minimum: 7 eligible days before provisional comparisons;
- preferred: 14 eligible days for initial baseline;
- confidence improves through ~21+ stable days.

The UI must show when a baseline is provisional.

### 6.2 Eligible days

A baseline day can be excluded when:

- the user labels a crash/PEM-like event;
- the user marks acute illness or another major confounder;
- sensor coverage is insufficient;
- morning measurement quality fails;
- a software/hardware problem affected the data.

The system should not assume that every symptom-free day is perfectly healthy; the goal is simply to avoid obvious unstable periods contaminating the reference distribution.

### 6.3 Robust baseline statistics

For each feature maintain:

- rolling median;
- median absolute deviation (MAD);
- eligible sample count;
- baseline time range;
- recency weighting metadata.

A robust z-like score may be computed as:

```text
robust_z = (x - median) / (1.4826 * MAD)
```

When MAD is extremely small or sample count is insufficient, use a guarded fallback rather than allowing division instability.

### 6.4 Baseline adaptation

The baseline should evolve slowly.

Rules:

- Never update from periods marked as a crash/event.
- Avoid large same-day adaptation that would normalise away an emerging deviation.
- Consider a trailing 28–42 day eligible window after the initial baseline.
- Store baseline snapshots so historical results are reproducible.
- If the user's underlying function changes substantially for weeks, allow the baseline to re-establish rather than permanently comparing with an obsolete state.

## 7. Feature definitions

### F-001 Resting HR deviation

```text
RHR_DEV = robust_z(current_resting_hr vs personal resting-HR baseline)
```

Higher positive values indicate unusually high resting HR.

### F-002 HRV deviation

```text
HRV_DEV = -robust_z(current_lnRMSSD vs personal lnRMSSD baseline)
```

The sign is inverted so a larger positive value consistently means greater strain/deviation.

### F-003 Movement load

Compute minute-level movement magnitude from accelerometer signals, then derive:

- active minutes;
- cumulative movement load;
- sustained activity bouts;
- recent 1 h / 6 h / 24 h load versus the user's normal distribution.

Do not treat step count alone as total exertion.

### F-004 HR-to-movement residual

The same physical movement can sometimes produce a different cardiovascular response.

During sufficiently stable baseline periods, fit a simple personal relationship between movement features and HR. For a current window:

```text
HR_RESIDUAL = observed_HR - expected_HR_given_movement
```

A sustained positive residual may indicate unusually high cardiovascular cost for ordinary activity.

MVP implementation should begin with stratified movement bins or a simple regularised regression rather than a black-box model.

### F-005 Heart-rate recovery

When a clear activity-to-rest transition is detected, record HR decline over fixed periods such as 1, 2 and 5 minutes, provided movement remains low.

Compare with personal recovery behaviour for similar activity intensity rather than population norms.

### F-006 Temperature deviation

Where supported, calculate deviation from the user's comparable time-of-day baseline. Temperature is contextual because infection, environment and sensor contact can strongly affect it.

### F-007 Data coverage

Every scoring interval carries:

- percentage HR coverage;
- valid IBI count;
- motion contamination;
- watch-worn confidence if inferable;
- optional sensor availability.

Coverage is a first-class feature because confidence must fall when measurement quality falls.

## 8. Real-time pacing reminder

This is deliberately separate from the strain score.

The user may configure a HR value (for example one chosen with a clinician or from their existing pacing plan). If HR remains above that value for a configurable duration, the watch can vibrate.

Rules:

- default off;
- user-configurable;
- label as `pacing HR reminder`;
- never say crossing it causes PEM;
- never say remaining below it prevents PEM;
- allow temporary snooze during intentional activities.

## 9. Experimental strain score

### 9.1 Feature normalization

Convert each reliable feature to a positive strain contribution approximately bounded to `[0, 1]` using a monotonic transform of its robust deviation.

Example conceptual mapping:

```text
0.0 = near baseline
0.25 = mild deviation
0.5 = meaningful deviation
0.75 = large deviation
1.0 = extreme personal deviation
```

Exact breakpoints are experimental constants and must be tested through replay before use.

### 9.2 Multi-signal rule

High strain should require either:

- at least two independent primary features showing sustained meaningful deviation; or
- one very large primary deviation plus corroborating symptom/user-context evidence.

A single low HRV reading or elevated HR reading is not enough.

### 9.3 Proposed weighted score

Initial transparent formulation:

```text
score =
    w_rhr  * rhr_component
  + w_hrv  * hrv_component
  + w_hrm  * hr_motion_component
  + w_hrr  * recovery_component
  + w_load * recent_load_component
  + w_temp * temperature_component_optional
```

Weights sum to 1 over available reliable components, but missing features do **not** automatically increase confidence in the remaining score.

Return separately:

- numerical experimental score;
- categorical state;
- data coverage/confidence;
- list of contributing factors.

The UI may hide the raw number initially to avoid false precision.

## 10. Proposed state machine

```text
ESTABLISHING_BASELINE
        |
        v
BASELINE <------ RECOVERY
   |               ^
   v               |
ELEVATED_STRAIN    |
   |               |
   v               |
HIGH_STRAIN -------+

Any state -> INSUFFICIENT_DATA when coverage is inadequate.
```

### Baseline

Adequate data; major features are near recent personal baseline.

### Elevated strain

One or more reliable features are meaningfully abnormal or recent load is unusual, but evidence is not strong/sustained enough for High strain.

### High strain

Multiple independent features show sustained abnormality and data quality is adequate.

### Recovery

Entered after a user-labelled event or High strain period when measurements are improving but have not remained stable near baseline long enough to consider the episode resolved.

### Insufficient data

No reassuring interpretation may be shown.

## 11. Recovery logic

Recovery should be evidence-based, not timer-based.

A candidate recovery-complete rule might require:

1. major resting features return inside a configurable personal baseline band;
2. no sustained high-strain periods occur for 24–48 hours;
3. ordinary HR-to-movement response is near baseline;
4. the user reports symptoms/function have returned near their pre-event level, where the user chooses to provide this;
5. data coverage is adequate.

Because PEM can be delayed, the app should not convert one good measurement into a strong `recovered` message.

The initial implementation should call the state `Recovery` and show which signals have or have not normalised. It should avoid declaring that PEM has ended.

## 12. Event labelling and learning

User-labelled events are the central MVP outcome label.

Store:

- estimated onset;
- recognition/log time;
- peak (optional);
- recovery/end time;
- severity;
- symptom profile;
- suspected exertion windows/tags if the user chooses.

Derived analysis can then ask:

- Which features changed 6/12/24/48 h before the event?
- Which changed only after symptoms started?
- How long did each feature take to return to baseline?
- Which signals repeatedly precede events for this user?

## 13. Personal predictive modelling — later phase

Only after adequate labelled data exists.

Candidate models:

- logistic regression with regularisation;
- generalized additive model;
- simple gradient boosting with strict feature constraints;
- Bayesian personal time-series model.

Prefer models with calibration and explanation over raw discriminative performance.

Prevent leakage by using time-based train/validation splits. Never randomly mix later observations into training for earlier predictions.

Metrics:

- AUROC;
- AUPRC (important with rare events);
- sensitivity/specificity at declared thresholds;
- calibration curve;
- Brier score;
- false alerts per week;
- median useful lead time;
- percentage of days with enough data to score.

## 14. Confounders

The app should permit optional tagging and explanations for factors that can alter HR/HRV independently of PEM, including:

- infection/fever;
- dehydration;
- heat;
- poor sleep;
- medication changes;
- caffeine/stimulants;
- alcohol;
- emotional stress;
- travel/time-zone change;
- sensor fit/contact problems.

These should not be automatically interpreted as causes unless the user labels them.

## 15. Guardrails

The algorithm must never:

- say `No PEM` because vitals are near baseline;
- say `Safe to exercise`;
- automatically prescribe increased activity;
- diagnose an infection or cardiac problem from abnormal data;
- hide the fact that a feature is experimental;
- silently change thresholds/model versions;
- use data from a labelled crash as stable baseline without explicit logic allowing it.

## 16. Testability

Every feature and state transition requires deterministic fixtures covering:

- clean normal data;
- single-metric outlier;
- multi-metric sustained deviation;
- missing IBI;
- motion-contaminated morning reading;
- disconnected watch;
- user-labelled crash;
- gradual recovery;
- baseline drift;
- timezone change;
- algorithm version migration.

The replay harness should output both result and explanation text/factors for snapshot testing.
