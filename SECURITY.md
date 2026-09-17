# Security Policy

## Supported versions

The project is currently pre-release. There are no supported production versions yet.

Once releases begin, this file will list supported versions and security-update policy.

## Reporting a vulnerability

Please do not post exploit details, secrets, private health data or identifiable user datasets in a public issue.

Preferred reporting route:

1. Use GitHub's private vulnerability/security reporting mechanism for this repository if it is available.
2. If a private reporting option is not available, open a minimal public issue requesting a private security contact **without including vulnerability details or personal data**.

A dedicated security contact can be added before the first external/public release.

## What to include

Where safe, include:

- affected commit/version;
- affected watch/phone component;
- prerequisites;
- reproduction steps using synthetic data;
- confidentiality/integrity/availability impact;
- whether the issue could cause misleading physiological status or lost/corrupted health data;
- suggested mitigation if known.

Do not attach real personal health exports unless a secure exchange method has been agreed.

## Security-sensitive areas

Particularly important components include:

- health sensor permissions;
- local databases;
- watch/phone sync protocol;
- export/share flow;
- deletion flow;
- Android exported components/intents;
- release signing;
- dependency supply chain;
- algorithm/configuration integrity;
- any future networking or cloud service.

## Safety impact

A security bug can also be a safety bug. Examples:

- tampering causes `Baseline` when data is missing/abnormal;
- replayed sync data corrupts the personal baseline;
- deletion claims success while sensitive data remains;
- another app can read health information;
- an attacker can change alert thresholds;
- a malformed payload crashes continuous monitoring silently.

Reports with potential health/safety consequences should be prioritised accordingly.

## Secure development expectations

See `docs/PRIVACY_SECURITY.md` for the project threat model and engineering controls.

Before public release, the project should add:

- automated dependency/security scanning;
- secret scanning;
- static analysis;
- an SBOM/release dependency record;
- release-signing documentation;
- a defined vulnerability triage/response SLA;
- a dedicated private security contact.
