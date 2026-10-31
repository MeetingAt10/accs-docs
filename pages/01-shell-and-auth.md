# Shell, sign-in, and system states

Back to [index](README.md) · [sitemap](../sitemap.md)

## App shell (`src/routes/_app.tsx`)

Wraps P03 to P52. Reference composition: design system `site/app/_site/examples/Portal.tsx`.

| Region | Components | Notes |
|---|---|---|
| Frame | `AppShell` | `density="comfortable"` for Student and Organization; `density="compact"` for handler and admin roles; the person can change it on P43. Skip link `.ds-skip-link` to `contentId`. |
| Site switcher | `SiteSwitcher` (`site-chrome.tsx`): `nav aria-label="ACCS sites"` of `NavItem` links in the top bar's second row | Seven module sites in fixed order: Home, Forums, Schedule, Cases, Evaluations, Records, Admin. A site is listed only when the session has an allowed destination there, and the row is hidden when fewer than two sites qualify. Each link goes to the site's first permitted landing and, when the site has its own host, loads that host. The current site has `aria-current="page"` and a sliding indicator, so it is not marked by color alone. Desktop only (1024 px and up); on mobile the drawer's **Sites** section replaces it. See [sitemap §4](../sitemap.md#4-primary-navigation-and-contextual-sidebar). |
| Contextual sidebar | APP-1 `RoleSidebar` = `Sidebar` + `SidebarSection` + `NavItem` (`icon`, `isCurrent`) | Shows only the current site's entries, filtered by `session.allowedPages` and labeled for the person's roles. The `Sidebar` label is "ACCS" plus the site name, and the role group named "Workspace" is titled with the site name. The sidebar is omitted when the site has no allowed entries for the person. This is navigation only; route and API checks still enforce access. |
| Sidebar footer | `Avatar decorative`, name, one `RoleBadge` per role held, `SidebarNote` | The note says "Sample data. Access follows your assigned role and scope." Brand: `DoorPlate code="ACCS"` plus the current site's name (the same brand shows in the mobile header). |
| Top bar | `TopBar` (`title`, `tools`) | Title repeats the page name as a `p`; while on simulated data add `Badge outline marker={false}` "Sample data". |
| Search | `Search size="sm" className="ds-topbar__search"` | Searches posts and spaces the person can read and opens P04 with `?q=`; a link there offers "Search people" (P37). Never searches reports, cases, or records. |
| Assistant | `IconButton icon="robot" label="Assistant"` → `Drawer side="end" title="Assistant"` with APP-17 | Passes the current page as context ("Explain this stage" on P20). Link to P41. |
| Notifications | `IconButton icon="bell" dot label="Notifications, 2 unread"` → `Popover title="Notifications"` with DS-4 `NotificationList` (latest 5), `Button variant="tertiary"` "Mark all read", `Link variant="standalone"` "See all" (P40) | The label states the count in words. |
| Account menu | `MenuButton iconOnly icon="user" placement="bottom end"` with `MenuItem` Profile (P39), Settings (P43), Help (P42), `MenuSeparator`, Sign out | Settings and Help also appear in the Home site's sidebar when permitted. Sign out ends the session and opens P02 with the signed-out message. |
| Phones | `MobileHeader` through `AppShell` (`mobileBrand={<DoorPlate code="ACCS" size="sm" />}`, `mobileNav`, `mobileActions` = permission-filtered account menu) | Drawer starts with a **Sites** section of the seven site links (permitted sites only), then shows destinations for the current site. The labeled account menu includes permitted Profile, Notifications (with unread count), Assistant, Settings, and Help links, plus Sign out. |
| Restricted banner | APP-20 `RestrictedBanner` = `Alert tone="warning"` under the top bar | Only for a student with an active restriction (STU-11): what is paused, until when, and what still works; link to P36. |
| Jump | `CommandPalette` (Ctrl+K / Cmd+K) | Role-filtered destinations and actions ("New post", "New report", "Book a consultation"); navigating to a destination updates the current site and contextual sidebar. |
| Toasts | `ToastRegion` (already in `__root.tsx`) | |

### Module sites in the shell

