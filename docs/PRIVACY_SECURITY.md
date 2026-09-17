# Privacy and Security Requirements

## 1. Principle

The app handles sensitive physiological and symptom information. The MVP therefore follows a **local-first, data-minimising, no-advertising, no-third-party-analytics** design.

This document defines engineering requirements, not a final legal privacy policy.

## 2. Privacy model

### MVP defaults

- No account required.
- No cloud backend required.
- No telemetry/analytics SDK.
- No advertising SDK.
- No health-data upload by default.
- Watch-to-phone transfer only through the paired-device communication channel.
- Export only on explicit user action.
- User can delete all app-controlled health data.

### Data minimisation

Collect only signals needed for the current feature set. Do not collect:

- contacts;
- SMS/call history;
- precise location;
- advertising identifiers;
- unrelated device identifiers;
- microphone/camera data;
- free-text medical history unless a future feature explicitly requires it.

## 3. Australian privacy context

Australian health information is sensitive in nature and should be handled accordingly. OAIC guidance for health and mobile applications emphasises transparency, appropriate collection/use, security and clear privacy notices.

Whether the Privacy Act 1988 and Australian Privacy Principles legally apply to a particular future operator/business depends on circumstances and must be assessed before distribution. The project should nonetheless design to strong APP-like principles from the beginning:

- collect only what is necessary;
- explain what is collected and why;
- use data only for stated purposes unless valid consent/authority supports another use;
- protect data from loss, misuse and unauthorised access;
- provide access/export and deletion controls where appropriate;
- be clear about any future overseas/cloud disclosure.

## 4. Data classification

### Class A — sensitive health data

- HR/IBI/HRV;
- temperature;
- activity-derived physiological features;
- symptoms;
- crash/event labels;
- notes;
- algorithmic states linked to a user.

### Class B — device/app metadata

- app version;
- OS version;
- device model;
- sensor SDK version;
- permission state;
- local pseudonymous installation ID.

### Class C — non-sensitive configuration

- visual preferences;
- notification settings not revealing health state.

Class A requires the strongest handling controls.

## 5. Threat model

### Assets

- physiological time series;
- symptom/event history;
- user notes;
- derived state history;
- exports;
- algorithm configuration.

### Adversaries / failure sources

- another app on the device;
- person with physical access to unlocked device;
- malicious dependency;
- accidental export/share;
- debug logs/backups containing health data;
- compromised build/release pipeline;
- future backend compromise;
- lost/stolen watch or phone;
- malformed/corrupted sync payload.

## 6. Core security requirements

### SEC-001 — Least privilege
Request only the sensor/runtime permissions needed for enabled features.

### SEC-002 — Application-private storage
Databases and raw files shall be stored in app-private storage unless the user explicitly exports them.

### SEC-003 — Platform encryption
Rely on Android/Wear OS platform file-based encryption as a minimum. Evaluate application-level database encryption before external distribution or cloud features.

### SEC-004 — Key management
Any application-level encryption keys shall be protected with Android Keystore where technically appropriate; never hard-code encryption keys in the app/repository.

### SEC-005 — No sensitive release logging
Release builds shall not log raw HR/IBI, symptoms, notes, exported file content or encryption material.

### SEC-006 — Debug separation
Debug builds may expose additional sensor diagnostics but must visibly indicate debug mode and must not be used as the public release configuration.

### SEC-007 — Sync integrity
Watch/phone payloads shall use versioning, stable IDs and integrity checks. Deserialisation must reject malformed/unexpected payloads safely.

### SEC-008 — Export safety
Exports shall be explicitly user initiated. The UI shall warn that files copied outside the app may be readable by the destination app/service and cannot be deleted remotely by PEM Advisor.

### SEC-009 — Dependency control
Dependencies shall be version-pinned through Gradle lock/version catalog strategy where practical and reviewed for maintenance/security posture.

### SEC-010 — Signed builds
Release artifacts shall be signed through a documented process. Signing keys must never be committed to the repository.

