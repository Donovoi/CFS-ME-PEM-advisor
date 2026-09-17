# Contributing

The project is currently in the prototype/planning stage. Contributions should preserve the central safety principle: the app reports personalised physiological deviations and user-labelled patterns; it does not make unsupported claims that a wearable can diagnose or exclude PEM.

## Development priorities

Follow the order in `docs/ROADMAP.md`:

1. reliable sensor collection;
2. loss-tolerant watch/phone sync;
3. controlled HRV measurement;
4. personal baseline;
5. event/symptom labelling;
6. transparent strain/recovery logic;
7. alerts only after shadow-mode evaluation.

Do not skip directly to a machine-learning classifier.

## Proposed project structure

```text
app-phone/
app-wear/
core-model/
core-algorithm/
core-protocol/
test-fixtures/
docs/
```

Keep `core-algorithm` and shared model/protocol code as free of Android framework dependencies as practical so that recorded datasets can be replayed in JVM tests.

## Requirements traceability

Significant features should reference relevant requirement IDs from `docs/PRODUCT_REQUIREMENTS.md` in the PR description.

Example:

```text
Implements FR-010, FR-011 and NFR-001.
```

Safety-related changes should also identify the relevant hazard from `docs/SAFETY_REGULATORY.md`.

## Algorithm changes

Any change to a physiological feature or state rule must:

- document the rationale;
- increment the appropriate feature/algorithm version;
- include deterministic test fixtures;
- preserve historical provenance;
- state how missing/poor-quality data is handled;
- avoid changing user-facing medical claims without safety/regulatory review.

Do not tune thresholds against a test event and then report the same event as independent validation.

## Sensor changes

When adding a sensor:

1. document why it is needed;
2. document Samsung permission/capability requirements;
3. measure battery/storage impact;
4. define data-quality rules;
5. add capability detection and error handling;
6. avoid collecting the signal continuously when a lower-cost sampling strategy is sufficient.

## Privacy rules

Do not add:

- third-party advertising;
- health-data analytics/telemetry;
- cloud upload;
- precise location;
- unrelated identifiers;

without first updating `docs/PRIVACY_SECURITY.md`, threat modelling the new data flow, and making the collection user-visible/consensual as appropriate.

Never commit:

- signing keys;
- API keys/tokens;
- raw personal health exports;
- identifiable test datasets;
- Samsung/private SDK licensing material that cannot be redistributed.

## Logging

Release code must not log raw physiological values, symptom text or secrets.

Debug diagnostics should use synthetic or explicitly local test data whenever possible.

## Testing expectations

For core algorithm code:

- unit tests for formulas;
- fixtures for state transitions;
- missing-data tests;
- reproducibility tests.

For watch collection:

- screen-off test;
- restart/reconnect test;
- permission failure test;
- battery test.

For sync:

- duplicate batch;
- interrupted batch;
- long disconnect;
- idempotent replay.

For UX:

- `Insufficient data` must never look like `Baseline`;
- large text/TalkBack checks;
- low-stimulation mode checks.

## Commit/PR guidance

Prefer small commits with clear intent, for example:

```text
feat(wear): collect continuous HR and IBI
feat(sync): add idempotent sensor batch protocol
feat(algorithm): add robust morning HR baseline
fix(safety): prevent baseline state on missing IBI
 docs: update Samsung permission requirements
```

PR descriptions should include:

- what changed;
- requirement IDs;
- safety/privacy impact;
- test evidence;
- screenshots for UI changes;
- battery impact for sensing/background changes.

## Research claims

When citing evidence:

- link the original paper/guideline where possible;
- distinguish ME/CFS from Long COVID or mixed chronic-illness cohorts;
- distinguish association from prediction and prediction from diagnosis;
- give study population/context;
- do not promote speculative mechanisms as established facts.

Update `docs/RESEARCH.md` when evidence materially changes feature selection or product claims.

## Licensing

A project licence has not yet been selected. Do not assume contribution/licensing terms until a licence is added to the repository.
