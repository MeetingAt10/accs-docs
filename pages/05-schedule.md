# Schedule: calendar and consultations

Back to [index](README.md) · [sitemap](../sitemap.md)

Consultation states (STU-05, PRO-05): **Requested → Approved → Booked → Completed**, or **Declined** / **Cancelled**.

| State | Badge |
|---|---|
| Requested | `Badge form="pending" tone="warning"` |
| Approved | `Badge form="pending" tone="info"` |
| Booked | `Badge tone="info"` (solid) |
| Completed | `Badge tone="success"` |
| Declined, Cancelled | `Badge form="closed"` |

---

## P13 Calendar — `/calendar`

| | |
|---|---|
| File | `src/routes/_app/calendar.tsx` (search params: `view`, `date`, `layers`) |
| Permission | `calendar.view`; to-dos need `todo.manage` |
| Modules | STU-04, PRO-04 (sync), CHR-06, DEAN-06, GUD-05, OSAS-04, ORG-02 |
| Stories | US-24, US-25 |
| Tests | STU-04-V/I/B, DEAN-06-V/I, GUD-05-V, OSAS-04-V/I |
| Template | T6 Schedule |

**Access by role**

| Role | Access | Layers |
|---|---|---|
| Student | ✓ | Academic year, Campus, CCS events, Faculty availability, My to-dos |
| Professor | ✓ | Same, with own availability and booked consultations |
| Organization | S† | CCS events, own organization's events |
| Program Chair, Dean, Guidance, OSAS | R | Academic year, Campus, CCS events, Faculty availability |
| Staff, Administrator, President | — | |

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} split title="Calendar"`, `actions` = `Button icon="plus"` "Add to-do" (students), `Button variant="secondary"` "Manage availability" (professors, P17) | |
| Toolbar | `Row`: `SegmentedControl` (Month, Week, List), `IconButton icon="caret-left" label="Previous"`, `IconButton icon="caret-right" label="Next"`, `Button variant="tertiary"` "Today" | |
| Layers | `TagGroup label="Show" selectionMode="multiple"` with lane tints | Switching calendars (STU-04). |
| Month, List | DS-5 `EventCalendar` | Tablet and desktop Month view use the seven-column calendar grid. Phone Month view replaces the grid with a date-grouped event list and a native date input. List view presents the month’s events in date and time order. |
| Week | `Timetable` with `TimeBlock` (`lane="schedule"` institutional, `lane="community"` organization, `lane="consult"` availability), `NowLine` | Tablet and desktop show the timetable grid; phones show events grouped by day in an agenda list. Selecting an event opens its details in a bottom drawer on phones. |
| Event detail | `Popover` on desktop, `Drawer side="bottom"` on phones: title, type `Badge`, time, description, contact (`Link` to P38), availability state | Availability shows `Link` "Request this time" (P15). No edit controls without authority. |
| Add to-do | `Modal title="Add to-do"`: `TextField`, `DatePicker label="Due"`; `PrivacyNotice` "Only you can see your to-dos." | |

**States**

| Case | Result | Test |
|---|---|---|
| View an event and add a to-do | Details and availability shown; the to-do saves to the student's calendar only | STU-04-V |
| Date with no events, or an unavailable slot | Nothing invented; unavailable slots are not bookable | STU-04-I |
| Another student's to-do, or editing an institutional event | Hidden or denied | STU-04-B |
| Nonexistent event | No false event | DEAN-06-I, OSAS-04-I |

**Open.** Who publishes academic, campus, and CCS events; what "calendar sync" covers (PRO-04); which students see which faculty availability.

---

## P14 Consultations — `/consultations`

| | |
|---|---|
| File | `src/routes/_app/consultations/index.tsx` |
| Permission | `consultation.request` (student view) or `consultation.decide` (professor view) |
| Modules | STU-05, PRO-05 |
| Stories | US-21, US-22, US-23 |
| Tests | STU-05-V, PRO-05-V |

**Access by role**

| Role | Access | View |
|---|---|---|
| Student | O | My requests, Find faculty |
| Professor (and a Chair or Dean who teaches) | O | To decide, Upcoming, History |
| Everyone else | — | |

**Layout: student**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} title="Consultations"` | |
| Tabs | `Tabs`: My requests, Find faculty | |
| My requests | `Table` (professor, date and time, status `Badge`, updated), rows open P16; labeled rows on phones | |
| Find faculty | `Search label="Find a professor"` + `Grid columns={3}` of DS-7 `PersonCard` (name, `RoleBadge` Faculty, office, next open time) | Only professors the student may consult. Opens P15. |
| Empty | `EmptyState title="No requests yet"` with action "Find a time" | |

