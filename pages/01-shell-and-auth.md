# Shell, sign-in, and system states

Back to [index](README.md) · [sitemap](../sitemap.md)

## App shell (`src/routes/_app.tsx`)

Wraps P03 to P52. Reference composition: design system `site/app/_site/examples/Portal.tsx`.

| Region | Components | Notes |
|---|---|---|
| Frame | `AppShell` | `density="comfortable"` for Student and Organization; `density="compact"` for handler and admin roles; the person can change it on P43. Skip link `.ds-skip-link` to `contentId`. |
| Site rail | `SiteRail` (`site-chrome.tsx`): 72 px navy rail, `nav aria-label="ACCS sites"` | One icon button with a short label per site the person's role may open (`canOpenSite` in `src/lib/sites.ts`: role in `siteRoles` and at least one allowed page). The current site has an accent fill, a 3 px indicator, and `aria-current="page"`; both slide between sites. The person's initials at the bottom open the account menu. No illustration art. |
| Contextual sidebar | `SiteSidebar` (`site-chrome.tsx`) | 248 px, white. Site name and one-line purpose, then the current site's entries filtered by `session.allowedPages`, then account links. The current entry has the site accent wash. |
| Sidebar footer | Initials avatar, name, role label, sign-out button | No sample-data note. |
| Top bar | 56 px, translucent with blur, sticky | Breadcrumb `<Site> / <Page>`, search (768 px and up), assistant, notifications with unread dot; on phones a menu button opens the drawer. |
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
| Document title | `src/routes/_app.tsx` | `<Page> · ACCS <Site>`, for example "My reports · ACCS Cases". When the page title equals the site name, the title is `<Page> · ACCS`. Titles use generic page names only, never record or case titles. |
| Motion | `src/motion.css`, `motion/react` | Route changes cross-fade and rise through View Transitions. Page headers and sections (`.accs-stagger`) rise in sequence 60 ms apart; rows, table rows, grid items, and KPIs cascade; progress bars grow; active indicators slide (`layoutId`). |
| Reduced motion | `prefers-reduced-motion: reduce` | Removes every entrance, cascade, slide, and hover movement; content appears at once. |

Cases and Records use the neutral lane (grievance mode: a gray tint instead of a brand color). Forums uses the community (violet) lane, Schedule the consult (teal) lane, and the other sites navy. See [design-system.md](../design-system.md#navigation-and-wayfinding).

Guard: `beforeLoad` checks the session and redirects to `/sign-in?redirect=<path>` before any shell renders. A path or host the role may not open (`canOpenPath`) renders the full-page **Access denied** view (`AccessDenied` in `guards.tsx`: lock icon, "You don't have access to this page", the role and site named in one sentence, and a "Go to your home" button) with no shell links. After sign-in the redirect is followed only if the new session may open it (S5 ALL-01-E).

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

**Open.** Email delivery for reset codes, lockout shared across API instances, session length, and QCU identity-provider integration.

### Built sign-in and password reset

There is no public registration. Accounts are created by the secretary or imported from the MIS/registrar list (team decision, 2026-11-05; [delta](../deltas/2026-11-05-consultation-50-percent.md)).

Files: `src/routes/_auth/sign-in.tsx`, `-auth-shell.tsx`, `auth.css`. No app shell, no site list, and no links to internal sites. Split layout from 960 px: a navy brand panel (`DoorPlate` "ACCS | CCS", a headline, three facts; no images) and the form column. Below 960 px the brand panel becomes a compact navy strip above the form.

| Page | Contents |
|---|---|
| `/sign-in` | "Email or ID" (email, user ID, username, or student/employee number), password with show/hide, Remember me (30-day cookie), Forgot password, Sign in (loading state). Errors: `INVALID_CREDENTIALS` "The email, ID, or password is incorrect."; `TOO_MANY_ATTEMPTS` after 5 failures in 15 minutes; network failure. |
| Forgot password | Inline panel swap on `/sign-in`: email or ID, then a 6-digit code (15 minutes, single use) with a new password and confirmation. The API always answers the same way, so it never reveals whether an account exists. No email provider is connected yet; the page says to contact the CCS office if no email arrives. |

Password rule: 10+ characters with upper and lower case, a number, and a symbol. Seeded pilot accounts sign in with their ID or email and the shared pilot password, which each environment sets through the stack parameter `SeedPassword` (Lambda `SEED_PASSWORD`; a built-in default applies to local runs and tests only), which must be rotated or disabled before real users join.

After sign-in the person goes to the `redirect` parameter if present, otherwise to the current site's first permitted landing, otherwise to Home (`/admin` for an Administrator). If that route is on another site's host, the browser loads that host ([sitemap §4.2](../sitemap.md#42-cross-site-behavior)). `/sign-in` is served on every host.

### First sign-in (`/first-login`)

File: `src/routes/first-login.tsx`. An account created by the secretary, and the secretary once the Dean activates it, signs in with a temporary username and password. Sign-in then sends the person to `/first-login`, and every other route redirects there (`mustChangeCredentials` on the session) until the change is done. The page asks for the temporary password, a new username (3 to 32 characters: letters, numbers, dot, dash, underscore), and a new password with confirmation, under the same password rule as above. On success the person goes to the `redirect` parameter or their landing page, and other devices are signed out. A person without temporary credentials who opens the page is redirected away; a signed-out visitor goes to `/sign-in`. Sign-in accepts the username as an identifier next to email, ID, and student/employee number. API: `POST /api/auth/complete-first-login` ([backend README](../../backend/README.md)).

### Dev toolbox (development stack only)

A floating button with the ACCS logo and a toolbox icon (`src/components/dev-toolbox/`) lists accounts grouped by role. Picking one makes the API end the current session and start one for that account, so testers change user without signing out. It is loaded only when the build sets `VITE_DEV_TOOLBOX=true` (or under `vite dev`); other builds do not contain it. The API routes (`GET /api/dev/users`, `POST /api/dev/switch-user`) exist only on the `accs-dev-` stack and every switch is audited. Anyone signed in there can become any active account, the Dean included, so keep real student data off that stack. Not part of the product UI.

### Local role launcher

The optional Vite launcher (`ACCS_ROLE_LAUNCHER=1`) opens `/__role?account=<ID>` on that account's own `*.localhost` hostname and signs in with the seeded pilot password. `/__roles` no longer lists accounts, because the account list endpoint was removed. Use `ACCS_DEV_API_TARGET` to select the backend (default `http://localhost:3000`). Local development only.

## System states (inside the shell)

| State | When | Components | Copy |
|---|---|---|---|
| Restricted | Permission key missing (APP-2 `RestrictedView`) | `ViewState kind="restricted"` | Page: "This page isn't available to your role." Record: "This record isn't available to you. It may not exist, or it may belong to someone else." The record wording is the same for missing and forbidden records, so existence is not revealed (S5 ALL-01-J, STU-09-B). |
| Not found | No route matches | `EmptyState title="Page not found"` with `action` link to Home | |
| Error | Loader or API failure | `ViewState kind="error"` with `actions` = `Button` "Try again" (`router.invalidate()`) | Keep the person's input and place. |
| Loading | Route pending | `Skeleton` in the page's shape; `ViewState kind="loading"` names the wait ("Loading reports") | |
| Offline write | A save fails | `Alert tone="danger"` beside the form; draft kept | Never a toast only. |
