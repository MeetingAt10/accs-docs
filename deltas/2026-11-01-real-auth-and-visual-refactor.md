# Delta: real sign-in and registration, and the app visual refactor

**Status:** implemented, 2026-11-01; session, reset-code, and throttling text corrected 2026-11-05 to match the backend
**Superseded in part:** the public registration page, the `/register` route, `POST /api/auth/register`, the self-registration rules, and the registration step of success criterion 1 below are removed by [`2026-11-05-consultation-50-percent.md`](2026-11-05-consultation-50-percent.md) (no public registration; accounts come from the secretary or the MIS/registrar import).
**Builds on:** [`2026-11-01-subdomain-sites.md`](2026-11-01-subdomain-sites.md)

## Why

The module-site pass added hero art on top of unchanged pages, so the app still read as boxy boilerplate. Sign-in was a demo-account picker with "Sample data" badges. This delta replaces both.

## 1. Authentication

### Backend (`backend/src`)

| Endpoint | Body | Result |
|---|---|---|
| `POST /api/auth/register` | `{ fullName, email, identifier, role, program?, yearLevel?, section?, password }` | 201 + session cookie + session view; 400 for invalid fields (with `fields`); 409 `CONFLICT` for a duplicate email or ID |
| `POST /api/auth/sign-in` | `{ emailOrId, password, remember? }` | 200 + session cookie + session view; 401 `INVALID_CREDENTIALS`; 429 `TOO_MANY_ATTEMPTS` |
| `POST /api/auth/forgot-password` | `{ emailOrId }` | 200 always (no account enumeration). Outside production, issues a 6-digit reset code when an account matches; in production, issues nothing |
| `POST /api/auth/reset-password` | `{ emailOrId, code, password }` | 200 and ends every session of the account; code is single use, 15 minutes; 400 `INVALID_CODE`; 429 `TOO_MANY_ATTEMPTS` |
| `POST /api/auth/change-password` | `{ currentPassword, password }` (signed in; 401 otherwise) | 200 and ends the account's other sessions; 400 `INVALID_CREDENTIALS` when the current password is wrong; 429 `TOO_MANY_ATTEMPTS` |
| `POST /api/auth/sign-out` | `{}` | 200, ends the session, clears the cookie |

The session cookie is HTTP-only and holds an opaque random token. With `remember: true` it lasts 30 days; otherwise it is a browser-session cookie. Sessions are server-side: the `ccs_identity` table stores a `SESSION` item with the SHA-256 of the token, and every request checks it. A session expires after 30 minutes idle or 12 hours in all (with `remember: true`, 7 days idle or 30 days in all). These values are provisional until decision D-08. `lastSeenAt` is written at most every 5 minutes, so an idle session can end up to 5 minutes before the idle limit. Sign-out ends that session; changing a password or completing first login ends the account's other sessions; a password reset or deactivating the account ends all of them. Without `TABLE_PREFIX`, sessions stay in memory locally, and on Lambda sign-in fails rather than issuing sessions valid on one instance only.

