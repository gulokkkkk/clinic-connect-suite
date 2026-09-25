# Caddy Care — SaaS Master Plan

**Status:** approved product direction  
**Initial market:** independent dental clinics in Pakistan  
**Pilot target:** 3 clinics; architecture validated for 5 before scaling  
**Deployment goal:** near-zero infrastructure cost during a controlled pilot

## 1. Product promise

Caddy Care gives a clinic its own branded digital front door and a complete operating system behind it: appointments, live queue, patient records, dental charting, treatment plans, prescriptions, billing, reminders, analytics, and controlled AI assistance.

The product has one shared backend and database for operational efficiency. Each clinic can receive a separately designed and deployed public frontend. Patients see the clinic's identity; staff use reliable shared workflows; platform operators control the service without entering medical records by default.

## 2. Decisions that define the system

1. **Shared core, separate brands.** Business logic and data contracts are centralized. Public presentation is not.
2. **Tenant isolation from day one.** Every clinic-owned row, query, cache key, job, file path, realtime channel, export, and log context is scoped by clinic.
3. **Server authority.** The browser never decides clinic access, staff permissions, prices, payment completion, or record ownership.
4. **Dental-first workflow.** The first vertical includes odontograms, tooth surfaces, treatment plans, procedure stages, dental imaging references, and follow-ups.
5. **AI is optional assistance.** It can answer clinic FAQs, help navigate booking, draft de-identified summaries, and assist documentation. It cannot diagnose, prescribe, or block core operations.
6. **Operational control is a product.** Plans, limits, health, incidents, support sessions, exports, key status, suspensions, and audit history are controlled centrally.
7. **Free tier has an exit plan.** Backups are portable and load thresholds trigger paid capacity before clinic operations become unreliable.

## 3. System shape

```text
Clinic A domain      Clinic B domain      Clinic C domain
unique frontend      unique frontend      unique frontend
        \                 |                 /
         +---------- HTTPS API -----------+
                         |
              Tenant and auth boundary
                         |
         Shared application services and jobs
            |            |             |
        PostgreSQL      Redis       Object storage
                         |
               Platform control center
```

The current repository is the first dental demo frontend and documentation home. Production implementation should use the current TanStack Start runtime for the web/server layer unless an architecture decision explicitly approves a separate backend service. See [Architecture](ARCHITECTURE.md).

## 4. Users and workspaces

| Actor | Primary workspace | Core capabilities |
|---|---|---|
| Patient/guardian | Clinic-branded site and patient area | Book, reschedule, join queue, view own records/files, manage family, pay, consent |
| Receptionist | Front desk | Register patient, book, check in, queue, collect payment, print; no private clinical notes |
| Dental assistant | Clinical support | Prepare visit, record permitted observations, manage sterilization/tasks; restricted notes |
| Dentist | Doctor workspace | Patient timeline, odontogram, diagnosis, treatment plan, procedure notes, prescription, follow-up |
| Clinic manager | Operations | Schedules, staff, fees, services, inventory, cash close, reports; medical access only if granted |
| Clinic owner/admin | Clinic administration | Memberships, settings, billing, exports, audit, reports, AI settings; no silent record editing |
| Platform support | Control center | Service health and approved support sessions; no default clinical access |
| Platform owner | Control center | Clinics, plans, limits, incidents, platform roles, suspensions, global health and audits |

One person may hold different roles in different clinics. Permissions are membership-based, never a global role copied onto the user profile.

## 5. Clinic lifecycle

1. **Application:** owner supplies clinic identity, contact, city, specialty, expected doctors, and domain.
2. **Verification:** platform operator verifies the clinic and owner before activating real patient data.
3. **Provisioning:** create clinic, owner membership, plan limits, public configuration, file namespace, audit stream, and trial dates in one controlled workflow.
4. **Setup:** clinic configures locations, chairs, services, prices, doctors, schedules, policies, reminders, brand, and consent text.
5. **Frontend delivery:** build or customize that clinic's separate public experience against the stable API contract.
6. **Readiness review:** permissions, backups, test booking, cancellation, payment, queue, record, export, and restore evidence must pass.
7. **Trial and activation:** convert to paid manually during the pilot; record invoice and plan state.
8. **Suspension:** block new writes except payment/export/support operations; preserve records and patient safety access according to policy.
9. **Offboarding:** produce a clinic export, verify receipt, enforce retention/legal hold, revoke domains and keys, then schedule deletion.

Self-service registration may be added after the pilot. It must not create an unverified clinic capable of storing real medical data without operational checks.

## 6. Dental clinic workflows

### Appointment to checkout

1. Patient or receptionist selects location, service, dentist, date, and slot.
2. Server validates schedule, duration, buffers, closures, chair availability, and duplicate holds.
3. A short-lived slot hold prevents double booking; confirmation creates an appointment idempotently.
4. Reminders allow confirm, cancel, or reschedule. Deposits are optional and clinic-configured.
5. Check-in creates a queue entry. Staff can prioritize emergencies only with a recorded reason.
6. Dentist reviews allergies and history, records examination and odontogram findings, creates a treatment plan, and documents work performed.
7. Front desk generates an itemized invoice, records payment method and collector, and issues a receipt.
8. Follow-up, post-operative instructions, and review request are scheduled.