**Layout: professor**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} split title="Consultation requests"`, `actions` = `Button variant="secondary"` "Manage availability" (P17) | |
| Tabs | `Tabs`: To decide (`count`), Upcoming, History | |
| To decide | `Table` on tablet and desktop; labeled request cards on phones (student, requested time, status, updated, topic), each opens P16 | |
| Upcoming | `Agenda` of booked consultations | |
| History | `Table` + `Pagination` on tablet and desktop; labeled request cards on phones, with the same P16 destination | |
| Empty | `EmptyState title="No requests to decide"` | |

---

## P15 A professor's open times — `/consultations/faculty/$facultyId`

| | |
|---|---|
| File | `src/routes/_app/consultations/faculty.$facultyId.tsx` |
| Permission | `consultation.request` and a professor the student may consult |
| Modules | STU-05, STU-04 |
| Tests | STU-05-V/I/B |
| Template | T6 Schedule |

**Access.** Student S. Everyone else —.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Trail | `Breadcrumbs` (Consultations / Prof. Reyes) | |
| Professor | DS-7 `ProfileHeader` (compact): `Avatar`, name, `RoleBadge`, office, QCU email `Link` | |
| Picker | `ConsultationSlotPicker` (`days`, `slots` with `state` open / booked / unavailable, `timeZoneNote="Local campus time"`) | |
| Topic | `TextArea label="What do you want to discuss?" maxLength={300}` † | Not in the data model yet. |
| Note | `Alert tone="neutral"` "Nothing is booked until your professor confirms." | |
| Action | `Button` "Request this time" (disabled until a slot is chosen) | Success: `toast` "Request sent", open P16. |

**States**

| Case | Result | Test |
|---|---|---|
| Request an open slot | Request recorded as Requested | STU-05-V |
| Unavailable or booked slot | Not selectable; no booking | STU-05-I |
| Two students request the same slot | At most one succeeds; the other sees `Alert tone="danger"` "That time was just taken. Pick another time." and refreshed slots | STU-05-B |
| No open times | `EmptyState title="No open times this week"` with next-week navigation and a `Link` to message the professor (P11) | |
| Professor not in the student's scope | Restricted state | |

---

## P16 Request — `/consultations/requests/$requestId`

| | |
|---|---|
| File | `src/routes/_app/consultations/requests.$requestId.tsx` |
| Permission | the requesting student (O) or the addressed professor (O) |
| Modules | STU-05, PRO-05 |
| Tests | STU-05-V, PRO-05-V/I/B |
| Template | T7 Record |

**Layout**

| Region | Components | Notes |
|---|---|---|
| Trail | `Breadcrumbs` (Consultations / request date) | |
| Heading | `SectionHeader level={1} split` ("Consultation with Prof. Reyes" or "Request from J. Dela Cruz"), `actions` = status `Badge` | |
| Facts | `dl.ds-record__facts`: date and time, requested at, topic, decision time, reason | |
| History | `CaseTimeline` (Requested, Approved, Booked, Completed; or Declined, Cancelled) | |
| Professor actions | Only valid next steps: `Button` "Approve", `Button` "Book", `Button` "Mark completed", `Button variant="secondary"` "Decline", `Button variant="destructive"` "Cancel" | Decline and Cancel open a `Modal` with a required `TextArea` reason; Cancel uses `role="alertdialog"`. |
| Student actions | `Button variant="secondary"` "Cancel request" → `Modal role="alertdialog"` | Policy TBD. |

**States**

| Case | Result | Test |
|---|---|---|
| Approve, decline, complete, or cancel | State and both calendars update; student notified | PRO-05-V |
| Approve a slot booked by another request | `Alert tone="danger"` "This time is already booked"; no double booking | PRO-05-I |
| Another professor's request, or an impossible transition | Restricted state, or the action is not offered | PRO-05-B |

**Open.** Cancellation and rescheduling rules and notice periods; meeting mode and place.

---

## P17 Availability — `/consultations/availability`

| | |
|---|---|
| File | `src/routes/_app/consultations/availability.tsx` |
| Permission | `consultation.decide` |
| Modules | PRO-04 |
| Stories | US-20 |
| Tests | PRO-04-V/I/B |
| Template | T6 Schedule |

**Access.** Professor O (own schedule). Everyone else —.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} split title="My availability"`, `actions` = `Button icon="plus"` "Add times" | |
| View | `SegmentedControl` (Week, List) | |
| Week | `Timetable` with `TimeBlock form="open"` (available), `form="disabled"` (blocked), `form="taken"` (booked, opens P16), `NowLine` | Selecting an open block opens a `Popover`: Mark unavailable, Delete. |
| Add times | `Modal title="Add times"`: `DatePicker`, `Select` start, `Select` end, `RadioGroup label="Repeat"` (Once, Weekly until the end of term †) | `Alert tone="neutral"` "Students see changes right away." |

**States**

| Case | Result | Test |
|---|---|---|
| Mark an unbooked slot available or unavailable | Eligible students' calendars reflect it | PRO-04-V |
| Overlapping or invalid slot | Field error ("Overlaps 10:00–11:00"); nothing saved | PRO-04-I |
| Change a booked slot | `Modal role="alertdialog"` "This time is booked. Changing it needs a cancellation." with Keep / "Cancel the booking" (P16 flow); a booking never disappears silently | PRO-04-B |

**Open.** Overlap policy, recurring slots, changes to booked slots.
