# My reports (student side)

Back to [index](README.md) · [sitemap](../sitemap.md) · stages and statuses: [sitemap 5](../sitemap.md#5-case-stages-and-statuses-shared-model)

These pages belong to the reporting student only. Handlers work on the same records through P21 and P22. Wording rule (D-03): a **report** until it reaches Guidance, a **Case File** from then on. Copy stays neutral: categories ("General concern"), references in mono, no names in titles, crumbs, notifications, or previews.

---

## P18 My reports — `/reports`

| | |
|---|---|
| File | `src/routes/_app/reports/index.tsx` |
| Permission | `report.viewOwn` |
| Modules | STU-09, STU-06 |
| Stories | US-29 |
| Tests | STU-09-V/I |
| Template | T3 Queue (own records) |

**Access.** Student O. Everyone else —.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} split title="My reports"`, `actions` = `Button icon="plus"` "New report" (P19) | Hidden when report submission is restricted; anonymous reporting alone being restricted only removes the anonymous option on P19. |
| Boundary | `PrivacyNotice title="Who can open these"` | "Only you and the office handling each report can open it. Reporting under your name or anonymously is your choice on each report." |
| View | `SegmentedControl` (Active, Closed) | |
| List (desktop) | `ReportsTable caption="Your reports"` (reference, category, status, updated, action "View") | |
| List (phones) | `ReportRecord` rows in `.ds-record-list` | |
| Empty | `EmptyState title="No reports yet" description="If something is wrong, you can tell the right office here."` with action "New report" | |

**States**

| Case | Result | Test |
|---|---|---|
| Own reports exist | Listed with current status and office | STU-09-V |
| Loading | `ViewState kind="loading"` "Loading reports" | |

---

## P19 New report — `/reports/new`

| | |
|---|---|
| File | `src/routes/_app/reports/new.tsx` (search param `from` = source post) |
| Permission | `report.create` |
| Modules | STU-06, STU-07, STU-08, ALL-04 |
| Stories | US-26, US-27, US-28, US-52 |
| Tests | STU-06-V/I/B, STU-08-V |
| Template | T4 Wizard (APP-6 `ReportWizard`) |

**Access.** Student ✓. The anonymous option is hidden for a student whose anonymous reporting is restricted (ADM-03). Everyone else —.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Trail | `Breadcrumbs` (My reports / New report) | |
| Heading | `SectionHeader level={1} title="New report"` | |
| Progress | `Stepper label="Report steps" steps={["Describe", "Classify", "Identity", "Review"]}` | |
| Source link | `Alert tone="info"` "Linked to your discussion 'Lab schedule changes'" | Only with `?from=` (STU-08). |
| 1 Describe | `TextArea label="What happened" isRequired showCount maxLength={4000} description="Say what happened, when, and where. Leave out names you don't need to share."`, `DatePicker label="When" ` (optional), `TextField label="Where"` (optional), `FileUpload` † | |
| Assistant | `Button variant="secondary" icon="robot"` "Help me write this" → `Drawer` with APP-17 `AssistantPanel` (drafting mode) | The draft is inserted only when the student chooses "Use this text". `Alert tone="neutral"`: "The assistant helps you write. It doesn't decide what happened or what happens next." (STU-06-B) |
| 2 Classify | `Select label="Category" isRequired` (categories TBD), `RadioGroup label="How serious is it?" card isRequired` (Low, Medium, High, each with `Severity` and a one-line description) | |
| 3 Identity | APP-5 `IdentityChoice`: `RadioGroup` (card) "Use my name" / "Stay anonymous for now"; `PrivacyNotice` | Notice text: "Anonymous means the people handling your report see an alias. The report is still tied to your verified account. You can share your name later." (workflow rule 5, STU-07) |
| 4 Review | `dl.ds-record__facts` (category, severity, identity, linked discussion), the text in `.ds-prose`, `Button variant="tertiary"` "Edit" per section | |
| Actions | `Row`: `Button variant="secondary"` "Back", `Button` "Next" / "Submit report"; `MenuButton` (Save draft, Discard draft) | Discard: `Modal role="alertdialog"` "Discard this draft?" |
| Submitted | `Alert tone="success" title="Report submitted"` with the reference in mono and the first office ("With your adviser" or "With the Program Chair"), then a link to P20 | The reason for a bypass is not shown to the student. |

**States**

| Case | Result | Test |
|---|---|---|
| Narrative, category, severity, and identity mode given | Report created, routed to an authorized office, visible on P18 and P20 | STU-06-V |
| Missing narrative or classification | Step error; no report or notification created | STU-06-I |
| AI drafting or classification | Student reviews before submitting; AI decides nothing | STU-06-B |
| Started from a discussion | Report keeps the link to the post | STU-08-V |
| Draft save fails | `Alert tone="danger"` "Couldn't save your draft. Your text is kept on this device." | |

**Open.** Report categories; attachments; who may submit (STU-06 "submission permissions need final policy"); route exceptions.

---

## P20 Report — `/reports/$reportId`

| | |
|---|---|
| File | `src/routes/_app/reports/$reportId.tsx` |
| Permission | `report.viewOwn` (owner only) |
| Modules | STU-09, STU-07, STU-08 |
| Stories | US-27, US-29 |
| Tests | STU-09-V/I/B, STU-07-V/I/B |
| Template | T7 Record |
| Reference | design system `ReportDetail.tsx` |

**Access.** The reporting student only (O). Handlers use P22. Anyone else, and any missing ID, gets the restricted record copy.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Trail | `Breadcrumbs` (My reports / RPT-0241) | Reference, never the subject. |
| Heading | `SectionHeader level={1} split title="General concern" description="Report RPT-0241"`, `actions` = status `Badge`, `Severity` | After Guidance: "Case File CF-0032 (from report RPT-0241)". |
| Facts | `dl.ds-record__facts`: reference, type (Report or Case File), current office, submitted, identity ("Anonymous to handlers" or "Named"), linked discussion | |
| Boundary | `PrivacyNotice title="Who can open this"` | "You and the office handling it. Internal notes stay with the office." |
| Tabs | `Tabs`: Progress, Messages (`count` unread) | |
| Progress | `CaseTimeline` with public-safe milestones (`state` done / current / pending) | No internal notes, no other students' identities (STU-09-B). |
| Messages | DS-3 `MessageList` + `MessageComposer` with the handling office | Header shows "You appear as Reporter A-12" while anonymous. |
| Disclose | `Button variant="secondary"` "Share my name" → APP-5 `DiscloseIdentityDialog` = `Modal role="alertdialog" title="Share your name with the Guidance Office?"` | Body says who will see it and that it is recorded. Disclosure is per office and logged (STU-07). |
| Source | `Link` "Started from your discussion" (P06) | When converted (STU-08). |
| Help | `Button variant="tertiary" icon="robot"` "What does this stage mean?" → assistant drawer | ALL-04. |

**States**

| Case | Result | Test |
|---|---|---|
| Own report, then its Case File | Correct type, stage, office, timestamped milestones | STU-09-V |
| Nonexistent ID | Restricted record copy | STU-09-I |
| Another student's report or an internal note | Restricted record copy; notes never sent to this page | STU-09-B |
| Send anonymous message, then disclose | The office sees the name only after the student's action; change logged | STU-07-V |
| Close the disclosure dialog | Identity stays as it was | STU-07-I |
| Unauthorized viewer opens the thread | Identity and messages hidden | STU-07-B |

**Open.** Whether disclosure can be undone; mandatory disclosure for in-person meetings (STU-07-B); withdrawing a report.
