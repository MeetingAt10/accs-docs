# Shell, sign-in, and system states

Back to [index](README.md) · [sitemap](../sitemap.md)

## App shell (`src/routes/_app.tsx`)

Wraps P03 to P52. Reference composition: design system `site/app/_site/examples/Portal.tsx`.

| Region | Components | Notes |
|---|---|---|
| Frame | `AppShell` | `density="comfortable"` for Student and Organization; `density="compact"` for handler and admin roles; the person can change it on P43. Skip link `.ds-skip-link` to `contentId`. |
| Sidebar | APP-1 `RoleSidebar` = `Sidebar` + `SidebarSection` + `NavItem` (`icon`, `count`, `isCurrent`) | Brand `DoorPlate code="CCS"`. Sections and items per role: [sitemap 4](../sitemap.md#4-navigation-per-role). Counts: unread messages, due evaluations, pending requests, open cases, pending moderation. |
| Sidebar footer | `Avatar decorative`, name, one `RoleBadge` per role held, `SidebarNote` | The note states the scope in words: "Own reports only" (Student), "Reports assigned to you" (Professor), "BSIT program" (Chair), "Guidance stage cases" (Guidance). |
| Top bar | `TopBar` (`title`, `tools`) | Title repeats the page name as a `p`; while on simulated data add `Badge outline marker={false}` "Sample data". |
| Search | `Search size="sm" className="ds-topbar__search"` | Searches posts and spaces the person can read and opens P04 with `?q=`; a link there offers "Search people" (P37). Never searches reports, cases, or records. |
| Assistant | `IconButton icon="robot" label="Assistant"` → `Drawer side="end" title="Assistant"` with APP-17 | Passes the current page as context ("Explain this stage" on P20). Link to P41. |
| Notifications | `IconButton icon="bell" dot label="Notifications, 2 unread"` → `Popover title="Notifications"` with DS-4 `NotificationList` (latest 5), `Button variant="tertiary"` "Mark all read", `Link variant="standalone"` "See all" (P40) | The label states the count in words. |
| Account menu | `MenuButton iconOnly icon="user" placement="bottom end"` with `MenuItem` Profile (P39), Settings (P43), Help (P42), `MenuSeparator`, Sign out | Sign out ends the session and opens P02 with the signed-out message. |
| Phones | `MobileHeader` through `AppShell` (`mobileBrand={<DoorPlate code="CCS" size="sm" />}`, `mobileNav`, `mobileActions` = bell) | Sidebar items move into the drawer. |
| Restricted banner | APP-20 `RestrictedBanner` = `Alert tone="warning"` under the top bar | Only for a student with an active restriction (STU-11): what is paused, until when, and what still works; link to P36. |
| Jump | `CommandPalette` (Ctrl+K / Cmd+K) | Role-filtered destinations and actions ("New post", "New report", "Book a consultation"); every item also exists in the sidebar. |
| Toasts | `ToastRegion` (already in `__root.tsx`) | |

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

## System states (inside the shell)

| State | When | Components | Copy |
|---|---|---|---|
| Restricted | Permission key missing (APP-2 `RestrictedView`) | `ViewState kind="restricted"` | Page: "This page isn't available to your role." Record: "This record isn't available to you. It may not exist, or it may belong to someone else." The record wording is the same for missing and forbidden records, so existence is not revealed (S5 ALL-01-J, STU-09-B). |
| Not found | No route matches | `EmptyState title="Page not found"` with `action` link to Home | |
| Error | Loader or API failure | `ViewState kind="error"` with `actions` = `Button` "Try again" (`router.invalidate()`) | Keep the person's input and place. |
| Loading | Route pending | `Skeleton` in the page's shape; `ViewState kind="loading"` names the wait ("Loading reports") | |
| Offline write | A save fails | `Alert tone="danger"` beside the form; draft kept | Never a toast only. |
