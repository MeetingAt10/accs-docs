# ACCS sitemap, access, and components

**Status:** proposed, 2026-10-26. Built from the team's documents; not an approved spec.
**App:** `accs-system/frontend` (React 19, Vite, TanStack Router file routes). The public landing page stays in `accs-landing`.
**Design system:** `@meetingat10/design-system` v1.0.1.
**Page specs:** one file per area in [`pages/`](pages/README.md). This file is the index: roles, access, routes, navigation, and components.

## Sources

| ID | Document | Used for |
|---|---|---|
| S1 | `FILE001: Draft-v1` (Drive, edited 2026-10-24), Chapter 1 objectives, scope, delimitations; 3.1.3 population | Feature scope, ten roles, prototype limits |
| S2 | `DOC003_MODULE-INVENTORY.xlsm`, Module Inventory + Open Decisions sheets | 58 modules (5 shared + 53 role modules): functional specs, boundaries, data; decisions D-01 to D-08 |
| S3 | `DOC002_USER-STORIES.md`, US-01 to US-55 + workflow acceptance rules | Page behavior, access scope |
| S4 | `DOC004_USER-STORIES-PER-MODULE.xlsx` | Story, acceptance check, and boundary per module |
| S5 | `DOC005_BLACK-BOX-TEST-BLUEPRINT.docx` (2026-10-23) | Valid, invalid, and boundary cases (183) each page must pass |
| S6 | `meetingatten-design-system` v1.0.1 specs (`components/**`) and React exports | Component names, variants, states |

Prototype limits from S1: CCS only (IT, CS, IS); web only; official QCU email only, no personal phone or external messaging; simulated enrollment data and sample records, no live registrar connection (Delimitation 5); the AI assistant supports processes and never decides. The registration flow, password recovery, lockout, and session expiry are TBD (S5 ALL-01-F, G, K, L).

## How the app is organized

One app, one set of routes, organized by feature. The signed-in person's roles decide which navigation items appear, which records each page lists, and which actions show on a record. A page never relies on hiding: the API checks role and relationship again on every call, and a record the person may not open looks the same as a record that does not exist ("This record isn't available to you").

---

## 1. Users and access control (RBAC)

RBAC is objective 2 and scope item 1 of the paper (S1), and "Role-Based Access Control" is in its definition of terms.

### 1.1 The ten roles

| # | Role | Who | Scope attribute | Pilot accounts (S5 fixtures) |
|---|---|---|---|---|
| 1 | Student | Enrolled BSIT, BSCS, BSIS undergraduates | program, year, section, subjects, organization memberships | 3+ (BSIT section A, BSIT section B, BSCS); one with a restriction |
| 2 | CCS Student Organization | Organization representative account | organization | 1 representative; members are Student accounts; 1 nonmember |
| 3 | Professor | CCS teaching faculty | handled sections and subjects; advisory section | 2+ (assigned adviser, another professor) |
| 4 | CCS Staff | CCS office staff | college | 1 |
| 5 | Program Chair | One each for IT, CS, IS | program | 2+ (two programs, to test isolation) |
| 6 | Dean | CCS Dean | college | 1 |
| 7 | QCU Administrator / Moderator | Platform administrator and content moderator | platform | 1 |
| 8 | Guidance | Guidance Office personnel | Case Files at the Guidance stage; students in Guidance cases | 1 |
| 9 | OSAS | Office of Student Affairs and Services personnel | cases transferred to OSAS; students under OSAS authority | 1 |
| 10 | Office of the University President | Authorized executive office user | executive-stage cases only | 1 |

The paper's survey population (3.1.3) lists six groups: students, professors, Program Chairs, Dean, Guidance, and OSAS. Organization, Staff, Administrator, and President accounts are system roles, not survey respondents. 3.1.3 says "San Bartolome Campus" while Delimitation 1 says "Main Campus"; the paper should settle on one.

### 1.2 Access is role plus relationship

| Check | Rule | Sources |
|---|---|---|
| Membership | A student enters a section, subject, or class chat only while enrolled; a former enrollment loses access. Private organization posts need membership. | STU-01, ORG-01, S5 STU-01-I/B |
| Assignment | A professor sees a report only while it is assigned to them. | PRO-06 |
| Program | A Program Chair sees only their program's sections, reports, analytics, evaluations, and offense records. | CHR-01 to CHR-05 |
| Case stage | Dean, Guidance, OSAS, and President see a case only from the stage where it reached them. | DEAN-02, GUD-01, OSAS-01, PRES-01 |
| Ownership | A student sees only their own reports, requests, evaluations, to-dos, and standing. | STU-04, STU-05, STU-09, STU-11 |
| Conflict | A report naming the student's adviser bypasses that adviser, who must not see it; it goes to the Program Chair. | PRO-06, CHR-02 |
| Account state | A restricted student loses only the restricted features (posting, commenting, anonymous reporting) and keeps announcements, academic information, the assistant, and permitted professor messages. | STU-11, ADM-03, US-49 |

### 1.3 Rules

1. **Deny by default.** A route or action is allowed only when a rule grants it.
2. **Check on the server.** Hiding a menu item is not access control. Every API call checks role and relationship; direct links, search results, and notifications are tested the same way (S5 ALL-01-E, ALL-01-J, ALL-05-I).
3. **Multiple roles.** One person can hold several roles, for example a Program Chair or Dean who also teaches. Permissions are the union of the roles, and the sidebar shows each role's section. The conflict check still applies per record.
4. **Assignments change.** After an administrator changes a role or program, the next sign-in uses only the new assignment (S5 ALL-01-I).
5. **Anonymous is not unaccountable.** Anonymous posts and reports still belong to a verified account; the identity is hidden only from audiences that should not see it (workflow rule 5).
6. **People decide.** The assistant, word filters, and similarity summaries never apply restrictions, offenses, or case outcomes (workflow rule 7).
7. **Audit.** Role changes, restrictions, case transitions, moderation decisions, identity disclosures, and every opening of an offense record are logged with who, what, and when (ADM-01, CHR-05, DEAN-05, OSAS-02).
8. **Notifications leak nothing.** A notification names the kind of event, never confidential details, and opening it re-checks access (ALL-05).

