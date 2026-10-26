# Community: feed, posts, spaces, organizations

Back to [index](README.md) · [sitemap](../sitemap.md)

Post labels (flair, US-13): **Discussion, Rant, Assistance, Support**. Announcements carry a priority instead (levels TBD; proposed General, Important, Urgent).

---

## P04 Feed — `/feed`

| | |
|---|---|
| File | `src/routes/_app/feed/index.tsx` (search params: `scope`, `label`, `q`, `page`) |
| Permission | `feed.read` |
| Modules | ALL-03, STU-02, PRO-02, CHR-06, DEAN-06, GUD-05, OSAS-04, ORG-02, STF-01 |
| Stories | US-08, US-09, US-10, US-12, US-13, US-14 |
| Tests | ALL-03-V/I/B, STU-02-V, PRO-02-B, STF-01-V |
| Template | T2 Feed |

**Purpose.** CCS-wide discussions and official announcements, plus each person's permitted spaces, filterable by scope and label. Shows source, audience, publish time, and priority on every item (ALL-03).

**Access by role**

| Role | Access | Scope tabs | Can post |
|---|---|---|---|
| Student | S | CCS, My program, My classes, Organizations (member) | Yes (P05) |
| Organization | S | CCS, Our organization | In own organization |
| Professor | S | CCS, My classes (taught), Faculty | Classes, Faculty |
| Staff | S | CCS | † (D-06) |
| Program Chair | S | CCS, Program, Faculty | Program |
| Dean | S | CCS, all programs, Faculty | CCS-wide |
| Guidance, OSAS | S | CCS, permitted spaces | Permitted posts |
| Administrator | R | CCS | No (moderates on P49) |
| President | R | CCS | No |

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} split title="Feed"`; `actions` = `Button icon="plus"` "New post" | Button only with `post.create`. |
| Scope | `Tabs` / `TabList` / `Tab` | Only the scopes the person has; CCS is the default (ALL-03-V). |
| Filters | DS-8 `FilterBar`: `Search label="Search posts"`, `TagGroup selectionMode="multiple"` (Announcements, Discussion, Rant, Assistance, Support), `Select` sort (Newest, Most active); removable chips for active filters | Filters live in the URL so results can be shared within scope. |
| Pinned | `AnnouncementCard` (`lane="schedule"` college and class, `lane="community"` organizations), priority `Badge` in `meta` | Highest priority first. |
| Discussions | `Divider label="Today"` / "Earlier this week", `ForumCard` (`topic` = label, `author` = name or "Anonymous student", `replies`, `status`) | Card opens P06. |
| Paging | `Pagination summary="Showing 1–20 of 118 posts"` | |
| Loading | `Skeleton` cards | |

**States**

| Case | Result | Test |
|---|---|---|
| Open the feed | CCS-wide tab; items show source, audience, time, priority | ALL-03-V |
| Search with no matches | `ViewState kind="empty"` "No posts match 'thesis'" + `Button` "Clear filters"; nothing unrelated substituted | ALL-03-I |
| `?scope=` of a private space without membership | Restricted state; no posts or metadata | ALL-03-B |
| Restricted student | "New post" `isDisabled`; the shell banner says why | STU-02-B |
| Own post held for review | Shown to the author only, with `Badge form="pending"` "In review" (if posts are held, TBD) | ADM-02 |

**Open.** Announcement priority levels; CCS-wide publishing by Staff and organizations (D-06); whether flagged posts are held or published with a flag.

---

## P05 New post — `/feed/new`

| | |
|---|---|
| File | `src/routes/_app/feed/new.tsx` (search params: `space`, `type`) |
| Permission | `post.create`; `announcement.publish` for announcements |
| Modules | STU-02, PRO-02, ORG-02, CHR-06, DEAN-06, GUD-05, OSAS-04 |
| Stories | US-12, US-13, US-15 |
| Tests | STU-02-V/I/B, PRO-02-V/I, ORG-02-V/I/B, CHR-06-V/I, DEAN-06-V/I, GUD-05-V/I/B, OSAS-04-V/B |
| Template | Single form (`Container width="reading"`) |

**Purpose.** Write a post or an announcement for a permitted audience with a required label (APP-3 `PostComposer`).

**Access by role**

| Role | Access | Audiences offered | Extras |
|---|---|---|---|
| Student | S | Member spaces | Identity choice where anonymous is allowed |
| Organization | S | Own organization; wider only with explicit authority | Announcements with event details |
| Professor | S | Taught classes, Faculty | Class announcements |
| Program Chair | S | Program, Faculty | Program announcements |
| Dean | S | CCS-wide, Faculty | College announcements |
| Guidance, OSAS | S | Permitted spaces | Case-content warning |
| Staff | † | TBD | |

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `Breadcrumbs` (Feed / New post), `SectionHeader level={1} title="New post"` | |
| Type | `SegmentedControl` (Post, Announcement) | Only with `announcement.publish`. |
| Audience | `Select label="Post in" isRequired` with `SelectOption description` ("Visible to BSIT 3A members") | Only permitted spaces are listed. |
| Label | `RadioGroup label="Label" row isRequired` (Discussion, Rant, Assistance, Support) | Hidden for announcements. |
| Title | `TextField label="Title" isRequired` | |
| Body | `TextArea label="Post" showCount maxLength={5000} description="Keep personal details out. Posts are checked for harmful words."` | |
| Identity | APP-5 `IdentityChoice` = `RadioGroup` (card): "Show my name" / "Post anonymously", with `PrivacyNotice` "Classmates see 'Anonymous student'. Your verified account stays linked for moderators." | Students only, where the space allows it. |
| Priority | `RadioGroup label="Priority" row` (General, Important, Urgent) | Announcements only (levels TBD). |
| Event details | `Disclosure title="Event details"`: `DatePicker`, `TextField` time and place, `TextField` contact, `Checkbox` "Notify members" | Organization and class announcements (ORG-02). |
| Attachments | `FileUpload` † | Only if attachments are approved. |
| Case warning | `Alert tone="neutral"` "Don't include case details. Case information stays in the case record." | Guidance and OSAS (GUD-05-B, OSAS-04-B). |
| Actions | `Row`: `Button type="submit"` "Publish", `Button variant="secondary"` "Cancel" | Leaving with text asks first: `Modal role="alertdialog"` "Discard this post?" |

**States**

| Case | Result | Test |
|---|---|---|
| Valid post | Created for the chosen audience with the chosen display identity; `toast` "Posted to BSIT 3A"; open P06 | STU-02-V, PRO-02-V, ORG-02-V |
| Missing label, audience, or event fields | Field errors; nothing created | STU-02-I, PRO-02-I, ORG-02-I |
| `?space=` outside membership or scope | `Alert tone="danger"` "You can't post there"; audience not preselected | STU-02-I, CHR-06-I, DEAN-06-I, GUD-05-I |
| Posting restricted | Form replaced by `ViewState kind="restricted"` "Posting is paused until 12 Nov" + link to P36 | STU-02-B |
| Harmful-word flag | `Alert tone="warning"` "Your post was sent for review" (if held, TBD); the flag goes to P49 | STU-02 spec, ADM-02 |
| College-wide audience without authority | Not offered; a crafted request is denied | ORG-02-B |

**Open.** Spaces that allow anonymous posts; priority levels; attachments; holding flagged posts.

---

## P06 Post — `/feed/$postId`

| | |
|---|---|
| File | `src/routes/_app/feed/$postId.tsx` |
| Permission | `feed.read` and read access to the post's space |
| Modules | ALL-03, STU-02, STU-08, PRO-02; entry to ADM-06 |
| Stories | US-09, US-15, US-19, US-28 |
| Tests | ALL-03-B, STU-02-V/B, STU-08-V/I/B, PRO-02-B |
| Template | T7 Record (public variant) |

**Purpose.** Read a post, agree, comment, report it, and, for the author of a concern, choose whether to turn it into a report.

**Access by role.** Anyone who can read the space (S); commenting needs `post.create` in that space. Administrator and President: R. A faculty-only post is restricted for students (PRO-02-B).

**Layout**

| Region | Components | Notes |
|---|---|---|
| Trail | `Breadcrumbs` (Feed / BSIT 3A / Post) | |
| Post | APP-4 `PostView`: `SectionHeader level={1}` (post title); meta `Row` = `Avatar`, name or "Anonymous student", `RoleBadge` (faculty and offices show their official role), space, time; label `Badge outline`; body `.ds-prose`; attachments | |
| Actions | `Row`: DS-10 `ReactionButton` "Agree" with count, `Button variant="tertiary"` "Reply", `MenuButton iconOnly` (Copy link, Report post, Edit and Delete for the author) | |
| Conversion prompt | `Alert tone="info" title="Many people agree with this. Turn it into a report?"` with `actions`: `Button` "Start a report" (P19 `?from=postId`), `Button variant="tertiary"` "Not now" | Author only, when the threshold is met (STU-08). Body: "Only you can decide. Nothing is filed unless you choose to." |
| Comments | DS-9 `CommentThread` / `Comment` (author or alias, `RoleBadge`, time, Reply, `MenuButton` Report) | "Show more" or `Pagination`. |
| Comment box | `TextArea label="Add a comment"`, APP-5 `IdentityChoice` (compact), `Button` "Comment" | Hidden without permission. |
| Report dialog | `Modal title="Report this post?"`: `RadioGroup` reason (Harassment, Hate, Spam, Personal information, Other), `TextArea` details; `ModalActions` Cancel / "Send report" | Body: "Moderators review reports. A report is not a finding." Sends to P49 (ADM-06). |

**States**

| Case | Result | Test |
|---|---|---|
| Not a member of a private space | Restricted record copy | ALL-03-B, PRO-02-B |
| Removed by moderation | `EmptyState title="This post was removed"` | ADM-02 |
| Commenting restricted | Comment box replaced by `Alert tone="neutral"` "Commenting is paused until 12 Nov" | STU-02-B |
| Author accepts the prompt | P19 opens linked to the post | STU-08-V |
| Author declines or ignores | Stays a discussion; no report | STU-08-I |
| High engagement without consent | No report is created | STU-08-B |
| Anonymous author | Readers never see the verified account | STU-02-B |

**Open.** Agreement threshold (D-07); report reasons; who may resolve an alias (D-08).

---

## P07 Spaces — `/spaces`

| | |
|---|---|
| File | `src/routes/_app/spaces/index.tsx` |
| Permission | `roster.view` (students: enrollment) |
| Modules | STU-01, PRO-01, CHR-01 |
| Stories | US-01, US-02, US-03 |
| Tests | STU-01-V/I, PRO-01-V/I, CHR-01-B |

**Purpose.** The sections and subjects a person belongs to or oversees.

**Access by role**

| Role | Access | Title | Shows |
|---|---|---|---|
| Student | O | My classes | Current-term enrolled sections and subjects |
| Professor | S | My classes | Handled sections and subjects |
| Program Chair | S | Program sections | All sections in the program |
| Dean | S | Sections | All CCS sections, filter by program |

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1}` with the term in the description | |
| Student and Professor | `Grid columns={3}` of `Card`: `CardHeader` with `DoorPlate size="sm" outline code="BSIT 3A"`, `CardTitle` (subject), `CardMeta` (professor or student count), `CardAction` links "Open space" (P08) and "Group chat" (P12) | |
| Chair and Dean | DS-8 `FilterBar` (`Search`, `Select` program for the Dean, `Select` year level) + `Table` (section, program, year, adviser, students, subjects), rows link to P08, `Pagination` | |

