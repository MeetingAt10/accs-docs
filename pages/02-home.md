# Home dashboard

Back to [index](README.md) · [sitemap](../sitemap.md)

## P03 Home — `/home`

| | |
|---|---|
| File | `src/routes/_app/home.tsx` (one route; a variant component per role) |
| Permission | signed in |
| Modules | ALL-01 (role dashboard), STF-01, CHR-01, DEAN-01; previews of ALL-03, ALL-05, STU-05, STU-09, STU-10, PRO-05, PRO-06, GUD-01, OSAS-01, PRES-01 |
| Stories | US-03, US-04, US-08, US-53, US-54 |
| Tests | ALL-01-A, ALL-01-H, STF-01-V/B, CHR-01-V/I/B, DEAN-01-V/I/B |
| Template | T1 Dashboard |

**Purpose.** Land each role on the work that matters to it, with counts that match the lists they link to.

**Shared frame**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1}` | Title "Home"; description names the scope ("BSIT program, 1st semester 2026–2027"). |
| Several roles | `Tabs` with one `Tab` per role ("As Professor", "As Program Chair") | Only when the person holds two or more roles; each panel is that role's variant. |
| Stat row | DS-1 `Stat` ×3–4 in `Grid columns={4}` | Each stat links to the filtered list it counts. Zero shows "0", never a blank. |
| Body | `Grid columns="main"` (main column + rail), `Stack gap={32}` | |
| Region headings | `SectionHeader level={2} split` with a "See all" `Link` | |
| Loading | `Skeleton` per region | |

### Variants

**Student**

| Region | Components | Content |
|---|---|---|
| Today | `Timetable` (one day, `now="live"`) with `TimeBlock` (`lane="consult"` booked consultations, `lane="schedule"` CCS events) | `EmptyState` "Nothing scheduled today" with action "Find a consultation time" (P14). |
| Announcements | `AnnouncementCard` ×3, highest priority first | Link to P04 (Announcements filter). |
| From your classes | `ForumCard` ×3 | Latest posts in enrolled spaces (STU-01). |
| Rail: My reports | `ReportRecord` ×2 (neutral category, status) | Only active reports; link to P18. |
| Rail: Consultations | Rows with `Badge` (Requested, Approved, Booked) | Link to P14. |
| Rail: Evaluations | `Progress label="2 of 5 submitted"` + `Button` "Continue" | Only while a cycle is open (STU-10). |

**Organization representative**

| Region | Components | Content |
|---|---|---|
| Stat row | DS-1 | Members, posts this week, upcoming events. |
| Announcements | `AnnouncementCard lane="community"` + `Button` "New announcement" (P05) | |
| Upcoming events | `Agenda` with `AgendaItem lane="community"` | |
| Rail: Discussion | `ForumCard` ×2 | Link to P10. |

**Professor**

| Region | Components | Content |
|---|---|---|
| Stat row | DS-1 | Pending consultation requests, open assigned reports, evaluations to complete. |
| Today | `Timetable` (one day) with `TimeBlock form="open"` (availability) and `form="taken"` (booked), `NowLine` | Link to P17. |
| Consultation requests | `Table` (student, time, status `Badge`), 5 rows, row link to P16 | Decisions happen on P16, not inline. |
| Rail: Assigned reports | `ReportRecord` list (reference, neutral category, `Severity`, received) | Link to P21. Only reports assigned now (PRO-06). |
| Rail: Evaluations | Rows with due dates | Link to P26. |
| Rail: Post | `Button variant="secondary"` "Post to a class" | P05. |

**Staff** (STF-01; permissions pending D-06)

| Region | Components | Content |
|---|---|---|
| Announcements | `AnnouncementCard` list | CCS-wide only. |
| Discussions | `ForumCard` list | CCS-wide only. |
| Rail: Find people | `Search` → P37 | |
| Rail: Scope | `SidebarNote`-style `Alert tone="neutral"` | "Staff actions are being defined; you see CCS-wide content only." |

**Program Chair** (CHR-01)

| Region | Components | Content |
|---|---|---|
| Stat row | DS-1 | Open reports in the program, bypassed reports awaiting review, escalated to Dean this term, median days to resolve. |
| Report queue | `Table` (top 5 by severity and age; columns reference, category, `Severity`, stage, received) | Bypassed rows carry `Badge form="restricted"` "Adviser bypassed". Link to P21. |
| Sections | `Table` (section, adviser, students, open reports) | Rows link to P08. Only the Chair's program (CHR-01-B). |
| Rail: Similar reports | DS-1 `Stat` "Groups to review" | Link to P23. |
| Rail: Evaluations | `Meter` completion per open cycle | Link to P28. |
| Rail: Post | `Button variant="secondary"` "Post to the program" | P05. |

**Dean** (DEAN-01)

| Region | Components | Content |
|---|---|---|
| Stat row | DS-1 | Open reports (college), at Dean stage, transferred to Guidance this term, median days to resolve. |
| By program | DS-2 `BarChart` "Open reports by program and stage" with its table | Bars link to P25 filtered. |
| Escalated to you | `Table` preview (DEAN-02) | Link to P21. |
| Rail: Evaluation cycle | `Badge` (Draft, Published, Closed) + `Meter` completion | Link to P31. |
| Rail: Post | `Button variant="secondary"` "Post to CCS" | P05. |

**Administrator.** `/home` redirects to P45 `/admin`, which is the administrator's dashboard.

**Guidance**

| Region | Components | Content |
|---|---|---|
| Stat row | DS-1 | Transferred reports awaiting intake, active Case Files, restriction reviews. |
| Intake | `Table` (reference, from Dean, received, `Severity`) | Rows link to P22, where "Open Case File" happens (GUD-01). |
| Active Case Files | `Table` (Case File reference, next milestone, last action) | Link to P21. |
| Rail: Restriction reviews | Rows → P35 | |
| Rail: Today | `Agenda` from the calendar | |

**OSAS**

| Region | Components | Content |
|---|---|---|
| Stat row | DS-1 | Cases transferred to OSAS, active, Good Moral reviews, restriction reviews. |
| OSAS queue | `Table` preview | Link to P21. |
| Rail: Compliance reviews | Rows → P34 Good Moral tab | |

**President's Office**

| Region | Components | Content |
|---|---|---|
| Stat row | DS-1 | Executive cases open, awaiting action. |
| Executive queue | `Table` (reference, escalated from, date, summary status) | Link to P21. Only executive-stage cases (PRES-01). |

**States**

| Case | Result | Test |
|---|---|---|
| Each role signs in | Lands on its variant; role-only regions match the permission matrix | ALL-01-H, STF-01-V |
| Filter with no matches, empty period | Stats show 0; lists show `ViewState kind="empty"` | CHR-01-I, DEAN-01-I |
| Chair edits the URL to another program | No foreign counts or rows; restricted state | CHR-01-B |
| Staff opens a roster or report link | Restricted state | STF-01-B |
| Counts vs lists | Each stat equals the count on the list it links to | CHR-01-V, DEAN-01-V |

**Open.** Metric definitions (median days, bottleneck) are TBD (CHR-03); Staff content beyond CCS-wide (D-06).
