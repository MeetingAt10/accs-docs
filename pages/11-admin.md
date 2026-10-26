# Administration

Back to [index](README.md) · [sitemap](../sitemap.md)

The QCU Administrator / Moderator manages roles, permissions, moderation, and platform monitoring for the CCS deployment. Administrators manage configuration and moderation. They never decide case outcomes or disciplinary findings (ADM-01 boundary), and admin analytics never show confidential report narratives or message text (ADM-04). Every admin change is written to the audit trail (P51).

The admin layout guard is `src/routes/_app/admin.tsx`. It lets a person in only with `role.manage`, `moderation.review`, or `audit.view`. Each page then checks its own key. The Administrator's `/home` redirects to P45 `/admin` (see [02-home.md](02-home.md)).

---

## P45 Admin overview: `/admin`

| | |
|---|---|
| File | `src/routes/_app/admin/index.tsx` |
| Permission | `audit.view` |
| Modules | ADM-04, ADM-05, ADM-02 |
| Stories | US-55 |
| Tests | ADM-04-V/I/B |
| Template | T1 Dashboard |

**Access.** Administrator ✓. Every other role gets the restricted state.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1}` "Administration", description "ACCS platform · CCS pilot" | |
| Status | APP-16 `SystemStatus`: status `Badge` (Operational, Degraded, Down), uptime `Meter` for the last 30 days | Opens P52. |
| Stats | DS-1 `Stat` row: active users this week (by role), posts this week, flags awaiting review, restriction change requests | Flags link to P49. Restriction requests link to P35. |
| Sign-ins | DS-2 `TrendChart` "Sign-ins per day by role" | Table fallback. |
| Body | `Grid columns="main"` | |
| Main | "Moderation queue" `Table` preview, 5 rows | Rows open P50; a link opens P49. |
| Rail | "Recent admin actions": APP-15 `AuditLog` compact, 5 rows | Links to P51. |
| Rail | Assistant DS-1 `Stat`s: sessions, failures | Links to P52. |

**States**

| Case | Result | Test |
|---|---|---|
| Period with no events | Zeros in every `Stat`, empty chart | ADM-04-I |
| Aggregates | Counts only; no confidential report narratives or message text | ADM-04-B |
| Counts and the lists they link to | Counts match the filtered list on the target page | ADM-04-V |

**Open.** How "restriction change requests" are raised is not defined in ADM-03; the source of that count is TBD (D-05).

---

## P46 Users and roles: `/admin/users`

| | |
|---|---|
| File | `src/routes/_app/admin/users/index.tsx` |
| Permission | `role.manage` |
| Modules | ADM-01 |
| Stories | US-05 |
| Tests | ADM-01-V/I |
| Template | T3 Queue + detail |

**Access.** Administrator ✓. Every other role gets the restricted state.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} split` "Users and roles"; `actions` = `Button` "Assign role" | The button opens the P47 add-role dialog after a user is picked with `ComboBox`. |
| Boundary | `PrivacyNotice` "Role changes take effect at the next sign-in and are recorded." | ALL-01-I. |
| Filters | DS-8 `FilterBar`: `Search` by name or QCU ID, `Select` role, `Select` program or office, `Select` status (Active, Inactive) | |
| Table | `Table`: `Avatar`, name, QCU ID, roles as `RoleBadge`s, program or office, status `Badge`, last sign-in | Rows open P47. |
| Rows (phones) | `ReportRecord` rows, or labeled rows | |
| Paging | `Pagination` | |
| Empty | `ViewState kind="empty"` when no user matches | |

**States**

| Case | Result | Test |
|---|---|---|
| Assign a role to a user | Recorded with actor, time, and reason; applies at next sign-in | ADM-01-V |
| Missing or invalid assignment | Field errors; no partial permission change | ADM-01-I |

**Open.** Accounts come from the simulated QCU source. How accounts are created and how duplicates are prevented is TBD (ALL-01-F, pending production identity policy).

---

## P47 User detail: `/admin/users/$userId`

| | |
|---|---|
| File | `src/routes/_app/admin/users/$userId.tsx` |
| Permission | `role.manage` |
| Modules | ADM-01 |
| Stories | US-05 |
| Tests | ADM-01-V/I/B, ALL-01-I |
| Template | T7 Record |