### 1.4 Who can see or reach whom

**Messaging reach** (STU-03, PRO-03, CHR-06, DEAN-06, GUD-05, OSAS-04):

| From | Can message | Not allowed or TBD |
|---|---|---|
| Student | Own professors; peers within permitted scope; own section and subject group chats | Unrelated class chats |
| Professor | Assigned students; faculty colleagues; own class group chats | Unrelated class groups |
| Program Chair | Faculty in the program | Students only in approved special circumstances (TBD) |
| Dean | Faculty | Students only in approved special circumstances (TBD) |
| Guidance, OSAS | Authorized faculty | Wider rights TBD; case details stay in the case record |
| Organization, Staff | TBD (D-06) | |
| Administrator, President | None defined | |

**Record visibility:**

| Record | Who sees it |
|---|---|
| A student's report | The student; the handler at the current stage; earlier handlers keep their own history entries. Never the conflicted adviser. |
| Internal case notes | Handlers with case access; never the reporter's timeline (GUD-02). |
| Public-safe milestones | The reporter and handlers (STU-09). |
| Offense history | Program Chair (program), Dean (college), Guidance and OSAS (authorized students). Each opening is logged. Never peers, never public profiles. |
| Account standing | The student in full; peers see only a neutral standing badge (US-47; wording TBD with D-08). |
| Evaluation answers | Nobody individually; professors, Chairs, and the Dean see published aggregates within scope (STU-10, PRO-08). |
| Personal to-dos | The owner only (STU-04). |

**Identity visibility of anonymous participants** (STU-02, STU-07, workflow rule 5): classmates and handlers see an alias; the verified account stays linked in authorized records. Which roles may resolve the alias (for example moderators or Guidance) is not defined; it belongs to D-08. A reporter's disclosure is an explicit, logged action, per handler.

### 1.5 Permission matrix

✓ = allowed · S = within the role's scope (1.1) · O = own records only · — = not allowed · † = proposed, depends on an open decision (mostly D-06).

| Key | Permission | Stu | Org | Prof | Staff | Chair | Dean | Admin | Guid | OSAS | Pres |
|---|---|---|---|---|---|---|---|---|---|---|---|
| `feed.read` | Read feed and announcements | S | S | S | S | S | S | ✓ | S | S | S |
| `post.create` | Post and comment | S | S | S | † | S | S | — | S | S | — |
| `post.anonymous` | Post or comment anonymously | S | — | — | — | — | — | — | — | — | — |
| `announcement.publish` | Publish announcements | — | org | class | † | program | college | — | — | — | — |
| `message.send` | Direct and group messages | S | † | S | † | S | S | — | S | S | — |
| `roster.view` | View rosters | own sections | members | handled | — | program | college | — | — | — | — |
| `calendar.view` | View calendars | ✓ | S† | ✓ | — | ✓ | ✓ | — | ✓ | ✓ | — |
| `todo.manage` | Personal to-dos | O | — | — | — | — | — | — | — | — | — |
| `consultation.request` | Request consultation | ✓ | — | — | — | — | — | — | — | — | — |
| `consultation.decide` | Manage availability, decide requests | — | — | O | — | — | — | — | — | — | — |
| `report.create` | Submit report | ✓ | — | — | — | — | — | — | — | — | — |
| `report.viewOwn` | View own reports, disclose identity | O | — | — | — | — | — | — | — | — | — |
| `case.view` | View routed reports or cases | — | — | assigned | — | program | Dean stage | — | Guidance stage | OSAS stage | executive stage |
| `case.act` | Respond, record action, escalate, transfer, resolve | — | — | assigned | — | program | Dean stage | — | Guidance stage | OSAS stage | executive stage |
| `case.convert` | Convert a report to a Case File | — | — | — | — | — | — | — | ✓ | — | — |
| `cluster.review` | Review similar-report groups | — | — | — | — | program | † | — | † | † | — |
| `analytics.view` | Grievance analytics | — | — | — | — | program | college | — | — | — | — |
| `evaluation.fill` | Fill evaluations | assigned | — | peer, upward | — | † | † | — | — | — | — |
| `evaluation.review` | Review proposed questions | — | — | ✓ | — | † | — | — | — | — | — |
| `evaluation.configure` | Configure and publish cycles | — | — | — | — | — | ✓ | — | — | — | — |
| `evaluation.results` | View published results | — | — | own | — | program | college | — | — | — | — |
| `offense.view` | View offense history | — | — | — | — | program | college | — | S | S | — |
| `goodmoral.review` | Record Good Moral prerequisite status | — | — | — | — | — | — | — | S | S | — |
| `restriction.manage` | Apply or lift restrictions | — | — | — | — | — | — | ✓ | refer † | S † | — |
| `standing.viewOwn` | View own account standing | O | — | — | — | — | — | — | — | — | — |
| `moderation.review` | Review flagged and reported content | — | — | — | — | — | — | ✓ | — | — | — |
| `role.manage` | Assign roles, configure permissions | — | — | — | — | — | — | ✓ | — | — | — |
| `audit.view` | Audit, adoption, assistant, uptime | — | — | — | — | — | — | ✓ | — | — | — |
| `directory.view` | Directory and profiles | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| `self.manage` | Own profile, notifications, settings | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| `assistant.use` | AI assistant (answers within own access) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

Sources: S2 module rows, S3 stories and workflow rules. "refer" means Guidance reviews and refers an apply or lift decision (GUD-04). Who may apply restrictions also depends on D-05. Chair and Dean evaluation forms are TBD: scope item 5 says evaluations run "among faculty, Program Chairs, and the Dean" without naming the forms.

---

## 2. Sitemap

Page IDs (P01 to P52) are used in every page spec.

