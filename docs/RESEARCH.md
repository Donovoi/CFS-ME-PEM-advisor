# Research and Evidence Notes

## 1. Purpose

This file records the evidence used to choose MVP signals and product constraints. It is not a claim that any listed biomarker can diagnose PEM or ME/CFS.

Evidence is grouped by how directly it informs the product.

## 2. High-confidence clinical/product constraints

### 2.1 PEM is delayed and variable

CDC describes PEM as worsening after physical or mental activity that would previously have been tolerated. Symptoms commonly worsen 12–48 hours after activity and can last days or weeks.

Product implication:

- immediate post-activity HR cannot determine whether PEM will occur;
- recovery cannot be declared from a single good reading;
- event analysis needs at least 24–48 h context.

Source:

- CDC, Manage ME/CFS: https://www.cdc.gov/me-cfs/management/index.html
- CDC, Strategies to Prevent Worsening of Symptoms: https://www.cdc.gov/me-cfs/hcp/clinical-care/treating-the-most-disruptive-symptoms-first-and-preventing-worsening-of-symptoms.html

### 2.2 Personal energy limits matter

NICE NG206 recommends personalised energy management and explicitly notes that each person has a different and fluctuating energy limit. It also recommends making self-monitoring easy, including use of existing activity trackers/heart-rate monitors where helpful.

Product implication:

- personal baseline over population cut-offs;
- no automatic graded activity progression;
- activity/symptom logging is as important as sensor collection.

Source:

- NICE NG206: https://www.nice.org.uk/guidance/ng206/chapter/recommendations

### 2.3 Heart-rate monitors can assist pacing but are not PEM tests

CDC notes that some people use heart-rate/activity monitors to avoid overexertion, while emphasising individual limits and delayed PEM.

Product implication:

- an optional HR pacing reminder is reasonable;
- it must remain separate from the multi-signal strain/recovery engine.

Source:

- CDC PEM toolkit: https://www.cdc.gov/me-cfs/pdfs/toolkit/managing-pem_508.pdf

## 3. Within-person wearable biomarker evidence

### 3.1 2026 Visible / npj Digital Medicine study

Nelson et al., *Digital physiological biomarkers predict within-person symptom changes in complex chronic illness* (2026).

Dataset:

- 4,244 Visible users;
- high-density repeated observations;
- 60-second morning PPG assessment;
- evening symptom reporting;
- HR, HRV and respiratory rate.

Key result relevant to this project:

- within-person increases in morning HR and decreases in HRV were associated with worsening evening symptoms/crashes;
- models including morning biometrics fit/predicted symptoms better than models relying on previous-day symptom reports/covariates alone.

Important limitations:

- complex chronic illness cohort rather than a pure, clinically confirmed ME/CFS cohort;
- `crash` is an app/self-report outcome;
- association/prediction in this dataset does not make HR/HRV specific biomarkers of PEM;
- product should not copy a proprietary score or imply equivalence.

Product implication:

- a controlled morning measurement using personal HR/HRV deviations is strongly justified as an experimental feature.

Source:

- https://www.nature.com/articles/s41746-026-02543-3

## 4. Autonomic / HRV evidence

### 4.1 2026 wearable HRV study in Long COVID

Ruijgt et al., *Wearable Heart Rate Variability Monitoring, Autonomic Dysfunction and Post-exertional Malaise in Long COVID: An Observational Study* (2026).

Study:

- 121 Long COVID patients;
- 21 healthy controls;
- continuous multi-day HR/HRV;
- CPET-defined first ventilatory threshold (VT1);
- daily activity logs.

Relevant findings:

- HRV was lower in Long COVID participants across daily activities and sleep;
- HRV remained lower for 24 h after exercise around/above VT1 compared with controls;
- authors concluded wearable HRV can assess autonomic function/overexertion in this population.

Limitations:

- Long COVID is not identical to ME/CFS;
- observational association does not establish a universal safe threshold;
- this does not prove consumer-watch HRV can diagnose PEM.

Product implication:

- HRV recovery trajectory is worth tracking;
- recovery state should be multi-hour/day rather than immediate;
- ventilatory/anaerobic threshold concepts may inform future research but should not become a hard default HR limit without individual assessment.

Source:

- PubMed: https://pubmed.ncbi.nlm.nih.gov/42501245/

### 4.2 Consecutive-day exercise ME/CFS research

Nelson et al. / related ME/CFS autonomic exercise literature has reported altered HR/HRV-related responses across consecutive exercise testing in ME/CFS.

One relevant paper:

- *Markers of Cardiac Autonomic Function During Consecutive Day Peak Exercise Tests in People With Myalgic Encephalomyelitis/Chronic Fatigue Syndrome* (2021)
- PubMed: https://pubmed.ncbi.nlm.nih.gov/34970156/

Product implication:

- cardiac/autonomic recovery features are biologically plausible research signals;
- maximal exercise testing should **not** be reproduced as an app validation method for a person with ME/CFS outside an appropriate clinical/research setting.

## 5. Symptom labelling evidence/problem

There is no single accepted consumer measurement that objectively marks the onset/end of PEM.

Vernon et al. (2023) examined PEM domains/triggers/recovery through questionnaire data in Long COVID and ME/CFS, illustrating symptom heterogeneity.

Source:

- https://pubmed.ncbi.nlm.nih.gov/36911963/

Product implication:

- the MVP must collect user-labelled events and symptom/function ratings;
- research outputs must say `user-labelled crash/PEM-like event`, not treat the label as laboratory-confirmed PEM.

## 6. Samsung Health Sensor SDK capability evidence

