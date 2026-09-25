# Data Model

## 1. Modeling rules

- UUID primary keys; UTC timestamps; explicit creation/update actor where relevant.
- Every clinic-owned table has non-null `clinic_id`, a foreign key, and an index beginning with `clinic_id`.
- Unique constraints include `clinic_id` unless uniqueness is truly global.
- Roles live in dedicated membership/role tables, never on user profiles.
- Money uses integer minor units plus ISO currency (`PKR` initially), never floating point.
- Clinical and financial history is append-oriented. Finalized records use corrections/addenda rather than destructive edits.
- Soft deletion is not universal: choose active/archive, legal hold, or hard deletion by record class and retention policy.
- Database constraints and row-level policies reinforce service authorization; application filtering alone is insufficient.

## 2. Identity, tenancy, and access

| Entity | Key fields and constraints |
|---|---|
| `users` | Identity subject, display name, verified contacts, status; contains no role |
| `clinics` | Legal/display name, status, timezone, locale, currency, trial/plan references |
| `clinic_domains` | Clinic, normalized host, type, verification state; host globally unique |
| `clinic_locations` | Clinic, name, address, timezone, contact, status |
| `clinic_memberships` | Clinic, user, status; unique `(clinic_id, user_id)` |
| `roles` | Platform-defined role code and scope (`clinic` or `platform`) |
| `membership_roles` | Membership, role; unique pair |
| `platform_memberships` | User and platform status; platform roles stored separately |
| `role_permissions` | Role-to-permission mapping; managed as controlled configuration |
| `support_sessions` | Operator, clinic, reason, approval, start/expiry, mode, termination |

Clinic status: `pending_verification`, `trial`, `active`, `past_due`, `suspended`, `offboarding`, `deleted`. Status transitions are validated and audited.

## 3. Clinic configuration

`clinic_settings`, `branding_configs`, `locations`, `chairs`, `service_catalog`, `service_prices`, `provider_services`, `business_hours`, `closures`, `provider_schedules`, `schedule_exceptions`, `notification_templates`, and `consent_templates`.

Version price and consent changes. An appointment/invoice stores the applied price snapshot; historical totals must not change when catalog prices change.

## 4. Patients and consent

| Entity | Purpose |
|---|---|
| `patients` | Clinic-scoped MRN, name, DOB, sex, contacts, address, preferred language, status |
| `patient_identifiers` | Normalized phone/email/optional identifiers with restricted access |
| `patient_relationships` | Guardian/dependent/family relationship and authority bounds |
| `patient_alerts` | Allergy, medical alert, infection-control notice, severity and verification |
| `conditions` / `medications` | Current/history state, source, verification time |
| `consents` | Template version, patient/guardian, scope, decision, capture method, timestamp, evidence |
| `patient_preferences` | Communication channels, language, marketing and reminder choices |
| `patient_merge_cases` | Proposed duplicate pair, evidence, reviewer, resolution; no automatic destructive merge |

Use a clinic-generated MRN unique within that clinic. Cross-clinic identity linkage, if introduced, must be invisible to clinics and must never expose another clinic relationship.

## 5. Scheduling and queue

`appointments` store patient, location, provider, service, start/end, status, source, notes category, cancellation reason, version, and optional parent appointment. Status transitions: `held`, `requested`, `confirmed`, `checked_in`, `in_service`, `completed`, `cancelled`, `no_show`.

`slot_holds` have an expiry and unique scheduling resource/window. `appointment_status_history` records transitions. `waitlist_entries` capture acceptable ranges. `queue_entries` store service date, token, check-in time, state, priority, priority reason, called/start/end times. Token uniqueness is per clinic/location/date.

Prevent overlap using transaction-safe constraints/locking across provider, chair, and clinic rules. Never rely only on a preflight availability response.

## 6. Dental clinical record

| Entity | Purpose |
|---|---|
| `encounters` | Appointment linkage, provider, complaint, history, examination, diagnosis, status/version |
| `odontogram_versions` | Immutable snapshot or version header for a point-in-time chart |
| `tooth_findings` | Tooth code, surface(s), finding code, severity, notes, source version |
| `periodontal_exams` | Structured measurements by tooth/site and exam date |
| `treatment_plans` | Status, estimate, patient approval, expiry, clinician and version |
| `treatment_plan_items` | Tooth/surface, procedure, priority, stage, price snapshot, assigned provider |
| `procedures` | Planned item reference, performed time, provider, outcome, materials, fee |
| `clinical_notes` | Type, content, author, draft/signed/addendum state |
| `prescriptions` / `prescription_items` | Prescriber, medicine, dose, route, frequency, duration, instructions |
| `follow_ups` | Due window, reason, state and source encounter/procedure |
| `vitals` | Type, numeric/text value, unit, measured time and recorder |
| `clinical_files` | Object reference, category, checksum, capture time, uploader, visibility |

