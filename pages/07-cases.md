# Cases: queue, workspace, similar reports, analytics

Back to [index](README.md) · [sitemap](../sitemap.md) · stages and actions: [sitemap 5](../sitemap.md#5-case-stages-and-statuses-shared-model)

Handlers see a report only while it is at their stage (or in their program for the Chair). The reporter is shown by name or by alias ("Reporter A-12"), depending on the reporter's identity choice and disclosures.

---

## P21 Case queue — `/cases`

| | |
|---|---|
| File | `src/routes/_app/cases/index.tsx` (search params: `tab`, `status`, `severity`, `category`, `q`, `page`) |
| Permission | `case.view` |
| Modules | PRO-06, CHR-02, DEAN-02, GUD-01, GUD-02, OSAS-01, PRES-01 |
| Stories | US-30 to US-36 |
| Tests | PRO-06-I/B, CHR-02-I/B, DEAN-02-I, GUD-01-I, OSAS-01-I, PRES-01-V/I/B |
| Template | T3 Queue + detail |

**Access by role**

| Role | Access | Page title | Tabs (with counts) |
|---|---|---|---|
| Professor | S (assigned) | Assigned reports | Open, Waiting on reporter, Closed |
| Program Chair | S (program) | Program reports · BSIT | Bypassed (adviser conflict), Escalated to me, Handled by me |
| Dean | S (Dean stage) | Escalated reports | At Dean stage, Transferred, Closed |
| Guidance | S (Guidance stage) | Case Files | Intake (transferred), Active Case Files, Transferred to OSAS, Closed |
| OSAS | S (OSAS stage) | OSAS cases | New transfers, Active, Transferred, Closed |
| President | S (executive stage) | Executive cases | Awaiting action, Closed |
| Student, Organization, Staff, Administrator | — | | |

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1}` (title per role), description states the scope | |
| Boundary | `PrivacyNotice` (compact) | "You see only reports routed to you. Every action is recorded." |
| Tabs | `Tabs` with `count` | |
| Filters | DS-8 `FilterBar`: `Search label="Search by reference"`, `Select` severity, `Select` category, `Select` status, `Select` received (7 days, 30 days, term) | No search by student name. |
| Table (desktop, >=1024px) | `Table` with `allowsSorting`: Reference (row header, mono), Category, `Severity`, Status `Badge`, Received, Last action, Waiting (days) | Bypassed rows: `Badge form="restricted"` "Adviser bypassed". Guidance and later: Case File reference. Rows open P22. |
| Rows (phones and tablets, <1024px) | `ReportRecord` (`facts`: severity, received, last action, waiting days, office; adviser-bypassed routing where applicable) | Retains table status, reference, category, and `Open` action. |
| Sorting (phones and tablets) | Labeled `Select` controls for sort field and direction | Same sortable fields and ascending/descending behavior as the desktop table. |
| Paging | `Pagination` | |
| Empty | `EmptyState` per tab ("No reports assigned to you"); `ViewState kind="empty"` when filters match nothing | |

**States**

| Case | Result | Test |
|---|---|---|
| Report without a conflict | Stays with the adviser; not in the Chair's Bypassed tab | CHR-02-I |
| Report naming this professor | Never in their queue | PRO-06-B |
| Report below Dean stage | Not in the Dean's queue | DEAN-02-I |
| Report not transferred to Guidance or OSAS | Not in their queue | GUD-01-I, OSAS-01-I |
| Case below executive stage, or a guessed URL | Not in the executive queue; restricted record copy | PRES-01-I, PRES-01-B |
| Executive stage cases | Only assigned executive cases with authorized history | PRES-01-V |

---

## P22 Case workspace — `/cases/$caseId`

| | |
|---|---|
| File | `src/routes/_app/cases/$caseId.tsx` |
| Permission | `case.view`; actions need `case.act`; "Open Case File" needs `case.convert` |
| Modules | PRO-06, CHR-02, DEAN-02, GUD-01, GUD-02, OSAS-01, PRES-01, PRES-02; handler side of STU-07 |
| Stories | US-30 to US-37 |
| Tests | PRO-06-V/I/B, CHR-02-V/B, DEAN-02-V/I/B, GUD-01-V/I/B, GUD-02-V/I/B, OSAS-01-V/I/B, PRES-01-V/B, PRES-02-V/I/B |
| Template | T7 Record |

**Access by role.** The handler at the current stage: S, with that stage's actions (table below). Earlier handlers: read-only summary of their own entries under "Handled by me" (†, D-04/D-08). The conflicted adviser, other programs, and other stages: restricted record copy.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Trail | `Breadcrumbs` (Cases / RPT-0241) | |
| Heading | `SectionHeader level={1} split title="General concern" description="Report RPT-0241 · received 21 Oct"`, `actions` = stage `Badge`, `Severity`, Case File `Badge` (Guidance and later) | |
| Conflict | APP-9 `ConflictNotice` = `Alert tone="warning" title="Routed to you because the assigned adviser may be involved"` "Detection only suggests routing. It isn't a finding." | Bypassed reports (CHR-02). |
| Body | `Grid columns="main"` | |
| Tabs (main) | `Tabs`: Report, Messages, Notes, History | |
| Report | `dl.ds-record__facts` (reference, category, severity, program, submitted, reporter name or alias, identity mode, linked discussion), narrative in `.ds-prose`, attachments | |
| Messages | DS-3 `MessageList` + `MessageComposer` with the reporter; `PrivacyNotice` "The reporter sees messages you send here." | Alias until the reporter discloses (STU-07). |
| Notes | APP-8 `CaseNotes`: notes with `Badge form="restricted"` "Internal", `TextArea label="Add a note"`, `Button` "Save note"; `PrivacyNotice` "Internal notes never appear in the reporter's timeline." | GUD-02-B. |
| History | `CaseTimeline` (actor role, action, reason, time); entries the reporter can see carry `Badge outline marker={false}` "Shown to reporter" | Route history and conflict reason (CHR-02). |
| Actions (rail) | APP-7 `CaseActionBar`: the stage's actions as `Button`s, the rest in a `MenuButton` | Each opens a `Modal`: `Select` action or next office (allowed transitions only), `TextArea label="Reason" isRequired`, `TextArea label="Update shown to the reporter" isRequired` for transitions. Transfers and resolution use `role="alertdialog"`. Success: `toast` ("Escalated to the Dean") and a new timeline entry. |
| Similar reports | `Alert tone="info"` "3 similar reports in BSIT this month" + `Link` (P24) | Chair, when a group exists (CHR-07). |
| Facts (rail) | `dl.ds-record__facts`: current office, handler role, days at this stage | |

**Actions by stage**

| Stage | Handler | Primary | Others |
|---|---|---|---|
| 1 Adviser | Professor | Respond | Record action, Escalate to Program Chair, Resolve |
| 2 Program Chair | Chair | Record decision | Resolve, Escalate to Dean |
| 3 Dean | Dean | Record decision | Resolve, Transfer to Guidance (or another authorized stage) |
| 4 Guidance, intake | Guidance | Open Case File | |
| 4 Guidance, active | Guidance | Add milestone | Resolve, Transfer to OSAS |
| 5 OSAS | OSAS | Add milestone | Resolve, Transfer to President's Office |
| 6 Executive | President | Record action | Return, Resolve, Transfer (as permitted) |

**States**

| Case | Result | Test |
|---|---|---|
| Respond and take a next action | Response and stage history persist | PRO-06-V |
| Unassigned report | Restricted record copy | PRO-06-I |
| Conflicted adviser opens it | Restricted record copy | PRO-06-B, CHR-02-B |
| Chair records a review on a bypassed report | Reason and history shown; action recorded | CHR-02-V |
| Dean records action and transfers | Route history and stage update | DEAN-02-V |
| Unsupported bypass, or asking the assistant for an outcome | Not offered; no automatic finding | DEAN-02-B |
| Guidance accepts a transferred report | New Case File ID linked to the report; earlier history kept; notices sent | GUD-01-V |
| "Open Case File" on a report not transferred to Guidance | Not offered; a crafted call is denied | GUD-01-I |
| A new report at submission | Stays a report, never a Case File | GUD-01-B |
| Missing status, reason, or required fields | Field errors; no partial history entry | GUD-02-I, PRES-02-I |
| Reporter opens an internal note, or the assistant proposes a verdict | Note never sent to P20; a person decides | GUD-02-B |
| OSAS accepts a transfer and records an action | Officer, time, stage, and a public-safe milestone saved | OSAS-01-V |
| Transfer without an authorized rule or reason | Not offered; no silent stage change | OSAS-01-B |
| Executive action with an undefined power | Not offered until the rule exists | PRES-02-B |

**Open.** Allowed transitions and bypass (D-04); what earlier handlers keep; whether any handler may resolve an alias (D-08); required fields per action.

---

## P23 Similar reports — `/cases/clusters`

| | |
|---|---|
| File | `src/routes/_app/cases/clusters/index.tsx` |
| Permission | `cluster.review` |
| Modules | CHR-07 |
| Stories | US-38 |
| Tests | CHR-07-V/I |
| Template | T3 Queue |

**Access.** Program Chair S (program). Dean, Guidance, OSAS: † ("authorized reviewer" in US-38). Everyone else —.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} title="Similar reports" description="Groups suggested by text similarity in BSIT. Suggestions, not findings."` | |
| Threshold | `Alert tone="neutral"` "Grouping uses a draft threshold (80%) that hasn't been approved." | D-07. |
| Filters | DS-8 `FilterBar`: `Select` status (To review, Confirmed, Dismissed), `Select` period | |
| Groups | APP-10 `ReportClusterCard` = `Card` with neutral theme title ("Lab scheduling, 4 reports"), `Meter label="Similarity" valueLabel="86% similar"`, `CardMeta` date range, status `Badge` | Opens P24. |
| Empty | `EmptyState title="No similar reports right now"` | No group without a meaningful match (CHR-07-I). |

