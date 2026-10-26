# People and account: directory, profiles, notifications, assistant, help, settings

Back to [index](README.md) · [sitemap](../sitemap.md)

These pages are available to every role, and each person sees only what their access allows.

---

## P37 Directory — `/directory`

| | |
|---|---|
| File | `src/routes/_app/directory.tsx` |
| Permission | `directory.view` |
| Modules | ALL-02 |
| Stories | US-06, US-07 |
| Tests | ALL-02-V/I |

**Access.** All roles ✓. The directory lists personnel (CCS faculty and staff, Guidance, OSAS), not students.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} title="Directory" description="CCS, Guidance, and OSAS personnel."` | |
| Filters | DS-8 `FilterBar`: `Search` by name, `Select` office (CCS, Guidance, OSAS), `Select` program (IT, CS, IS), `Select` role | |
| Results | `Grid columns={3}` of DS-7 `PersonCard` (`Avatar`, name, `RoleBadge`, title, office or program, QCU email as a mailto `Link`) | Each card opens P38. |
| Paging | `Pagination` | |
| Empty | `ViewState kind="empty"` "No one matches" | No invented entry. |

**States**

| Case | Result | Test |
|---|---|---|
| Browse or search for personnel | Matching cards show name, role, office or program, and QCU email | ALL-02-V |
| Search for a person who does not exist | `ViewState kind="empty"` "No one matches"; no entry is invented | ALL-02-I |
| Loading | `Skeleton` cards in the grid | |

**Open.** Which offices and titles appear for Staff, Administrator, and President accounts; whether students can ever be listed.

---

## P38 Profile — `/people/$userId`

| | |
|---|---|
| File | `src/routes/_app/people/$userId.tsx` |
| Permission | `directory.view` |
| Modules | ALL-02, STU-11 (neutral badge) |
| Stories | US-06, US-47 |
| Tests | ALL-02-V/B, STU-11-B |

**Access.** All roles R, approved fields only.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Trail | `Breadcrumbs` (Directory / name) | |
| Header | DS-7 `ProfileHeader`: `Avatar`, name, one `RoleBadge` per role, title, office, program, QCU email `Link` | |
| About | `Section` with approved professional or educational fields in `dl.ds-record__facts` | Only fields the person has added and the platform approves. |
| Standing | APP-13 `StandingBadge` | Student profiles only, neutral wording only (US-47). Wording TBD (D-08). |
| Actions | `Row`: `Button` "Message", `Button variant="secondary"` "See open consultation times" | "Message" shows only when the pair is allowed by the messaging reach rules in [sitemap 1.4](../sitemap.md#14-who-can-see-or-reach-whom) and opens a new message (P11). The consultation button shows only when the viewer is a student and the profile is a professor the student may consult; it opens P15. |

**Never shown:** personal phone, social links, private photos or videos, offense details (ALL-02 boundary).

**States**

| Case | Result | Test |
|---|---|---|
| Open a verified personnel profile | QCU email, role, office or program, and approved fields appear | ALL-02-V |
| Open another user's public profile | Personal phone, social links, private media, and offense details are absent | ALL-02-B |
| A peer opens a student's profile | Neutral `StandingBadge` only; no offense history | STU-11-B |
| Unknown or hidden user | `ViewState kind="restricted"` with the record copy from [01-shell-and-auth.md](01-shell-and-auth.md): "This record isn't available to you. It may not exist, or it may belong to someone else." | |

**Open.** Neutral standing wording and who sees the badge (D-08); which profile fields are approved.

---

## P39 My profile — `/profile`

| | |
|---|---|
| File | `src/routes/_app/profile.tsx` |
| Permission | `self.manage` |
| Modules | ALL-02 |
| Tests | ALL-02-V |

**Access.** All roles O.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} title="My profile" description="What others see."` | |
| Preview | DS-7 `ProfileHeader` | Live preview of the saved profile and the edits in progress. |
| Official fields | `dl.ds-record__facts`: name, QCU email, roles, program or office | Read-only. Note: "From QCU records. Contact the CCS office to correct them." |
| Editable fields | `TextField` title (faculty and offices), `TextArea` "About" with `maxLength`, `TextField` degree †, `TextField` school †, `FileUpload` profile photo † | Education fields and the photo are proposed. Whether a profile photo is allowed is TBD. |
| Note | `Alert tone="neutral"` "Don't add phone numbers or social links. Profiles show your QCU email only." | |
| Action | `Button` "Save" | Success: `toast`. |