**States**

| Case | Result | Test |
|---|---|---|
| Current enrollment | Current sections and subjects listed | STU-01-V, PRO-01-V |
| No enrollment yet | `EmptyState title="No classes linked to your account yet"` "Classes come from enrollment records. Contact the CCS office if this is wrong." | STU-01-I |
| Enrollment or assignment changes | List refreshes from the approved roster | STU-01-I, PRO-01-I |
| Chair asks for another program | Restricted state | CHR-01-B |

---

## P08 Space — `/spaces/$spaceId`

| | |
|---|---|
| File | `src/routes/_app/spaces/$spaceId.tsx` |
| Permission | membership (student), assignment (professor), program or college scope (Chair, Dean) |
| Modules | STU-01, STU-02, PRO-01, PRO-02, CHR-01 |
| Stories | US-01, US-02, US-09, US-10 |
| Tests | STU-01-B, PRO-01-V/I/B, PRO-02-V/B |
| Template | T2 Feed with tabs |

**Access by role**

| Role | Access | Tabs |
|---|---|---|
| Student (enrolled) | S | Discussion, Announcements, Members (names only, †) |
| Professor (assigned) | S | Discussion, Announcements (can publish), Roster |
| Program Chair | R | Discussion, Roster |
| Dean | R | Discussion, Roster |

