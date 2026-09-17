# UX and Accessibility Specification

## 1. UX objective

The product is for people who may be fatigued, cognitively overloaded, dizzy, light/sound sensitive, or unable to tolerate repeated interaction. The interface must therefore minimise decision load, typing, animation and unnecessary alerts.

The watch should answer one question quickly:

> **How different is my physiology from my recent personal baseline, and why?**

The phone should answer the follow-up questions:

> **What changed, when did it change, how does this compare with previous events, and is it trending back toward baseline?**

## 2. Language rules

Preferred language:

- `Baseline`
- `Elevated strain`
- `High strain`
- `Recovery`
- `Insufficient data`
- `Your resting HR is higher than your recent baseline`
- `HRV is lower than your recent baseline`
- `Your HR is higher than usual for this amount of movement`
- `Several signals are still outside your recent baseline range`

Avoid in MVP:

- `PEM detected`
- `PEM ruled out`
- `You are safe to exercise`
- `You have recovered`
- `Your mitochondria are failing`
- unsupported causal explanations

A normal state should say `near recent baseline`, not `healthy` or `safe`.

## 3. Watch information hierarchy

### Primary watch screen

```text
[STATE]
Elevated strain

2 signals changed
HR +9 bpm vs baseline
HRV lower than baseline

[Log symptoms]   [Details]
```

The screen should not show a pseudo-precise percentage unless later validation demonstrates that users understand it appropriately.

### Baseline-building state

```text
Learning your baseline
Day 5 of minimum 7

Today's data is being collected,
but strain alerts are not active yet.
```

### Insufficient-data state

```text
Not enough reliable data

Watch was off-wrist or HRV quality was low.
No reassuring interpretation is available.
```

## 4. Watch interactions

Required actions in two taps or fewer from the main screen:

- log symptoms;
- start morning check;
- view current HR;
- snooze alerts;
- open explanation.

Typing should not be required for common watch workflows.

## 5. Morning check UX

Proposed flow:

1. `Morning check — about 2 minutes.`
2. `Sit or lie quietly. Keep the watch still. Don't talk during the measurement.`
3. Simple progress ring with no distracting animation.
4. If motion/data quality fails: `Too much movement for a reliable HRV reading. Retry when comfortable.`
5. Result focuses on comparison rather than judgment:

```text
Morning check complete

Resting HR: +6 bpm vs baseline
HRV: 14% below baseline

Overall: Elevated strain
```

The app should not pressure the user to repeat failed measurements immediately.

## 6. Pacing HR reminder

Separate UI from the strain engine.

Example notification:

```text
Pacing reminder
HR has been above your 115 bpm reminder for 2 minutes.

This is your configured pacing limit, not a PEM diagnosis.
```

Controls:

- dismiss;
- snooze 15/30/60 minutes;
- pause until end of current activity;
- disable.

## 7. Phone dashboard

### Today card

Show:

- current state;
- confidence/data coverage;
- top 2–3 contributing factors;
- last morning check;
- symptom check-in shortcut;
- `Why this status?` link.

### Explanation screen

Example:

```text
Why Elevated strain?

Resting HR
72 bpm today
62 bpm personal median
Higher than 91% of your eligible baseline mornings

HRV (RMSSD)
28 ms today
39 ms personal median
Lower than your usual range

Movement response
Near baseline

Data quality
Good
```

Use actual values and plain language. Do not infer a disease mechanism.

## 8. Timeline

Phone timeline should align:

- state bands;
- HR/HRV features;
- activity/movement load;
- symptom check-ins;
- user-labelled crash onset/peak/end;
- alerts;
- excluded baseline periods;
- missing data.

Users should be able to tap a crash-like event and compare:

- 48 h before;
- event duration;
- recovery period;
- prior similar events.

## 9. Recovery view

The recovery screen should present components independently:

```text
Recovery tracking

Resting HR       Back near baseline
HRV              Still below baseline
Movement response Improving
Symptoms          User reports still worse than usual
Data coverage     Good

Several signals have not yet returned to your recent baseline.
```

This is preferable to a binary `recovered/not recovered` label.

## 10. Symptom check-in

Target common-path completion: under 60 seconds.

Use large tap targets and optional items.

Suggested scales:

- Overall compared with usual: Better / Usual / Slightly worse / Much worse
- Fatigue: 0–4
- Brain fog: 0–4
- Dizziness/orthostatic symptoms: 0–4
- Pain/flu-like symptoms: 0–4
- Sleep quality: 0–4
- Function today: 0–4

Then:

- `Are you in a crash/PEM-like episode?` Yes / No / Unsure
- optional tags;
- optional note.

`Unsure` is important because forcing a binary label can degrade both usability and research data.

## 11. Event labelling

Allow retrospective edits because PEM can be delayed and the user may recognise the event after it has started.

Fields:

- estimated onset;
- peak (optional);
- end/recovery (optional until later);
- severity;
- suspected preceding exertion (optional);
- notes.

Always display `user-labelled` in data exports and research screens.

## 12. Low-stimulation mode

A dedicated mode should:

- remove non-essential animations;
- reduce haptics;
- reduce alert frequency;
- use dark/low-luminance-friendly layouts while respecting system appearance;
- avoid flashing/pulsing elements;
- consolidate multiple non-urgent warnings;
- keep language short.

The user can configure whether High strain uses vibration, sound, both, or neither.

## 13. Accessibility

Requirements:

- support Android font scaling;
- large watch tap targets;
- TalkBack labels for every control;
- no information conveyed by colour alone;
- status uses icon + text + optional colour;
- graphs have accessible summaries;
- minimum interaction count for common tasks;
- landscape/tablet phone layouts are a later optimisation, not a blocker.

## 14. Notifications

Notification classes:

1. **Immediate:** user-configured pacing HR reminder.
2. **Important but not immediate:** sustained multi-signal strain change.
3. **Informational:** morning check reminder, sync/data-quality issue.

Rate-limit physiological notifications. Repeated abnormal readings should update an existing notification/state rather than generate a stream of alerts.

## 15. Error UX

Examples:

### Sensor permission lost

`Heart-rate access is off. Physiological status is unavailable until permission is restored.`

### Samsung SDK not authorised

`Samsung Health Sensor access is unavailable for this build/device. Open diagnostics for details.`

### Phone disconnected

`Phone disconnected. Your watch is storing data locally and will sync when reconnected.`

### Low data coverage

`Only 34% of expected heart-rate data was available today. Status confidence is low.`

Errors must never fall back to a normal-looking green state.

## 16. Research/advanced screen

Keep technical metrics out of the default experience but provide an advanced view with:

- robust-z values;
- RMSSD/lnRMSSD;
- baseline median/MAD;
- HR-motion residual;
- feature/version identifiers;
- coverage percentages;
- raw export actions.

## 17. Safety copy

Persistent About/Help copy should state:

- the app is not an emergency service;
- it does not diagnose or exclude PEM, ME/CFS, infection or heart disease;
- unusually severe or concerning symptoms should be assessed medically;
- a normal-looking wearable status does not override the user's symptoms or known limits.

Emergency red-flag wording should be reviewed before public release and adapted to the intended market.