**States**

| Case | Result | Test |
|---|---|---|
| Open own profile | Official fields shown read-only; editable approved fields shown; preview matches what others see on P38 | ALL-02-V |
| Save a change | Saved fields appear in the preview and on P38 | ALL-02-V |
| Save fails | `Alert tone="danger"` beside the form; entered text kept | |

**Open.** Which fields are approved; whether a profile photo is allowed.

---

## P40 Notifications — `/notifications`

| | |
|---|---|
| File | `src/routes/_app/notifications.tsx` |
| Permission | `self.manage` |
| Modules | ALL-05, STU-12 |
| Stories | US-17 |
| Tests | ALL-05-V/I/B, STU-12-V |

**Access.** All roles O. The top-bar popover shows the latest five from the same list and links here ([01-shell-and-auth.md](01-shell-and-auth.md)).

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} split title="Notifications"`, `actions` = `Button variant="tertiary"` "Mark all read" and `Link` "Settings" (P44) | |
| Filter | `Tabs`: All, Priority, Unread (`count`) | Priority holds report replies and case milestones. |
| List | DS-4 `NotificationList` of DS-4 `NotificationItem` (priority marker, type icon, neutral title such as "Update on report RPT-0241", time, unread state, link to the source) | A title names the kind of event, never confidential details. |
| Grouping | `Divider label` "Today" and "Earlier" | |
| Paging | "Load more" `Button variant="tertiary"` or `Pagination` | Which one is used is left to build. |
| Empty | `EmptyState title="You're all caught up"` | |

**States**

| Case | Result | Test |
|---|---|---|
| An authorized report reply or case milestone arrives | Source-linked notice appears prioritized above general news | ALL-05-V |
| Open an item whose source is no longer accessible | Restricted record copy; the notification never contained confidential details | ALL-05-I |
| An event for another role or program occurs | This person receives no revealing notification | ALL-05-B |
| Mark an item or all items read, then reload | Read state persists | STU-12-V |
| Loading | `Skeleton` rows | |
| Load fails | `ViewState kind="error"` with `Button` "Try again" | |

**Open.** Delivery channels and timing (ALL-05-B); how long read notifications are kept (D-08).

---

## P41 Assistant — `/assistant`

| | |
|---|---|
| File | `src/routes/_app/assistant.tsx` |
| Permission | `assistant.use` |
| Modules | ALL-04, STU-06 (drafting help), ADM-05 (monitoring) |
| Stories | US-51, US-52 |
| Tests | ALL-04-V/I/B |
| Template | T5 Conversation |

**Access.** All roles S: answers use only data the person may view. The same APP-17 `AssistantPanel` also opens as a `Drawer` from the top bar on every page ([01-shell-and-auth.md](01-shell-and-auth.md)). Drafting help (US-52) applies to students.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} title="Assistant" description="Ask how ACCS works, where to find things, and what a stage means."` | |
| Panel | APP-17 `AssistantPanel` | Contains the three regions below. |
| Prompts | `TagGroup` of suggested prompts tailored to the role | For example "How do I book a consultation?", "What does 'With Program Chair' mean?", "Where are my evaluations?" |
| Conversation | DS-3 `MessageList` | Answers include source `Link`s to pages or Help (P42). |
| Composer | DS-3 `MessageComposer` | |
| Note | `Alert tone="neutral"` "The assistant explains and helps you write. It doesn't decide reports, restrictions, or outcomes, and it can't see records you can't." | Always visible. |
| Reset | `Button variant="tertiary"` "Clear conversation" | |

**States**

| Case | Result | Test |
|---|---|---|
| A platform FAQ, or an authorized schedule or milestone question | Relevant answer from permitted data | ALL-04-V |
| An out-of-scope or unanswerable question | Fallback reply; no fabricated platform fact. Wording TBD | ALL-04-I |
| A request for another student's case, or for a guilt or restriction decision | Declined; no confidential data and no decision returned | ALL-04-B |
| Service failure | `Alert tone="danger"` "The assistant isn't available right now." with a `Link` to Help (P42) | |
| Waiting for a reply | `Spinner` or a pending `MessageBubble` in the list | |

**Open.** AI provider data handling and log retention (D-08); fallback wording (ALL-04-I); who may read assistant logs (ADM-05 boundary).

---

## P42 Help — `/help`

