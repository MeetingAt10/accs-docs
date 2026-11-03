# Delta: module sites on separate hosts, and the shell redesign

**Status:** in progress, 2026-11-01
**Superseded in part:** the site switcher row, site hero, Home site directory, and demo-account sign-in below are replaced by [`2026-11-01-real-auth-and-visual-refactor.md`](2026-11-01-real-auth-and-visual-refactor.md) (site rail, no illustration art, real sign-in and registration, role-gated sites).
**Authoritative source:** `DOC003_MODULE-INVENTORY.xlsm`, Module Inventory sheet (58 modules)

## Existing behavior touched

- [`2026-10-27-shared-navigation.md`](2026-10-27-shared-navigation.md) groups routes into four areas (Home, Community, Schedule, Workspace) inside one host. Workspace mixes reports, cases, evaluations, records, and administration.
- [`../sitemap.md` §4](../sitemap.md#4-primary-navigation-and-contextual-sidebar) and [`../pages/01-shell-and-auth.md`](../pages/01-shell-and-auth.md) describe that shell.
- `architecture/` hosts the app on one CloudFront distribution (`AppSite`), and the API accepts one CORS origin.

## Requested change

Split the signed-in app into seven **module sites**. Each site has its own host, landing page, contextual navigation, hero art, and lane color. The four areas become seven sites, and Workspace is split by module family. Routes, permissions, page IDs, and API checks do not change. A site is a navigation and presentation boundary, not an authorization boundary.

| Site | Host label | Routes | DOC003 modules |
|---|---|---|---|
| Home | `portal` | `/home`, `/notifications`, `/assistant`, `/help`, `/settings/*`, `/profile`, `/standing` | ALL-01, ALL-04, ALL-05, STU-11, STU-12, STF-01, CHR-01, DEAN-01 (dashboard) |
| Forums | `forums` | `/feed/*`, `/spaces/*`, `/organizations/*`, `/messages/*`, `/directory`, `/people/*` | ALL-02, ALL-03, STU-01, STU-02, STU-03, ORG-01, ORG-02, PRO-01, PRO-02, PRO-03, CHR-06, DEAN-06, GUD-05, OSAS-04 |
| Schedule | `schedule` | `/calendar`, `/consultations/*` | STU-04, STU-05, PRO-04, PRO-05 |
| Cases | `cases` | `/reports/*`, `/cases/*` | STU-06, STU-07, STU-08, STU-09, PRO-06, CHR-02, CHR-07, DEAN-02, GUD-01, GUD-02, OSAS-01, PRES-01, PRES-02 |
| Evaluations | `evaluations` | `/evaluations/*` | STU-10, PRO-07, PRO-08, CHR-04, DEAN-03, DEAN-04 |
| Records | `records` | `/records/*`, `/restrictions`, `/analytics` | CHR-03, CHR-05, DEAN-05, GUD-03, GUD-04, OSAS-02, OSAS-03, ADM-03 (restrictions live on P35) |
| Admin | `admin` | `/admin/*` | ADM-01, ADM-02, ADM-04, ADM-05, ADM-06 |

`/sign-in` is served on every host. The adviser asked (2026-09-30) to drop "portal" from the product title, so the Home site shows "Home" in the UI. `portal` is used only as the host label.

### Host resolution

1. **Configured origins.** Build-time `VITE_SITE_ORIGINS` is a JSON map `{ "<site>": "https://<host>" }`. CI fills it from the stack outputs. When the current origin matches an entry, that entry is the current site.
2. **Host label.** Otherwise, if the first DNS label is a site's host label (`forums.localhost`, `forums.access-ccs.com`), that site is current. Other sites are reached by swapping that label.
3. **Unified.** Otherwise, as on plain `localhost:5173`, every route works on one host, as before. The site is derived from the path for theming and navigation only.

When a route belongs to another site and that site has an origin, the router does a full-page redirect to `<origin><path><search>`. Links render absolute `href`s to the other site. The session cookie belongs to the API host (`SameSite=None; Secure` in production), so one sign-in works across all deployed sites.

`access-ccs.com` was a proposal when this delta was written. It was registered on 2026-11-03: prod sites use `<label>.access-ccs.com` and dev sites `<label>-dev.access-ccs.com` (see `architecture/README.md`). The CloudFront default hostnames still work.

### Shell and visual redesign

- **Site switcher:** a top bar row lists only the sites with at least one allowed destination. The current site is marked by a sliding indicator plus `aria-current`, not by color alone.
- **Contextual sidebar:** shows the current site's allowed entries for the person's roles, plus account links.
- **Site hero:** on each site's landing route, it shows the site title, a one-line purpose, and the module IDs it serves. It also shows the site art (WebP, transparent, converted from `meeting-at-ten-assets`) with a slow float.
- **Lane tints** (design-system rules): community is violet (Forums), consult is teal (Schedule), and navy is the default. Case and record surfaces stay neutral (grievance mode).
- **Motion:** route changes use View Transitions where supported (cross-fade and 8 px rise, 200 to 320 ms, ease-out tokens). Page sections reveal in a stagger. Hero art floats. `prefers-reduced-motion` removes all transforms and loops.
- **Sign-in:** split layout. The brand panel uses the topographic background and crest art. Demo accounts are filtered by search and grouped by role, and the page names the site being entered.
- **Home:** a site directory with one tile per accessible site (art, purpose, destinations) sits above the existing P03 role dashboard.

### Infrastructure

- `AppSite` (existing distribution and URL) serves the **portal** site. Six new `static-site.yaml` nested stacks (`Spa=true`) serve forums, schedule, cases, evaluations, records, and admin.
- The API `CorsOrigin` becomes a comma-separated list of all seven site URLs. The backend parses `CORS_ORIGIN` as a list.
- The root stack outputs `SiteOrigins` (JSON) for the frontend build. The frontend CI uploads the same `dist/` to all seven buckets and invalidates each distribution.

## Success criteria

1. Each deployed site URL loads its own landing page, hero, and navigation. A cross-site link or direct URL lands on the owning site's host with the path preserved.
2. One sign-in on any site is honored on every site, in a browser that allows the API's cross-site cookie.
3. Navigation only lists permitted destinations. Direct route and API access stay independently authorized, and the backend access tests still pass.
4. Unified mode on `localhost:5173` keeps every route working on one host.
5. Reduced motion disables transforms and loops. Current location is perceivable without color.
6. `pcore npm run typecheck` and `pcore npm run build` pass (frontend), and `pcore npm run typecheck` and `pcore npm test` pass (backend).