As of September 2026, Samsung Health Sensor SDK documentation states:

- `HEART_RATE_CONTINUOUS`: processed HR including IBI at 1 Hz;
- `ACCELEROMETER_CONTINUOUS`: raw x/y/z at 25 Hz;
- `PPG_CONTINUOUS`: raw green/red/IR PPG at 25 Hz;
- `SKIN_TEMPERATURE_CONTINUOUS`: processed skin and ambient temperature, available on Watch5 series and later;
- continuous trackers are supported until listener removal;
- measured data is for fitness/wellness, not diagnosis/treatment.

Source:

- Data specifications: https://developer.samsung.com/health/sensor/guide/data-specifications.html
- SDK overview: https://developer.samsung.com/health/sensor/overview.html

Product implication:

- HR+IBI + accelerometer are suitable primary prototype channels;
- continuous raw PPG creates ~25x the sample cadence of HR and should be optional/research-oriented initially;
- Samsung skin temperature is **skin/ambient temperature**, not core body temperature;
- IBI event packaging must be tested carefully because Samsung notes that complete tracking-time IBI values may be stored in the first returned data point when data arrives in screen-off batches.

## 7. Samsung background and distribution constraints

Samsung documents that reliable screen-off continuous HR collection may require:

- foreground service;
- `foregroundServiceType="health"`;
- appropriate health/body-sensor/background permissions;
- wake-lock handling in continuous monitoring implementations.

Samsung also requires partner registration for public distribution of apps using the Health Sensor SDK; local development can use developer mode.

Sources:

- Continuous HR with screen off: https://developer.samsung.com/galaxy-watch/blog/en/2026/04/23/continuous-heart-rate-tracking-on-galaxy-watch-even-with-the-screen-off
- Permissions: https://developer.samsung.com/health/sensor/guide/permission-request.html
- App verification: https://developer.samsung.com/health/sensor/guide/app-verification.html

## 8. Wear OS phone/watch transport

Google's Wear OS Data Layer API provides paired watch/Android phone communication. Google explicitly notes that Data Layer should not be treated as storage and recommends applications keep their own persistent copy of data.

Sources:

- Overview: https://developer.android.com/training/wearables/data/overview
- Sync guidance: https://developer.android.com/training/wearables/data/sync

Product implication:

- durable watch buffer + canonical phone database;
- idempotent sync;
- Data Layer only transports state/data.

## 9. Regulatory evidence

### 9.1 Australia / TGA

TGA 2026 guidance says software can be a medical device when intended to diagnose, prevent, monitor, predict or treat disease/injury/disability.

Sources:

- https://www.tga.gov.au/resources/guidance/understanding-how-we-regulate-software-based-medical-devices
- https://www.tga.gov.au/resources/consumer-information-and-resources/software-based-medical-devices-consumers

Product implication:

- `physiology differs from personal baseline` and `predicts PEM` are not equivalent claims;
- medical prediction/monitoring wording is a regulatory decision gate.

TGA also lists software/risk/usability/security standards relevant to medical-device software, including IEC 62304, IEC 62366-1, ISO 14971 and IEC 81001-5-1.

Source:

- https://www.tga.gov.au/products/medical-devices/software-and-artificial-intelligence-ai/overview/standards-software-based-medical-devices

## 10. Privacy evidence

OAIC guidance treats health information as sensitive and emphasises transparent collection/use and security. The project should adopt strong privacy principles even while the exact legal obligations of a future operator remain to be assessed.

Sources:

- APP Guidelines (updated May 2026): https://www.oaic.gov.au/privacy/australian-privacy-principles/australian-privacy-principles-guidelines
- Guide to health privacy: https://www.oaic.gov.au/privacy/privacy-guidance-for-organisations-and-government-agencies/health-service-providers/guide-to-health-privacy/introduction-and-key-concepts

## 11. Evidence grading for MVP features

This is an engineering prioritisation, not a clinical evidence grade.

| Feature | Rationale strength for prototype | Specificity for PEM | MVP role |
|---|---|---|---|
| Resting/morning HR vs self | Moderate | Low | Primary |
| Morning HRV/RMSSD vs self | Moderate | Low | Primary |
| Movement/activity load | Strong as context | Low | Primary |
| HR relative to movement | Plausible / exploratory | Low | Primary experimental |
| HR recovery | Plausible / supportive | Low | Primary experimental |
| Symptom/event self-report | Essential outcome context | Not objective biomarker | Primary |
| Sleep | Supportive/context | Low | Secondary |
| Skin temperature | Contextual | Very low | Secondary |
| Respiratory rate | Contextual | Low | Secondary |
| Raw PPG morphology | Research | Unknown | Deferred/experimental |
| EDA | Research | Unknown | Deferred |

## 12. Key research questions for this project

1. Which within-person features change before naturally occurring user-labelled events?
2. How early do they change?
3. Which signals are most useful for recovery tracking?
4. Does HR-for-movement become abnormal before symptoms?
5. How much does sleep/caffeine/illness confound alerts?
6. What false-alert rate is tolerable/useful?
7. Can morning measurement add useful information beyond symptoms and previous-day load?
8. Is continuous sensing materially better than one/two controlled measurements plus activity data?

## 13. What evidence would change the design

Revisit the algorithm if new peer-reviewed evidence establishes:

- a validated wearable PEM biomarker;
- a better personal thresholding method;
- a robust orthostatic metric from wrist-only sensors;
- improved Samsung sensor capabilities/data quality;
- evidence that a chosen feature is misleading or unsafe;
- updated NICE/CDC/TGA guidance.

Research review should be repeated before each external pilot/release, not treated as finished documentation.
