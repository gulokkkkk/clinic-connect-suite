# Caddy Care — SaaS Master Plan (v2, replaces old IMPLEMENTATION_PLAN)

> Change from v1: **one shared backend for every clinic, but every clinic gets its own separate frontend (own UI, own design, own domain) — nothing visual is shared.** Hosting target: **Oracle Cloud Always Free**. AI: **Google Gemini, 3 API keys per clinic** (15+ keys total). First target: **3–5 clinics**, near-zero running cost.

---

## 1. The model in one picture

```text
  clinic-a.com          drsmile.pk            cityhospital.caddy.care      admin.caddy.care
  (Frontend A)          (Frontend B)          (Frontend C)                 (Super Admin app)
  own design            own design            own design                   (you only)
       \                    |                        /                          |
        \   HTTPS + X-Clinic-Key + JWT              /                           |
         v                  v                      v                            v
 +-------------------------------------------------------------------------------------+
 |  ONE backend  —  api.caddy.care  (Oracle ARM VM, Docker)                            |
 |  Caddy (TLS) -> Django 5 + DRF + Channels -> Celery workers                         |
 |  Tenant resolver: clinic key + Origin allow-list -> request.clinic                   |
 |  Gemini Key Pool: 3 keys per clinic, round-robin + failover + budget                |
 +-------------------------------------------------------------------------------------+
     PostgreSQL 16 (self-hosted, same VM)   Redis   Object storage (Oracle / R2)
```

- **Backend = the product.** All logic, data, AI, queue, billing live here once.
- **Frontend = the clinic's shop window.** Each clinic gets its own React app (separate repo/folder, separate deploy, separate domain). Different layout, fonts, colors, pages — you can design each one from scratch.
- Frontends only talk to the API. They never share code that affects looks. They *may* share one small, invisible **`@caddy/sdk`** package (API client, types, auth helpers) so each new frontend takes days, not weeks.

## 2. How any clinic registers (self-serve onboarding)

1. Clinic owner opens `caddy.care/register` (your marketing site) → name, city, phone, specialty, plan.
2. Backend creates: `Clinic` (status `trial`), owner user (`clinic_admin`), a **public clinic key** (`ck_live_...`), a 14-day trial.
3. Owner lands in the **Clinic Admin panel** (a generic panel hosted by you, used by all clinics — this is a tool, not their brand) to add doctors, timings, fees, staff, FAQ, and **paste their 3 Gemini keys** (or you add them).
4. You (or a template script) spin up their **own frontend**: `npx create-caddy-clinic drsmile` → copies a starter, sets `VITE_CLINIC_KEY`, `VITE_API_URL`. Designer customizes it. Deploy to Cloudflare Pages (free), connect their domain.
5. Backend `Clinic.allowed_origins` gets their domain → CORS + tenant lock active. Go live.

Result: registration is automatic; the custom look is the paid "setup" you sell.

## 3. Multi-tenant rules (must never break)

| Rule | How |
|---|---|
| Every tenant row has `clinic_id` | Base model `TenantModel`; DB index on `clinic_id` |
| Clinic comes from the request, never the body | `TenantMiddleware`: `X-Clinic-Key` header → clinic; must match `Origin` in `allowed_origins` |
| Staff can only act in their clinic | JWT contains `clinic_id` + `role`; checked against middleware clinic |
| Queries auto-filtered | `TenantScopedViewSet.get_queryset()` filters by `request.clinic` |
| Extra safety net | Postgres Row-Level Security with `SET app.clinic_id` per request (phase 2) |
| Tests | Every endpoint has an "other clinic gets 404" test |
| Patient across clinics | One `Patient` identity per phone, `PatientClinicLink` per clinic — Clinic A never sees Clinic B's visits |

## 4. Roles and what each panel shows

| Role | Panel | Sees / does |
|---|---|---|
| **Super Admin (you)** | `admin.caddy.care` | All clinics, plans, trials, invoices, key-pool health, AI usage per clinic, error logs, impersonate (audited), suspend clinic |
| **Clinic Admin / Owner** | Clinic Admin panel | Everything in *their* clinic: doctors, staff, schedules, fees, patients, full patient profiles, all visits, revenue, cash closing, no-shows, reviews, AI chat logs, Gemini key status, audit log, branding/FAQ |
| **Doctor** | Doctor console | Today's queue, call next, full patient profile + timeline, AI 3-line pre-visit summary, write Rx (voice or type), lab orders, follow-up, own earnings & stats |
| **Receptionist** | Front desk | Walk-in check-in, token printing, queue control, bookings, cash collection, reminders; limited medical view (no notes) |
| **Patient** | Clinic's own frontend | Book, live token/ETA, health vault, prescriptions PDF, lab reports, family members, chat with Caddy, reviews |

