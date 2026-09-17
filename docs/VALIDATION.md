# Validation Plan

## 1. Purpose

The project needs separate evidence for four different questions:

1. **Can the watch collect the intended signals reliably?**
2. **Are the derived features calculated correctly?**
3. **Do the personalised strain/recovery signals correspond to user-labelled symptom changes?**
4. **Could the product eventually support a medical claim?**

These are different validation problems and must not be collapsed into one accuracy number.

## 2. Validation stages

### Stage A — Sensor pipeline verification

Goal: prove the software captures, timestamps, stores and synchronises data correctly.

Tests:

- continuous HR/IBI with screen on/off;
- watch reboot during monitoring;
- phone disconnected for 1 h, 8 h and 24 h;
- reconnect and verify no missing/duplicate batches;
- permission revoke/restore;
- low battery mode;
- low storage;
- timezone change;
- app/process restart;
- Samsung Health Sensor Service version mismatch.

Pass criteria should be defined per test before execution.

### Stage B — Physiological feature verification

Goal: confirm derived features are implemented as specified.

Use deterministic synthetic/replayed datasets for:

- RMSSD;
- artefact filtering;
- resting HR;
- movement aggregation;
- HR-motion residual;
- recovery windows;
- robust median/MAD baselines;
- state transitions.

Every formula requires unit tests with known expected output.

### Stage C — Hardware/reference comparison

Goal: quantify measurement agreement in representative conditions.

Potential references:

- ECG-quality or validated chest-strap reference for RR/HR during controlled resting and activity periods;
- manual protocol timestamps for movement/activity transitions;
- thermometer/reference only if temperature becomes materially important.

Protocol should include:

- quiet rest;
- standing;
- gentle walking;
- ordinary household movement;
- recovery after movement;
- repeated morning checks.

Important: consumer-wearable validation is condition-dependent. Agreement at rest does not imply equal accuracy during movement.

### Stage D — N-of-1 personal observational phase

Goal: determine whether the selected features are useful for one user's own event patterns.

Collect for several weeks before changing the algorithm aggressively.

Minimum useful dataset should contain:

- stable baseline days;
- symptom check-ins;
- several user-labelled crash/PEM-like episodes if they occur naturally;
- recovery intervals;
- contextual tags;
- adequate sensor coverage.

Do **not** deliberately provoke PEM for app testing.

Questions:

- Does morning HR rise before/with labelled events?
- Does HRV fall before/with labelled events?
- Is HR elevated relative to ordinary movement?
- Does HR recovery change?
- Which signals return to baseline first?
- How often are abnormal days unrelated to a crash label?

### Stage E — Prospective personal alert validation

Freeze an algorithm version before evaluating it.

For each day, record the state/alert before the outcome label is known. Avoid retrospectively tuning thresholds on the same event and then claiming that event as validation.

Metrics:

- event sensitivity;
- event-level precision/positive predictive value;
- false-alert days per week;
- alert lead time;
- percentage of days scoreable;
- calibration if a probabilistic score is introduced;
- user burden and alert fatigue.

### Stage F — Multi-participant research

Only after the personal prototype and governance are mature.

Requirements before recruiting others should include:

- defined research protocol;
- consent materials;
- ethics/HREC review where applicable;
- privacy impact assessment;
- data management plan;
- pre-specified primary/secondary outcomes;
- frozen algorithm/version;
- inclusion/exclusion criteria;
- statistical analysis plan.

A convenience sample of app users can generate hypotheses, but stronger medical claims require appropriately designed validation.

## 3. Ground truth problem

There is no single consumer biomarker that objectively establishes PEM.

MVP outcome label:

> **User-labelled crash/PEM-like event with symptom/function measures and timing.**

This must remain explicitly labelled as self-report.

Possible future reference outcomes:

- validated PEM questionnaires;
- clinician-adjudicated episodes;
- repeated CPET/research protocols where ethically appropriate;
- structured functional measures.

The absence of a perfect gold standard means reported performance must state exactly what outcome was predicted.

## 4. Data split methodology

Time-series validation must avoid leakage.

Do not randomly split minute/day observations across train and test sets when later samples could inform earlier ones.

Preferred approaches:

- chronological holdout;
- rolling-origin evaluation;
- leave-one-event-out for personal models;
- leave-one-participant-out or external cohort validation for population models.

Thresholds must be selected on training/development data, then evaluated on untouched data.

## 5. Statistical metrics

For binary future event prediction:

- AUROC;
- AUPRC;
- sensitivity;
- specificity;
- positive predictive value;
- negative predictive value;
- Brier score;
- calibration slope/intercept;
- false alerts per 7 days;
- median warning lead time.

AUPRC and false-alert burden are especially important if crash days are uncommon.

For continuous symptoms:

- within-person correlation;
- mixed-effects models for multi-user data;
- mean absolute error where prediction is meaningful;
- calibration across symptom ranges.

Never report only the best metric after testing many alternatives without correction/disclosure.

## 6. Baseline validation

Test the baseline engine against scenarios including:

- stable physiology;
- one extreme outlier;
- seven-day illness interval;
- gradual long-term shift;
- repeated missing mornings;
- crash interval accidentally included then corrected;
- seasonal/temperature change;
- medication/caffeine context changes.

Desired property: one abnormal day should not immediately redefine normal.

## 7. HRV quality validation

For each morning check capture:

- total IBI count;
- accepted count;
- rejected count;
- motion metric;
- RMSSD before/after filtering in debug/research mode;
- watch fit/quality errors exposed by SDK.

Compare watch-derived IBI/RMSSD with a suitable reference under rest first. Define an explicit quality threshold before relying on HRV for alerts.

## 8. Battery validation

Test at least:

- HR/IBI only;
- HR/IBI + accelerometer;
- + skin temperature;
- short raw PPG windows;
- worst-case debug logging disabled in release-like build.

Record:

- starting/ending watch battery;
- monitoring duration;
- screen-on time;
- sync frequency;
- dropped sensor coverage;
- temperature/thermal issues.

The product should prefer reliable 24-hour coverage over collecting every possible raw sensor continuously.

## 9. Usability validation

Specific ME/CFS-oriented questions:

- Can a user log symptoms during a bad day with minimal effort?
- Are alerts too frequent or stimulating?
- Does the language create false reassurance or anxiety?
- Can the user understand why a status changed?
- Is `Insufficient data` clearly different from `Baseline`?
- Can users retrospectively correct event onset without confusion?

Track completion time for morning check and symptom check-in.

## 10. Safety validation

Before release, test that the application never produces unsafe copy in cases such as:

- severe symptoms but normal wearable values;
- missing sensors;
- disconnected watch;
- elevated HR due to exercise;
- low HRV after poor sleep;
- high temperature deviation;
- algorithm exception;
- corrupted baseline.

No error path may display a reassuring normal state.

## 11. Reproducibility

For every evaluated algorithm version archive:

- source commit SHA;
- algorithm version;
- feature definitions;
- parameter file;
- dataset/export format version;
- evaluation script;
- evaluation output.

A result should be reproducible from exported input data without the original live app session.

## 12. Criteria before making stronger claims

Do not claim `predicts PEM` or `detects PEM` merely because personal graphs look correlated.

Before considering such wording, require at minimum:

- a precisely defined intended use;
- prospectively frozen algorithm;
- independent validation dataset;
- adequate sample/event count;
- clinically meaningful performance and calibration;
- documented failure modes;
- human-factors testing;
- regulatory assessment for target markets.

See `SAFETY_REGULATORY.md`.