**Access.** Administrator ✓. Every other role gets the restricted state.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Trail | `Breadcrumbs` (Users and roles / name) | |
| Heading | DS-7 `ProfileHeader`, admin view with QCU ID and status | |
| Role assignments | "Role assignments" `Table`: `RoleBadge`, scope (program, section, office, or organization), since, assigned by, `MenuButton` | "Remove" opens `Modal role="alertdialog"` with a required reason. |
| Add role | `Button` "Add role" opens a `Modal`: `Select` role; scope fields that depend on the role; required `TextArea` "Reason"; `Alert tone="neutral"` "Takes effect at the next sign-in." | Scope fields: `Select` program for Program Chair, `ComboBox` sections for Professor, `Select` office for Guidance and OSAS, `ComboBox` organization for Organization. |
| Account status | "Account status" `Select` (Active, Inactive) † | Depends on the identity policy (ALL-01). |
| Activity | "Activity": APP-15 `AuditLog` filtered to this user | |

**States**

| Case | Result | Test |
|---|---|---|
| Approved role or program change | Next sign-in reflects it; actor, time, and reason are logged | ADM-01-V, ALL-01-I |
| Missing or invalid assignment | No partial permission change | ADM-01-I |
| Looking for a way to decide case guilt or a disciplinary outcome | No such control exists on the page | ADM-01-B |

**Open.** When an existing session picks up a changed assignment is TBD (ALL-01-I; record observed timing). Account status control: identity policy.

---

## P48 Permissions: `/admin/roles`

| | |
|---|---|
| File | `src/routes/_app/admin/roles.tsx` |
| Permission | `role.manage` |
| Modules | ADM-01 |
| Tests | ADM-01-B |
| Template | Single table page (no template) |

**Access.** Administrator ✓. Every other role gets the restricted state.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1}` "Permissions", description "Which roles can do what." | |
| Caution | `Alert tone="warning"` "This matrix is proposed. Changes need approval from CCS (D-06)." | |
| Matrix | APP-14 `PermissionMatrix`: one row per permission key from [sitemap 1.5](../sitemap.md#15-permission-matrix) (key in mono plus a plain description), one column per role; cells show ✓, scope words, or — | Read-only by default. |
| Edit | `Button` "Edit" switches the cells to `Checkbox`. "Save" opens a `Modal` with a summary of changes and a required `TextArea` "Reason". | Saved changes go to the audit trail. |

**States**

| Case | Result | Test |
|---|---|---|
| Open the page | Matrix is read-only until "Edit" is chosen | ADM-01 acceptance (review permission matrix) |
| Save without a reason | Blocked; nothing changes | ADM-01 acceptance |
| Save with a reason | Every changed cell is logged with actor, time, and reason | ADM-01 acceptance |
| Looking for a case or disciplinary decision permission | The matrix holds access keys only; no key decides outcomes | ADM-01-B |

**Open.** Approval workflow for permission changes (D-06). Staff and Organization defaults depend on the same decision.

---

## P49 Moderation queue: `/admin/moderation`

| | |
|---|---|
| File | `src/routes/_app/admin/moderation/index.tsx` |
| Permission | `moderation.review` |
| Modules | ADM-02, ADM-06 |
| Stories | US-18, US-19 |
| Tests | ADM-02-I, ADM-06-I |
| Template | T3 Queue + detail |

**Access.** Administrator ✓. Every other role gets the restricted state.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1}` "Moderation" | |
| Boundary | `PrivacyNotice` "A flag or a report is not proof. Decide after reading the context." | |
| Tabs | `Tabs` with `count`: Flagged by filter (ADM-02 word filter and AI flags), Reported by users (ADM-06), Decided | |
| Filters | DS-8 `FilterBar`: `Select` type (Post, Comment, Message), `Select` reason, `Select` source (Filter, User), `Select` period | |
| Table | `Table`: item type, space, flagged text excerpt only, reason `Badge`, source, received, status `Badge` | Rows open P50. |
| Rows (phones) | `ReportRecord` rows, or labeled rows | |
| Paging | `Pagination` | |
| Empty | `EmptyState` "Nothing to review" | |

**States**

| Case | Result | Test |
|---|---|---|
| Unsubstantiated flag dismissed | Leaves the queue; no restriction or grievance created | ADM-02-I |
| Dismissed user complaint | No formal grievance created | ADM-06-I |

**Open.** Whether flagged posts are held or published with a flag is undecided (sitemap 9).

---

## P50 Moderation review: `/admin/moderation/$itemId`

| | |
|---|---|
| File | `src/routes/_app/admin/moderation/$itemId.tsx` |
| Permission | `moderation.review` |
| Modules | ADM-02, ADM-06 |
| Stories | US-18, US-19 |
| Tests | ADM-02-V/I/B, ADM-06-V/I/B |
| Template | T7 Record |