```
accs-landing (public, separate site)
└── /                                   Product landing

accs-system/frontend
├── P01 /                               Redirect: /home when signed in, else /sign-in
├── P02 /sign-in                        Sign in (no app shell)
│        (not found, error, and restricted are states inside the shell, not routes)
│
├── P03 /home                           Dashboard, one page with role variants
│
├── Community ................................................ pages/03-community.md
│   ├── P04 /feed                       Feed and announcements
│   ├── P05 /feed/new                   New post or announcement
│   ├── P06 /feed/$postId               Post and comments
│   ├── P07 /spaces                     My classes / teaching load / program sections
│   ├── P08 /spaces/$spaceId            Section or subject space
│   ├── P09 /organizations              CCS organizations
│   └── P10 /organizations/$orgId       Organization space
│
├── Messages ................................................. pages/04-messages.md
│   ├── P11 /messages                   Conversations
│   └── P12 /messages/$threadId         Conversation
│
├── Schedule ................................................. pages/05-schedule.md
│   ├── P13 /calendar                   Calendars and to-dos
│   ├── P14 /consultations              Requests (student) / incoming requests (professor)
│   ├── P15 /consultations/faculty/$facultyId    A professor's open times
│   ├── P16 /consultations/requests/$requestId   Request detail
│   └── P17 /consultations/availability          Manage availability (professor)
│
├── My reports (student) ..................................... pages/06-reports.md
│   ├── P18 /reports                    My reports
│   ├── P19 /reports/new                Report wizard
│   └── P20 /reports/$reportId          Report detail and secure conversation
│
├── Cases (handlers) ......................................... pages/07-cases.md
│   ├── P21 /cases                      Case queue (scoped by role and stage)
│   ├── P22 /cases/$caseId              Case workspace
│   ├── P23 /cases/clusters             Similar-report groups
│   ├── P24 /cases/clusters/$clusterId  Group review
│   └── P25 /analytics                  Grievance analytics
│
├── Evaluations .............................................. pages/08-evaluations.md
│   ├── P26 /evaluations                Assigned forms and entry points
│   ├── P27 /evaluations/forms/$assignmentId     Fill in a form
│   ├── P28 /evaluations/results        Results overview
│   ├── P29 /evaluations/results/$facultyId      One faculty member's results
│   ├── P30 /evaluations/cycles         Cycles (Dean)
│   ├── P31 /evaluations/cycles/$cycleId         Cycle editor and monitor (Dean)
│   └── P32 /evaluations/cycles/$cycleId/review  Question review (professor)
│
├── Student records .......................................... pages/09-records.md
│   ├── P33 /records                    Find a student
│   ├── P34 /records/$studentId         Offenses, standing, restrictions, Good Moral
│   ├── P35 /restrictions               Restriction queue
│   └── P36 /standing                   My account standing (student)
│
├── People and account ....................................... pages/10-people-and-account.md
│   ├── P37 /directory                  Personnel directory
│   ├── P38 /people/$userId             Profile
│   ├── P39 /profile                    My profile
│   ├── P40 /notifications              Notification inbox
│   ├── P41 /assistant                  AI assistant (also a drawer on every page)
│   ├── P42 /help                       FAQ and how-to
│   ├── P43 /settings                   Account and appearance
│   └── P44 /settings/notifications     Notification priorities
│
└── Admin .................................................... pages/11-admin.md
    ├── P45 /admin                      Admin overview
    ├── P46 /admin/users                Users and roles
    ├── P47 /admin/users/$userId        User detail and role assignment
    ├── P48 /admin/roles                Permission matrix
    ├── P49 /admin/moderation           Moderation queue
    ├── P50 /admin/moderation/$itemId   Moderation review
    ├── P51 /admin/audit                Audit trail
    └── P52 /admin/monitoring           Assistant and uptime
```

The app shell, sign-in, and system states are in [pages/01-shell-and-auth.md](pages/01-shell-and-auth.md); the dashboard is in [pages/02-home.md](pages/02-home.md).

### 2.1 File routes

```
src/routes/
├── __root.tsx                      DesignSystemProvider, ToastRegion (exists)
├── index.tsx                       P01 redirect
├── _auth.tsx                       Layout without the shell
│   └── _auth/sign-in.tsx           P02
├── _app.tsx                        AppShell + session guard (redirects to /sign-in?redirect=…)
└── _app/
    ├── home.tsx                                    P03
    ├── feed/index.tsx · new.tsx · $postId.tsx      P04–P06
    ├── spaces/index.tsx · $spaceId.tsx             P07–P08
    ├── organizations/index.tsx · $orgId.tsx        P09–P10
    ├── messages.tsx (two-pane layout) · messages/index.tsx · messages/$threadId.tsx   P11–P12
    ├── calendar.tsx                                P13
    ├── consultations/index.tsx · faculty.$facultyId.tsx · requests.$requestId.tsx · availability.tsx   P14–P17
    ├── reports/index.tsx · new.tsx · $reportId.tsx P18–P20
    ├── cases/index.tsx · $caseId.tsx · clusters/index.tsx · clusters/$clusterId.tsx   P21–P24
    ├── analytics.tsx                               P25
    ├── evaluations/index.tsx · forms.$assignmentId.tsx · results/index.tsx · results/$facultyId.tsx · cycles/index.tsx · cycles/$cycleId/index.tsx · cycles/$cycleId/review.tsx   P26–P32
    ├── records/index.tsx · $studentId.tsx          P33–P34
    ├── restrictions.tsx · standing.tsx             P35–P36
    ├── directory.tsx · people/$userId.tsx · profile.tsx   P37–P39
    ├── notifications.tsx · assistant.tsx · help.tsx       P40–P42
    ├── settings/index.tsx · settings/notifications.tsx    P43–P44
    ├── admin.tsx (guard: role.manage or moderation.review or audit.view)
    └── admin/index.tsx · users/index.tsx · users/$userId.tsx · roles.tsx · moderation/index.tsx · moderation/$itemId.tsx · audit.tsx · monitoring.tsx   P45–P52
```

