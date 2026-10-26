# Evaluations

Back to [index](README.md) · [sitemap](../sitemap.md)

360-degree evaluation (objective 6): students evaluate their section's faculty; professors evaluate peers and give upward feedback on the Program Chair and Dean; the Dean configures, publishes, and monitors cycles. Participation (100% vs 50/50, D-01) and enforcement (grades, Good Moral, D-02) are undecided, so no page states a penalty.

Cycle states: **Draft → In review (faculty) → Published → Closed**. Form states for an evaluator: **Not started**, **Submitted**.

---

## P26 Evaluations — `/evaluations`

| | |
|---|---|
| File | `src/routes/_app/evaluations/index.tsx` |
| Permission | `evaluation.fill`, `evaluation.results`, or `evaluation.configure` |
| Modules | STU-10, PRO-07, CHR-04, DEAN-03 |
| Stories | US-40, US-42, US-43 |
| Tests | STU-10-V, PRO-07-V, CHR-04-V |

**Access by role**

| Role | Access | Shows |
|---|---|---|
| Student | O | Forms assigned to the student's section this cycle |
| Professor | O | Peer and upward forms; "Questions to review" when a cycle is in review; link to own results |
| Program Chair | † | Assigned forms (TBD); link to program results |
| Dean | ✓ | Links to cycles and results; assigned forms (TBD) |
| Everyone else | — | |

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} split title="Evaluations" description="1st semester 2026–2027 · closes 30 Nov"` | Dean `actions`: `Button` "Manage cycles" (P30), `Button variant="secondary"` "Results" (P28). |
| Progress | `Progress label="2 of 5 submitted"` | Students and professors. |
| Tabs | `Tabs`: To complete, Questions to review (professors, `count`) | |
| Forms | `Table`: evaluated person (`Avatar`, name), subject or relation (Peer; Upward: Program Chair; Upward: Dean), due, status `Badge`, action `Link` ("Start" or "View receipt") | Opens P27. |
| Results link | `Link variant="standalone"` "See my results" (P28) | Professors. |
| Empty | `EmptyState title="No evaluations open" description="You'll get a notification when the next cycle opens."` | |

---

## P27 Evaluation form — `/evaluations/forms/$assignmentId`

| | |
|---|---|
| File | `src/routes/_app/evaluations/forms.$assignmentId.tsx` |
| Permission | `evaluation.fill` and assignment to this evaluator |
| Modules | STU-10, PRO-07, CHR-04 |
| Stories | US-40, US-42 |
| Tests | STU-10-V/I/B, PRO-07-V/I/B, CHR-04-V |
| Template | T4 Wizard |

**Access.** The assigned evaluator only (O).

**Layout**

| Region | Components | Notes |
|---|---|---|
| Trail | `Breadcrumbs` (Evaluations / Prof. Reyes · Data Structures) | |
| Heading | `SectionHeader level={1} title="Evaluate Prof. Reyes" description="Data Structures · BSIT 3A · 1st semester"` | |
| Boundary | `PrivacyNotice` "Your answers are combined with others. Summaries never show individual answers." | STU-10 acceptance. |
| Instructions | `.ds-prose` from the cycle | |
| Progress | `Progress label="8 of 12 answered"`, or `Stepper` when the form has several criteria groups | |
| Questions | DS-6 `RatingScale` per criterion (`isRequired`, words at both ends) | Questions are role-specific: student, peer, or upward (PRO-07). |
| Comments | `TextArea label="Anything else?" maxLength={1000}` (optional) | |
| Submit | `Button` "Submit evaluation" → `Modal role="alertdialog" title="Submit your evaluation?"` "You can't change answers after you submit." | |
| Receipt | `Alert tone="success" title="Submitted 12 Nov, 10:42"` and the read-only answers | |

**States**

| Case | Result | Test |
|---|---|---|
| Published, assigned form submitted | Completion saved once | STU-10-V, PRO-07-V |
| Missing required answers | `Alert tone="danger"` "3 questions need an answer" at the top and each question marked invalid; nothing recorded | STU-10-I, PRO-07-I |
| Submit twice | Second attempt not recorded; receipt shown | STU-10-I, PRO-07-I |
| Unassigned faculty, target, or cycle | Restricted state | STU-10-B, PRO-07-B |
| Cycle closed | `ViewState kind="empty"` "This evaluation closed on 30 Nov" | |

---

## P28 Results — `/evaluations/results`

| | |
|---|---|
| File | `src/routes/_app/evaluations/results/index.tsx` (search params: `cycle`, `program`) |
| Permission | `evaluation.results` |
| Modules | PRO-08, CHR-04, DEAN-04 |
| Stories | US-44, US-45 |
| Tests | PRO-08-V/I, CHR-04-V/I/B, DEAN-04-V/I/B |
| Template | T1 Dashboard |

**Access by role**

| Role | Access | Shows |
|---|---|---|
| Professor | O | Own results (the P29 view for themselves) |
| Program Chair | S | Program aggregates and the program's faculty |
| Dean | ✓ | College aggregates, by program, all faculty |
| Everyone else | — | |

**Layout (Chair and Dean)**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} title="Evaluation results"`, `Select label="Cycle"` (published cycles only) | |
| Boundary | `PrivacyNotice` "Individual answers aren't shown. Results appear after a cycle is published." | |
| Stats | DS-1 `Stat`: completion rate, faculty evaluated, overall average | |
| By criterion | DS-2 `BarChart` | Table fallback. |
| Faculty | DS-8 `FilterBar` (`Search`, `Select` program for the Dean) + `Table` (name, sections, responses, overall average), rows open P29 | No automatic "misconduct" labels (DEAN-04). |

**States**

