# Delta: account provisioning and the concern flow for the 50% check

**Status:** implemented on dev, 2026-11-05
**Source:** capstone consultation of 2026-11-04. Summary and transcript are in `~/Development/whisper/output/2026-11-04/` (`capstone_summary_20261104.txt`, `capstone_20261104_132842.md`). The no-registration rule is a team decision made after the consultation; it is not in the transcript.

## Existing behavior touched

- [`2026-11-01-real-auth-and-visual-refactor.md`](2026-11-01-real-auth-and-visual-refactor.md) added public self-registration. This delta removes it.
- [`../sitemap.md`](../sitemap.md) lists ten roles with no secretary, and only an administrator can change roles.
- Domain data (reports, consultations, messages, notifications, audit) lives in one in-memory object per Lambda container (`backend/src/store.ts`, `backend/src/app.ts`). Only users, credentials, sessions and reset codes persist. A filing made on one container is not visible on another, so the two-person demo cannot work reliably on the hosted stack.
- Role and `active` changes (`updateRoles` in `backend/src/services.ts`) are written to that same in-memory object.
- The API is HTTP only. There is no WebSocket API, email service or Excel parser.

## Requested change

### 1. No public registration (slice A, done first)

There is no registration on the landing page or anywhere else. Student accounts come only from the MIS/registrar list imported by the secretary, or are added by the secretary. Remove `POST /api/auth/register`, the `/register` page, every link to it, and the "Create account" calls to action on the landing and Meeting@10 sites.

### 2. Account provisioning (slice B)

- A seeded Dean signs in and sees account types (secretary, faculty, advisor, program chair), all disabled at first. The Dean activates the secretary.
- Add a `secretary` role. Activation emails the secretary a link and a generated temporary username and password. Each account gets its own temporary password.
- First sign-in forces a change of both username and password before any other page opens. Sign-in accepts the username as an identifier.
- The secretary creates and activates accounts:
  - Faculty: search all or filter by program, select all.
  - Advisor: pick year level 1-4, then pick from faculty. One person can hold several year levels. Assignments can change every year.
  - Program chair: pick IS, CS or IT, then pick from that program's faculty.
  - Students: upload the MIS Excel list (includes Gmail), or add one by hand. A student's advisor follows from program and year level.
- Faculty and advisor stay one account (the `professor` role plus advisor assignments). One login, modules shown by role and assignment.
- Role, `active`, and assignment changes persist across instances and cold starts.

### 3. Concern flow (slice C)

- Student files a concern: writes the statement, then a digital consent form opens. Without consent, the advisor cannot read it. Consent is recorded with time and wording.
- Student may also allow the program chair or Dean to see it. Otherwise only the advisor sees it.
- If the concern is about the student's advisor, the advisor is removed from the recipients and is not notified. The student's program chair receives it instead.
- The advisor gets an in-app notification and an email if email is configured, sees a concern list, and opens the statement.
- Student picks how to talk: system chat, video call, voice call, or face-to-face. Each mode asks for its own consent, including consent to record a call.
  - Chat is a real two-way conversation, refreshed by polling. No new WebSocket infrastructure.
  - Video and voice calls use a meeting link the advisor adds, plus a schedule. No WebRTC or telephony provider.
  - Face-to-face uses a schedule and place.
- Scheduling reuses the existing consultation slots, linked to the concern.
- All of this state persists across instances, following [ADR 0004](../../architecture/adr/0004-dynamodb-domain-tables.md) for the tables this flow needs.

### 4. UI (slice D)

E-services style: one sign-in, then a full-page module picker for the role. Dean, program chair and other unfinished modules appear as tiles with no function. A signed-in user does not sign in again when moving between module sites.

### 5. Dev toolbox (slice E, development stack only)

A floating toolbox button, using the ACCS logo with a toolbox icon, appears on the `-dev` hosts. It lists accounts grouped by role. Picking one swaps the session cookie to that account, so testers can change users and roles without signing out.

- The session cookie is `httpOnly`, so the API performs the swap: it ends the current session and starts one for the chosen account. The browser holds no stored passwords.
- The API routes exist only when `TABLE_PREFIX` is the `accs-dev-` stack. On any other stack they return 404. The caller must already be signed in, and every switch is audited.
- The toolbox code is excluded from the production frontend build.
- Risk accepted for development: anyone signed in on the dev stack can become any dev account, including the Dean. Keep real student data off the dev stack.

### Out of scope for 50%

Escalation past advisor and chair, OSAS and Guidance status updates, confidential flag, strikes and clearance blocks, AI urgency ranking, faculty load upload, teacher evaluation changes, forum changes, QR registration.

## Success criteria

1. No public route, API endpoint, link or copy offers self-registration on any site. The `POST /api/auth/register` route no longer exists: it returns 404 with a session, and 401 without one because the shared API session check runs first, as for any unknown API path.
2. Starting from a fresh store on `accs-dev`, this scenario works with two separate browser sessions, and the data survives a Lambda cold start:
   Dean activates secretary; secretary changes username and password on first sign-in; secretary creates an advisor and imports students from Excel; student changes credentials, files a concern and consents, picks chat; advisor is notified, reads the statement, replies; student sees the reply and answers.
3. A concern without consent is not readable by the advisor. A concern naming the advisor as the subject never reaches that advisor. Both are covered by API tests.
4. A second app instance sharing the same store sees the first instance's concern (test).
5. Backend and frontend typecheck, tests and build pass.

## Implementation notes

Slices B to E are implemented on `dev`. These points differ from the plan above.

- **Scheduling.** The student gives a preferred date and time, and the handler sets the scheduled time, link or place. Booking the handler's open slots through the API exists (`slotId`), but the screen has no slot picker yet.
- **Email.** There is no email channel. The UI shows the credentials and the sign-in link once to the Dean or secretary to pass on.
- **Program chair.** For a concern about the adviser, the program chair is chosen automatically from the student's program, not by the student.
- **Stack limits.** The AWS account has a concurrency limit of 10 Lambda executions shared by accs-dev and accs-prod. Bursts above that returned 503 during testing.
- **Excel columns.** The column names are assumptions until a sample MIS file is provided.

## Open items

- Next check date, presenter and default credential format were not stated in the consultation.
- No sample MIS Excel file has been seen, so column names are assumptions until one is provided.
- Invite email needs SES on access-ccs.com (new infrastructure on `architecture` main, which deploys both accs-dev and accs-prod). It is not approved; see Implementation notes for what the app does meanwhile.

## Fold-in record

- Slice A: folded into `sitemap.md` and `pages/` on 2026-11-05.
- Slices B to E: folded into `sitemap.md` (roles, permission matrix, who can see a concern, talk modes, first sign-in), `pages/01-shell-and-auth.md` (first-login page, dev toolbox), `pages/02-home.md` (module picker), `pages/06-reports.md` (P19, P20), `pages/07-cases.md` (P22), `pages/11-admin.md` (`/accounts`), and `frontend/README.md` on 2026-11-05.