**Access.** Administrator ✓. Every other role gets the restricted state.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Trail | `Breadcrumbs` (Moderation / item reference) | |
| Heading | `SectionHeader level={1} split` (for example "Flagged comment", description "In BSIT 3A · 24 Oct"); `actions` = status `Badge` and source `Badge` | |
| Context | APP-4 `PostView` with DS-9 `CommentThread`, or DS-3 `MessageBubble`s for a reported message; flagged text marked with `Highlight` | Read-only. |
| Flag details | `dl.ds-record__facts`: reason, source, matched filter rule, reporter (user reports only) | Whether the moderator sees the reporter's name is TBD (D-08). |
| Caution | `Alert tone="neutral"` "A flag is not proof." | |
| Decision | `RadioGroup` (card): Keep (no action), Remove content, Remove and warn the author, Refer as a possible grievance (user reports only, ADM-06); required `TextArea` "Reason"; `Checkbox` "Notify the author" (as permitted); `Button` "Record decision" | Remove and refer use `Modal role="alertdialog"`. A referral creates a linked record only after approval. |
| History | `CaseTimeline` | |
| Restriction | `Link` "Apply a restriction" to P35 | Restrictions are not applied here. P35 needs an approved basis (D-05). |

**States**

| Case | Result | Test |
|---|---|---|
| Decision recorded | Decision and time saved; permitted parties notified | ADM-02-V, ADM-06-V |
| Unsupported flag or complaint dismissed | No restriction or grievance created | ADM-02-I, ADM-06-I |
| Flag alone | No automatic punishment | ADM-02-B |
| Referral on public reactions or an AI flag alone | Blocked | ADM-06-B |

**Open.** Referral rules and party notices; who sees reporter identity (D-08); the policy for serious referrals (ADM-02-B).

---

## P51 Audit trail: `/admin/audit`

| | |
|---|---|
| File | `src/routes/_app/admin/audit.tsx` |
| Permission | `audit.view` |
| Modules | ADM-01, ADM-04, plus record-access logs from CHR-05, DEAN-05, OSAS-02 |
| Stories | US-55 |
| Tests | ADM-04-V/I/B |
| Template | T3 Queue + detail |

**Access.** Administrator ✓. Every other role gets the restricted state.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1}` "Audit trail" | |
| Boundary | `PrivacyNotice` "The log shows references, never report narratives or message text." | ADM-04. |
| Filters | DS-8 `FilterBar`: `DatePicker` from and to, `Select` module, `ComboBox` actor, `Select` action (Role change, Permission change, Restriction, Case transition, Moderation decision, Identity disclosure, Record opened) | |
| Log | APP-15 `AuditLog` (`Table`): time, actor with `RoleBadge`, action, module, record reference, result | Record opened entries come from CHR-05, DEAN-05, and OSAS-02. |
| Rows (phones) | `ReportRecord` rows, or labeled rows | |
| Paging | `Pagination` | |
| Empty | `ViewState kind="empty"` | |

**States**

| Case | Result | Test |
|---|---|---|
| Filter by time and module | Matching entries and counts shown | ADM-04-V |
| Period with no events | Zero or empty view | ADM-04-I |
| Entry for a confidential record | Reference only; no narrative or message text | ADM-04-B |

**Open.** Retention period (D-08).

---

## P52 Monitoring: `/admin/monitoring`

| | |
|---|---|
| File | `src/routes/_app/admin/monitoring.tsx` |
| Permission | `audit.view` |
| Modules | ADM-05, ADM-04 |
| Stories | US-55 |
| Tests | ADM-05-V/I/B |
| Template | T1 Dashboard |

**Access.** Administrator ✓. Every other role gets the restricted state.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1}` "Monitoring" | |
| Tabs | `Tabs`: Uptime, Assistant | |
| Uptime | APP-16 `SystemStatus`; DS-2 `TrendChart` availability per day; `Table` of incidents (start, end, duration, service, note) | |
| Assistant | DS-1 `Stat`s (sessions, failures, median response time); DS-2 `TrendChart` sessions per day; `Table` of errors by category (time, category, count) | No chat content. |
| Notice | `Alert tone="neutral"` "Chat content isn't shown here. Access to conversation logs needs separate approval." | Assistant tab. |

**States**

| Case | Result | Test |
|---|---|---|
| Period with telemetry | Metrics match the telemetry for that period | ADM-05-V |
| Period with no data | Empty or zero view; uptime is never filled in | ADM-05-I |
| Opening confidential chat content without authorization | Denied | ADM-05-B |

**Open.** Log access and retention policy (D-08).
