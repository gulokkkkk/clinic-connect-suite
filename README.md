# Caddy Care

Caddy Care is a production-focused, multi-tenant SaaS for dental clinics in Pakistan. One control plane and one backend serve many clinics, while every clinic can have a completely independent public website, domain, brand, navigation, and patient experience.

This repository currently contains the **first dental-clinic demo frontend** and the product's implementation documentation. The production backend and shared staff portals described in the documents are the next build phases; they are not yet implemented here.

## Product boundaries

| Surface | Audience | Shared or clinic-specific? |
|---|---|---|
| Clinic public website | Patients and visitors | Separate design and deployment per clinic |
| Patient booking and account | Patients | Clinic-branded UI, shared backend contracts |
| Front desk and doctor workspace | Clinic staff | Shared product, always scoped to one clinic |
| Clinic owner workspace | Clinic owners | Shared product, always scoped to one clinic |
| Platform control center | Caddy Care operators | Shared internal product; cross-clinic access is privileged and audited |
| API, jobs, database, files | All products | One multi-tenant backend |

Separate clinic branding does **not** mean duplicated business logic. Appointment rules, records, payments, notifications, permissions, and audit behavior remain authoritative in the shared backend.

## Non-negotiable rules

- Every clinic-owned record carries a `clinic_id`; tenant context comes from trusted domain configuration and authenticated membership, never from a request body alone.
- Roles and memberships are separate from user profiles. Authorization is checked server-side for every protected action.
- Patient medical access follows least privilege and every sensitive read or write is auditable.
- Appointments, clinical records, and billing must keep working when AI is unavailable.
- Free-tier AI never receives patient names, phone numbers, CNICs, MRNs, addresses, or unredacted files.
- Financial and clinical mutations are transactional; retryable operations use idempotency keys.
- Oracle Free Tier is the pilot deployment target, not a guarantee of unlimited capacity or uptime.

## Documentation map

Read in this order:

1. [SaaS master plan](docs/SAAS_MASTER_PLAN.md) — product, customers, decisions, scope, and commercial direction.
2. [Architecture](docs/ARCHITECTURE.md) — system boundaries, tenancy, request flows, services, and deployments.
3. [Data model](docs/DATA_MODEL.md) — entities, ownership, integrity rules, indexes, and retention.
4. [API and events](docs/API_AND_EVENTS.md) — contracts, authentication, errors, idempotency, realtime, and webhooks.
5. [Security and privacy](docs/SECURITY_AND_PRIVACY.md) — access control, medical data, AI rules, audit, and incident response.
6. [System control](docs/SYSTEM_CONTROL.md) — platform operator controls, plans, limits, support access, and health.
7. [Operations](docs/OPERATIONS.md) — Oracle deployment, monitoring, backup/restore, scaling, and cost controls.
8. [Dental clinic demo](docs/DENTAL_CLINIC_DEMO.md) — the first clinic UI, realistic workflows, and completion checklist.
9. [Implementation roadmap](docs/IMPLEMENTATION_ROADMAP.md) — build sequence, tests, gates, and deferred work.

## Current technology

The demo is built with TanStack Start, React 19, TypeScript, Tailwind CSS v4, and Vite. The architecture documents define the intended production services; implementation must follow the repository's actual runtime rather than introducing a second web framework without an explicit architecture decision.

## Local development

```sh
bun install
bun run dev
```

The local app runs at `http://localhost:8080`. Never commit credentials. Public clinic identifiers and public API URLs may be exposed to a frontend; private service keys, AI keys, encryption keys, and privileged database credentials must only exist in secure server-side configuration.

## Delivery status

- Documentation blueprint: in progress in this phase.
- Dental clinic demo UI: partially built; complete against the demo specification next.
- Shared production backend, authentication, persistent records, staff portals, and control center: planned, not yet delivered.