**Layout**

| Region | Components | Notes |
|---|---|---|
| Trail | `Breadcrumbs` (My classes / BSIT 3A) | |
| Heading | `SectionHeader level={1} split title="BSIT 3A · Data Structures" description="Prof. Reyes · 42 students"`, `actions` = `Button` "New post" (P05 `?space=`), `Button variant="secondary" icon="chat"` "Group chat" (P12) | |
| Tabs | `Tabs`: Discussion, Announcements, Roster or Members | |
| Discussion | DS-8 `FilterBar` (compact), `ForumCard` list, `Pagination` | |
| Announcements | `AnnouncementCard` list | |
| Roster | `Search` + `Table` (`Avatar`, name, student ID, program, status `Badge` Enrolled / Dropped), `Pagination` | Student ID only for Professor, Chair, Dean. |
| Members | Names and roles only | Students; visibility of classmates is TBD. |

**States**

| Case | Result | Test |
|---|---|---|
| Not enrolled, or a guessed URL | Restricted record copy | STU-01-B |
| Unassigned professor | Restricted record copy | PRO-01-B |
| Roster after an enrollment change | Shows the approved current roster | PRO-01-I |

**Open.** Whether students see the class member list.

---

## P09 Organizations — `/organizations`

| | |
|---|---|
| File | `src/routes/_app/organizations/index.tsx` |
| Permission | `feed.read` |
| Modules | ORG-01 |
| Tests | ORG-01-I |