### SEC-011 — Backups
Decide explicitly whether Android automatic backup includes the database. For sensitive local-only health data, default design should avoid unintentional cloud backup unless the user has clearly opted into an appropriate backup design.

### SEC-012 — Data deletion
`Delete all data` must remove application-controlled local health data from phone and watch. Deletion failures must be surfaced.

## 7. Authentication

The MVP does not require a PEM Advisor account.

Optional local app lock may be added later using device authentication/biometrics, but the app should not build a separate password database unless there is a strong requirement.

## 8. Network policy

MVP should be capable of operating with no internet permission unless a concrete requirement arises.

If internet access is later required:

- document every endpoint;
- use TLS;
- certificate/platform validation must not be bypassed;
- no clear-text traffic;
- minimise transmitted fields;
- define retention and server-side deletion;
- document data residency/overseas disclosure;
- perform threat modelling before enabling upload.

## 9. Wear OS Data Layer

Wear OS Data Layer is app-scoped paired-device communication, but application code still needs to:

- authenticate protocol/schema logically through expected package/channel context;
- validate payload size/type/version;
- avoid treating incoming content as trusted simply because it arrived from the Data Layer;
- store received records idempotently;
- keep persistent source copies until acknowledgement.

## 10. Data retention

Retention should be proportional to usefulness.

Raw high-frequency data carries higher privacy/storage cost than derived features. Defaults proposed in `DATA_MODEL.md` deliberately retain raw data for shorter periods while retaining user-understandable summaries longer.

The retention screen should let the user see:

- raw HR/IBI retention;
- raw PPG retention;
- long-term feature/event retention;
- approximate storage use;
- export/delete actions.

## 11. Privacy UI requirements

Before enabling a sensor, explain:

- what it measures;
- why the app wants it;
- whether it is optional;
- where the data is stored;
- whether it leaves the device.

Do not bundle unrelated permissions behind one vague statement.

## 12. Research/data donation — future only

Any future cohort/research upload must be a distinct opt-in feature.

Requirements before implementation:

- explicit consent separate from normal app use;
- purpose limitation;
- data dictionary;
- pseudonymisation strategy;
- re-identification risk assessment;
- withdrawal/deletion process;
- retention period;
- ethics/governance review where applicable;
- secure transport/storage;
- no conditioning core app functionality on research participation.

## 13. Vulnerability handling

Repository shall maintain `SECURITY.md` with a private reporting route once an appropriate contact mechanism is chosen.

Process goals:

1. acknowledge report;
2. reproduce/triage;
3. assess confidentiality/integrity/availability and potential health/safety consequence;
4. patch/test;
5. communicate affected versions and mitigations;
6. preserve a vulnerability record.

Design should align with the spirit of ISO/IEC 29147 and ISO/IEC 30111 if the project progresses toward medical-device status.

## 14. Build/release security checklist

Before any public build:

- release logging audited;
- no secrets in repository/history;
- dependency vulnerability review;
- permissions reviewed;
- exported Android components reviewed;
- backup policy reviewed;
- secure update/signing process documented;
- SBOM generation considered;
- static analysis/lint clean or exceptions documented;
- data deletion verified;
- privacy notice matches actual data flows.

## 15. Current references

- OAIC, Australian Privacy Principles guidelines (updated May 2026): https://www.oaic.gov.au/privacy/australian-privacy-principles/australian-privacy-principles-guidelines
- OAIC, Guide to health privacy: https://www.oaic.gov.au/privacy/privacy-guidance-for-organisations-and-government-agencies/health-service-providers/guide-to-health-privacy/introduction-and-key-concepts
- OAIC, Mobile privacy guide: https://www.oaic.gov.au/privacy/privacy-guidance-for-organisations-and-government-agencies/more-guidance/mobile-privacy-a-better-practice-guide-for-mobile-app-developers
- TGA, medical device cyber security guidance: https://www.tga.gov.au/resources/guidance/complying-medical-device-cyber-security-requirements

Reassess privacy-law scope before distribution, particularly if cloud, research, clinician access or commercial operation is introduced.