Each route declares its permission key in `beforeLoad` (for example `requirePermission('case.view')`). A failed check renders the restricted state inside the shell and keeps the URL, so testers can confirm S5 ALL-01-E and ALL-01-J. Loaders call the API, which applies the scope rules again.

---

## 3. Route access matrix

✓ = full use for that role · S = scoped (1.1) · O = own records only · R = read only · † = proposed, open decision · — = restricted state, nothing revealed.

| ID | Route | Stu | Org | Prof | Staff | Chair | Dean | Admin | Guid | OSAS | Pres |
|---|---|---|---|---|---|---|---|---|---|---|---|
| P01 | `/` | → | → | → | → | → | → | → | → | → | → |
| P02 | `/sign-in` | signed-out visitors | | | | | | | | | |
| P03 | `/home` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | → P45 | ✓ | ✓ | ✓ |
| P04 | `/feed` | S | S | S | S | S | S | R | S | S | R |
| P05 | `/feed/new` | S | S | S | † | S | S | — | S | S | — |
| P06 | `/feed/$postId` | S | S | S | R | S | S | R | S | S | R |
| P07 | `/spaces` | O | — | S | — | S | S | — | — | — | — |
| P08 | `/spaces/$spaceId` | S | — | S | — | R | R | — | — | — | — |
| P09 | `/organizations` | R | R | R | R | R | R | R | R | R | R |
| P10 | `/organizations/$orgId` | S | ✓† | R | R | R | R | R | R | R | R |
| P11 | `/messages` | ✓ | † | ✓ | † | ✓ | ✓ | — | ✓ | ✓ | — |
| P12 | `/messages/$threadId` | S | † | S | † | S | S | — | S | S | — |
| P13 | `/calendar` | ✓ | S† | ✓ | — | R | R | — | R | R | — |
| P14 | `/consultations` | O | — | O | — | — | — | — | — | — | — |
| P15 | `/consultations/faculty/$facultyId` | S | — | — | — | — | — | — | — | — | — |
| P16 | `/consultations/requests/$requestId` | O | — | O | — | — | — | — | — | — | — |
| P17 | `/consultations/availability` | — | — | O | — | — | — | — | — | — | — |
| P18 | `/reports` | O | — | — | — | — | — | — | — | — | — |
| P19 | `/reports/new` | ✓ | — | — | — | — | — | — | — | — | — |
| P20 | `/reports/$reportId` | O | — | — | — | — | — | — | — | — | — |
| P21 | `/cases` | — | — | S | — | S | S | — | S | S | S |
| P22 | `/cases/$caseId` | — | — | S | — | S | S | — | S | S | S |
| P23 | `/cases/clusters` | — | — | — | — | S | † | — | † | † | — |
| P24 | `/cases/clusters/$clusterId` | — | — | — | — | S | † | — | † | † | — |
| P25 | `/analytics` | — | — | — | — | S | ✓ | — | — | — | — |
| P26 | `/evaluations` | O | — | O | — | † | ✓ | — | — | — | — |
| P27 | `/evaluations/forms/$assignmentId` | O | — | O | — | † | † | — | — | — | — |
| P28 | `/evaluations/results` | — | — | O | — | S | ✓ | — | — | — | — |
| P29 | `/evaluations/results/$facultyId` | — | — | O | — | S | ✓ | — | — | — | — |
| P30 | `/evaluations/cycles` | — | — | — | — | — | ✓ | — | — | — | — |
| P31 | `/evaluations/cycles/$cycleId` | — | — | — | — | — | ✓ | — | — | — | — |
| P32 | `/evaluations/cycles/$cycleId/review` | — | — | O | — | † | — | — | — | — | — |
| P33 | `/records` | — | — | — | — | S | S | — | S | S | — |
| P34 | `/records/$studentId` | — | — | — | — | S | S | — | S | S | — |
| P35 | `/restrictions` | — | — | — | — | — | — | ✓ | S | S | — |
| P36 | `/standing` | O | — | — | — | — | — | — | — | — | — |
| P37 | `/directory` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| P38 | `/people/$userId` | R | R | R | R | R | R | R | R | R | R |
| P39 | `/profile` | O | O | O | O | O | O | O | O | O | O |
| P40 | `/notifications` | O | O | O | O | O | O | O | O | O | O |
| P41 | `/assistant` | S | S | S | S | S | S | S | S | S | S |
| P42 | `/help` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| P43 | `/settings` | O | O | O | O | O | O | O | O | O | O |
| P44 | `/settings/notifications` | O | O | O | O | O | O | O | O | O | O |
| P45–P52 | `/admin/**` | — | — | — | — | — | — | ✓ | — | — | — |

A Chair or Dean who also teaches gets the Professor column as well (rule 3).

---

## 4. Navigation per role

Sidebar sections (`Sidebar`, `SidebarSection`, `NavItem` with `count`). Icon names come from the design system's vocabulary; two are not in it yet and should be added: `robot` (assistant) and `chart-bar` (analytics).