Use a configured tooth notation system and preserve the original code. Surfaces are constrained values (`M`, `O/I`, `D`, `B/F`, `L/P`) rather than free-form text. Signed clinical notes are immutable; addenda reference the original.

## 7. Finance and inventory

`estimates`, `estimate_lines`, `invoices`, `invoice_lines`, `payments`, `payment_allocations`, `refunds`, `financial_adjustments`, `cash_sessions`, `cash_movements`, `cash_closings`, and `doctor_share_rules/entries` form an auditable ledger-like model.

- Invoice number unique per clinic and fiscal sequence.
- Payment provider reference unique per provider/account where present.
- Refund cannot exceed available paid amount.
- Adjustments require reason and permission; never edit a collected amount silently.
- Cash closing stores expected, counted, variance, attestation, and reviewer.

Optional pilot inventory uses `inventory_items`, `stock_lots`, and append-only `stock_movements`; stock on hand is derived/reconciled, not freely overwritten.

## 8. Communications, AI, and operations

| Entity | Purpose |
|---|---|
| `notification_jobs` | Recipient reference, template version, channel, schedule, status |
| `delivery_attempts` | Provider, external ID, status, timestamps, safe error category |
| `inbound_messages` | Verified provider event and clinic routing; restricted retention |
| `clinic_ai_keys` | Clinic, encrypted secret reference, label, state, cooldown, last success/error |
| `ai_usage_daily` | Clinic/key/model/use case/date, request/token counts, blocked/error totals |
| `feature_flags` | Global default and clinic override, rollout and expiry |
| `plan_entitlements` | Plan limits/features and effective version |
| `clinic_subscriptions` | Plan, period, state, manual/payment references |
| `audit_events` | Actor, clinic, action, target type/id, result, reason, safe change summary, request ID |
| `outbox_events` | Clinic, event type/version, aggregate, payload, delivery state |
| `export_jobs` | Clinic, requester, scope, status, expiry, checksum |
| `incidents` | Severity, scope, state, timeline, impact and resolution |

AI key ciphertext is never returned by application queries. Store key-encryption material outside the database and support re-encryption/key rotation.

## 9. Audit policy

Audit authentication changes, medical reads and writes, exports, print/download of sensitive files, permission changes, support sessions, impersonation, plan/limit changes, AI configuration, payments/refunds, clinic suspension, and deletion. Audit events are append-only and inaccessible to ordinary update/delete paths.

Avoid storing full medical text in audit diffs. Record field categories or carefully redacted before/after values. Retain pilot audit history for at least seven years unless legal review establishes a different rule.

## 10. Indexes and query shape

At minimum:

- `(clinic_id, status, created_at)` for operational lists.
- `(clinic_id, normalized_phone)` for patient lookup with restricted index access.
- `(clinic_id, location_id, starts_at, status)` and provider/time indexes for appointments.
- `(clinic_id, patient_id, occurred_at)` for encounters/procedures.
- `(clinic_id, service_date, state, token_number)` for queue.
- `(clinic_id, invoice_date, status)` and payment-date indexes for finance.
- `(clinic_id, created_at, action)` for audit.
- Partial indexes for active/pending rows where they materially reduce pilot queries.

Use cursor pagination for growing timelines. Query plans and slow-query evidence decide later indexes.

## 11. Retention and deletion

Define retention by data class before onboarding real patients. Clinic deletion is a staged workflow: suspend access, legal-hold check, export, revoke credentials/domains, expire files/keys, then delete or anonymize according to the signed agreement. Backups expire on their own schedule and are not selectively edited; restoration procedures reapply deletion tombstones before service use.

## 12. Migration and seed rules

Migrations are reviewed, reversible where practical, and deploy with explicit data access grants and tenant policies. Production migrations do not depend on app startup. Demo data is synthetic and visibly non-production; never seed real phone numbers, CNICs, or medical documents.