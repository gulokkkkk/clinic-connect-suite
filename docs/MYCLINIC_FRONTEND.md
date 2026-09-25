# My Clinic Frontend — Dental Demo Product Specification

**Clinic:** Crescent & Pearl Dental (fictional demonstration clinic)  
**Market:** Independent urban dental clinic in Pakistan  
**Status:** UI demonstration scope; no real patient, clinician, contact, or payment data  
**Purpose:** Define every patient-facing and clinic-facing screen before implementation.

## 1. Product outcome

The demo must feel like a complete clinic experience rather than a generic healthcare landing page. A patient should be able to discover treatment, understand indicative pricing, choose a dentist and time, complete a booking, and check their private queue state. Staff should be able to demonstrate the same journey from arrival through examination, treatment planning, payment, and follow-up.

This frontend demonstrates one clinic's independent identity while following the shared Caddy Care contracts. It must not imply that planned persistence, authentication, messaging, payment, AI, or cross-device synchronization is live until those flows are connected and tested.

## 2. Clinic identity and design character

- **Positioning:** calm, precise, modern family dentistry with specialist access.
- **Personality:** reassuring rather than playful; premium but not intimidating; locally credible rather than imported or corporate.
- **Visual subject:** real dental care, healthy smiles, instruments and rooms where appropriate—not abstract software imagery.
- **Languages:** English first in this demo, with layouts and content structures ready for Urdu and Roman Urdu.
- **Motion:** purposeful transitions that explain progress. Teeth may animate subtly in the brand mark, service selection, booking progress, and odontogram. Motion must stop or simplify when reduced motion is requested.
- **Trust:** sterilization, clinician review, transparent estimates, privacy, and emergency guidance are visible without unsupported medical or compliance claims.

## 3. Public navigation

The public header contains the clinic identity, Services, Dentists, Pricing, About, Contact, Patient login, and a prominent Book appointment action. Mobile uses a compact menu with booking always reachable. An urgent-care strip may explain what to do for severe pain, swelling, bleeding, or trauma, but it must not diagnose.

Every navigation destination will be a real route when the full demo is implemented. The first implementation may present these as complete in-page demo sections only if links do not point to missing pages.

## 4. Homepage

### First view

- Clinic name as the primary heading.
- A direct promise focused on comfortable, transparent dental care.
- Book appointment and View services actions.
- A clinic/dentistry image with honest demo labelling.
- Today's status: demo opening state, next available appointment, and urgent-care guidance. Hours remain “demo schedule” until verified.
- Trust signals: itemized treatment plans, sterilization process, family appointments, and dentist-reviewed records.

### Service discovery

Service categories use recognizable dental language, not technical product language. Each item shows purpose, expected visit duration, indicative starting price, and whether consultation is required. Selecting a service opens details or starts booking with the service preselected.

### Dentist discovery

Each fictional clinician card includes a clear demo label, specialty, languages, focus treatments, next indicative availability, and a booking action. Credentials remain placeholders and are never presented as verified.

### Patient journey

A short visual sequence shows Book → Visit → Treatment plan → Follow-up. This is patient language; operational software details stay out of the public site.

### Clinic proof and safety

- Sterilization workflow in plain language.
- What to bring to the first appointment.
- Accessibility and family/guardian support.
- Accepted payment-method placeholders without claiming a live payment integration.
- Frequently asked questions with deterministic answers; an AI assistant is optional and clearly unavailable when its service is down.

### Final action and footer

Repeat booking and contact actions. Use `.example` contact details and label them as demonstration data until the clinic supplies verified phone, WhatsApp, address, hours, social links, and policies.

## 5. Services and indicative demo rates

These are fictional PKR values for interface design and sales walkthroughs. They are not an offer, quote, or medical recommendation. The production clinic owner must approve rates, tax handling, deposits, duration, and aftercare text before publication.

| Service | Demo duration | Indicative demo rate |
|---|---:|---:|
| Dental consultation | 30 min | PKR 1,500 |
| Scaling and polishing | 45–60 min | From PKR 5,500 |
| Tooth-coloured filling | 45 min | From PKR 4,500 |
| Simple extraction | 30–45 min | From PKR 4,500 |
| Surgical extraction | 60 min | From PKR 18,000 |
| Root canal treatment | 60–90 min/stage | From PKR 18,000 |
| Porcelain crown | 2 visits | From PKR 28,000 |
| Teeth whitening | 60–90 min | From PKR 24,000 |
| Braces consultation | 30 min | PKR 2,000 |
| Clear-aligner assessment | 45 min | PKR 3,000 |
| Child dental check-up | 30 min | PKR 1,200 |
| Dental emergency assessment | 30 min | PKR 2,000 |

Rates use “from” whenever final cost depends on examination, imaging, material, tooth position, complexity, or treatment stages. Booking never promises a diagnosis or final price.

## 6. Booking experience

The booking flow is a focused five-step sequence:

1. **Treatment:** choose service or “I am not sure”; emergency symptoms receive safe guidance.
2. **Dentist:** choose a suitable clinician or first available.
3. **Date and time:** show real availability when connected; demo slots are visibly synthetic. A visible hold timer appears only when the backend actually holds the slot.
4. **Patient details:** name, contact preference, new/returning patient, guardian/dependent relationship, brief non-diagnostic visit reason, and communication consent.
5. **Review:** clinic, service, dentist, date, indicative duration/rate, policy consent, then confirmation.

The UI covers loading, no slots, invalid details, duplicate booking warning, hold expiry, connection retry, success, and cancellation. Submission becomes idempotent when connected so repeated taps cannot create duplicate appointments.

