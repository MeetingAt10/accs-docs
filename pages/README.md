# ACCS page specs

**Status:** proposed, 2026-10-26. Index, roles, and access rules: [../sitemap.md](../sitemap.md).

| File | Pages |
|---|---|
| [01-shell-and-auth.md](01-shell-and-auth.md) | App shell, top bar, assistant drawer, command palette, P01 `/`, P02 `/sign-in`, system states |
| [02-home.md](02-home.md) | P03 `/home` and its ten role variants |
| [03-community.md](03-community.md) | P04 to P10: feed, composer, post, spaces, organizations |
| [04-messages.md](04-messages.md) | P11, P12: conversations |
| [05-schedule.md](05-schedule.md) | P13 to P17: calendar and consultations |
| [06-reports.md](06-reports.md) | P18 to P20: a student's own reports |
| [07-cases.md](07-cases.md) | P21 to P25: case queue, case workspace, similar reports, analytics |
| [08-evaluations.md](08-evaluations.md) | P26 to P32: forms, results, cycles |
| [09-records.md](09-records.md) | P33 to P36: student records, restrictions, own standing |
| [10-people-and-account.md](10-people-and-account.md) | P37 to P44: directory, profiles, notifications, assistant, help, settings |
| [11-admin.md](11-admin.md) | P45 to P52: administration |

## How to read a page spec

Each page has:

- **Facts:** file route, permission key (see [sitemap 1.5](../sitemap.md#15-permission-matrix)), modules (S2), user stories (S3), test cases (S5), template (sitemap 7.4).
- **Access by role:** only the roles that can open the page, with what each one sees and can do. Every other role gets the restricted state.
- **Layout:** regions from top to bottom, with the components for each. Plain names are design system exports (v1.0.1). `DS-n` is a component to add to the design system; `APP-n` is an ACCS component built in the app (sitemap 7.2 and 7.3).
- **States:** loading, empty, error, restricted, and the S5 cases the page must pass.
- **Open:** decisions or details the documents leave undecided.

## Access codes

✓ full use · S scoped to the role's program, assignment, stage, or membership · O own records only · R read only · † proposed, open decision · — restricted state.

## Shared conventions

- **Page heading:** `SectionHeader level={1}` in the content; `TopBar` repeats the title as a `p`.
- **Region states:** `ViewState` inside the region (`loading`, `empty`, `error`, `restricted`); `EmptyState` when a whole list has nothing yet and the person can act; `Skeleton` while the layout is known.
- **Confirmation:** `toast` for completed actions; `Alert` for anything people must read or fix; `Modal role="alertdialog"` for destructive or irreversible actions.
- **Tables on phones:** use labeled rows or cards below 768px, retaining key fields, status, and actions; `ReportRecord` is one pattern. P07 oversight sections, P14 faculty requests, P30 cycles, and P31 completion use cards; calendar month and week views use date-grouped event lists.
- **Message navigation on phones:** P11 shows the inbox list; opening a conversation switches to P12, which provides a "Back to conversations" link.
- **Restricted content:** `PrivacyNotice` states who can open what. It never promises anonymity or outcomes.
- **Sample data:** label synthetic records with `Badge outline marker={false}` "Sample data" while the prototype runs on simulated data.