---

## P24 Group review — `/cases/clusters/$clusterId`

| | |
|---|---|
| File | `src/routes/_app/cases/clusters/$clusterId.tsx` |
| Permission | `cluster.review` |
| Modules | CHR-07 |
| Tests | CHR-07-V/B |
| Template | T7 Record |

**Layout**

| Region | Components | Notes |
|---|---|---|
| Trail | `Breadcrumbs` (Similar reports / Lab scheduling) | |
| Heading | `SectionHeader level={1} split`, `actions` = status `Badge` | |
| Caution | `Alert tone="neutral"` "These summaries come from the reports below. Read the reports before deciding anything." | CHR-07-B. |
| Basis | Shared phrases with `Highlight`, `Meter` similarity, threshold version | Similarity basis (CHR-07). |
| Sentiments | `Grid columns={2}`: "Supporting" and "Dissenting" summaries in `.ds-prose` | Both always shown. |
| Reports | `ReportsTable caption="Reports in this group"` | Only reports the reviewer may open; rows open P22. |
| Decision | APP-7 subset: `Button` "Confirm group", `Button variant="secondary"` "Dismiss group" (`Modal` with a required reason), `TextArea label="Note"` | |
| History | `CaseTimeline` of review actions | |

**States.** A group never spans programs the reviewer can't see; "declare true" is not an action (CHR-07-B). Reviewing shows both sentiments, basis, and reports (CHR-07-V).