- Passwords: `crypto.scryptSync` (N=16384, r=8, p=1, 16-byte salt, 64-byte key), compared with `timingSafeEqual`. Strength rule: 10+ characters, upper, lower, digit, symbol.
- Seeded institutional accounts get one shared seed password (stack parameter `SeedPassword`, Lambda `SEED_PASSWORD`; a built-in default applies to local runs and tests only) so tests and existing users can sign in by email or ID. The seeded secretary is inactive and never accepts it: that account signs in only with credentials the Dean issued. No account list is exposed: `GET /api/demo-accounts` is removed, and the `accountId`-only sign-in is removed.
- Self-registration roles: **student**, **professor** (Faculty), **staff** (CCS staff). Chair, dean, guidance, OSAS, President's Office, and administrator access is granted by an administrator (P46/P47), never self-selected: a public form that grants case-file or admin access is a privilege-escalation hole.
- Students must give program (`BSIT`/`BSCS`/`BSIS`), year level (1–4), and section (`A`–`F`). Identifier format: `NN-NNNN` (e.g. `24-1234`) for students, `NNNN-NNNN` or `NN-NNNN` for employees. Email must be unique; identifier must be unique.
- Persistence: accounts and password hashes live in an auth store. With `AUTH_STORE_BUCKET` set (Lambda), the store is one JSON object in a private S3 bucket (read with ETag caching, written with a conditional put). Without it, a JSON file at `AUTH_STORE_FILE` (default `/tmp/accs-auth-store.json`). Sessions and reset codes are not in this store: they live in the DynamoDB table `<TABLE_PREFIX>ccs_identity`, as hashes only, so they hold on every Lambda instance. The store is read at the start of every `/api` request (cached for 2 seconds, then a conditional GET) and accounts are merged into the repository (and their class spaces) before session resolution. Tests use in-memory stores. A seeded user keeps the shared password until they change it; a changed password is stored under their user ID.
- Reset codes: no email provider is connected yet (open item). Outside production, forgot-password logs the code to the server console. In production it issues nothing and its response tells the person to ask an ACCS administrator, who issues a code with `POST /api/admin/accounts/:id/reset-code`; the code is shown once and the action is audited.
- Throttling: 5 failures per 15 minutes returns 429 on sign-in and reset-password (per identifier) and on change-password (per account). These counters are in memory and per instance, so this limit does not hold across Lambda instances. Reset-code limits are stored on the `RESET` item and hold on every instance: 5 guesses per code, a 60-second cooldown, and 5 self-service codes per account per hour, with the same 200 response from forgot-password. An administrator-issued code skips the cooldown and the hourly cap. These values are provisional until decision D-08.

### Frontend

- `/sign-in`: split screen (`src/routes/_auth/sign-in.tsx`, shared pieces in `-auth-shell.tsx`, styles in `auth.css`). Left: navy brand panel with the ACCS door plate, "ACCS / QCU · College of Computer Studies", a one-line headline, and three short facts. It has no site list and no links. Right: form with "Email or ID", password with show/hide, Remember me, Forgot password, submit with loading state, inline errors, and a link to `/register`. Forgot password swaps the right column in place: enter email or ID, then enter the 6-digit code with a new password (strength meter and confirm), then back to sign-in with a success notice. No demo list, no sample badges. The two columns apply from 960 px up.
- `/register`: one page, three fieldsets (About you, Your role at CCS, Password). Role selector (Student / Faculty / CCS staff); program, year level, and section appear only for students. A note says chairs, deans, guidance, OSAS, and administrators get access from the CCS administrator. Password strength meter (4 segments with a checklist) and a confirm field. On success the session is set and the person lands on the `redirect` target or their landing page.
- `/sign-in` and `/register` send an already signed-in visitor to their landing page. The `redirect` search parameter is accepted only when it starts with a single `/`.
- Settings (P43, `/settings`) has a **Change password** panel: current password, new password with strength meter, confirm.
- The dev role launcher (`frontend/vite.local-role-launcher.ts`) signs in with the seeded password. Its `/__roles` page no longer lists accounts, because `GET /api/demo-accounts` is gone; `/__role?account=<ID>` on `<lowercased ID>.localhost` still opens a session.

## 2. Visual system

**Mode:** Operate. People finish tasks between classes, on laptops in labs and on phones in hallways. Light theme, daylight use.

**Palette** (`src/styles.css`, `--accs-*`): ink `#0B1220`, QCU navy `#032E87`, rail navy `#081A3F`, paper `#F3F5F9`, surface `#FFFFFF`, hairline `#E2E7EF`. Each site has one accent (`--site-accent`, `-ink`, `-wash`) set by `[data-site]`: Home blue `#3157D5`, Forums violet `#7040C7`, Schedule teal `#0D9488`, Cases wine `#9A3B52`, Evaluations gold `#B7791F`, Records steel `#0E7490`, Admin graphite `#334155`.

**Type:** Plus Jakarta Sans (display, 700–800, tracking −0.02em) for page titles, panel titles, and numbers; Inter for body (14–15 px in the app); JetBrains Mono only for references, IDs, times, and counts.

