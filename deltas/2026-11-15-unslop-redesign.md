# Delta: copy cleanup and portal layout redesign

**Status:** dropped, 2026-11-25. Never merged: dev replaced the zinc theme and the Home hero this layout was built on. Kept as a record of the copy and layout ideas.

## Existing behavior touched

- `frontend/src/**` user-visible copy and code comments, `frontend/README.md`, `frontend/index.html`.
- `meetingatten-design-system` documentation prose: `README.md`, `PRODUCT.md`, `CONTRIBUTING.md`, `docs/`, `foundations/`, `themes/`, `components/**/*.md`.
- The Home page layout for every role (`frontend/src/components/app/PageScreen.tsx`, `portal.css`, `community-home.tsx`, `theme.css`). Other page layouts follow one at a time, after the owner reviews each in the dev server.

## Requested change

Decided with the owner on 2026-11-15.

1. Rewrite copy that reads as machine-written, in the frontend and in the design system docs.
2. Redesign the portal layout. The neutral theme stays (zinc surfaces, Geist, Geist Mono, Poppins titles, `frontend/src/theme.css`). The redesign changes layout, hierarchy and density only. The owner first chose the Superdesign canvas, then on 2026-11-15 chose to skip mockups and build in code.
3. In the design system, edit documentation prose only. `CHANGELOG.md`, archived deltas, the structure of `DESIGN.md`, CSS comments, JSDoc, `dist/` and the site code are left alone, so no rebuild or release is needed.

## Copy rules

Order of authority, highest first:

1. Text the server owns or the code compares. Never reword it.
2. This spec's glossary.
3. [`voice-and-tone.md`](../../../meetingatten-design-system/docs/voice-and-tone.md) in the design system.
4. The unslop rules: no em or en dash as punctuation, no connector colons, no arrows or symbols in place of words, no figurative verbs ("carries", "owns", "lives", "rides"), no contrast framing used for effect, sentence case, straight quotes, active voice when the actor is known, one idea per sentence.

Rules 3 and 4 conflict in a few places, and rule 3 wins. "Your text is kept." stays, because the voice guide prescribes it. A do and don't pair such as `"Discard draft", not "OK"` stays, because it is an instruction with an example.

### Never reworded

- Consent wording in `frontend/src/components/app/concern-talk.tsx` (`fallbackWording`). The server saves the same text.
- Statements of who can read, open or be told about a report, and of what is logged, kept or recorded. These are policy. They get glossary spelling and casing only.
- Notices such as "A report is not a finding." and "A flag is not proof of a violation."
- Labels that double as API values, URL parameters, filter ids, or values a regular expression matches.
- Strings a test asserts, unless the glossary changes the term. Then the string and its test change together.
- `src/lib/doc003-modules.json`, `src/routeTree.gen.ts`, sample reports in the dev toolbox.

### Left as they are

- The `—` glyph in an empty table cell and the `–` between two numbers or two times.
- Short noun-phrase captions under a tile or field label, when their meaning is clear.

## Glossary

Settled by the page specs, the backend and `sitemap.md`.

| Term | Rule | Source |
|---|---|---|
| adviser | Spelled "adviser", never "advisor". Lowercase in a sentence ("your adviser"). | Backend uses "adviser" 107 times and "advisor" 12 times; the docs 41 and 3. |
| Role names | Capitalized when they name the role or office: Student, Professor, Program Chair, Dean, CCS Secretary, CCS Staff, System Administrator, Guidance, OSAS. | `sitemap.md` §1.1, `src/lib/role-labels.ts` |
| report | The record a student files, until it reaches Guidance. | `pages/06-reports.md`, wording rule D-03 |
| Case File | The same record from Guidance on. Both words capitalized. | D-03 |
| concern | The matter a report describes. Used in category names ("General concern") and in the question about the adviser. | `pages/06-reports.md` |

Role-specific labels for the `/cases` route are deliberate and stay: "Assigned reports" (Professor), "Report queue" (Program Chair), "Escalated reports" (Dean), "Case Files" (Guidance), "OSAS cases" (OSAS).

### Open pairs

The specs do not settle these, so the copy pass leaves them unchanged. The owner decides.

| Pair | Where |
|---|---|
| "File a concern" and "New report" for the student's action | `grievance.tsx:339`, `module-picker.tsx:56`; `pages/06-reports.md` says "New report" |
| faculty and professor for one person | `schedule.tsx:1575` and `:1580`; `sites.ts:141` and `role-labels.ts:7` |
| appointment and consultation | Nav says "Appointments"; `pages/05-schedule.md` says "Consultations" |
| "Not active" and "Inactive" | `accounts.tsx:313`, `admin.tsx:905` |
| "Submit report" and "Send report" | `concern-talk.tsx:171`, `community.tsx:1679` |
| "Delete post" and "This post was removed" | `community.tsx:1455`, `:1268` |
| "Pick" and "Choose" | `schedule.tsx:1343`, `:1325` |
| "Mark not available" and "Mark unavailable" | `schedule.tsx:2293`, `:2011` |
| "Profile" and "My profile" | `navigation.tsx:872`, `:573` |
| "Restricted features" and "Features paused" | `records.tsx:406`, `:1103` |
| "System admin" (short form) and "System Administrator" | `admin.tsx:99`, `:111` |
| 12-hour times with an en dash in the app, 24-hour with a hyphen in the voice guide | `schedule.tsx:216`; `voice-and-tone.md`, "Times and dates" |
| "the secretary" (short form) and "CCS Secretary" | 21 and 15 uses across `src/`; `getting-started.tsx:12` is asserted by its test |
| "Chair" (short form) and "Program Chair" | `admin.tsx:97`, `accounts.tsx:529`, `grievance.tsx:48` |
| "CCS administrator" and "System Administrator" | `guards.tsx` |
| "Guidance Office" and "Guidance" | `grievance.tsx:665` |
| "Admin", "Administration" and "System Administrator" | `sites.ts:88`, `page-catalog.ts:44`, `role-labels.ts:11` |
| "credentials" and "sign-in details" | `README.md`, `account-import.ts`; UI copy says "sign-in details" |
| "The request could not be completed." (passive fallback) | `api.ts:89`, `dev-toolbox-api.ts:24`, `vite.local-role-launcher.ts:37` |

