<!-- LOVABLE:BEGIN -->
> [!IMPORTANT]
> This project is connected to [Lovable](https://lovable.dev). Avoid rewriting
> published git history — force pushing, or rebasing/amending/squashing commits
> that are already pushed — as it rewrites history on Lovable's side and the
> user will likely lose their project history.
>
> Commits you push to the connected branch sync back to Lovable and show up in
> the editor, so keep the branch in a working state.
<!-- LOVABLE:END -->

# Caddy Care project invariants

- Read `README.md` and the relevant document under `docs/` before changing product behavior.
- This is one shared-backend, multi-tenant dental SaaS. Never implement clinic-owned data without an explicit `clinic_id` and server-enforced tenant scope.
- Resolve tenant context from trusted host/domain configuration and authenticated clinic membership. Never trust a client-supplied clinic ID by itself.
- Store roles in a dedicated membership/role model, not on user or patient profiles. Enforce authorization on the server.
- Public clinic frontends may have entirely independent visual systems. Do not force shared presentation components across clinics.
- Keep clinical, booking, queue, and payment workflows functional without AI. Do not allow AI to diagnose or autonomously prescribe.
- Do not expose private keys, medical records, internal IDs, or operator controls in browser bundles.
- Make sensitive reads and all clinical, financial, permission, export, impersonation, and deletion actions auditable.
- Add cross-tenant negative tests for every clinic-scoped endpoint or server function.
- Use transactional writes and idempotency for payments, booking confirmation, notification dispatch, and other retryable operations.
- Treat Oracle Free Tier as pilot infrastructure; preserve portable backups and the scaling thresholds in `docs/OPERATIONS.md`.
- Do not claim a planned backend capability is implemented until its central user flow is verified end to end.