The shell is one component (`AppChrome` in `src/components/app/navigation.tsx`) that serves all seven sites. It derives the current site from the path (`siteForPath`) and sets `data-site` and `data-lane` on the shell, which drive the lane tint. Host resolution and cross-site redirects are in [sitemap §4](../sitemap.md#4-primary-navigation-and-contextual-sidebar); the redirect runs in `beforeLoad` of `src/routes/__root.tsx`, and the root route also rewrites links the design system renders so they point at the owning host.

| Element | Where | Behavior |
|---|---|---|
| Site hero | `SiteHero`, above the page content on each site's landing routes | Shows the site title, a one-line purpose, the DOC003 module IDs the person's roles own on that site (shared ALL modules always), and the site art. The title is a time-of-day greeting with the person's first name on Home, and the site name elsewhere. The art is decorative (`alt=""`), hidden below 720 px, and floats slowly. Landing routes: Home `/home`, `/notifications`; Forums `/feed`, `/spaces`, `/messages`, `/directory`; Schedule `/calendar`, `/consultations`; Cases `/cases`, `/reports`; Evaluations `/evaluations`; Records `/analytics`, `/records`, `/restrictions`; Admin `/admin`. |
| Home site directory | `SiteDirectory`, "Your sites", on `/home` below the hero and above the P03 dashboard | One `CardLink` tile per other site the person can open: art, site name, purpose, and up to four of the person's destination labels there. Hidden when the person has no other site. |
| Document title | `src/routes/_app.tsx` | `<Page> · ACCS <Site>`, for example "My reports · ACCS Cases". When the page title equals the site name, the title is `<Page> · ACCS`. Titles use generic page names only, never record or case titles. |
| Motion | `src/components/app/sites.css` | Route changes cross-fade through View Transitions where the browser supports them (old page out in 120 ms, new page in over 200 ms). Sections of the new page rise 8 px with a stagger of up to 160 ms; module chips and directory tiles stagger the same way. The sliding indicator moves between switcher items, and the hero art floats. Tiles lift 2 px on hover. All durations and easings use the design system's `--ds-motion-*` tokens. |
| Reduced motion | `prefers-reduced-motion: reduce` | Removes the page and hero reveals, the art float, the View Transition animations, and the tile hover movement, so content appears at once. The switcher indicator follows the design system provider's Motion setting, which follows the operating system preference. |

Cases and Records use the neutral lane (grievance mode: a gray tint instead of a brand color). Forums uses the community (violet) lane, Schedule the consult (teal) lane, and the other sites navy. See [design-system.md](../design-system.md#navigation-and-wayfinding).

Guard: `beforeLoad` checks the session and redirects to `/sign-in?redirect=<path>`. After sign-in the redirect is followed only if the new session may open it (S5 ALL-01-E).

## P01 `/` — entry redirect

| | |
|---|---|
| File | `src/routes/index.tsx` (today: the coming-soon page) |
| Permission | none |
| Modules | ALL-01 |

Signed in: redirect to `/home`. Signed out: redirect to `/sign-in`. Keep the current coming-soon page at `/` until launch, behind a flag.

## P02 Sign in — `/sign-in`

| | |
|---|---|
| File | `src/routes/_auth/sign-in.tsx` |
| Permission | signed-out visitors; a signed-in person is redirected to `/home` |
| Modules | ALL-01 |
| Stories | US-01 to US-05 (role resolution) |
| Tests | ALL-01-A to ALL-01-L |
| Reference | design system `SignIn.tsx` |

**Purpose.** Sign in with a verified QCU account and land in the role's view.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Page | `Container width="reading"`, `.ds-sheet` canvas | No app shell. |
| Card | `Card overlay` | One floating panel. |
| Brand | `DoorPlate size="lg" code="Portal"` | ACCS wordmark until a logo is approved. |
| Heading | `SectionHeader level={1} title="Sign in" description="Use your official QCU account."` | |
| Messages | `Alert` (see states) | Above the fields, so screen readers hear it first. |
| Identifier | `TextField label="QCU email" type="email" autoComplete="username" isRequired` | Whether the identifier is the QCU email or the student/employee ID is TBD (ALL-01-F, K). |
| Password | `TextField label="Password" type="password" autoComplete="current-password" isRequired` | |
| Remember | `Checkbox` "Keep me signed in on this device" † | Only if the session policy allows it (ALL-01-G). |
| Submit | `Button type="submit"` "Sign in" | Shows `loading` while checking. |
| Help | `Disclosure title="Can't sign in?"` | Plain text: who to contact. Password recovery is TBD, so no reset link yet. |
| Prototype note | `PrivacyNotice title="Prototype"` | "ACCS uses simulated enrollment data. Don't enter real records." |

**States**

| Case | Result | Test |
|---|---|---|
| Empty fields submitted | Field errors (`isInvalid`, `errorMessage` "Enter your QCU email" / "Enter your password"); nothing sent | ALL-01-D |
| Wrong password or unknown account | One generic `Alert tone="danger"`: "Couldn't sign in. Check your QCU email and password." Email kept, password cleared. Whether to tell an unknown account apart is TBD | ALL-01-B, C |
| Inactive or suspended account | `Alert tone="warning"`: "This account can't sign in right now. Contact the CCS office." (TBD) | ALL-01-L |
| Signed out | `Alert tone="success"`: "You've signed out." | ALL-01-G |
| Session ended | `Alert tone="info"`: "Your session ended. Sign in again to continue." | ALL-01-G |
| Spaces or letter case in the identifier | Record what happens; normalization is TBD | ALL-01-K |
| Success | Land on `/home` in the role's view, or the permitted `redirect` | ALL-01-A, H |

**Open.** Identifier format, registration flow, recovery, lockout, session length (identity policy).

### Prototype sign-in: split layout

The layout table and states above are the target spec. The prototype build (`src/routes/_auth/sign-in.tsx`, `sign-in.css`) has no email and password fields; the person picks a simulated demo account. It still has no app shell and uses a split layout: two columns from 1024 px up, stacked below that.

| Region | Contents |
|---|---|
| Brand panel | `DoorPlate size="lg" code="ACCS"`, the headline "Your access to the College of Computer Studies", a one-line description, and a list of the seven site names (`aria-label="ACCS sites"`, with `aria-current` on the site this host serves). The crest art (`/brand/sites/crest.webp`, decorative) sits over the topographic background (`/brand/topographic.webp`). |
| Sign-in panel | Heading "Sign in", or "Sign in to <Site>" when the host serves one site. `Badge outline` "Sample data" and a note that the accounts are simulated. |
| Search | `Search` "Find a demo account" filters by name, email, description, and role. With no match the panel says `No demo account matches "<query>".` |
| Accounts | Grouped under role headings, in the order Students, Student organization, Professors, CCS staff, Program chairs, Dean, Guidance, OSAS, President's Office, Administration. A multi-role account appears once, under the earliest of its roles in that order. Each account is one button named "Continue as <name>, <roles>" with avatar, name, description, and `RoleBadge`s. The pressed account shows a `Spinner` and the others are disabled until the request ends. |
| States | Loading: `ViewState kind="loading"`. Account list failed: `ViewState kind="error"`. Sign-in failed: `Alert tone="danger"` "Sign-in failed". |

After sign-in the person goes to the `redirect` parameter if present, otherwise to the current site's first permitted landing, otherwise to Home (`/admin` for an Administrator). If that route is on another site's host, the browser loads that host ([sitemap §4.2](../sitemap.md#42-cross-site-behavior)). `/sign-in` is served on every host. The page elements reveal in a short stagger, which `prefers-reduced-motion: reduce` turns off.

### Local demo-role launcher

The frontend's optional Vite launcher at `/__roles` lists accounts from the selected demo API. Each link opens `/__role?account=<id>` on that account's own `*.localhost` hostname, signs in through the existing endpoint, and opens Home or Administration. Host-only session cookies keep different account hostnames independent in one browser window. Tabs for the same account hostname share a session, and all accounts share the selected API's records.

Enable it with `ACCS_ROLE_LAUNCHER=1`, keep `VITE_API_URL=/api`, and use `ACCS_DEV_API_TARGET` to select the backend (default `http://localhost:3000`). Vite binds to `127.0.0.1` and rejects nonlocal hosts. This launcher is a local development tool; production builds keep the normal sign-in flow. See the [frontend startup instructions](../../frontend/README.md#compare-demo-accounts-in-separate-tabs).

## System states (inside the shell)

| State | When | Components | Copy |
|---|---|---|---|
| Restricted | Permission key missing (APP-2 `RestrictedView`) | `ViewState kind="restricted"` | Page: "This page isn't available to your role." Record: "This record isn't available to you. It may not exist, or it may belong to someone else." The record wording is the same for missing and forbidden records, so existence is not revealed (S5 ALL-01-J, STU-09-B). |
| Not found | No route matches | `EmptyState title="Page not found"` with `action` link to Home | |
| Error | Loader or API failure | `ViewState kind="error"` with `actions` = `Button` "Try again" (`router.invalidate()`) | Keep the person's input and place. |
| Loading | Route pending | `Skeleton` in the page's shape; `ViewState kind="loading"` names the wait ("Loading reports") | |
| Offline write | A save fails | `Alert tone="danger"` beside the form; draft kept | Never a toast only. |