| Role | Sections and items (icon) |
|---|---|
| Student | **Workspace:** Home (`house`), Feed (`chats`), My classes (`graduation-cap`), Organizations (`flag`), Messages (`envelope`, unread), Calendar (`calendar`), Consultations (`chalkboard`) · **Support:** My reports (`file`), Evaluations (`list-checks`, due), My standing (`info`) |
| Organization | **Workspace:** Home, Feed, Our organization (`flag`), Calendar, Messages† |
| Professor | **Workspace:** Home, Feed, My classes, Messages, Calendar, Consultations (pending requests) · **Support:** Assigned reports (`tray`, open), Evaluations |
| Staff | **Workspace:** Home, Feed, Organizations |
| Program Chair | **Workspace:** Home, Feed, Program sections (`graduation-cap`), Messages, Calendar · **Oversight:** Report queue (`tray`), Similar reports (`copy`), Analytics (`chart-bar`), Evaluation results (`list-checks`), Student records (`archive`) |
| Dean | **Workspace:** Home, Feed, Sections, Messages, Calendar · **Oversight:** Escalated reports (`tray`), Analytics, Student records · **Evaluations:** Cycles (`notepad`), Results (`list-checks`) |
| Administrator | **Administration:** Overview (`house`), Moderation (`flag`, pending), Users and roles (`users`), Permissions (`lock`), Restrictions (`prohibit`), Audit trail (`list`), Monitoring (`desktop`) · **Community:** Feed |
| Guidance | **Workspace:** Home, Feed, Messages, Calendar · **Cases:** Case Files (`tray`, intake) · **Records:** Student records (`archive`), Restrictions (`prohibit`) |
| OSAS | **Workspace:** Home, Feed, Messages, Calendar · **Cases:** OSAS cases (`tray`) · **Records:** Student records, Restrictions |
| President | **Workspace:** Home, Feed · **Cases:** Executive cases (`tray`) |
| Everyone | **Account:** Directory (`users`), Help (`question`), Settings (`gear`), Sign out (`sign-out`). Top bar: search, assistant (`robot`), notifications (`bell`), account menu (`user`). |

---

## 5. Case stages and statuses (shared model)

Used by P18–P25 and the home dashboards. Sources: workflow rules 2 and 3, PRO-06, CHR-02, DEAN-02, GUD-01, GUD-02, OSAS-01, PRES-01, PRES-02.

| Stage | Handler | Entry | Actions (proposed from module specs) |
|---|---|---|---|
| 1 Adviser | Assigned adviser or appropriate professor | New report (default route) | Respond, record next action, escalate to Chair, resolve |
| 2 Program Chair | Chair of the reporter's program | Escalation from stage 1, or **bypass** when the adviser may be involved | Triage, record action, resolve, escalate to Dean |
| 3 Dean | CCS Dean | Escalation from stage 2 | Record decision, resolve, transfer to Guidance or another authorized stage |
| 4 Guidance | Guidance Office | Transfer; the report becomes a **Case File** here | Convert to Case File, notes, milestones, resolve, transfer to OSAS |
| 5 OSAS | OSAS | Transfer from Guidance | Record action, milestones, resolve, transfer to the President's Office |
| 6 Executive | President's Office | Escalation from OSAS | Review, record action, return, resolve, or transfer as permitted |

Direct bypass to higher offices "when necessary" is not defined (D-04). Every transition records date, status, responsible office, actor, and reason.

| Status the reporter sees | Badge |
|---|---|
| Draft | `Badge form="pending" tone="warning"` |
| Submitted · With adviser · With Program Chair · With Dean · With OSAS · With President's Office | `Badge form="pending"` (in progress) |
| Case File at Guidance | `Badge form="pending" tone="info"` plus the Case File reference |
| Resolved | `Badge tone="success"` (solid) |
| Closed | `Badge form="closed"` |
| Internal-only entries (handlers) | `Badge form="restricted"` |

---

## 6. Module coverage

Every module in S2 maps to at least one page. Details per page in `pages/`.

| Page | Modules |
|---|---|
| P02, guards on every route | ALL-01 |
| P03 | ALL-01, STF-01, CHR-01, DEAN-01, ADM-04 |
| P37, P38, P39 | ALL-02 |
| P04, P05, P06 | ALL-03, STU-02, STU-08, PRO-02, CHR-06, DEAN-06, GUD-05, OSAS-04, ORG-02, STF-01 |
| P41, assistant drawer, P42 | ALL-04 (P41 also uses STU-06 drafting help) |
| P40, P44 | ALL-05, STU-12 |
| P07, P08 | STU-01, PRO-01, CHR-01 |
| P09, P10 | ORG-01, ORG-02 |
| P11, P12 | STU-03, PRO-03, CHR-06, DEAN-06, GUD-05, OSAS-04 |
| P13 | STU-04, PRO-04, CHR-06, DEAN-06, GUD-05, OSAS-04, ORG-02 |
| P14–P17 | STU-05, PRO-04, PRO-05 |
| P18–P20 | STU-06, STU-07, STU-08, STU-09 |
| P21, P22 | PRO-06, CHR-02, DEAN-02, GUD-01, GUD-02, OSAS-01, PRES-01, PRES-02 |
| P23, P24 | CHR-07 |
| P25 | CHR-03, DEAN-01 |
| P26, P27 | STU-10, PRO-07, CHR-04 |
| P28, P29 | PRO-08, CHR-04, DEAN-04 |
| P30–P32 | DEAN-03 |
| P33, P34 | CHR-05, DEAN-05, GUD-03, GUD-04, OSAS-02, OSAS-03 |
| P35 | ADM-03, GUD-04, OSAS-03 |
| P36 | STU-11 |
| P46–P48, P51 | ADM-01 |
| P45, P51, P52 | ADM-04 |
| P49, P50 (P45 previews the queue) | ADM-02, ADM-06 |
| P52 (P41 and P45 show the assistant counts) | ADM-05 |

---

## 7. Components

### 7.1 From the design system (exists in v1.0.1)

