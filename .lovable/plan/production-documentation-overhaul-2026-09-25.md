# Production Documentation Overhaul

## Goal
Turn the repository documentation into one coherent, implementation-ready blueprint for Caddy Care: a shared-backend, multi-tenant dental-clinic SaaS with separately branded clinic frontends, initially serving 3–5 clinics in Pakistan.

## Documentation changes

1. **Rewrite the project README**
   - Explain the product, repository role, system boundaries, target users, architecture summary, document map, implementation order, and local development.
   - Clearly distinguish the reusable clinic-management platform, shared staff portals, and each clinic’s independent public website/UI.

2. **Expand the master plan**
   - Replace high-level ideas with firm product decisions, tenant lifecycle, roles and permissions, clinic onboarding, dental workflows, system-control requirements, AI governance, reliability targets, pricing direction, delivery phases, and acceptance gates.
   - Keep the Oracle Free Tier and clinic-owned Gemini-key strategy, while documenting limits, graceful fallback, privacy, and a migration path when the free tier is no longer sufficient.

3. **Add implementation-grade technical specifications**
   - `ARCHITECTURE.md`: services, request flow, tenant resolution, frontend separation, background jobs, file storage, realtime events, and deployment topology.
   - `DATA_MODEL.md`: clinics, memberships and roles, patients, dental charting, appointments, queue, encounters, prescriptions, payments, files, AI usage, notifications, audit events, constraints, indexes, retention, and tenant-scoping rules.
   - `API_AND_EVENTS.md`: API conventions, authentication, idempotency, errors, pagination, key resource contracts, webhooks, and realtime events.
   - `SECURITY_AND_PRIVACY.md`: least privilege, tenant isolation, medical-data access, encryption, consent, auditability, session security, incident response, backups, and AI de-identification.
   - `SYSTEM_CONTROL.md`: super-admin control plane, clinic status changes, plans/limits, feature flags, support access, health monitoring, key-pool controls, abuse prevention, and safe impersonation.
   - `OPERATIONS.md`: Oracle deployment, environments, CI/CD, monitoring, backups and restore drills, disaster recovery, capacity thresholds, maintenance, and near-zero-cost resource choices.
   - `DENTAL_CLINIC_DEMO.md`: the first demo’s pages, role-based dashboards, dental odontogram and treatment plan, booking/queue, patient profile, billing, realistic demo data, and UI completion checklist.
   - `IMPLEMENTATION_ROADMAP.md`: sequenced milestones, dependencies, tests, exit criteria, and explicit deferred features.

4. **Align repository guidance**
   - Extend `AGENTS.md` without removing its managed Lovable block, adding project invariants future contributors must follow.
   - Update the routes guide to define the intended public clinic demo, staff/admin areas, naming conventions, access boundaries, metadata requirements, and loading/error behavior.

## Non-negotiable design decisions

- One backend and database, with strict clinic-level isolation on every clinic-owned record and request.
- Clinic identity is resolved from trusted domain/configuration and authenticated membership, never accepted blindly from request bodies.
- Roles are modeled separately from users; authorization is enforced server-side and audited.
- Public clinic frontends can be completely different designs; shared code is limited to invisible contracts, SDK utilities, and staff tooling.
- Clinical actions remain usable when AI is unavailable; AI never diagnoses, prescribes, or exposes identifiable patient data to free-tier models.
- Money, appointments, stock, and clinical changes use transactional writes, immutable audit history, and idempotency where retries are possible.
- Free-tier infrastructure is a launch strategy, not an availability promise; documented thresholds trigger scaling or paid services.

## Validation

- Cross-check every Markdown file for contradictory terminology, architecture, role permissions, phases, and feature scope.
- Verify all internal document links and the final documentation inventory.
- Confirm the application build remains healthy because this phase changes documentation only.