**Signature:** the **site rail**. A 72 px dark navy rail on the far left holds the ACCS mark and one icon button per site the person's role may open (Phosphor icons from the design system, label under the icon). The current site gets its accent fill and a 3 px indicator that slides between sites. Next to it, a light 248 px contextual sidebar shows the current site's name, purpose, and navigation, with the signed-in person (profile link, sign out) at the bottom. The rail's avatar button opens an account menu (Profile, Notifications, Settings, Help, Sign out).

**Shell** (`src/components/app/navigation.tsx` `AppChrome`, `site-chrome.tsx`, `sites.css`): rail, sidebar, and a 56 px topbar from 1024 px up. The topbar has a breadcrumb (site / page), a search box for posts and spaces (from 768 px), the Assistant drawer button, and the notifications popover. Below 1024 px the rail and sidebar are hidden; a menu button opens a drawer with the sites, the current site's navigation, and account links. The sidebar is omitted when a site has no allowed entries for the person. A student with an active restriction sees a banner under the top of the shell. The shell sets `data-site` on its root, which switches the accent. The command palette and the site hero, site switcher row, and Home site directory from the module-site pass are gone.

**Assets: minimal.** Signed-in pages use no illustration art at all: no hero banners, no floating art, no art tiles. No generated picture assets remain: `frontend/public/brand/` is deleted, and the `/sign-in` and `/register` brand panel uses the design-system `DoorPlate` over a CSS gradient, as before the asset pass. Workspace pages read like Linear or Vercel: type, hairlines, one accent.

**Motion** (`src/motion.css`): one layer for every page. The page header and sections rise in sequence (`.accs-page-head`, `.accs-reveal`, `.accs-stagger`), rows and cards cascade (`.accs-rows`, `.accs-table` rows, `.accs-grid`), KPIs scale in, progress bars grow from the left, interactive panels lift 2 px on hover, and skeletons shimmer. Route changes use View Transitions (`defaultViewTransition` on the router). Durations come from `--accs-dur-1` to `--accs-dur-3` (160, 280, 520 ms) and `--accs-ease-out`. Auth screens and the shell chrome animate with the `motion` package. `prefers-reduced-motion: reduce` removes all of it.

**Primitives** (`src/components/app/page-shell.tsx` + foundation CSS): `PageHeader`, `Panel` (border, no resting shadow, radius 14, never nested), `Kpis`/`Kpi` (one hairline-divided strip), `StatusChip` (dot + text, tone by `toneFor`), `DateTile`, `Empty`, `siteCrumb`, plus CSS classes `.accs-pill(s)`, `.accs-tag`, `.accs-toolbar`, `.accs-table(-wrap)`, `.accs-rows`/`.accs-row`, `.accs-timeline`, `.accs-progress`, `.accs-avatar`, `.accs-mono`, `.accs-meta`. Dates go through `src/lib/format.ts`; raw ISO strings never render.

**Bans:** "Sample data", "Synthetic", "Prototype", "Demo" badges or copy anywhere in the UI; DOC003 module ID chips in the UI; eyebrow labels above headings; gradient text; nested cards; colored `border-left` callouts; emoji as icons; progress rings.

### Per site

| Site | Layout |
|---|---|
| Home | Greeting header with date (no art), KPI strip, two columns: left "Today" agenda + announcements feed + recent activity timeline; right rail quick actions, my reports, evaluations due, to-dos. |
| Forums | Three columns on wide screens: category pills + spaces list, feed of thread cards (author, label tag, title, excerpt, reply and reaction counts, time), right rail with announcements and organizations. Composer as a compact "Start a discussion" bar that expands. |
| Schedule | Calendar as the main panel with a slim toolbar (month/week/list), faculty availability as slot cards, booking as a stepper (faculty, slot, purpose, confirm). |
| Cases | Status pipeline strip (Submitted → Adviser → Chair → Dean → Guidance → OSAS → Resolved) with counts, a dense case table, severity chips, case detail with a timeline and action bar. Neutral, calm tone. |
| Evaluations | Assignment cards with linear progress, cycle list as a table with status chips, rating matrix with sticky criterion column, results with bar charts. |
| Records | Filter toolbar, KPI strip, analytics charts in panels, records table with standing chips. |
| Admin | Dense console: KPI strip, users table, RBAC matrix with sticky header, audit log table with mono timestamps, monitoring panels. |

