# System Control

## 1. Purpose

The control center is the operating console for the SaaS owner. It manages clinic lifecycle, plans, platform health, support, incidents, AI capacity, and audit. It is not a universal medical-record browser.

Platform roles are separate from clinic memberships: `platform_owner`, `platform_operator`, `support_agent`, `security_auditor`, and `billing_operator`. A person receives only the minimum role needed; platform access requires MFA.

## 2. Control center views

### Portfolio overview

Show active/trial/suspended clinics, new registrations, active staff/patients (aggregated), bookings, job backlog, notification failures, AI availability, storage growth, backup freshness, current incidents, and deployment version. Avoid patient names and raw clinical data.

### Clinic detail

- Identity, owner contacts, verification, domains, frontend URL/version, locations.
- Lifecycle state, plan, trial/renewal, entitlements, usage against limits.
- Operational health, latest booking/queue activity counts, job failures, provider health.
- Storage, export, backup inclusion, AI key status and quota—not key values.
- Staff membership summary and recent privileged changes.
- Support sessions, incidents, invoices, notes, and complete operator audit.

### Global operations

Deployments, API latency/error rate, database/Redis/storage health, worker lag, dead-letter jobs, backup/restore state, provider outages, security alerts, feature rollouts, and incident timeline.

## 3. Clinic lifecycle state machine

```text
pending_verification -> trial -> active -> past_due -> suspended
                           |        |          |          |
                           +--------+----------+--> offboarding -> deleted
```

Transitions are server commands with permission, reason, confirmation, idempotency, and audit. Suspension blocks new ordinary writes and public booking but preserves configured patient-safety access, clinic export, payment resolution, and operator recovery. Deletion requires completed export, legal-hold check, grace period, and second approval.

## 4. Plans, limits, and entitlements

Plans version entitlements such as active doctors, locations, monthly appointments, storage, reminder volume, AI requests, reports, integrations, and support level. Existing subscriptions reference an effective entitlement version.

- Enforce limits server-side at the action creating consumption.
- Warn at 70%, 85%, and 100% where useful.
- Do not disable access to existing medical records because a quota is reached.
- Hard-limit costly creation (new staff/location, AI, excess messages) with a clear remedy.
- Temporary overrides require reason, expiry, and operator audit.
- Pilot billing may be manual, but plan state and invoice history remain explicit.

## 5. Feature flags

Flags have owner, purpose, environment, default, clinic overrides, rollout percentage if safe, start/expiry, dependencies, and rollback notes. Security checks never depend solely on a feature flag. High-risk clinical schema changes are not percentage-rolled within a single clinic workflow unless old/new versions interoperate.

Use kill switches for AI, outbound reminders, public booking, payment provider, file uploads, and specific unstable integrations. Core record reads remain available whenever safe.

## 6. AI key control

Each clinic may register up to three clinic-owned Gemini keys through secure server-side handling. The control center shows aliases/fingerprints, status, last success, safe error class, cooldown, daily usage, quota estimate, and rotation due date.

States: `pending_validation`, `active`, `cooldown`, `quota_exhausted`, `invalid`, `revoked`. Selector uses only active keys for the same clinic. A `429` applies bounded cooldown; authentication errors disable that key; anomalous traffic disables the clinic AI feature and alerts operators. A clinic key never silently falls back to another clinic's key.

Emergency platform-funded capacity, if ever enabled, is a deliberate entitlement with a global budget and audit—not an automatic hidden pool.

## 7. Support access

Support starts from a ticket and diagnostic snapshot. If a session is required, choose clinic, scope, read/write mode, reason, approver, and maximum duration. The system issues a short-lived support grant and displays a banner. It never reveals passwords or secrets.

Write support is exceptional. Membership changes, prescriptions, signed notes, refunds, export, key reveal, and deletion remain blocked or separately approved. Session termination revokes access immediately.

## 8. Operational actions

| Action | Safeguards |
|---|---|
| Verify/activate clinic | Verification evidence, owner confirmation, readiness checklist |
| Domain change | Ownership verification, preview, TLS check, rollback |
| Suspend/reactivate | Reason, impact preview, step-up auth, owner notification |
| Plan/limit override | Old/new value, expiry, billing note |
| Retry job | Idempotency check; sanitized payload preview |
| Replay webhook | Signature/event validation and duplicate protection |
| Generate export | Step-up auth, scope, expiry, encrypted artifact, download audit |
| Begin deletion | Two-person approval, legal hold/export checks, delayed execution |
| Rotate/revoke AI key | Clinic ownership, no value display, immediate state change |
| Start support session | Ticket, scope, approval, expiry, full audit |

## 9. Health and alert thresholds

At pilot scale, alert on: API unavailable for two consecutive checks; error rate above 5% for 5 minutes; p95 core API latency above 1.5 seconds for 10 minutes; database disk above 70/85/95%; backup older than 30 hours; worker oldest-job age above 5 minutes; dead-letter growth; repeated cross-tenant denials; notification failure above 20%; all AI keys unavailable for a clinic; and TLS/domain expiry risk.

Thresholds are starting points and must be tuned. Alerts route by severity and include runbook links, affected clinics, start time, request IDs, and safe context.

## 10. Abuse and fraud controls

Track booking spam, OTP abuse, repeated cancellations, mass patient search, bulk export, unusual refunds/adjustments, impossible operator activity, notification spikes, and AI bursts. Controls include throttles, temporary feature lock, re-authentication, review queues, and clinic/operator alerts. Do not make medical access dependent on opaque automated risk scores without human review.

## 11. Audit and reporting

Every control action writes an immutable operator audit. Provide filters by clinic, actor, action, target, request/ticket, result, and time; export itself is audited. Security auditors can read logs but cannot operate clinics. Clinic owners receive a tenant-only view of support sessions and privileged changes.

## 12. Control-center acceptance

- Platform role cannot be granted by a clinic admin.
- Support has no medical access before a valid session.
- Expired/terminated sessions stop immediately.
- Suspended clinics cannot create ordinary new activity yet can safely export.
- Entitlement limits cannot be bypassed in the UI or API.
- AI keys never appear in responses, logs, screenshots, or audit payloads.
- Every high-risk action has actor, reason, confirmation, result, and rollback/recovery path.