| Case | Result | Test |
|---|---|---|
| Published cycle | Aggregates match completed responses | PRO-08-V, CHR-04-V, DEAN-04-V |
| Unpublished or empty cycle | `ViewState kind="empty"` "Results for this cycle aren't published yet" | PRO-08-I, CHR-04-I, DEAN-04-I |
| Another program, or individual answers | Restricted; never shown | CHR-04-B, DEAN-04-B |

---

## P29 One faculty member's results — `/evaluations/results/$facultyId`

| | |
|---|---|
| File | `src/routes/_app/evaluations/results/$facultyId.tsx` |
| Permission | `evaluation.results` (professors: own ID only) |
| Modules | PRO-08, CHR-04, DEAN-04 |
| Tests | PRO-08-V/B, DEAN-04-V/B |

**Access.** Professor O (self), Program Chair S (program faculty), Dean ✓.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Trail | `Breadcrumbs` (Results / Prof. Reyes) | |
| Person | DS-7 `ProfileHeader` (compact): name, `RoleBadge`, program | |
| Cycle | `Select label="Cycle"` | |
| Stats | DS-1 `Stat`: overall average, responses, completion | |
| By criterion | DS-2 `BarChart` with the program average as a reference | |
| Feedback themes | `Accordion` of `Disclosure` per theme, each with a count and summarized comments | PRO-08: summarized feedback. |
| Caution | `Alert tone="neutral"` "Results describe feedback. They aren't findings about conduct." | DEAN-04. |
| Follow-up | `TextArea label="Follow-up note"` + `Button` "Save" † | Chair and Dean, if permitted (CHR-04). |

**States.** Another professor's results for a professor: restricted (PRO-08-B). Evaluator identities never appear (DEAN-04-B). Release threshold TBD.

---

## P30 Cycles — `/evaluations/cycles`

| | |
|---|---|
| File | `src/routes/_app/evaluations/cycles/index.tsx` |
| Permission | `evaluation.configure` |
| Modules | DEAN-03 |
| Stories | US-43 |

**Access.** Dean ✓. Everyone else —.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} split title="Evaluation cycles"`, `actions` = `Button icon="plus"` "New cycle" | Opens a `Modal` (`Select` term, `TextField` name) that creates a Draft and opens P31. |
| List | `Table`: term, forms, status `Badge` (Draft, In review, Published, Closed), completion `Meter`, open and close dates | Rows open P31. |
| Empty | `EmptyState title="No cycles yet"` with action "New cycle" | |

---

## P31 Cycle editor and monitor — `/evaluations/cycles/$cycleId`

| | |
|---|---|
| File | `src/routes/_app/evaluations/cycles/$cycleId/index.tsx` |
| Permission | `evaluation.configure` |
| Modules | DEAN-03 (and US-41 review status) |
| Stories | US-41, US-43 |
| Tests | DEAN-03-V/I/B |
| Template | Tabs over T4 |

**Access.** Dean ✓.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Trail | `Breadcrumbs` (Cycles / 1st semester 2026–2027) | |
| Heading | `SectionHeader level={1} split`, `actions` = status `Badge`, `Button` "Publish" (`Modal role="alertdialog"` "Publish this cycle? Assigned students and faculty get their forms."), `Button variant="secondary"` "Close cycle" | Only valid transitions show. |
| Tabs | `Tabs`: Setup, Questions, Faculty review, Completion | |
| Setup | `TextField` name, `TextArea` instructions, `DatePicker` opens and closes, `CheckboxGroup label="Forms"` (Student to faculty, Faculty peer, Upward: Program Chair, Upward: Dean), `Select label="Participation rule"` † with `Alert tone="warning"` "Participation rule not decided (D-01)" | |
| Questions | APP-18 `QuestionEditor` per form: rows of criterion, question, type (`Select`: Rating 1–5, Text), required `Switch`, `MenuButton` (Move up, Move down, Delete); `Button` "Add question"; `Disclosure title="Preview"` rendering DS-6 `RatingScale` | |
| Faculty review | `Table` (professor, status `Badge` Reviewed / Commented / Not yet, comments), `Disclosure` per comment; `Button` "Send for review" | US-41: professors review questions against their duties. |
| Completion | DS-1 `Stat` overall; DS-8 `FilterBar` (program); `Table` by section (assigned, submitted, `Meter`) and by faculty | Counts only. |

**States**

| Case | Result | Test |
|---|---|---|
| Configure, publish, monitor, close | Assigned users get their forms; states change in order | DEAN-03-V |
| Publish with missing fields | `Alert tone="danger"` listing what's missing; "Publish" stays disabled | DEAN-03-I |
| Participation change or penalty | No enforcement controls until QCU confirms (D-01, D-02) | DEAN-03-B |

---

## P32 Question review — `/evaluations/cycles/$cycleId/review`

| | |
|---|---|
| File | `src/routes/_app/evaluations/cycles/$cycleId/review.tsx` |
| Permission | `evaluation.review` |
| Modules | DEAN-03 (consult faculty on criteria), PRO-07 |
| Stories | US-41 |
| Tests | PRO-07-B (criteria need faculty review) |

**Access.** Professor O (cycles in review that evaluate them). Program Chair † (upward forms). Everyone else —.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Trail | `Breadcrumbs` (Evaluations / Review questions) | |
| Heading | `SectionHeader level={1} title="Review the questions about your duties"` | |
| Note | `Alert tone="neutral"` "Your comments go to the Dean before the cycle is published." | |
| Questions | Per question: text, criterion, `RadioGroup row label="Fits my duties?"` (Yes, Needs change), `TextArea label="Comment"` | |
| Submit | `Button` "Send review" → `toast` "Review sent" | Read-only afterwards. |

**States.** Cycle not in review: `ViewState kind="empty"` "Nothing to review right now".
