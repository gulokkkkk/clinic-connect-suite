# Security and Privacy

## 1. Security objectives

Caddy Care processes identity, medical, financial, and operational information. The minimum objectives are confidentiality between clinics, least-privilege access inside each clinic, integrity of clinical/financial history, recoverability, clear accountability, and safe handling of external services.

This document is an engineering baseline, not a claim of certification or legal compliance. Before real-patient onboarding, obtain Pakistani legal review for consent, retention, breach handling, cross-border processing, telemedicine, prescriptions, and applicable data-protection obligations.

## 2. Data classification

| Class | Examples | Baseline handling |
|---|---|---|
| Public | Clinic name, services, public hours | Publish only approved fields |
| Internal | Feature flags, operational metrics, runbooks | Authenticated staff/operator access |
| Confidential | Staff contacts, contracts, detailed finance | Role-limited, encrypted, audited changes |
| Medical/sensitive | Patient identity, history, notes, images, prescriptions | Strict need-to-know, encrypted, audited access/export |
| Secret | Password verifiers, private keys, AI keys, signing/encryption keys | Secret store only; never logged or returned |

## 3. Tenant isolation

Every clinic-owned table has `clinic_id`. Tenant context is server-derived, and every lookup combines resource ID with clinic ID. Database row-level security is defense in depth. Jobs, caches, object paths, exports, analytics, search, and realtime channels use the same boundary.

Required tests for each clinic-scoped capability: list, detail, create, update, delete/archive, search, export, file URL, realtime subscription, background job, and indirect relationship. Use two clinics with colliding human-readable values and assert no disclosure. Cross-tenant access returns neutral not-found behavior.

## 4. Authentication and sessions

- Verify email/phone before privileged use; verify clinic ownership operationally during the pilot.
- Require MFA for platform operators and clinic owners before real-patient launch; strongly recommend it for dentists/managers.
- Require step-up authentication for exports, role changes, support sessions, suspensions, key changes, refunds above threshold, and deletion.
- Short-lived access sessions with secure rotation/revocation; terminate sessions after password reset, account disable, role removal, or suspected compromise.
- Secure, HTTP-only cookies where possible; CSRF protection for cookie-authenticated mutations; strict origin checks.
- Rate-limit login and OTP attempts without revealing whether an identity exists.
- Never implement admin status in local storage or a client-only flag.

## 5. Authorization

Permissions are action-oriented, such as `patient.read_identity`, `clinical.read`, `clinical.write`, `prescription.sign`, `payment.refund`, `membership.manage`, and `audit.read`. Roles bundle permissions, but the server evaluates permission, active membership, clinic context, record state, and exceptional restrictions on every action.

Receptionists do not read private clinical notes. Managers do not automatically gain clinical access. Owners can administer the clinic but cannot silently alter signed records. Platform support has no standing medical access.

## 6. Support and impersonation

Prefer diagnostic metadata over impersonation. When support access is unavoidable:

1. Capture ticket, reason, requested scope, approver, and expiry.
2. Require operator MFA/step-up and clinic approval for medical visibility except a documented emergency process.
3. Show a persistent support-session banner and acting-as identity.
4. Default to read-only and block high-risk actions.
5. Record every read/action under both real operator and represented identity.
6. Expire automatically and notify the clinic with a session summary.

## 7. Clinical and financial integrity

Signed notes and issued prescriptions are immutable; corrections use linked addenda. Financial adjustments/refunds are append-only with reasons and approval thresholds. Sensitive concurrent edits use record versions. Server time and actor identity are authoritative.

Emergency priority changes, allergy override acknowledgements, deleted appointments, price overrides, and manual payment confirmations must be explicitly reasoned and audited.

## 8. Encryption and secrets

- TLS for all traffic; modern headers and HSTS after domain validation.
- Encrypted disks/object storage and encrypted off-site backups.
- Application-level encryption for clinic AI keys and other provider credentials; encryption keys live separately from ciphertext.
- Rotate secrets on schedule and immediately after suspected exposure. Track key version, not secret value.
- Private keys never enter browser bundles, analytics, logs, issue trackers, exports, or demo data.
- Signed file links are short-lived, permission-checked, and scoped to one object/action.

## 9. AI privacy and safety

Free-tier Gemini is limited to public clinic information, booking assistance, and de-identified text approved by policy. Redaction removes names, contacts, CNIC, MRN, addresses, exact identifiers, and unnecessary dates. Raw images/files and full records are forbidden.

Use-case allowlists, prompt-size limits, output filtering, per-clinic quotas, and human confirmation protect clinical workflows. AI output is labelled as assistance, not diagnosis. Emergency language directs users to local emergency care (for example Rescue 1122 where applicable) and the clinic's verified contact; it does not invent medical advice.

Log model, use case, token counts, outcome, safety category, latency, and request ID—not raw sensitive prompts by default. Clinics can disable AI independently.

## 10. Logging and audit

Application logs use structured fields: request ID, environment, service, safe actor ID, clinic ID, route/action, status, latency, and error category. Redact authorization headers, cookies, passwords, OTPs, API keys, patient narrative, phone numbers, CNIC, and signed URLs.

Audit events are immutable and include who, clinic, action, target, time, reason, result, support session, and safe change summary. Restrict audit access and alert on gaps, mass reads/exports, repeated denials, privilege changes, key changes, and unusual operator activity.

## 11. Consent and patient rights

Capture versioned consent for clinic care/data handling and separately for optional communications, marketing, AI-assisted features, and guardian access where applicable. Withdrawal stops future optional processing without rewriting legitimate historical clinical records.

Provide workflows for access/export, correction requests, communication preferences, and deletion requests subject to medical/legal retention. Verify requester identity and record fulfillment.

## 12. Secure development

- Review dependencies and lock versions; patch critical vulnerabilities promptly.
- Validate all inputs with schemas; encode outputs; use parameterized database access.
- Protect uploads against type confusion, oversize files, malware, path injection, and public ACLs.
- Apply CSP, clickjacking protection, secure referrer policy, and safe CORS allowlists.
- Verify webhook signatures and prevent replay; protect outbound URLs against SSRF.
- Separate environments and use synthetic data outside production.
- Require security review for auth, tenant context, exports, files, AI, billing, and platform controls.

## 13. Incident response

Severity is based on patient safety, cross-tenant exposure, data loss, financial integrity, and outage scope. On detection: preserve evidence, contain credentials/access, stop unsafe processing, assess affected clinics/data/time, restore safe service, notify leadership and clinics according to legal advice, document decisions, and complete remediation/post-incident review.

Never conceal a cross-tenant incident. Maintain current clinic contacts and an offline copy of the response checklist.

## 14. Launch gate

Before real patient data: independent tenant-isolation review, MFA for privileged users, restoration drill, vulnerability/dependency scan, log-redaction test, file-access test, expired-session test, permission matrix test, incident tabletop, signed clinic agreement/privacy terms, and documented retention decisions.