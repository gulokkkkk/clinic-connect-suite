# Implementation Roadmap

## 1. Delivery strategy

Deliver one vertical slice at a time, with tenant isolation, authorization, audit, errors, and operations included in each slice. Do not build every screen first and postpone the backend safety model.

Initial team assumption: 1–2 developers plus clinic-domain feedback. Dates are estimates; exit criteria, not calendar time, decide completion.

## 2. Milestone 0 — Dental demo foundation (weeks 1–2)

**Build:** finish clinic visual direction; public home/services/dentists/about/contact; interactive booking prototype; staff shells; realistic synthetic dataset; responsive and accessibility baseline.

**Exit:** public identity is clear; booking scenario works in demo state at phone/desktop widths; staff navigation covers reception, dentist, owner, and control; no dead links; every content route has metadata and core states.

## 3. Milestone 1 — Platform foundation (weeks 3–4)

**Build:** production environments; database migrations; clinics/domains/users/memberships/roles; trusted tenant resolution; authentication/MFA path; audit base; operator roles; CI security and tenant tests; backup automation.

**Exit:** two synthetic clinics pass read/write/file/job/realtime isolation tests; roles are server-enforced; operator access is separate; a backup restores; deployment/rollback rehearsed.

## 4. Milestone 2 — Clinic configuration and onboarding (weeks 5–6)

**Build:** clinic verification/lifecycle, owner setup, locations/chairs, services/prices/durations, dentists/staff, schedules/closures, public configuration, domains, plan entitlements, control-center clinic detail.

**Exit:** operator can provision a clinic without direct database changes; owner configures clinic; verified domain renders only publishable configuration; suspension and reactivation behave safely.

## 5. Milestone 3 — Scheduling and queue (weeks 7–9)

**Build:** availability engine, slot holds, patient registration, appointment commands, walk-ins, waitlist, check-in, queue, TV-safe feed, realtime updates, reminders/outbox, resilient retry.

**Tests:** concurrent requests, idempotent confirmation, schedule exceptions, cancellation windows, cross-tenant IDs/topics, provider outage and duplicate worker delivery.

**Exit:** no double booking in load/concurrency test; receptionist completes booked and walk-in journeys; queue remains authoritative after reconnect; failed reminders do not affect appointments.

## 6. Milestone 4 — Dental records (weeks 10–13)

**Build:** patient profile/alerts/consent, encounters, odontogram and history, findings, periodontal summary, treatment plans/stages, procedure completion, notes/signing/addenda, prescriptions, follow-ups, secure files.

**Exit:** dentist completes exam-to-follow-up; signed records cannot be silently altered; receptionist lacks private-note access; files use short-lived authorized links; patient sees only patient-safe views.

## 7. Milestone 5 — Billing and clinic reporting (weeks 14–16)

**Build:** estimates, invoices/price snapshots, payments/allocations, refunds/adjustments, receipts, cash sessions/closing, outstanding balances, doctor attribution/share reports, owner KPIs.

**Exit:** appointment-to-checkout and partial-payment flows reconcile; duplicate callbacks cannot duplicate payment; variance and refund require reason/permission; reports match ledger test fixtures.

## 8. Milestone 6 — Communications and AI (weeks 17–18)

**Build:** notification preferences/templates, email/push and optional WhatsApp, delivery status, clinic Gemini key pool, quota/health/cooldown, redaction, FAQ/booking assistant, de-identified draft summary, kill switches.

**Exit:** no direct identifiers reach free-tier AI tests; all-key failure degrades cleanly; provider replay is safe; quotas and key health appear without secret exposure; clinical drafts require human confirmation.

## 9. Milestone 7 — System control and operational readiness (weeks 19–20)

**Build:** portfolio health, plans/limits, feature flags, support sessions, incidents, exports/offboarding, storage/capacity, backup evidence, alerts, deployment/frontend version inventory.

**Exit:** high-risk actions require step-up/reason/audit; support expires automatically; export and offboarding drill succeeds; alerts reach two humans; incident tabletop and monthly restore pass.

## 10. Milestone 8 — First clinic pilot (weeks 21–24)

**Build:** customize clinic #1 public UI, import verified setup data, train staff, parallel-run selected workflows, capture issues and baseline metrics. Add clinics #2 and #3 only after clinic #1 stability gate.

**Track:** booking completion, no-show rate, median wait, reminder delivery, checkout reconciliation, failed jobs, support tickets, staff task time, uptime, restore evidence, and patient feedback.

**Exit:** clinic staff operate without developer presence for two consecutive weeks; no unresolved high-severity security/data issue; financial close reconciles; clinic signs pilot acceptance; capacity remains below thresholds.

## 11. Testing matrix required throughout

- Unit: schedules, permissions, transitions, money, redaction, plan limits.
- Database: constraints, transactions, RLS/tenant context, migrations.
- Contract: OpenAPI responses/errors, compatibility, events/webhooks.
- Integration: booking, queue, clinical, billing, notifications, files, exports.
- Security: cross-tenant matrix, role escalation, IDOR, session/CSRF, upload, webhook replay, secret/log leakage.
- End-to-end: each role and the nine demo scenarios.
- Operational: backup restore, rollback, provider outage, Redis loss, disk alert, dead-letter recovery.
- UX/accessibility: phone/tablet/desktop, keyboard, focus, print, Urdu text expansion, offline/error states.

## 12. Release gates

No real patient data until security/privacy launch gate passes. No second clinic until cross-tenant tests and clinic #1 stability pass. No paid SLA until measured uptime, recovery, and support coverage justify it. No “offline mode” claim until conflict behavior is implemented and tested. No AI clinical claim beyond reviewed draft assistance.

## 13. Explicitly deferred

Native mobile apps; autonomous diagnosis/prescribing; full practice accounting/payroll; insurer/panel claim automation; pharmacy-grade inventory; laboratory system integrations; complex orthodontic imaging/cephalometrics; multi-region active-active; self-service clinic signup with instant production activation; unbounded custom workflows; and fully offline clinical editing.

## 14. Decision log required before implementation

Record architecture decisions for: production identity provider; exact database hosting after pilot; RLS/session context implementation; object storage provider/region; WhatsApp/payment providers; dental coding/notation standards; retention schedule; patient identity strategy; offline scope; and whether the shared staff/control products remain in this repository or become separate deployments.