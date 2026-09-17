# Safety and Regulatory Considerations

## 1. Scope

This document defines the safety boundary for the initial CFS/ME PEM Advisor prototype and records regulatory questions that must be revisited before public distribution.

It is not legal advice and does not determine the final regulatory classification of a future product.

## 2. Current intended purpose

The MVP is intended to:

- collect wearable physiology and user-entered symptom information;
- compare measurements with the user's own recent baseline;
- show physiological deviations and recovery trends;
- provide optional user-configured pacing reminders;
- help the user record and review patterns around self-labelled symptom/crash events.

The MVP is **not intended to diagnose, confirm, exclude, treat or prevent ME/CFS or PEM**.

The first build should use the terms `physiological strain`, `recovery`, `baseline deviation` and `user-labelled crash/PEM-like event` rather than claiming disease detection.

## 3. Why intended purpose matters in Australia

Australian Therapeutic Goods Administration (TGA) guidance states that software can meet the definition of a medical device when its intended purpose includes diagnosing, preventing, monitoring, predicting or treating a disease, injury or disability.

This matters directly to the project. A feature labelled:

> `Your HRV is below your personal baseline`

is materially different in intended purpose from:

> `PEM will begin in the next 12 hours`

A future claim that the software **predicts or monitors PEM/ME/CFS as a disease state** requires a fresh regulatory assessment and may bring the product within Australian medical-device regulation unless an exclusion/exemption applies.

The repository name does not by itself settle classification; claims, instructions, UI, marketing and actual intended use all matter.

## 4. Samsung platform limitation

Samsung's Health Sensor SDK documentation states that data measured through the SDK is for fitness and wellness information and is not for diagnosis or treatment of a medical condition.

Public distribution of an app using Samsung Health Sensor SDK also requires Samsung partner registration for the package/signing identity. During development, Samsung provides developer mode for local testing.

Consequences:

- Do not imply Samsung has clinically validated the PEM algorithm.
- Keep the prototype's intended purpose compatible with the permitted development/wellness context while regulatory strategy is unresolved.
- Treat Samsung partnership approval as a release gate, not a development blocker.

## 5. Safety hazards

### H-001 False reassurance

**Hazard:** physiology looks near baseline but the user is symptomatic or vulnerable to delayed PEM.

**Controls:**

- never display `safe to exercise`;
- no `PEM ruled out` state;
- normal state wording is `near recent baseline`;
- symptoms/known limits take precedence over app status in safety copy;
- missing data cannot produce Baseline.

### H-002 False alarm / anxiety

**Hazard:** normal autonomic variation generates repeated high-strain notifications, causing anxiety or excessive restriction.

**Controls:**

- multi-signal and persistence requirements;
- notification rate limits;
- explainability;
- ability to snooze/mute;
- avoid alarmist language;
- tune against false alerts/week, not only sensitivity.

### H-003 Single-metric overinterpretation

**Hazard:** user interprets high HR or low HRV as definite PEM.

**Controls:**

- separate pacing HR reminder from strain engine;
- no high-severity state from one ordinary outlier by default;
- show confounders and data quality;
- educational copy states HR/HRV are non-specific.

### H-004 Missing/poor sensor data

**Hazard:** sensor failure appears normal.

**Controls:**

- explicit `Insufficient data` state;
- coverage confidence;
- visible permission/SDK/device errors;
- no imputation to normal.

### H-005 Delayed PEM

**Hazard:** immediate post-activity metrics look normal and create premature reassurance.

**Controls:**

- no immediate `recovered` declaration;
- recovery considers sustained subsequent data;
- UI explains delayed symptom worsening can occur;
- recovery state remains component-based.

### H-006 Confounding illness/medication/environment

**Hazard:** app attributes HR/HRV changes specifically to PEM when caused by another factor.

**Controls:**

- only report observed deviations;
- optional contextual tags;
- temperature is contextual, not infection diagnosis;
- no causal attribution without evidence.

### H-007 Emergency misuse

**Hazard:** user relies on app for acute medical triage.

**Controls:**

- clearly state the app is not an emergency monitor;
- do not detect/exclude arrhythmia, myocardial infarction, infection, stroke, etc.;
- before release, include reviewed guidance that severe/concerning symptoms require appropriate medical assessment regardless of app state.

### H-008 Overactivity encouraged by score

**Hazard:** user increases activity because app says Baseline and triggers worsening.

**Controls:**

