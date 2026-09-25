# Operations

## 1. Pilot deployment goal

Run 3 clinics inexpensively on Oracle Cloud Always Free while preserving data safety and an exit path. Free tier is suitable for a supervised pilot, not a contractual high-availability promise.

Target topology: one Ampere A1 VM (capacity permitting) with Caddy reverse proxy, application/API process, worker, PostgreSQL, and Redis in containers; private object storage for files/backups; clinic public sites on Cloudflare Pages; Cloudflare DNS/proxy; external uptime/error monitoring.

Do not expose PostgreSQL or Redis publicly. Restrict SSH, use key authentication, disable password/root login, apply automatic security updates carefully, and maintain a tested recovery access path.

## 2. Environments

| Environment | Data | Purpose |
|---|---|---|
| Development | Synthetic only | Local build and tests |
| Staging | Synthetic/de-identified fixtures | Migration, integration, and release checks |
| Production | Real clinic data | Approved releases only |

Each environment has separate database, storage namespace, domains, provider keys, signing/encryption material, and webhooks. Never copy production medical data into development or staging.

## 3. Container layout

- `proxy`: TLS termination, routing, safe headers, request limits.
- `app`: stateless web/API runtime with health/readiness endpoints.
- `worker`: background jobs and scheduler with controlled concurrency.
- `postgres`: persistent encrypted block volume, not image filesystem.
- `redis`: password/private network, persistence only where required; never source of truth.

Pin image versions and resource limits. Add health checks and restart policies. Keep migrations as an explicit release step, not automatic uncontrolled startup behavior.

## 4. Release process

1. CI installs locked dependencies and runs lint/type/build, unit, permission, tenant, contract, and migration tests.
2. Build immutable image tagged with commit/version; scan dependencies/image.
3. Deploy to staging, migrate, run smoke and workflow checks.
4. Confirm fresh backup and rollback compatibility.
5. Deploy production with health-gated replacement; run expand-only migrations first.
6. Verify booking, staff access, queue, record read, job dispatch, and control health.
7. Monitor elevated errors/latency and roll back application if needed. Database rollback uses prepared forward-fix or restore only when unavoidable.

Clinic frontend deployments are independent. API compatibility must cover every live frontend version during upgrades.

## 5. Backup strategy

### Schedule and copies

- Nightly encrypted logical PostgreSQL backup; retain 7 daily, 4 weekly, and 6 monthly during pilot, subject to policy/storage.
- Continuous or frequent WAL/archive capability when moving beyond pilot RPO.
- Object metadata inventory plus storage replication/versioning where available.
- One copy on Oracle object storage and one off-provider copy (Cloudflare R2 or Backblaze B2).
- Configuration, migration history, and secret *names/rotation procedure* backed up; secret values remain in secure recovery escrow, not ordinary archives.

Pilot targets: **RPO 24 hours**, **RTO 8 hours**. These must appear in clinic agreements as targets, not guarantees, until operational evidence supports an SLA.

### Monthly restore drill

Restore into an isolated network; verify checksum/decryption; apply migrations; run row counts and referential checks; test one synthetic clinic workflow; verify files referenced by sample rows; apply deletion tombstones; record duration, failures, owner, and evidence; then destroy the drill environment securely.

A backup that has not been restored is not considered proven.

## 6. Monitoring and logs

Collect uptime, request count/error/latency, database connections/size/slow queries, VM CPU/memory/disk, Redis memory/evictions, worker queue age/retries/dead letters, storage usage, backup age, TLS status, notification outcomes, AI health, and frontend error rate.

Use structured, redacted logs with request and clinic IDs. Start with UptimeRobot or Better Stack, Sentry, and a free metrics/log allowance where practical. Telegram may receive operational alerts but never patient or secret data.

See [System control](SYSTEM_CONTROL.md) for initial thresholds.

## 7. Incident operations

Maintain an on-call owner and backup contact. Severity:

- **SEV-1:** cross-tenant exposure, data loss, unsafe clinical integrity, total outage during clinic hours.
- **SEV-2:** one clinic/core workflow materially unavailable, payment integrity issue, backup failure beyond target.
- **SEV-3:** degraded optional integration, limited UI issue, delayed noncritical jobs.

Open an incident timeline, contain, communicate affected clinics with known facts, recover, monitor, and publish an internal review with corrective actions. Security incidents follow [Security and privacy](SECURITY_AND_PRIVACY.md).

## 8. Capacity and scaling triggers

Move from the all-in-one free VM before sustained risk, not after failure. Trigger review when any occurs:

- CPU above 70% or memory above 75% during clinic hours for a week.
- Database disk above 70%, backup exceeds window, or restore exceeds RTO.
- p95 core API latency above 1.5 seconds under normal load.
- Worker queue age repeatedly exceeds 5 minutes.
- More than 5 active clinics, a second large location, or contractual uptime requirements.
- Oracle capacity/reclamation risk becomes unacceptable.

First moves: upgrade to paid Oracle capacity or another provider; separate database with managed backups; separate worker; move Redis to managed service if needed; add application replicas; add point-in-time recovery. Architecture remains provider-portable through containers, standard PostgreSQL, and object-storage abstraction.

## 9. Cost controls and free resources

| Need | Pilot option | Limitation/action |
|---|---|---|
| Compute | Oracle Always Free ARM | Capacity may be unavailable/reclaimed; keep backups and redeploy scripts |
| Public frontends | Cloudflare Pages | Each clinic deployment remains independently owned/versioned |
| DNS/proxy | Cloudflare free | Do not rely on free plan as a compliance product |
| Object storage | Oracle allowance + R2/B2 second copy | Track storage/egress and lifecycle rules |
| Email | Brevo/Resend free allowance | Verify sender/domain; monitor quotas |
| Push | Firebase Cloud Messaging | Optional; no sensitive text in lock-screen payloads |
| WhatsApp | Meta Cloud API | Templates/conversations may cost; pass through and cap |
| Monitoring | Free Sentry/uptime/metrics tiers | Retention and volume limited; alerts must still work |
| CI/CD | GitHub Actions allowance | Protect deployment credentials and production approvals |

Budget domains, WhatsApp/SMS, payment fees, backup egress, legal review, and operator time. “Almost free” excludes support labor and business risk.

## 10. Maintenance

- Daily: alerts, backups, dead letters, disk, provider/key failures.
- Weekly: dependency notices, error trends, slow queries, capacity, failed messages, access anomalies.
- Monthly: restore drill, privileged-access review, clinic usage/limits, secret/key rotation due list, OS/container updates, cost report.
- Quarterly: incident tabletop, permission review, retention/deletion jobs, frontend/API version inventory, disaster recovery rehearsal.

Maintenance windows are announced to clinics. Urgent security patches may bypass normal notice with documented approval.

## 11. Pilot operations gate

Provisioning and recovery are scripted; production secrets are secure; ports are restricted; monitoring reaches two humans; backup has restored successfully; release and rollback have been rehearsed; clinic contacts and status page/channel exist; no production data appears in non-production; and capacity thresholds are visible in the control center.