---

## P25 Analytics — `/analytics`

| | |
|---|---|
| File | `src/routes/_app/analytics.tsx` (search params: `period`, `program`, `stage`, `category`) |
| Permission | `analytics.view` |
| Modules | CHR-03, DEAN-01 |
| Stories | US-53, US-54 |
| Tests | CHR-03-V/I/B, DEAN-01-V/I/B |
| Template | T1 Dashboard |

**Access by role**

| Role | Access | Scope |
|---|---|---|
| Program Chair | S | Own program; the program filter is fixed |
| Dean | ✓ | College; filter by program |
| Everyone else | — | |

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} title="Analytics" description="BSIT program · counts only"` | |
| Filters | DS-8 `FilterBar`: `Select` period (This term, Last 30 days, Custom with `DatePicker`), `Select` program (Dean), `Select` stage, `Select` category | |
| Stats | DS-1 `Stat` ×5: open, escalated, resolved, median days to resolve, bypassed | Each links to P21 filtered. |
| Over time | DS-2 `TrendChart` "Opened and resolved" | Table fallback. |
| Bottlenecks | DS-2 `BarChart` "Median days waiting by stage" | |
| By category | DS-2 `BarChart` | |
| Oldest open | `Table` (reference, stage, days waiting), rows open P22 | Drill-down only to authorized reports. |
| Privacy | `PrivacyNotice` "Counts never show who reported. Groups under 5 are hidden." | Small-number hiding is a proposal (†). |

**States**

| Case | Result | Test |
|---|---|---|
| Choose a period or stage | Counts and rates match the timestamped reports; drill-down works | CHR-03-V, DEAN-01-V |
| Period with no records | Zeros and `ViewState kind="empty"`; never another period's data | CHR-03-I, DEAN-01-I |
| Other program, or identities beyond permission | Restricted; details hidden | CHR-03-B, DEAN-01-B |

**Open.** Metric formulas (CHR-03); small-number hiding.
