# Architecture

## 1. Scope and principles

Caddy Care uses one logical backend and database for many clinics. Each public clinic frontend is independently designed and deployed, while staff products and platform controls reuse stable workflows. The architecture optimizes first for tenant safety, clinical integrity, recoverability, and low pilot cost.

Core principles: fail closed on tenant ambiguity; server-authoritative permissions and calculations; transactional clinical/financial writes; asynchronous side effects; portable storage; observable operations; and graceful degradation of optional services.

## 2. Repository and deployment topology

This repository currently hosts the first dental demo and the specification. The target production topology is:

```text
Cloudflare DNS/proxy
  ├── clinic-a.pk -> Clinic A frontend deployment
  ├── clinic-b.pk -> Clinic B frontend deployment
  ├── app.caddy.care -> shared staff workspace
  └── control.caddy.care -> platform control center
                          |
                    api.caddy.care
                          |
       +------------------+------------------+
       |                  |                  |
  Web/API process   Background worker   Realtime process
       |                  |                  |
       +---------- PostgreSQL + Redis -------+
                          |
                     Object storage
```

For the 3-clinic pilot, services may share one Oracle ARM VM using containers, but they remain logically separable. Production web/server code should stay on TanStack Start unless an architecture decision record proves the need for another framework. Do not operate two independent sources of business truth.

## 3. Trust boundaries

1. **Public browser:** untrusted. Public clinic identifiers are routing hints, not authorization.
2. **Authenticated user:** identity is verified, but permissions depend on an active clinic membership.
3. **Clinic boundary:** every clinic-owned resource and derived artifact is scoped to one clinic.
4. **Platform boundary:** platform operators use separate privileges and step-up controls.
5. **External providers:** Gemini, email, WhatsApp, payment, and storage callbacks are verified and treated as untrusted input.

## 4. Tenant resolution

### Public requests

Resolve the normalized request host through an active `clinic_domains` record. Load only publishable clinic configuration. Unknown, unverified, or suspended domains fail closed. Preview domains use explicit environment-bound mappings, never fuzzy slug matching.

### Protected clinic requests

1. Validate the session server-side.
2. Resolve clinic from trusted host or an explicit clinic switch chosen from server-returned memberships.
3. Load an active membership for `(user_id, clinic_id)`.
4. Derive permissions from membership roles and policy—not from client claims alone.
5. Set clinic context for every query/transaction and include it in logs, jobs, cache keys, file paths, and realtime topics.

Never accept `clinic_id` from a mutation body as authority. Resource IDs are always queried together with the resolved clinic. A cross-tenant ID returns not found to avoid existence disclosure.

## 5. Application modules

| Module | Responsibilities |
|---|---|
| Identity and access | Users, sessions, MFA, clinic memberships, roles, permission evaluation |
| Clinic configuration | Clinics, domains, locations, chairs, hours, closures, services, pricing, branding |
| Patient registry | Clinic-scoped patient profile, guardians, alerts, consent, duplicate review |
| Scheduling | Provider schedules, slots, holds, appointments, waitlist, cancellation rules |
| Queue | Check-in, token sequence, priority reason, state transitions, ETA and display feed |
| Clinical | Encounters, odontograms, findings, treatment plans, procedures, prescriptions, files |
| Finance | Estimates, invoices, lines, payments, refunds, adjustments, cash closing, doctor shares |
| Communications | Templates, preferences, reminders, delivery attempts, inbound events |
| AI gateway | Redaction, clinic key selection, quota, model calls, safety, usage metadata |
| Audit and export | Immutable activity records, clinic export jobs, retention/legal hold |
| Platform control | Plans, limits, flags, support sessions, incidents, health, suspensions |

Modules may begin in one deployable application. Keep boundaries explicit in code and contracts so high-load jobs or realtime delivery can be separated later.

## 6. Critical request flows

### Booking

Availability reads use server-calculated slots. Booking starts a database transaction, locks or atomically consumes the slot/hold, validates clinic rules again, creates the appointment, records an outbox event, and commits. Notification workers process the event after commit. Repeated requests with the same idempotency key return the original result.

### Clinical record update

The server verifies clinical permission, clinic ownership, patient relationship, and record version. The mutation writes the new state and audit event in one transaction. Signed/finalized notes are corrected through addenda, not silent overwrite.

### Payment

The server calculates outstanding balance. A unique provider reference and idempotency key prevent duplicates. Payment, allocation, invoice state, ledger entry, and outbox event commit together. Provider callbacks are signature-verified.

### AI request

Authorize clinic and feature; classify use case; redact identifiers; enforce size and daily quota; select a healthy clinic-owned key; call the provider; safety-check output; return a draft; record non-clinical usage metadata. Failures never alter the clinical source of truth.

## 7. Background jobs and outbox

Use a transactional outbox for reminders, receipts, exports, file processing, webhooks, and analytics updates. Workers claim jobs with leases, retry with exponential backoff and jitter, and send exhausted jobs to a dead-letter state visible in the control center.

Every job payload includes `clinic_id`, event ID, schema version, and minimal identifiers. Jobs are idempotent. Never place raw clinical narratives or secrets in queue names, logs, or dead-letter summaries.

## 8. Files and media

Store metadata in the database and bytes in private object storage. Object paths begin with environment and clinic UUID. Browsers upload/download through short-lived signed URLs after permission checks. Validate type, size, and checksum; scan uploads when feasible; never trust filename extensions. Dental images remain private and are excluded from free-tier AI.

## 9. Cache and realtime

Redis holds short-lived slot holds, rate limits, safe public configuration, job coordination, and realtime fan-out. Every key includes environment and clinic UUID. Database remains authoritative.

Realtime topics are clinic-specific and permission-checked at subscription time. Events contain the minimum UI delta and no unnecessary medical details. Clients reconnect using sequence/version information and refetch authoritative state after gaps.

## 10. Frontend separation and contract reuse

Clinic websites do not share a compulsory design system. They may share generated API types, authentication helpers, error normalization, analytics consent utilities, and accessible booking primitives through a versioned SDK. The SDK must not impose colors, typography, layouts, navigation, or clinic copy.

The shared staff workspace uses one operational design system for consistency. Clinic brand accents may appear, but permissions and workflows stay identical.

## 11. Environments and change safety

Use separate development, staging, and production data/secrets. No production medical data is copied to non-production. Database migrations are forward-compatible: expand schema, deploy compatible code, backfill, switch reads, then remove obsolete fields in a later release.

OpenAPI and event schemas are versioned. Breaking API changes require a new major version or a compatibility window covering all deployed clinic frontends.

## 12. Architecture acceptance tests

- Host spoofing and unknown domains fail closed.
- A valid Clinic A user cannot read, mutate, subscribe to, export, or infer Clinic B data.
- Duplicate booking/payment requests produce one result.
- Worker retries do not send duplicate confirmed notifications.
- Redis, Gemini, email, and WhatsApp outages leave core records usable.
- A backup restores into an isolated environment and passes integrity checks.

Related: [Data model](DATA_MODEL.md), [API and events](API_AND_EVENTS.md), [Security and privacy](SECURITY_AND_PRIVACY.md), and [Operations](OPERATIONS.md).