## Redesign

Built in code and reviewed in the dev server. One Superdesign draft exists (a reproduction of the current student Home, 36.5 credits) in the project "ACCS portal layout redesign". No further drafts are planned.

### Home (P03), done 2026-11-15

Before, the wide column held college news and was mostly empty, and the person's own dated items sat at the bottom of the narrow column.

- The wide first column holds the person's own items, in this order: Today or Coming up, Appointments, Evaluations due, Case queue or Assigned reports, My reports. College news (Announcements, From your spaces) takes the narrow second column. On a phone the own items come first.
- When one group is empty, the other uses the full width.
- When the own-items column is at least 44rem wide (a 1440px window), each row is one line: reference, title, then status and date at the right, with the status chips ending on one line. Below that a row takes two lines, because the status and date would leave the title too little room.
- A report's reference comes before its title, as on the My reports page, because several reports share a title.
- The Home greeting, section headings and rows use the standard sizes again. The rules that enlarged them to fill the empty half are removed.
- The four header buttons keep their labels and use the small size.
- Row dividers are straight lines on every list page. The rows lost their 8px corners, which curled the divider ends.
- No logic filters or sorts the lists differently. The panels, their labels and their links are unchanged.

Not changed: the shell (rail and top bar), the module tiles and KPI strip that office roles see above the panels, and every other page.

### Next candidates

- Office roles' Home: six module tiles and a KPI strip sit above the Case queue and repeat the rail.
- Home shows a Case File with its report reference (`RPT-0245`), while My reports shows `CF-0245`.
- The other ten screens, one at a time.

## Success criteria

1. `npm run typecheck` and `npm test` pass in `frontend`.
2. The frontend diff for the copy pass changes strings, comments and the tests that pin a renamed term. It changes no logic.
3. No "advisor" remains in frontend copy.
4. In the design system, the diff changes no front matter, heading, code block or file outside the documentation list.
5. Each redesigned page is checked by screenshot at 390, 1024 and 1440px for a student, a professor and one office role, at the widths around each layout switch, and once in dark, with no sideways overflow.

## Open items

- The open pairs above.
- `frontend/README.md` said 58 DOC003 modules. The inventory and `sites.test.ts` say 55, so the README now says 55.
- Design system docs that disagree with each other. The copy pass did not change them:
  - `docs/migrating-apps.md` names v1.0.0 and v1.1.0 as the tag to pin. `README.md` pins v2.0.0.
  - `docs/principles.md:19` gives a block radius of 6. `foundations/radius.md:30` gives 8.
  - `foundations/treatments.md:48` lists radii 12, 6, 8, 4, 16, 6. `foundations/radius.md` lists 16, 8, 10, full, 20, 10.
  - `foundations/color.md:41` and `foundations/treatments.md:11` call the canvas cool. The light canvas is ivory `#F7F6F1`.
  - `README.md:131` says CI runs on pushes to `master` and `main`. `CONTRIBUTING.md:46` says `main` only.
  - `src/react/lib/lanes.ts:1` says community is violet and consultations teal. `site/app/_site/HomeSheet.tsx:23` says amber and emerald.

## Progress

- 2026-11-15: copy pass done in both repos. Frontend: 78 files. They change strings and comments only, plus two test files whose asserted strings changed with the glossary, one test title, and a `.superdesign/` line in `.gitignore`. `npm run typecheck` and `npm test` (155 tests) pass. Design system: 92 Markdown files, 214 lines, with no heading, front matter, code block, number or link changed. No test or site build ran there, because only prose changed. Front matter descriptions still use "carries" where the body now says "shows".
- 2026-11-15: Superdesign init files and `design-system.md` written to `frontend/.superdesign/`. One reproduction draft generated, then the owner chose to build in code.
- 2026-11-15: Home layout redesigned in code. `npm run typecheck` and `npm test` (155 tests) pass. Screenshots checked for a student (390, 1024, 1152, 1280, 1366 and 1440px in light, 1440px in dark), a professor and Guidance (390, 1024 and 1440px), and the My reports list. Three states the seed data lacks were shown with data changed in the browser only: a Coming up panel, own items with no news, and news with no own items. The Dean account does not sign in through the local launcher, so Guidance stood in for the office roles.

## Fold-in

- `pages/02-home.md` still describes the old column order. Update it once the owner accepts the Home layout.