| Group | Exports |
|---|---|
| Shell and navigation | `AppShell`, `Sidebar`, `SidebarSection`, `SidebarNote`, `NavItem`, `TopBar`, `MobileHeader`, `Breadcrumbs`, `Tabs`/`TabList`/`Tab`/`TabPanel`, `Pagination`, `Stepper`, `MenuButton`/`MenuItem`/`MenuSection`/`MenuSeparator`, `CommandPalette`, `Link` |
| Identity | `DoorPlate` (the ACCS wordmark), `Avatar`, `AvatarGroup`, `RoleBadge` |
| Layout | `Container`, `Stack`, `Row`, `Grid`, `Section`, `SectionHeader`, `Divider`, `Agenda`/`AgendaItem`, `Highlight` |
| Content | `Card` (+ `CardHeader`, `CardLane`, `CardWhen`, `CardTitle`, `CardBody`, `CardMeta`, `CardAction`), `CardLink`, `ForumCard`, `AnnouncementCard`, `Tile`, `Icon` |
| Records | `ReportsTable`, `ReportRecord`, `CaseTimeline`, `Table` (+ `TableHeader`, `Column`, `TableBody`, `TableRow`, `Cell`), facts list (`dl.ds-record__facts`) |
| Status | `Badge`, `Severity`, `ViewState`, `EmptyState`, `Skeleton`, `Spinner`, `Progress`, `Meter` |
| Messages to the person | `Alert`, `PrivacyNotice`, `ToastRegion`/`toast`, `Tooltip`/`TooltipTrigger` |
| Overlays | `Modal`/`ModalBody`/`ModalActions`/`DialogTrigger`, `Drawer`, `Popover`, `Disclosure`/`Accordion` |
| Forms | `TextField`, `TextArea`, `Search`, `Select`/`SelectOption`, `ComboBox`, `Checkbox`/`CheckboxGroup`, `RadioGroup`/`Radio`, `Switch`, `SegmentedControl`/`Segment`, `TagGroup`/`Tag`, `DatePicker`, `Calendar`, `FileUpload`, `Field`, `Button`, `IconButton`, `Kbd` |
| Scheduling | `Timetable`, `TimeBlock`, `NowLine`, `ConsultationSlotPicker` |

Reference compositions in the design system site: `site/app/_site/examples/Portal.tsx` (app shell), `ReportDetail.tsx` (record page), `SignIn.tsx`, `AdminOverview.tsx`.

### 7.2 Add to the design system (generic, reusable)

| ID | Component | Used on |
|---|---|---|
| DS-1 | `Stat` — label, value, change, scope note | P03, P25, P28, P29, P31, P45, P52 |
| DS-2 | `BarChart`, `TrendChart` — token colors, table fallback, keyboard reachable | P25, P28, P29, P45, P52 |
| DS-3 | Chat kit: `ConversationList`, `MessageList`, `MessageBubble`, `MessageComposer` | P11, P12, P20, P22, P41, assistant drawer |
| DS-4 | `NotificationList`, `NotificationItem` — priority, unread, source link | P40, top-bar popover |
| DS-5 | `EventCalendar` — month and list views of dated events | P13 |
| DS-6 | `RatingScale` — Likert row with required state | P27 |
| DS-7 | `PersonCard`, `ProfileHeader` | P15, P37, P38, P39, P47 |
| DS-8 | `FilterBar` — search, selects, active-filter chips | P04, P21, P23, P25, P33, P35, P46, P49, P51 |
| DS-9 | `CommentThread`, `Comment` | P06, P50 |
| DS-10 | `ReactionButton` — toggle with count ("Agree") | P06 |

### 7.3 Build in the app (ACCS-specific, from design system parts)

| ID | Component | Built from | Used on |
|---|---|---|---|
| APP-1 | `RoleSidebar` | `Sidebar`, `SidebarSection`, `NavItem`, `SidebarNote`, `RoleBadge`, `Avatar` | shell |
| APP-2 | `RequirePermission`, `RestrictedView` | route guard + `ViewState kind="restricted"` | every route |
| APP-3 | `PostComposer` | `Select`, `SegmentedControl`, `RadioGroup`, `TextField`, `TextArea`, `FileUpload`, `DatePicker`, `Button`, APP-5 | P05, P06, P08, P10 |
| APP-4 | `PostView` | `SectionHeader`, `Avatar`, `RoleBadge`, `Badge`, `.ds-prose` | P06, P50 |
| APP-5 | `IdentityChoice`, `DiscloseIdentityDialog` | `RadioGroup` (card), `PrivacyNotice`, `Modal role="alertdialog"` | P05, P06, P19, P20 |
| APP-6 | `ReportWizard` | `Stepper`, form controls, `PrivacyNotice`, `Alert` | P19 |
| APP-7 | `CaseActionBar` and its dialogs | `Button`, `MenuButton`, `Modal`, `Select`, `TextArea` | P22, P24 |
| APP-8 | `CaseNotes` | `TextArea`, `Badge form="restricted"`, list rows | P22 |
| APP-9 | `ConflictNotice` | `Alert tone="warning"` | P21, P22 |
| APP-10 | `ReportClusterCard`, `ClusterReview` | `Card`, `Meter`, `Highlight`, `Grid`, `ReportsTable` | P23, P24 |
| APP-11 | `OffenseHistory` | `CaseTimeline` or `Table`, `Badge` | P34 |
| APP-12 | `RestrictionPanel` | `CheckboxGroup`, `TextField`, `DatePicker`, `Alert`, `Button` | P34, P35 |
| APP-13 | `StandingBadge` | `Badge` with neutral wording | P34, P36, P38 |
| APP-14 | `PermissionMatrix` | `Table`, `Checkbox` | P48 |
| APP-15 | `AuditLog` | `Table` | P34, P47, P51 |
| APP-16 | `SystemStatus` | `Badge`, `Meter`, DS-1 | P45, P52 |
| APP-17 | `AssistantPanel` | DS-3, `TagGroup`, `Alert` | P41, P19, assistant drawer |
| APP-18 | `QuestionEditor` | `Table`, `TextField`, `Select`, `Switch`, `MenuButton` | P31 |
| APP-19 | `PurposeOfAccessDialog` | `Modal`, `Select`, `TextArea` | P34 |
| APP-20 | `RestrictedBanner` | `Alert tone="warning"` in the shell | shell (restricted students) |

### 7.4 Page templates

