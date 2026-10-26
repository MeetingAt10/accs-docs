# Student records, restrictions, and standing

Back to [index](README.md) · [sitemap](../sitemap.md)

Offense records are restricted to the Program Chair (program), the Dean (college), and Guidance and OSAS (authorized students). Every opening is logged (CHR-05, DEAN-05, OSAS-02), and peers never see details. Restrictions are feature-specific (posting, commenting, anonymous reporting). A restricted student always keeps announcements, academic information, the assistant, and permitted professor messages (US-49, ADM-03).

People decide: the assistant, word filters, and similarity summaries never create an offense or a restriction ([sitemap 1.3](../sitemap.md#13-rules), rule 6). The Administrator applies and lifts restrictions on P35 and has no access to offense records. Breadcrumbs use the student ID, and notifications name the kind of event only (rule 8). Restricted states use the shell copy ([system states](01-shell-and-auth.md#system-states-inside-the-shell)).

---

## P33 Student records — `/records`

| | |
|---|---|
| File | `src/routes/_app/records/index.tsx` |
| Permission | `offense.view` |
| Modules | CHR-05, DEAN-05, GUD-03, OSAS-02 |
| Stories | US-46 |
| Tests | CHR-05-I/B, DEAN-05-B, OSAS-02-B |
| Template | T3 Queue + detail |

**Access by role**

| Role | Access | Lists |
|---|---|---|
| Program Chair | S (program) | Students in the Chair's program |
| Dean | S (CCS) | CCS students under approved policy |
| Guidance | S | Students in Guidance cases, or otherwise authorized (GUD-03) |
| OSAS | S | Students under OSAS authority (OSAS-02) |
| Everyone else | — | |

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} title="Student records"`, `description` states the scope | Chair: "BSIT program". Dean: "CCS". Guidance: "Students in Guidance cases". OSAS: "Students under OSAS authority". |
| Boundary | `PrivacyNotice` "Every record you open is logged with your name and the time." | The log entry is made when a record opens on P34, not when this list loads. |
| Filters | DS-8 `FilterBar`: `Search label="Search by name or student ID"`, `Select label="Program"` (Dean only), `Select label="Standing"`, `Select label="Has active restriction"` | The Chair's program is fixed. Standing options follow APP-13 (wording TBD, D-08). |
| Table (desktop) | `Table`: student (`Avatar decorative`, name), student ID (mono), program, section, APP-13 `StandingBadge`, active restriction `Badge` | Sorted by name. Rows open P34. The restriction `Badge` shows only while a restriction is active. |
| Rows (phones) | `ReportRecord` (`facts`: program, section, standing) | Below 768px. |
| Paging | `Pagination` | |
| Empty | `ViewState kind="empty"`: "No students match" for a search or filter, "No students in your scope yet" for an empty list | No placeholder row and no invented record. |

**States**

| Case | Result | Test |
|---|---|---|
| Chair searches for another program's student | Not listed. The result is the ordinary no-match state, the same as for a name that doesn't exist | CHR-05-B |
| Student in scope with no offense record | Listed with the standing badge and no restriction badge. P34 shows "No offense records" | CHR-05-I |
| Student, Professor, Administrator, or any other role without `offense.view` opens `/records` | Restricted state ("This page isn't available to your role."). No student is revealed | DEAN-05-B, OSAS-02-B |
| Loading | `ViewState kind="loading"` "Loading students" | |

**Open.** Which students each office may list: CHR-05 and DEAN-05 depend on "approved access rules", and "otherwise authorized" for Guidance (GUD-03) has no definition (D-08). Whether the list shows every student in scope or only students with a record, case, or restriction. Standing values and badge wording (D-08).

---

## P34 Student record — `/records/$studentId`

| | |
|---|---|
| File | `src/routes/_app/records/$studentId.tsx` (search param: `tab`) |
| Permission | `offense.view`. The Good Moral tab also needs `goodmoral.review`. Restriction actions follow `restriction.manage` (Guidance refer †, OSAS S †) |
| Modules | CHR-05, DEAN-05, GUD-03, GUD-04, OSAS-02, OSAS-03 |
| Stories | US-46, US-48, US-50 |
| Tests | CHR-05-V/I/B, DEAN-05-V/I/B, GUD-03-V/I/B, GUD-04-V/I, OSAS-02-V/I/B, OSAS-03-V/I/B |
| Template | T7 Record |

**Access by role**

| Role | Access | Tabs | Can do |
|---|---|---|---|
| Program Chair | S (program) | Overview, Offenses, Restrictions | Read only |
| Dean | S (CCS) | Overview, Offenses, Restrictions | Read only. Sees the access history |
| Guidance | S | Overview, Offenses, Restrictions, Good Moral | Save a Good Moral review (GUD-03). Request a restriction change (GUD-04). Direct apply or lift †, not offered |
| OSAS | S | Overview, Offenses, Restrictions, Good Moral | Save a Good Moral review (OSAS-03). Record a permitted decision or referral † |
| Everyone else | — | | |

**Layout**

| Region | Components | Notes |
|---|---|---|
| Purpose | APP-19 `PurposeOfAccessDialog` = `Modal title="Open this student record?"` with `Select label="Purpose" isRequired` (Case handling, Good Moral review, Restriction review, Program oversight), `TextArea label="Note"` (optional), `Button` "Open record", `Button variant="secondary"` "Cancel" | Body: "Opening a record is logged with your name and the time." Opens on the first record opened in a session, before any request for the student's data, so it looks the same for a record the person may not open. Nothing loads until "Open record" (OSAS-02: verify role and purpose). Cancel returns to P33. The purpose is saved with the log entry (proposed). |
| Trail | `Breadcrumbs` (Student records / student ID) | The student ID, never the name. |
| Heading | `SectionHeader level={1} split`: `title` = student name, `description` = student ID (mono), program, and section ("BSIT · Section A"); `actions` = APP-13 `StandingBadge` and, while a restriction is active, a restriction `Badge` | Loads after the purpose is confirmed. |
| Boundary | `PrivacyNotice` "Opening this record was logged at 10:42." | The time comes from the log entry. |
| Tabs | `Tabs`: Overview, Offenses, Restrictions, Good Moral (Guidance and OSAS only) | The `tab` param picks the tab. A tab the role doesn't have opens Overview. P03 links OSAS compliance reviews to the Good Moral tab. |
| Overview | `dl.ds-record__facts`: program, section, year, standing, active restrictions (a summary, or "None") | Standing wording TBD (D-08). |
| Access history | APP-15 `AuditLog` (`Table`: person with name and `RoleBadge`, time, purpose) | Under the facts on Overview. Shown to the Dean (DEAN-05-V). Other offices: TBD, see Open. |
| Offenses | APP-11 `OffenseHistory` (a `CaseTimeline`, oldest first): one entry each for the first, second, and third offense, with a level `Badge`, the recorded details, decision provenance (decision reference, authorized actor role, date), and a status `Badge` | Actors appear by role. Status values TBD. With no entries: `ViewState kind="empty"` "No offense records". |
| Restrictions | APP-12 `RestrictionPanel` in read-only mode: per restriction, the features (`Badge`s), basis or decision reference, start–end, authorizer role, and status; `Alert tone="neutral"` "Announcements, academic information, the assistant, and permitted professor messages stay available." | Active first, then ended. With none active: `ViewState kind="empty"` "No active restrictions" (GUD-04-I). |
| Restriction actions | Guidance: `Button variant="secondary"` "Request change" on each active restriction → `Modal` with `Select label="Requested change"` (Lift, Shorten, Extend), `TextArea label="Reason" isRequired`, `Button` "Send request". OSAS: `Button variant="secondary"` "Record decision or referral" † → `Modal` with a required decision reference and reason | Guidance sends a referral only (GUD-04). It appears under Change requests on P35, and `toast` "Request sent" confirms it. OSAS fields are TBD until QCU defines its authority (OSAS-03); a referral also goes to P35. Chair and Dean have no actions. Administrators apply and lift on P35, not here. |
| Good Moral | `Select label="Prerequisite status"` (options TBD, D-02), `TextField label="Decision source" isRequired`, `TextArea label="Notes"`, `Button` "Save review"; `dl.ds-record__facts` for the last review (reviewer role, time); `Alert tone="neutral"` "Certificate rules need QCU approval." | Guidance and OSAS (`goodmoral.review`). No certificate or penalty action exists. Success: `toast` "Review saved". |

**States**

| Case | Result | Test |
|---|---|---|
| Authorized person confirms a purpose and opens a record in scope | Offense history with decision source and dates. The opening is logged, and the notice shows its time. The Dean also sees the access history | CHR-05-V, DEAN-05-V, OSAS-02-V, GUD-03-V |
| Student with no offenses | `ViewState kind="empty"` "No offense records". Nothing is invented. Good Moral shows no status until a review is saved | CHR-05-I, DEAN-05-I, GUD-03-I, OSAS-02-I |
| Student outside the person's scope (another program, or not under the officer's authority), or a guessed ID | Restricted record copy, the same as for an ID that doesn't exist | CHR-05-B, OSAS-02-B |
| Peer, or any role without `offense.view`, opens the URL | Restricted page state. P38 shows only the neutral `StandingBadge`, never offense details | DEAN-05-B, OSAS-02-B |
| Restrictions tab for a student with an effective restriction | Feature, decision reference, dates, and authorizer role shown | GUD-04-V |
| Restriction expired, or none applied | "No active restrictions". An expired one is listed as ended, never as active | GUD-04-I |
| Guidance sends "Request change" | Request saved and shown on P35. The restriction stays as it was until an authorized person decides | |
| OSAS records a review, permitted decision, or referral | The prerequisite or restriction state and the source action are saved and shown | OSAS-03-V |
| Change saved without a decision source or basis | Field error. Nothing is saved and no status changes | OSAS-03-I |
| Certificate, penalty, or apply or lift beyond the office's authority | Not offered. The outcome is TBD until QCU defines the rules, and academic access remains | GUD-03-B, OSAS-03-B |
| Purpose dialog cancelled | Return to P33. Nothing is loaded and no opening is logged | |
| Loading | The dialog first, then `Skeleton` in the page's shape and `ViewState kind="loading"` "Loading record" | |

**Open.** Purpose list and which purposes each role may choose; whether the purpose is asked once per session or for each student; whether it is saved with the log entry; whether a refused opening is logged (D-08). Good Moral status wording and rules (D-02). Offense statuses and retention (D-08). Which offices besides the Dean see the access history (D-08). Whether the Chair's Restrictions tab is in scope: CHR-05 names "standing context", while DEAN-05 and OSAS-03 name restriction status.

---

## P35 Restrictions — `/restrictions`

| | |
|---|---|
| File | `src/routes/_app/restrictions.tsx` |
| Permission | `restriction.manage` (Administrator ✓; Guidance refer †; OSAS S †). Guidance and OSAS open the page to review within their scope |
| Modules | ADM-03, GUD-04, OSAS-03 |
| Stories | US-48, US-49 |
| Tests | ADM-03-V/I/B, GUD-04-B, OSAS-03-I |
| Template | T3 Queue + detail (the detail opens in a `Drawer`, not a page) |

**Access by role**

| Role | Access | Sees | Can do |
|---|---|---|---|
| Administrator | ✓ | Every restriction | Apply and lift with a decision reference (ADM-03). Decide change requests. Never sees offense records |
| Guidance | S | Restrictions of students in Guidance cases | Review. Request a change (referral, GUD-04). Direct apply or lift †, not offered |
| OSAS | S | Restrictions of students under OSAS authority | Review. Request a change. A permitted change † (OSAS-03) |
| Everyone else | — | | |

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} split title="Restrictions"`, `description` states the scope, `actions` = `Button` "Apply restriction" | The button shows for the Administrator only. Description: "All students" (Administrator), "Students in Guidance cases", "Students under OSAS authority". |
| Boundary | `PrivacyNotice` "Restrictions change only the selected features. Announcements, academic information, the assistant, and permitted professor messages stay available." | |
| Tabs | `Tabs`: Active, Change requests (`count`), Ended | Change requests are referrals from Guidance and OSAS that wait for a decision. A decided request leaves the tab, and its effect shows on Active or Ended. Guidance and OSAS see requests for students in their scope. |
| Filters | DS-8 `FilterBar`: `Search label="Search by student name or ID"`, `Select label="Feature"` (Posting, Commenting, Anonymous reporting), `Select label="Program"` | Apply to the selected tab. |
| Table (desktop) | `Table`: student (`Avatar decorative`, name, student ID), features as `Badge`s (Posting, Commenting, Anonymous reporting), decision reference (mono), start–end, authorizer role, status `Badge` | Rows open the detail `Drawer`. In Change requests, start–end and authorizer become requested change, requested by (role), and date sent, and the status `Badge` reads "Pending". |
| Rows (phones) | `ReportRecord` (`facts`: features, end date, decision reference) | Below 768px. |
| Paging | `Pagination` | |
| Empty | `EmptyState` per tab ("No active restrictions", "No change requests", "No ended restrictions"), with `action` "Apply restriction" for the Administrator; `ViewState kind="empty"` when the filters match nothing | |
| Details | `Drawer side="end"`: `dl.ds-record__facts` (student, features, decision reference, basis, start, end, authorizer role, status) and the actions for the person's role | Opens from a row. Administrator, active restriction: `Button variant="secondary"` "Lift restriction". Administrator, change request: "Apply requested change" and "Decline request". Guidance and OSAS, active restriction: `Button variant="secondary"` "Request change". Ended rows have no actions. |
| Apply | `Button` "Apply restriction" → `Drawer side="end"` with APP-12 `RestrictionPanel`: `ComboBox label="Student"`, `CheckboxGroup label="Features"` (Posting, Commenting, Anonymous reporting), `TextField label="Decision reference" isRequired`, `Select label="Basis"` (TBD, D-05), `DatePicker` start and end, `Checkbox` "Notify the student", `Alert tone="neutral"` "Announcements, academic information, the assistant, and permitted professor messages stay available.", `Button` "Apply" | The student list shows name and student ID only. Only the three features can be selected, and an end date before the start date is invalid. Success: `toast` "Restriction applied" and a new Active row. The student notice names the event only (for example "Your account standing changed") and opens P36. |
| Lift | `Modal role="alertdialog" title="Lift this restriction?"` with `TextArea label="Reason" isRequired`, `Button` "Lift restriction", `Button variant="secondary"` "Cancel" | The body says the student can use the listed features again, that the step is logged, and that a new restriction can be applied later. Success: `toast` "Restriction lifted" and the row moves to Ended. |
| Request change | `Modal` with `Select label="Requested change"` (Lift, Shorten, Extend), `TextArea label="Reason" isRequired`, `Button` "Send request" | Guidance and OSAS. Success: `toast` "Request sent". The restriction is unchanged until an authorized person decides. |
| Decide request | "Apply requested change": Lift opens the lift dialog. Shorten and Extend open APP-12 `RestrictionPanel` with the student and features fixed and the end date editable. "Decline request": `Modal` with `TextArea label="Reason" isRequired` | Administrator only. Declining is not described in ADM-03 or GUD-04 (proposed, see Open). |

**States**

| Case | Result | Test |
|---|---|---|
| Administrator applies a restriction with features, decision reference, basis, and valid dates | Only the selected features are restricted. The decision is logged | ADM-03-V |
| Administrator lifts a restriction with a reason | Only that restriction ends. The decision is logged | ADM-03-V |
| Missing feature, decision reference, or basis, or an end date before the start date | Field errors (`isInvalid`, `errorMessage`). Nothing is saved | ADM-03-I |
| Posting restricted | The student still reads announcements and academic information, uses the assistant, and messages permitted professors. The feature list can't include them | ADM-03-B |
| Guidance tries to apply or lift directly | Not offered, only "Request change". A crafted call is denied | GUD-04-B |
| OSAS submits a change without a basis (a reason, or a decision reference for a permitted decision †) | Field error. No status change | OSAS-03-I |
| Guidance or OSAS sends a request | Appears under Change requests. The restriction is unchanged until decided | |
| Loading | `ViewState kind="loading"` "Loading restrictions" | |

**Open.** Trigger and authority to restrict (D-05): the evidence required, whether Guidance or OSAS may apply or lift directly, and whether being reported alone can be a basis. Basis options. Whether a restriction can have no end date. Notice wording, and whether notifying the student can be skipped. How an Administrator closes a change request without applying it, and how the requester learns the outcome.

---

## P36 My account standing — `/standing`

| | |
|---|---|
| File | `src/routes/_app/standing.tsx` |
| Permission | `standing.viewOwn` |
| Modules | STU-11 |
| Stories | US-47, US-49 |
| Tests | STU-11-V/I/B |
| Template | None |

**Access by role**

| Role | Access | Shows |
|---|---|---|
| Student | O | Own standing, paused features with end dates, and what stays available |
| Everyone else | — | |

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} title="My account standing"` | |
| Standing | APP-13 `StandingBadge` with one sentence | Good standing: "Good standing" and "All features are available." Restricted: neutral wording TBD (D-08). The page never shows offense details. |
| Paused | `Alert tone="warning" title="Some features are paused"` listing each paused feature and its end date, for example "Posting, until 12 Nov" | Only while a restriction is active. APP-20 `RestrictedBanner` in the shell shows the same facts and links here. |
| Still available | `SectionHeader level={2} title="Still available to you"`, `Agenda` with `AgendaItem` rows: Announcements (P04), Classes and academic information (P07), Assistant (P41), Messages with your professors (P11) | Only while a restriction is active. |
| Boundary | `PrivacyNotice` "Classmates see only a neutral standing badge, never offense details." | |
| Contact | `Link variant="standalone"` "Contact the Guidance Office" | Opens P37 filtered to Guidance. |

**States**

| Case | Result | Test |
|---|---|---|
| Restricted student opens the page | Neutral badge, each paused feature with its end date, and what stays available | STU-11-V |
| Restricted student tries a restricted action elsewhere | Only that feature is blocked: P05 replaces the form with "Posting is paused until 12 Nov", P06 replaces the comment box, and P19 hides the anonymous option. Everything else works | STU-11-I |
| Peer opens the student's profile (P38) | A neutral standing badge only, never offense details. The restricted student keeps announcements, academic information, the assistant, and permitted professor messages, and each "Still available" link opens its page | STU-11-B |
| No restriction | `StandingBadge` "Good standing" and "All features are available." No alert and no "Still available" list | |
| Loading | `ViewState kind="loading"` "Loading your standing" | |

**Open.** Badge wording (D-08). Whether the student sees the decision basis for a restriction. What the notice says for a restriction with no end date.