### In-depth patient profile (what doctor/admin see)
Demographics · MRN · family links · blood group · allergies (highlighted red) · chronic conditions · current medicines · vitals trend chart (BP, sugar, weight, temp) · visit timeline (complaint, diagnosis, notes, Rx) · lab reports with flagged values · attachments/images · follow-ups due · no-show count · payments/outstanding balance · AI summary · consent record · "who viewed this record" audit.

## 5. Gemini key pool (3 per clinic)

```text
ClinicAIKey: clinic_id, label, key_encrypted (Fernet), status (active|cooldown|dead),
             cooldown_until, requests_today, last_error, daily_budget
```
- Selector picks the clinic's next `active` key (round-robin). On `429` → mark `cooldown` (honor Retry-After, else 60 s) and try the next key. On `401/403` → mark `dead`, alert owner + you on Telegram.
- If all 3 are cooling down → friendly reply "Caddy is busy, please try in a minute" + booking still works via normal buttons. Never block core booking on AI.
- Per-clinic daily cap so one clinic cannot burn shared spare keys. Keep 3 **spare pool keys** (yours) as emergency fallback, billed to your account only if you allow it.
- Use cheaper **Flash / Flash-Lite** models for chat; cache FAQ answers in Redis; short system prompt.
- **Important — privacy and terms:**
  - On Gemini's **free tier Google may use prompts to improve its products.** Never send patient name, phone, CNIC or MRN — send de-identified text ("Patient, 45M, diabetic…"). For real medical summaries, move to a paid key later.
  - Best practice: each clinic creates its **own** Google account + 3 keys (clinic-owned). Creating many free accounts yourself just to multiply limits can break Google's terms and get keys banned.
- Keys stored encrypted; never sent to any frontend.

## 6. Running it (almost) free