| Template | Structure | Pages |
|---|---|---|
| T1 Dashboard | `SectionHeader level={1}`, DS-1 row, `Grid columns="main"`, previews | P03, P25, P45 |
| T2 Feed | `Tabs` by scope, DS-8, card list, `Pagination` | P04, P08, P10 |
| T3 Queue + detail | DS-8, `Table` (desktop) / `ReportRecord` (phone), detail page with `Breadcrumbs` | P21, P23, P33, P35, P46, P49, P51 |
| T4 Wizard | `Stepper`, one group per step, review step | P19, P27, P31 |
| T5 Conversation | two panes: list + thread + composer | P11, P12, P41 |
| T6 Schedule | `SegmentedControl` view switch, `Timetable` or DS-5, side detail | P13, P15, P17 |
| T7 Record | `Breadcrumbs`, `SectionHeader` with status, facts, `PrivacyNotice`, `CaseTimeline` | P16, P20, P22, P24, P34, P50 |

### 7.5 Component → pages

Generated from the page specs in `pages/`. "shell" means the app shell in `pages/01-shell-and-auth.md`. Subparts (`TableHeader`, `TabPanel`, `Radio`, `CardBody` and similar) come with their parent and are not listed.

| Component | Pages |
|---|---|
| `Accordion` | P29, P42 |
| `Agenda` | P03, P10, P14, P36 |
| `AgendaItem` | P03, P10, P36 |
| `Alert` | shell, P02, P03, P05, P06, P12, P15, P16, P17, P19, P22, P23, P24, P27, P29, P31, P32, P34, P35, P36, P39, P41, P44, P47, P48, P50, P52 |
| `AnnouncementCard` | P03, P04, P08, P10 |
| `AppShell` | shell |
| `Avatar` | shell, P06, P08, P10, P11, P12, P15, P26, P33, P35, P37, P38, P46 |
| `AvatarGroup` | P11, P12 |
| `Badge` | shell, P03, P04, P06, P08, P09, P10, P13, P14, P16, P20, P21, P22, P23, P24, P26, P30, P31, P33, P34, P35, P44, P45, P46, P49, P50 |
| `Breadcrumbs` | P05, P06, P08, P15, P16, P19, P20, P22, P24, P27, P29, P31, P32, P34, P38, P44, P47, P50 |
| `Button` | shell, P02, P03, P04, P05, P06, P08, P10, P12, P13, P14, P15, P16, P17, P18, P19, P20, P22, P24, P26, P27, P29, P30, P31, P32, P34, P35, P38, P39, P40, P41, P42, P43, P44, P46, P47, P48, P50 |
| `Card` | P02, P07, P23 |
| `CardAction` | P07 |
| `CardHeader` | P07 |
| `CardLink` | P09 |
| `CardMeta` | P07, P23 |
| `CardTitle` | P07 |
| `CaseTimeline` | P16, P20, P22, P24, P34, P50 |
| `Checkbox` | P02, P05, P35, P48, P50 |
| `CheckboxGroup` | P31, P35 |
| `ComboBox` | P11, P35, P46, P47, P51 |
| `CommandPalette` | shell |
| `ConsultationSlotPicker` | P15 |
| `Container` | P02, P05 |
| `DatePicker` | P05, P13, P17, P19, P25, P31, P35, P51 |
| `Disclosure` | P02, P05, P29, P31, P42 |
| `Divider` | P04, P12, P40 |
| `DoorPlate` | shell, P02, P07 |
| `Drawer` | shell, P12, P13, P19, P35, P41 |
| `EmptyState` | shell, P03, P06, P07, P11, P14, P15, P18, P21, P23, P26, P30, P35, P40, P49 |
| `FileUpload` | P05, P19, P39 |
| `ForumCard` | P03, P04, P08, P10 |
| `Grid` | P03, P07, P09, P14, P22, P24, P37, P45 |
| `Highlight` | P24, P50 |
| `IconButton` | shell, P11, P12, P13 |
| `Kbd` | P12 |
| `Link` | shell, P03, P10, P13, P15, P20, P22, P26, P36, P37, P38, P40, P41, P42, P43, P50 |
| `MenuButton` | shell, P06, P12, P19, P22, P31, P47 |
| `MenuItem` | shell |
| `MenuSeparator` | shell |
| `Meter` | P03, P23, P24, P30, P31, P45 |
| `MobileHeader` | shell |
| `Modal` | P05, P06, P11, P12, P13, P16, P17, P19, P20, P22, P24, P27, P30, P31, P34, P35, P47, P48, P50 |
| `ModalActions` | P06, P11 |
| `NavItem` | shell |
| `NowLine` | P03, P13, P17 |
| `Pagination` | P04, P06, P07, P08, P14, P21, P33, P35, P37, P40, P46, P49, P51 |
| `Popover` | shell, P13, P17 |
| `PrivacyNotice` | P02, P05, P13, P18, P19, P20, P21, P22, P25, P27, P28, P33, P34, P35, P36, P42, P46, P49, P51 |
| `Progress` | P03, P26, P27 |
| `RadioGroup` | P05, P06, P17, P19, P32, P43, P44, P50 |
| `ReportRecord` | P03, P18, P21, P33, P35, P46, P49, P51 |
| `ReportsTable` | P18, P24 |
| `RoleBadge` | shell, P06, P11, P12, P14, P15, P29, P34, P37, P38, P43, P46, P47, P51 |
| `Row` | P05, P06, P12, P13, P19, P38, P42 |
| `Search` | shell, P03, P04, P07, P08, P09, P11, P14, P21, P28, P33, P35, P37, P42, P46 |
| `Section` | P38 |
| `SectionHeader` | P02, P03, P04, P05, P06, P07, P08, P09, P10, P11, P13, P14, P16, P17, P18, P19, P20, P21, P22, P23, P24, P25, P26, P27, P28, P30, P31, P32, P33, P34, P35, P36, P37, P39, P40, P41, P42, P43, P44, P45, P46, P48, P49, P50, P51, P52 |
| `SegmentedControl` | P05, P13, P17, P18 |
| `Select` | P04, P05, P07, P17, P19, P21, P22, P23, P25, P28, P29, P30, P31, P33, P34, P35, P37, P46, P47, P49, P51 |
| `SelectOption` | P05 |
| `Severity` | P03, P19, P20, P21, P22 |
| `Sidebar` | shell |
| `SidebarNote` | shell, P03 |
| `SidebarSection` | shell |
| `Skeleton` | shell, P03, P04, P34, P37, P40 |
| `Spinner` | P41 |
| `Stack` | P03 |
| `Stepper` | P19, P27 |
| `Switch` | P12, P31 |
| `Tab` | P03, P04 |
| `TabList` | P04 |
| `Table` | P03, P07, P08, P10, P14, P21, P25, P26, P28, P30, P31, P33, P34, P35, P45, P46, P47, P49, P51, P52 |
| `Tabs` | P03, P04, P08, P10, P11, P14, P20, P21, P22, P26, P31, P34, P35, P40, P49, P52 |
| `TagGroup` | P04, P13, P41 |
| `TextArea` | P05, P06, P11, P12, P15, P16, P19, P22, P24, P27, P29, P31, P32, P34, P35, P39, P47, P48, P50 |
| `TextField` | P02, P05, P13, P19, P30, P31, P34, P35, P39 |
| `Tile` | P09, P10 |
| `TimeBlock` | P03, P13, P17 |
| `Timetable` | P03, P13, P17 |
| `ToastRegion` | shell |
| `TopBar` | shell |
| `ViewState` | shell, P03, P04, P05, P09, P18, P21, P25, P27, P28, P32, P33, P34, P35, P36, P37, P38, P40, P42, P46, P51 |
| `toast` | P05, P15, P22, P32, P34, P35, P39, P44 |
| APP-1 `RoleSidebar` | shell |
| APP-2 `RequirePermission`, `RestrictedView` | shell |
| APP-3 `PostComposer` | P05 |
| APP-4 `PostView` | P06, P50 |
| APP-5 `IdentityChoice`, `DiscloseIdentityDialog` | P05, P06, P19, P20 |
| APP-6 `ReportWizard` | P19 |
| APP-7 `CaseActionBar` and its dialogs | P22, P24 |
| APP-8 `CaseNotes` | P22 |
| APP-9 `ConflictNotice` | P22 |
| APP-10 `ReportClusterCard`, `ClusterReview` | P23 |
| APP-11 `OffenseHistory` | P34 |
| APP-12 `RestrictionPanel` | P34, P35 |
| APP-13 `StandingBadge` | P33, P34, P36, P38 |
| APP-14 `PermissionMatrix` | P48 |
| APP-15 `AuditLog` | P34, P45, P47, P51 |
| APP-16 `SystemStatus` | P45, P52 |
| APP-17 `AssistantPanel` | shell, P19, P41 |
| APP-18 `QuestionEditor` | P31 |
| APP-19 `PurposeOfAccessDialog` | P34 |
| APP-20 `RestrictedBanner` | shell, P36 |
| DS-1 `Stat` | P03, P25, P28, P29, P31, P45, P52 |
| DS-2 `BarChart`, `TrendChart` | P03, P25, P28, P29, P45, P52 |
| DS-3 Chat kit: `ConversationList`, `MessageList`, `MessageBubble`, `MessageComposer` | P11, P12, P20, P22, P41, P50 |
| DS-4 `NotificationList`, `NotificationItem` | shell, P40 |
| DS-5 `EventCalendar` | P13 |
| DS-6 `RatingScale` | P27, P31 |
| DS-7 `PersonCard`, `ProfileHeader` | P14, P15, P29, P37, P38, P39, P47 |
| DS-8 `FilterBar` | P04, P07, P08, P21, P23, P25, P28, P31, P33, P35, P37, P46, P49, P51 |
| DS-9 `CommentThread`, `Comment` | P06, P50 |
| DS-10 `ReactionButton` | P06 |

