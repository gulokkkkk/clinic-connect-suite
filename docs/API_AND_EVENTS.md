# API and Events

## 1. Contract goals

The shared API lets independently deployed clinic frontends evolve without duplicating business rules. Contracts are HTTPS-only, versioned, schema-validated, tenant-safe, observable, and compatible across a documented support window.

Initial base: `/api/v1`. Publish an OpenAPI document and generate browser-safe types/SDK helpers. Additive fields are allowed; removals or semantic changes require a deprecation window or new major version.

## 2. Authentication and clinic context

- Public catalog and availability endpoints resolve clinic from the verified host/domain context or a server-issued public clinic reference.
- Protected endpoints validate the session and active membership server-side.
- Platform endpoints require a separate platform role and step-up authentication for high-risk actions.
- The server ignores body/query `clinic_id` for authorization. If the API uses an explicit clinic selector, it must be one selected from server-returned memberships and revalidated on every call.

Never place access tokens in URLs. Cookies are secure, HTTP-only, same-site where architecture permits, and protected against CSRF. Bearer tokens are short-lived and verified for issuer, audience, expiry, and revocation/session state.

## 3. Request conventions

- JSON uses camelCase externally and consistent ISO 8601 UTC timestamps.
- IDs are opaque UUIDs; patients see human references such as MRN or booking code only when needed.
- Mutating retryable requests accept `Idempotency-Key`; scope uniqueness by clinic, actor/consumer, endpoint, and key.
- Optimistic concurrency uses a `version` or `If-Match` value for records where lost updates matter.
- List endpoints use cursor pagination: `data`, `page.nextCursor`, `page.hasMore`.
- Search input is length-limited and normalized. Medical and financial exports are asynchronous.
- Every response includes or echoes a safe request ID for support.

## 4. Error envelope

```json
{
  "error": {
    "code": "APPOINTMENT_SLOT_UNAVAILABLE",
    "message": "That time is no longer available.",
    "requestId": "req_...",
    "fields": [{ "field": "startsAt", "code": "unavailable" }]
  }
}
```

Use stable machine codes and safe user messages. Do not expose stack traces, SQL, provider secrets, existence of another clinic's resource, or raw provider responses.

Standard statuses: `400` validation, `401` authentication, `403` known but forbidden action, `404` missing or cross-tenant resource, `409` state/version conflict, `422` valid shape but invalid workflow, `429` rate limit, `503` temporary dependency outage.

## 5. Resource groups

### Public clinic

- `GET /clinic` — publishable clinic identity, theme primitives, locations and contact.
- `GET /services`, `GET /providers` — active public catalog only.
- `GET /availability` — server-calculated slots with short cache lifetime.
- `POST /booking-holds` — limited hold with expiry and abuse controls.
- `POST /appointments` — confirm using hold or atomic revalidation; idempotent.

Public endpoints return no staff contact details, internal prices, AI keys, patient counts, or operational health.

### Patients

`/me`, `/patients`, `/appointments`, `/queue`, `/encounters`, `/treatment-plans`, `/prescriptions`, `/files`, `/invoices`, `/payments`, and `/consents`. Guardians only access dependents within an active, consented relationship. Patient responses use patient-safe fields and exclude private staff notes.

### Clinic staff

`/staff/dashboard`, `/patients`, `/appointments`, `/queue`, `/clinical`, `/billing`, `/reports`, `/settings`, `/memberships`, and `/audit`. Endpoint policies enforce receptionist, assistant, dentist, manager, and owner differences. UI hiding is not authorization.

### Platform control

`/platform/clinics`, `/plans`, `/entitlements`, `/health`, `/incidents`, `/support-sessions`, `/exports`, `/feature-flags`, and `/audit`. Platform endpoints never share a generic clinic-admin permission.

## 6. State-transition APIs

Prefer commands over arbitrary status edits:

- `POST /appointments/{id}/confirm|cancel|check-in|start|complete|mark-no-show`
- `POST /queue/{id}/call|skip|start|complete`
- `POST /clinical-notes/{id}/sign` and `/addenda`
- `POST /treatment-plans/{id}/approve|decline`
- `POST /invoices/{id}/issue|void`
- `POST /payments/{id}/refund`
- `POST /platform/clinics/{id}/suspend|reactivate|begin-offboarding`

Each command validates allowed prior state, permission, version, reason where required, and writes audit/outbox data transactionally.

## 7. Realtime events

Event names are versioned past tense, for example:

- `queue.entry.checked_in.v1`
- `queue.entry.called.v1`
- `appointment.confirmed.v1`
- `appointment.cancelled.v1`
- `treatment_plan.approved.v1`
- `notification.delivery_updated.v1`
- `clinic.feature_changed.v1`

Envelope: event ID, type, version, clinic ID (server/internal), aggregate type/ID, occurred time, sequence, and minimal payload. Clients receiving a sequence gap refetch. Realtime is an optimization; authoritative reads still work without it.

## 8. Webhooks

External callbacks live under a public API path but authenticate themselves through provider signatures or a strong shared secret. Verify signature against the raw body before parsing. Enforce timestamp/replay windows, persist provider event IDs uniquely, acknowledge only after safe persistence, and process asynchronously.

Outbound clinic webhooks, if introduced, are HTTPS-only, signed, retried with backoff, redacted, and disabled after repeated failure. Never allow private-network destinations without SSRF protection.

## 9. Rate limits and abuse controls

Apply limits by IP/device for public traffic, by session/user for authenticated traffic, and by clinic/feature for costly operations. Protect OTP, booking holds, search, file signing, exports, AI, and login more aggressively. Rate-limit messages are generic and include a retry window where safe.

## 10. SDK boundary

The optional `@caddy/sdk` includes generated types, API client, auth/session helpers, idempotency helpers, error mapping, and event subscriptions. It contains no clinic theme, layouts, content, or secret. Frontend versions report themselves in a safe header so deprecations can be measured.

## 11. Contract acceptance

Contract tests cover every role, cross-clinic IDs, invalid transitions, concurrent booking, duplicate retries, pagination stability, redacted errors, expired sessions, webhook replay, and dependency failure. A clinic frontend is not released against an undocumented endpoint.