**Access.** All roles R. Students see a "Member" badge on their own organizations.

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} title="Organizations" description="Approved CCS student organizations."` | |
| Search | `Search label="Find an organization" showLabel` | |
| List | `Grid columns={3}` of `CardLink`: `Tile icon="flag" tone="community"`, title, member count, `Badge` "Member" | Opens P10. |

**States.** No match: `ViewState kind="empty"` "No organization named 'x'"; no invented space (ORG-01-I).

---

## P10 Organization — `/organizations/$orgId`

| | |
|---|---|
| File | `src/routes/_app/organizations/$orgId.tsx` |
| Permission | `feed.read`; membership for private posts and the member list |
| Modules | ORG-01, ORG-02 |
| Stories | US-11, US-12 |
| Tests | ORG-01-V/I/B, ORG-02-V/I/B |
| Template | T2 Feed with tabs |

**Access by role**

| Role | Access | Gets |
|---|---|---|
| Organization representative | ✓† | All tabs, "New announcement", "New post"; member management TBD (D-06) |
| Student member | S | Feed (public and private), Events, Members (names) |
| Everyone else | R | Public posts and events; no Members tab |

**Layout**

| Region | Components | Notes |
|---|---|---|
| Heading | `SectionHeader level={1} split` (organization name), verified identity as `Tile icon="flag" tone="community" label="Verified organization"` + `Badge tone="info"` "Verified" | ORG-01: show the organization badge. |
| Actions | `Button` "New announcement" (P05 `?space=org&type=announcement`), `Button variant="secondary"` "New post" | Representative; members get "New post" only. |
| Tabs | `Tabs`: Feed, Events, Members, About | |
| Feed | `AnnouncementCard lane="community"`, `ForumCard` | Private posts only for members. |
| Events | `Agenda` / `AgendaItem lane="community"` (chip = date), `Link` "See in calendar" (P13) | |
| Members | `Table` (`Avatar`, name, role in organization: Officer, Member) | Members and representative. |
| About | `.ds-prose` | |

**States**

| Case | Result | Test |
|---|---|---|
| Representative or member opens the space | Badge and member content; roster from verified membership | ORG-01-V |
| Unknown organization ID | Restricted record copy | ORG-01-I |
| Nonmember opens a private post or tries administration | Restricted; no private content | ORG-01-B |
| Announcement without required event fields | Field errors on P05 | ORG-02-I |
| College-wide audience without authority | Denied; flagged content goes to review | ORG-02-B |

**Open.** Representative powers, member management, wider publishing (D-06).