### Longitudinal patient profile

Identity and contacts; guardians/family; consent; allergies; conditions; medicines; medical alerts; vitals; appointment and queue history; encounter timeline; odontogram versions; periodontal findings; treatment plans and approvals; procedures; prescriptions; imaging/file references; balances/refunds; reminders; and a record-access audit.

Clinic A may not see Clinic B's clinical relationship with the same person. Identity matching can reduce duplicates internally, but clinic-visible records remain isolated.

## 7. Platform control requirements

The control center must provide clinic lifecycle state, plans and limits, feature flags, domains, deployment versions, uptime, job queues, backup status, storage growth, notification delivery, AI usage/key health, security events, incidents, and immutable operator actions.

Support access is time-boxed, reason-bound, approved where required, visually obvious, read-only by default, and audited. Suspension, export, deletion, role changes, and impersonation require step-up authentication. Full detail is in [System control](SYSTEM_CONTROL.md).

## 8. AI and Gemini policy

Each clinic may supply up to three clinic-owned Gemini API keys. Keys are encrypted server-side and selected through a per-clinic pool with health, cooldown, quota, and revocation state. Creating many accounts merely to evade provider limits is not part of the product strategy.

- Use low-cost models for public FAQ and booking guidance.
- Redact direct identifiers before any free-tier request; reject content that cannot be made safe.
- Never upload patient scans, photos, prescriptions, or raw files to free-tier AI.
- Do not store model prompts/responses by default; retain only safe operational metadata unless the clinic enables a reviewed feature.
- Require human confirmation before an AI draft enters a clinical record.
- If all keys fail, switch off AI for that clinic and keep deterministic workflows available.
- Alert on quota exhaustion, authentication failure, unusual usage, or repeated safety blocks.

See [Security and privacy](SECURITY_AND_PRIVACY.md) and [System control](SYSTEM_CONTROL.md).

## 9. Pakistan-focused value

| Clinic pain | Product response |
|---|---|
| Crowded waiting room and token disputes | Live queue, transparent token state, estimated turn, TV board |
| Calls interrupting staff | Web/WhatsApp booking with Urdu, Roman Urdu, and English content |
| No-shows | Confirm/cancel reminders, waitlist, optional deposit |
| Cash leakage | Itemized invoices, collector identity, immutable adjustments, daily cash close |
| Load-shedding and unstable internet | Resilient front desk states, printable lists/tokens, safe retry and reconciliation |
| Paper dental history | Odontogram and treatment timeline with file attachments |
| Complex staged treatments | Treatment plan stages, estimates, approvals, installments, completed work |
| Family members on one phone | Guardian and dependent relationships with consent boundaries |
| Forgotten follow-ups | Recall rules for cleaning, braces, root canal stages, and post-operative checks |
| Doctor revenue disputes | Procedure attribution and configurable share reports with adjustment audit |
| Reports scattered in chat | Controlled patient vault and clinic upload workflow |

WhatsApp templates and SMS are not free. Their actual provider cost must be passed through or limited by plan.

## 10. Commercial model

Sell an operational result, not a generic website: fewer calls, shorter perceived waits, visible cash, better follow-up, and one patient history.

**Pilot offer:** use a clinic-branded demo, run a 30-day controlled pilot, measure baseline versus outcome, and secure a testimonial/case study. Do not promise legal compliance, unlimited uptime, or fully offline medical editing until verified.

**Starting price hypothesis, to validate:** one-time design/onboarding fee of PKR 25,000–60,000; monthly clinic fee based on locations, doctors, storage, reminders, and support. Third-party messaging, domains, payment fees, and excess storage are pass-through costs. Final prices should follow interviews with at least 10 clinics and observed support load.

## 11. Reliability and success targets

- No confirmed double bookings under concurrent requests.
- No cross-clinic record disclosure in automated and manual security tests.
- Daily encrypted backup with a tested monthly restore; pilot target RPO 24 hours and RTO 8 hours.
- Core clinic screens remain usable when Gemini, email, or WhatsApp is unavailable.
- Every payment adjustment and clinical change identifies actor, time, clinic, reason, and before/after state where safe.
- First clinic staff can complete registration-to-checkout without developer assistance.
- Owners can reconcile daily collections against invoices and payment entries.

## 12. Delivery and scope control

Build the first dental demo before production persistence, then follow [Implementation roadmap](IMPLEMENTATION_ROADMAP.md). Pilot scope includes one location per clinic initially, core dental records, booking/queue, billing, reminders, reports, and operator controls.

Defer pharmacy, insurance/panel automation, full accounting, native mobile apps, autonomous diagnosis, complex offline conflict resolution, and multi-region high availability until the pilot validates demand.

## 13. Definition of production-ready pilot

The pilot is ready only when tenant-isolation tests, permission tests, booking concurrency tests, audit coverage, backup restore, role-specific usability, incident contacts, legal documents, clinic export, AI fallback, and rollback procedures have passed. A polished demo alone is not a production backend.