## 7. Queue status

Patients enter a booking reference or secure link. The result displays only privacy-safe information: checked in, waiting, called, in treatment, completed, or delayed; token label; people ahead; and an ETA range. It never exposes patient names or a full clinic schedule. Staff priority changes require a recorded reason in the production workflow.

## 8. Patient area

- Upcoming appointment with reschedule/cancel policy.
- Appointment and procedure history.
- Approved treatment-plan stages and estimates.
- Signed prescriptions and post-operative instructions.
- Files shared with the patient.
- Itemized invoices, payments, refunds, and outstanding balance.
- Dependents/family relationships with consent boundaries.
- Reminder language and channel preferences.
- Account security and record-access history.

Clinical notes are not automatically patient-visible. Access is clinic-scoped and authenticated in production; the demo must never place records or roles in browser storage.

## 9. Reception workspace

The reception view prioritizes speed and scanning:

- Today's counts and schedule status.
- Patient search with minimal result details.
- Registration with duplicate warning and guardian support.
- Appointment calendar, check-in, walk-in, token issue, queue call/skip/reorder.
- Itemized estimate and invoice.
- Cash/card/bank payment placeholders, partial payment, receipt, refund request, and follow-up booking.
- Opening cash, movements, expected amount, counted amount, variance reason, and close confirmation.

Medical alerts may be visible when needed for safety; private narratives and signed-note editing are not available to reception.

## 10. Dentist workspace

- Today list, queue, and medical-alert markers.
- Longitudinal patient summary with allergies, conditions, medicines, previous dentistry, recent files, and secondary balance context.
- Interactive permanent-tooth odontogram with tooth and surface selection, finding state, legend text, and historical comparison.
- Examination findings, periodontal summary, imaging references, and structured notes.
- Staged treatment plan with priority, tooth/surface, procedure, indicative fee, approval, and progress.
- Procedure completion with materials, outcome, provider, and next step.
- Prescription draft requiring explicit dentist review and signature.
- Recall, follow-up, and post-operative instructions.
- Optional de-identified AI draft, visibly marked and never auto-signed.

Signed clinical entries are immutable; corrections become attributed addenda.

## 11. Clinic owner view

- Appointments, no-shows, collections, outstanding balances, average wait, and chair use.
- Staff memberships and role assignment separate from user identity.
- Services, rates, durations, deposits, schedules, closures, policies, and reminders.
- Revenue by service/provider and payment method; refund and cash-variance audit.
- Follow-ups due, review funnel, message delivery, storage use, plan limits, and AI-key health without exposing key values.
- Domains, clinic branding, exports, tenant audit, and visible support sessions.

Ownership does not automatically grant unrestricted editing of clinical records.

## 12. Animated dental interactions

- Brand tooth mark uses a gentle reveal on first load.
- Hero content enters in a short, ordered sequence without delaying actions.
- Service selection may highlight the relevant tooth region with a restrained pulse.
- Booking progress advances with directional motion and preserves entered values.
- Odontogram tooth/surface selection provides immediate scale, outline, and textual feedback.
- Queue movement animates position changes while preserving readable status text.
- Success states use a concise check/reveal—not confetti.
- All animation uses opacity and transform where possible, remains brief, and obeys `prefers-reduced-motion`.

## 13. Required UI states and accessibility

Every major surface includes populated, empty, loading, validation, permission-denied, not-found, offline/retrying, unavailable dependency, and success states. Focus is visible; controls are keyboard reachable; touch targets are at least 44px where practical; statuses use text and icons in addition to color; dialogs identify the affected person and action; destructive actions explain impact and request a reason.

Text must remain readable at 360px mobile, tablet, 1280px desktop, browser zoom, and in printable monochrome prescriptions/receipts. Urdu readiness requires layouts that can later support right-to-left direction without rebuilding the information architecture.

## 14. Data, security, and production boundaries

- All current UI data is synthetic and visibly marked as demo content.
- Every future clinic-owned record carries `clinic_id`, resolved from trusted domain and authenticated membership on the server—not a browser-supplied clinic selector.
- Roles live in membership/role records and authorization is server-enforced.
- Sensitive reads and all clinical, payment, export, permission, support, and deletion actions are auditable.
- Booking and payment writes are transactional and idempotent.
- Private keys, internal operator controls, medical records, and hidden IDs never ship in browser bundles.
- Core booking, queue, records, and payments continue when AI is unavailable.
- No production claim is made until the central flow is connected and tested end to end.

## 15. Implementation sequence

1. Deliver the clinic-branded public homepage with service rates, dentists, booking preview, trust, FAQ, contact placeholders, complete responsive states, and route metadata.
2. Add dedicated public service, dentist, booking, queue, about, contact, and patient routes without dead links.
3. Build reception, dentist, and owner demonstration workspaces with synthetic data.
4. Add the interactive odontogram and the nine end-to-end demo scenarios.
5. Connect authentication and persistence only after the backend foundation, tenant isolation, role permissions, audit, backup, and privacy gates are implemented.

## 16. Acceptance criteria

- The first screen unmistakably belongs to one dental clinic.
- Indicative PKR pricing is understandable and consistently labelled.
- Booking is reachable and usable on every supported screen size.
- Dental animation improves feedback without becoming decorative clutter.
- There are no missing destinations, fake live integrations, unsupported credentials, or real personal data.
- Clinical, financial, and role boundaries remain visible in the UI design.
- The frontend can later consume shared API contracts without sharing its presentation system with another clinic.