| Need | Free choice |
|---|---|
| Server | **Oracle Always Free ARM (Ampere A1)** — up to 4 cores / 24 GB RAM in total. One VM runs everything for 3–5 clinics |
| Reverse proxy + HTTPS | Caddy server (auto Let's Encrypt) |
| DB | PostgreSQL 16 in Docker on the same VM, plus Oracle block volume |
| Cache / jobs | Redis in Docker + Celery |
| Backups | Nightly `pg_dump` → Oracle Object Storage (free 20 GB) **and** Cloudflare R2 / Backblaze B2 (second copy) |
| Files (labs, Rx PDFs) | Oracle Object Storage or Cloudflare R2 (10 GB free) |
| Clinic frontends | Cloudflare Pages (free, unlimited sites) |
| DNS / protection | Cloudflare free (hide VM IP, WAF, DDoS) |
| Email | Brevo (≈300/day free) or Resend free tier |
| Push notifications | Firebase Cloud Messaging (free) |
| WhatsApp | Meta WhatsApp Cloud API — replies to patient-started chats are low/no cost; templated reminders cost per message (pass cost to the clinic) |
| SMS | Avoid (paid in PK). Use WhatsApp + push first |
| Monitoring | UptimeRobot, Sentry free, Grafana Cloud free, Telegram bot for alerts |
| CI/CD | GitHub Actions → SSH deploy / Docker image on GHCR |

**Oracle warnings:** idle free VMs can be reclaimed — keep real traffic/cron running, or upgrade to Pay-As-You-Go (still free within limits, and protects the VM). ARM capacity in some regions is often "out of capacity" — retry or pick another region early. Always keep off-site backups.

Estimated monthly cost for 3–5 clinics: **Rs 0 – 3,000** (domain + optional WhatsApp messages).

## 7. Features real Pakistani clinics need (sell with these)

**Daily pains → your answer**
1. **Waiting-room crowd & token fights** → live token + "leave home now" WhatsApp alert + TV token board.
2. **Phone rings all day** → WhatsApp/web Caddy bot books in Urdu / Roman Urdu / English.
3. **No-shows** → reminder with Confirm/Cancel buttons, optional JazzCash/Easypaisa advance deposit.
4. **Cash leakage by staff** → every fee entered at check-in, daily cash closing report, owner sees it on phone.
5. **Load-shedding / internet down** → front desk works offline (PWA), syncs later; printable token slips.
6. **Paper prescriptions lost** → digital Rx PDF + QR, Urdu dosage instructions ("subah, dopahar, raat").
7. **Visiting doctors (doctor sits in 2–3 clinics)** → per-clinic schedules, doctor sees all their clinics.
8. **Follow-ups forgotten** → auto follow-up reminders, 1-tap rebook.
9. **Lab reports on WhatsApp chaos** → labs/staff upload into patient vault; flagged values.
10. **Family bookings on one number** → family accounts.
11. **Panel / corporate patients** → panel company field, monthly panel invoice.
12. **Doctor share / commission** → per-doctor revenue split report.
13. **Google reviews** → post-visit rating, happy patients redirected to Google.
14. **Pharmacy & stock (optional add-on)** → basic in-house dispensary inventory.
15. **Tax** → printable receipts with NTN/FBR-ready format (add-on).

## 8. How you sell it

- **Pitch (Urdu-friendly):** "Waiting room khali, phone band, har mareez ka record ek click pe. Aap ki clinic ki apni app — aap ke naam aur design ke saath."
- **Why they pick you over generic software:** their own branded app/website (not a shared portal), WhatsApp bot, Urdu, works in load-shedding, cheap.
- **Pricing (PKR):** Setup (custom frontend) Rs 25k–60k one time · Starter Rs 4,999/month per doctor · Clinic Rs 12,999/month (up to 5 doctors) · WhatsApp messages at cost. First 3 clinics: free setup in return for a case study + testimonial video.
- **Sales path:** 1) find 10 clinics near you (dentists, skin, gynae, child specialists book the most) · 2) walk in with a demo on your phone showing *their* name · 3) offer 30-day free pilot · 4) measure wait time and no-shows before/after · 5) turn numbers into a 1-page case study · 6) ask for referrals (doctors know doctors).
- **Legal basics:** Terms, Privacy Policy, Data Processing Agreement with each clinic, patient consent screen, "Not for emergencies — call 1122 / 115".

## 9. Implementation phases (1–2 devs, free stack)

| Phase | Weeks | Deliver |
|---|---|---|
| 0 — Infra | 1 | Oracle VM, Docker compose (Caddy, Django, Postgres, Redis, Celery), Cloudflare, backups, CI |
| 1 — Tenant core | 2–3 | Clinic registration, clinic keys, Origin lock, roles, JWT, phone OTP (WhatsApp OTP), audit log, super-admin app |
| 2 — Clinic ops | 4–6 | Doctors, schedules, slot engine, booking, queue + WebSocket, walk-ins, TV board, cash entry |
| 3 — Records | 6–8 | In-depth patient profile, visits, Rx PDF + QR, labs upload, vitals, family accounts |
| 4 — AI | 8–9 | Key pool, Caddy chat (web), safety filter, booking tools with confirmation, de-identified pre-visit summary |
| 5 — Frontend kit | 9–10 | `@caddy/sdk` + `create-caddy-clinic` starter; build clinic #1 custom frontend |
| 6 — Sell-ready | 10–12 | WhatsApp bot, reminders, analytics dashboards, cash closing, Urdu, billing/trials, offline PWA front desk |
| 7 — Pilot | 12+ | 3 pilot clinics, fix feedback, case studies, then clinics #4–5 |

## 10. Key new tables (add to DATA_MODEL.md)

- `Clinic` + `public_key`, `allowed_origins text[]`, `plan`, `trial_ends_at`, `status (trial|active|suspended)`, `frontend_url`
- `ClinicAIKey` (see §5), `AIUsage` (clinic, key, date, requests, tokens)
- `CashEntry` (appointment, amount, method cash|jazzcash|easypaisa|card, received_by), `CashClosing` (date, expected, counted, diff)
- `Vital` (patient, type, value, unit, taken_at)
- `PanelCompany`, `DoctorShare` (doctor, percent)
- `Review` (appointment, stars, text, sent_to_google)

## 11. Definition of Done
Tenant isolation tests pass · works on 360 px phone · offline-safe front desk · no secrets in frontend · backup restore tested monthly · every AI reply carries emergency disclaimer.
