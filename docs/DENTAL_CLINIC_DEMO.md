# Dental Clinic Demo Specification

## 1. Demo objective

Build a believable, polished dental-clinic experience that can be shown on a phone, tablet, reception desktop, and dentist screen. The demo should prove the whole patient journey and staff value—not merely a marketing homepage.

Use a fictional clinic and synthetic patients. Do not invent a real clinic's legal details, prices, doctor credentials, phone number, or address. Mark demo-only operational data clearly in staff views.

## 2. Visual direction

The clinic public site should feel calm, hygienic, modern, and locally credible without looking like generic SaaS. Use authentic dental imagery, clear service explanations, visible clinic identity, strong Urdu/Roman Urdu readiness, accessible contrast, and restrained motion. Avoid sharing its visual system with future clinics; they may look completely different.

Staff screens favor dense, fast scanning: neutral surfaces, high-contrast statuses, stable tables/timelines, obvious allergies/medical alerts, keyboard-friendly actions, and no decorative clutter. Clinical and financial information must never overlap or truncate dangerously.

## 3. Public patient pages

| Page | Must include |
|---|---|
| Home | Clinic identity first, urgent contact, book action, services, dentists, trust evidence, hours/location glimpse |
| Services | Dental services with duration/starting-price placeholder only when verified, preparation and aftercare |
| Dentists | Credentials placeholders, languages, services, availability entry point |
| Booking | Location/service/dentist/date/slot, patient details, consent, confirmation; visible hold expiry |
| Queue status | Booking/token reference, state, people ahead/ETA range, privacy-safe display |
| About | Clinic story, hygiene/sterilization approach, accessibility and languages |
| Contact | Verified phone/WhatsApp/address/hours, map only when details are real |
| Patient area | Upcoming/history, treatment plans, prescriptions, files, invoices, family, preferences |

Every content page gets its own route and metadata. Core booking does not require AI.

## 4. Reception workspace

- Today overview: confirmed, arrived, waiting, in chair, completed, no-show.
- Fast patient search by MRN/name/phone with privacy-aware results.
- New patient/guardian registration and duplicate warning.
- Appointment calendar and slot creation with service duration/chair constraints.
- Walk-in check-in, token print, queue reorder requiring reason, call/skip/complete.
- Itemized estimate/invoice, payment collection, receipt, refund request, outstanding balances.
- Daily cash session: opening amount, cash movements, expected versus counted, variance reason, close.
- Resilient pending states and safe retry when connection is unstable.

Reception cannot see private clinical narratives or edit signed records.

## 5. Dentist workspace

- Today list and queue with medical-alert indicators.
- Patient summary: allergies, conditions, medicines, prior dental work, recent files, balance as secondary context.
- Interactive odontogram using adult/permanent teeth first; status by tooth and surface; history comparison.
- Examination findings, diagnosis codes/labels, periodontal summary, clinical notes, and imaging attachments.
- Treatment plan with stages, priorities, tooth/surface, procedure, fee estimate, approval, and progress.
- Procedure completion with materials, outcome, next step, and provider attribution.
- Prescription draft with dosage/instructions and explicit dentist signing.
- Follow-up/recall and post-operative instructions.
- AI-generated de-identified draft summary only, visibly reviewable and never auto-signed.

## 6. Clinic owner/manager workspace

- Today and date-range KPIs: appointments, no-shows, collections, outstanding, average wait, chair utilization.
- Doctors, assistants, receptionists, memberships, roles, locations/chairs, schedules/closures.
- Services, prices, durations, deposits, cancellation and reminder settings.
- Revenue by service/provider, payment method, refunds/adjustments, cash-close variance.
- Recall/follow-up due, review funnel, notification delivery, storage and plan usage.
- Tenant-only audit view, exports, AI key status, domains/branding, and support sessions.

Medical access is permission-based; “owner” does not silently imply unrestricted clinical editing.

## 7. Platform-control demo

Include three fictional clinics to demonstrate tenancy. Show portfolio health, clinic status/plan/limits, domain/frontend version, backup freshness, job failures, notification health, AI key states, feature flags, incidents, and audited support-session creation. Never show actual key values or patient records in the portfolio.

## 8. Required demo scenarios

1. **New patient booking:** select cleaning, dentist, slot; create confirmation without double booking.
2. **Walk-in:** receptionist registers dependent, checks in, issues token, updates queue.
3. **Dental visit:** dentist opens alerts, records tooth finding, creates two-stage treatment plan, signs note.
4. **Checkout:** receptionist issues itemized invoice, records partial cash payment, prints receipt, books follow-up.
5. **Owner close:** owner reviews collections and confirms cash variance with reason.
6. **No-show recovery:** reminder outcome, no-show state, waitlist fill, rebooking.
7. **AI outage:** FAQ assistant becomes unavailable while booking and records continue normally.
8. **Tenant protection:** Clinic A staff cannot open Clinic B patient ID, file, event, or export.
9. **Support:** operator starts time-limited read-only session with reason; clinic sees the audit.

## 9. Synthetic demo dataset

Create 3 clinics with distinct names/themes; 2 locations only for demonstrating architecture, while pilot feature scope remains one primary location per clinic. Include 6 dentists across general dentistry, orthodontics, endodontics, pediatric dentistry, and oral surgery; assistants/receptionists/owners; at least 24 synthetic patients with family relationships; 2 weeks of appointments; queue states; odontogram findings; staged plans; invoices/payments; no-shows; follow-ups; and safe audit events.

Use obvious fictional contacts such as reserved/example domains. Keep conditions medically plausible but never use real people's data.

## 10. UI state checklist

Every major screen needs populated, empty, loading, validation, permission-denied, not-found, offline/retrying, dependency-unavailable, and success states. Tables and boards keep stable dimensions. Mobile actions remain reachable; text never overlaps; dialogs identify the affected patient/action; destructive actions explain impact and require reason where appropriate.

Accessibility: keyboard navigation, visible focus, semantic labels, 44px touch targets where practical, meaningful status text beyond color, reduced-motion support, and printable prescriptions/receipts that remain legible in monochrome.

## 11. Demo acceptance

- All nine scenarios work end to end with synthetic data.
- Public pages feel like one real dental clinic, not a dashboard template.
- Staff can complete common actions at 1280px and tablet widths; booking works at 360px.
- Odontogram is understandable without color alone and preserves history.
- Role restrictions and tenant failures are visible and tested.
- No fake contact/credential is presented as real; no private key exists in browser code.
- The experience is ready for a clinic walkthrough and feedback session, but labelled demo until production gates pass.