| | |
|---|---|
| File | `src/routes/_app/help.tsx` |
| Permission | signed in |
| Modules | ALL-04 (FAQ) |
| Stories | US-51 |

**Access.** All roles ✓. Topics follow the person's permissions. For example, a "Cases" topic appears only for handlers.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} title="Help"` | |
| Search | `Search label="Search help"` | |
| Topics | `Accordion` of `Disclosure` questions grouped by topic | Topics: Getting started, Feed and posts, Messages, Consultations, Reports and privacy, Evaluations, Account standing, Account and sign-in. |
| Privacy | `PrivacyNotice` inside the Reports and privacy topic | States who can open a report. No promise of anonymity or outcomes. |
| More help | `Row` "Still need help?": `Button variant="secondary"` "Ask the assistant", `Link` "Contact the CCS office" | The button opens the assistant drawer. The link opens P37 filtered to the CCS office. |
| Empty | `ViewState kind="empty"` | Suggests asking the assistant. |

**States**

| Case | Result | Test |
|---|---|---|
| Search with a match | Matching questions listed under their topics | |
| Search with no match | `ViewState kind="empty"` with a suggestion to ask the assistant | |
| A topic the role has no access to | Topic not listed | |

**Open.** FAQ content and who maintains it; whether "Contact the CCS office" needs a dedicated office entry.

---

## P43 Settings — `/settings`

| | |
|---|---|
| File | `src/routes/_app/settings/index.tsx` |
| Permission | `self.manage` |
| Modules | ALL-01 (account), design system themes and density |

**Access.** All roles O.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} title="Settings"` | |
| Account | `dl.ds-record__facts`: name, QCU email, each role as a `RoleBadge` with its scope (for example "Program Chair · BSIT"), account status | Read-only. |
| Account links | `Link` "How roles are assigned" (P42), `Button variant="secondary"` "Sign out" | Sign out ends the session and opens P02 with the signed-out message. |
| Other devices | `Button variant="secondary"` "Sign out of other devices" † | Session policy TBD (ALL-01-G). |
| Appearance | `RadioGroup row` Theme (Light, Dark, Match my device), `RadioGroup row` Density (Comfortable, Compact) | Applied at once. Density default per role is set in the shell ([01-shell-and-auth.md](01-shell-and-auth.md)). |
| Notifications | `Link variant="standalone"` "Notification priorities" (P44) | |

**States**

| Case | Result | Test |
|---|---|---|
| Open settings | Account facts match the signed-in account | |
| Change theme or density | The app updates immediately and the choice is kept | |
| Person holds several roles | One `RoleBadge` and scope per role | |

**Open.** Whether the theme and density choice is stored per account or per device; "Sign out of other devices" and session length (ALL-01-G).

---

## P44 Notification priorities — `/settings/notifications`

| | |
|---|---|
| File | `src/routes/_app/settings/notifications.tsx` |
| Permission | `self.manage` |
| Modules | STU-12, ALL-05 |
| Stories | US-17 |
| Tests | STU-12-V/I/B |

**Access.** All roles O. The list of categories differs by role.

| Category | Shown to |
|---|---|
| Report replies, Case milestones, Consultation updates, Messages, Announcements, Evaluations | Roles that receive those events |
| New assigned reports | Handlers |
| Moderation queue | Administrators |

**Layout**

| Region | Components | Notes |
|---|---|---|
| Trail | `Breadcrumbs` (Settings / Notifications) | |
| Heading | `SectionHeader level={1} title="Notification priorities"` | |
| Categories | One row per category with a `RadioGroup row` (Urgent, Normal, Off) | Role-specific categories are added to the common set. |
| Required | `Badge` "Required" with disabled controls and a one-line reason | For categories that policy requires. Which categories are mandatory is TBD. |
| Note | `Alert tone="neutral"` "Delivery channels such as email are still being decided." | |
| Action | `Button` "Save" | Success: `toast`. |

**States**

| Case | Result | Test |
|---|---|---|
| Change a permitted category and mark a notice read | Setting and read state persist | STU-12-V |
| Invalid or unavailable category choice | Not saved | STU-12-I |
| Lower the priority of report replies | Access to the report and mandatory notices are unchanged | STU-12-B |
| Save fails | `Alert tone="danger"` beside the form; choices kept | |

**Open.** Which categories are mandatory; delivery channels (ALL-05-B); category lists for Organization and Staff (D-06).