Not used in the app: `Hero`, `Navbar`, `Footer` and `Logo` (marketing sites only), `Calendar` (comes inside `DatePicker`), and `Tooltip`.

---

## 8. Build order

1. **Shell and access:** P01, P02, APP-1, APP-2, top bar, P03 skeleton, DS-1, DS-8.
2. **Grievance path (core of the study):** P19, P18, P20, P21, P22; APP-5 to APP-9; `CaseTimeline`, `PrivacyNotice`.
3. **Community:** P04, P05, P06, P07, P08, P09, P10; APP-3, APP-4, DS-9, DS-10.
4. **Schedule:** P14 to P17, P13; DS-5.
5. **Messages, notifications, assistant:** P11, P12, P40, P41, P44; DS-3, DS-4, APP-17.
6. **Evaluations:** P26 to P32; DS-6, APP-18, DS-2.
7. **Records and admin:** P33 to P36, P45 to P52, P23 to P25; APP-10 to APP-16, APP-19.
8. **People and settings:** P37, P38, P39, P42, P43.

## 9. Open decisions that change pages

| Decision (S2) | Pages affected |
|---|---|
| D-01 Evaluation population (100% vs 50/50) | P26, P27, P31 completion rules |
| D-02 Evaluation enforcement (grades, Good Moral) | P26 banners, P34 Good Moral tab |
| D-03 Report vs Case File wording | P18 to P22 labels |
| D-04 Escalation and bypass authority | APP-7 transitions, P21 queue scopes |
| D-05 Restriction trigger and authority | P35, APP-12 evidence fields |
| D-06 Staff and organization permissions | Staff and Org navigation, P05, P10, P11, P48 defaults |
| D-07 Similarity and agreement thresholds | P23, P24, STU-08 prompt on P06 |
| D-08 Privacy and retention | P38 standing badge, alias resolution, P12, P34, P51 retention |

Also unspecified: registration flow, password recovery, session expiry, lockout (ALL-01); cancellation and rescheduling rules (STU-05, PRO-04, PRO-05); report categories; announcement priority levels; who publishes academic and campus calendar events; whether flagged posts are held or published with a flag.