## 3. Role-based access (sites, routes, API)

Requested role names map to the ACCS role set: system administrator / IT support → `administrator`; counselor → `guidance`; coordinator / chairperson → `chair`; career officer → `osas`; faculty → `professor`.

| Site | Roles that may open it |
|---|---|
| Home, Forums | every signed-in role |
| Schedule | `student`, `organization`, `professor`, `chair`, `dean`, `guidance`, `osas` |
| Cases | `student` (own reports), `professor`, `chair`, `dean`, `guidance`, `osas`, `president` |
| Evaluations | `student`, `professor`, `chair`, `dean` |
| Records | `chair`, `dean`, `guidance`, `osas`, `president`, `administrator` |
| Admin | `administrator` only |

- `siteRoles` in `frontend/src/lib/sites.ts` is the site gate. `canOpenSite` is true when one of the person's roles is listed for the site (or the site is open to all) **and** at least one of the site's landing pages is in `allowedPages`. `canOpenPath` also requires the path's own page in `allowedPages`, and requires `canOpenSite` for both the path's site and the site this host serves. The rail, drawer, and landing redirect use `canOpenSite`.
- `_app.tsx` resolves the session before any shell renders; signed out → `/sign-in?redirect=<path>`. A path (or a host) the role may not open renders a full-page **403** from `AccessDenied` in `components/app/guards.tsx`: a lock icon, the title "You don't have access to this page", a line naming the person's role (and the site, when a site is the blocker) and pointing to the CCS administrator, and a **Go to your home** button. It replaces the shell. Inside the shell, a page the role may not open (page permission missing) shows the same message, and a record outside the person's scope shows "This record isn't available to you". No shell link, rail button, or dashboard shortcut points at a site the role cannot open.
- `/sign-in` and `/register` show no site list and no internal links.
- The API keeps per-page checks: no cookie → 401 on every `/api/pages/*`; a role outside a page's permissions → 403. `backend/tests/rbac-matrix.test.ts` asserts the full role × page matrix and the signed-out 401s.

## Success criteria

1. A new person registers at `/register`, lands in their workspace, signs out, and signs back in with email or ID plus password. Wrong passwords show an inline error.
2. No page shows sample, demo, synthetic, or prototype wording or module ID chips.
3. Every site uses the rail, contextual sidebar, `PageHeader`, and panel primitives; no illustration art appears after sign-in.
4. A student sees no Admin or Records link anywhere, and `/admin/*` or `/records/*` shows the 403 screen; signed-out visits to any app path land on `/sign-in`.
5. No horizontal overflow at 1440×900, 768×1024, and 390×844.
6. Backend tests, frontend typecheck, and the DOC003 58-module test pass.

## 4. Copy and layout cleanup (unslop pass)

All user-visible copy in the frontend and the backend seed and service strings was rewritten in plain, specific, sentence-case wording (no em dashes in prose, no sample, demo, synthetic, pilot, or prototype wording). Seed data reads like CCS traffic: rooms such as IC 301, terms such as "1st Semester AY 2026-2027", and real office names.

Layout repetition was removed:

- KPI strips stay only where the numbers drive a decision: Home, Records analytics, Admin overview and monitoring, and Evaluations results. Elsewhere counts moved into pill labels or a one-line summary (Schedule availability, Cases, Evaluations home, student record, restrictions).
- Card grids that were really lists became row lists: My classes, Organizations, Similar reports, Directory.
- Decorative icon tiles, glass blur on toolbars, gradient bars, and intro lines that restated the page title were removed. The Cases stage strip became a Stage filter in the queue toolbar.
- Evaluations sticky bars sit under the 56 px top bar.
