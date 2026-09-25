# Application routes

TanStack Start uses file-based routing. Every route file must map exactly to its `createFileRoute()` ID, and `src/routeTree.gen.ts` is generated automatically.

## Conventions

| File | URL |
|---|---|
| `index.tsx` | `/` |
| `services.tsx` | `/services` |
| `patients.$patientId.tsx` | `/patients/:patientId` |
| `_authenticated.dashboard.tsx` | `/dashboard`, behind the pathless authenticated layout |
| `__root.tsx` | Root document and shared providers; must preserve `<Outlet />` |

- Use dots consistently for nested route files in this repository.
- Use `$patientId`, not `:patientId`, for dynamic segments.
- Major public pages such as services, doctors, booking, about, and contact receive separate routes and unique metadata.
- Every data loader has a visible error state and not-found state. Data-backed screens use route/query loading rather than effect-driven initial fetches.
- Never edit `routeTree.gen.ts` manually.

## Product route boundaries

The first implementation is a dental-clinic demo. Keep these surfaces explicit:

```text
Public clinic site:  /, /services, /doctors, /book, /about, /contact
Patient area:        /patient/*       (authenticated, clinic-branded)
Clinic staff:        /app/*           (authenticated and clinic-scoped)
Platform control:    /control/*       (platform operators only)
```

Only create a route when its screen is implemented; every navigation target must exist in the same change. Protected clinic routes must use the authenticated route boundary, and platform-control routes require a separate server-verified platform role—not merely clinic-admin access.

## Tenant-aware rendering

- Public clinic context is resolved from a trusted hostname mapping. A clinic slug or public key may locate a clinic but grants no protected access.
- Staff context is the intersection of the resolved clinic and the signed-in user's active membership.
- A mismatched host and membership fails closed with a neutral not-found or access-denied screen; never fall back to another clinic.
- Clinic branding loads before the main screen where possible to avoid showing one clinic's theme on another clinic's domain.
- Route code never sends private AI keys, medical notes, or platform-only health information to the browser.

See [Architecture](../../docs/ARCHITECTURE.md), [API and events](../../docs/API_AND_EVENTS.md), and [Security and privacy](../../docs/SECURITY_AND_PRIVACY.md).