- no automatic activity progression recommendation;
- never convert low strain into an activity target;
- pacing remains based on user-known limits and professional advice where applicable.

### H-009 Baseline contamination

**Hazard:** prolonged unwell periods are learned as normal, suppressing useful warnings.

**Controls:**

- user event exclusions;
- slow baseline adaptation;
- baseline snapshots/versioning;
- stale/re-establishing baseline state;
- review timeline of baseline eligibility.

## 6. Safety requirements for copy

Every user-facing inference should answer:

1. What did the watch actually measure?
2. How does it compare with the user's baseline?
3. How reliable was the data?
4. What does the app **not** know from that measurement?

Example acceptable copy:

> `Your resting HR is 9 bpm above your recent median and HRV is below your usual range. Several physiological signals are unusually strained today. This is not a PEM diagnosis.`

Avoid:

> `Your body is entering PEM. Rest now to prevent damage.`

The latter makes both a diagnostic/predictive claim and an unsupported causal/treatment claim.

## 7. Regulatory decision gates

### Gate R0 — local prototype

- single developer/user testing;
- no public medical claim;
- no public supply;
- data remains local.

Action: document intended purpose and hazards.

### Gate R1 — closed external testing

Before involving other participants:

- decide whether activity is product testing or human-subject research;
- determine ethics/HREC requirements;
- privacy impact assessment;
- consent/data-management plan;
- review TGA implications of supplied prototype and claims.

### Gate R2 — public wellness release

Before app-store distribution:

- formal Australian regulatory classification assessment;
- Samsung partner registration/approval;
- privacy policy and collection notices;
- security review;
- app-store health-policy review;
- user-facing limitations and support process.

### Gate R3 — medical prediction/monitoring claim

If intended purpose becomes prediction/monitoring of PEM or ME/CFS:

- engage appropriate Australian regulatory expertise;
- determine device classification and ARTG pathway/exclusions/exemptions;
- define clinical evidence requirements;
- establish formal quality/risk/software lifecycle processes appropriate to classification;
- perform clinical and human-factors validation;
- implement post-market monitoring obligations if applicable.

## 8. Standards worth designing toward

TGA's 2026 guidance identifies standards commonly relevant to software-based medical devices, including:

- IEC 62304 — medical device software lifecycle processes;
- IEC 62366-1 — usability engineering;
- ISO 14971 — medical-device risk management;
- ISO 13485 — quality management systems;
- IEC 82304-1 — health software product safety;
- IEC 81001-5-1 — health software/health IT security lifecycle activities;
- ISO/IEC 29147 — vulnerability disclosure;
- ISO/IEC 30111 — vulnerability handling.

The MVP does not need to claim formal compliance. However, adopting traceability, versioning, risk logs, validation evidence and vulnerability handling early reduces rework if the project later enters a regulated pathway.

## 9. Regulatory traceability

Maintain a simple traceability chain:

```text
Hazard -> safety requirement -> implementation -> verification test -> evidence
```

Example:

```text
H-004 Missing data looks normal
 -> FR: explicit Insufficient data state
 -> StateEngine coverage gate
 -> replay test missing_ibi_and_hr.json
 -> test report / CI evidence
```

## 10. Medical/research evidence limitations

- HR and HRV can vary for many reasons and are not specific to PEM.
- Evidence from complex chronic illness cohorts and Long COVID is relevant to feature selection but does not establish a ME/CFS diagnostic biomarker.
- Self-labelled crash events are useful for personal pattern learning but are not an independent clinical gold standard.
- The product should describe associations, not unproven disease mechanisms.

## 11. Current authoritative references

- TGA, Understanding how we regulate software-based medical devices (24 Feb 2026): https://www.tga.gov.au/resources/guidance/understanding-how-we-regulate-software-based-medical-devices
- TGA, Software-based medical devices for consumers (updated 30 Jan 2026): https://www.tga.gov.au/resources/consumer-information-and-resources/software-based-medical-devices-consumers
- TGA, Standards for software-based medical devices (updated 5 Feb 2026): https://www.tga.gov.au/products/medical-devices/software-and-artificial-intelligence-ai/overview/standards-software-based-medical-devices
- Samsung Health Sensor SDK introduction/limitations: https://developer.samsung.com/health/sensor/guide/introduction.html
- Samsung app verification/partner process: https://developer.samsung.com/health/sensor/guide/app-verification.html

Regulatory guidance can change; re-check current TGA and Samsung requirements at every release gate.
