# Hardware and software requirements

Date: 2026-10-18  
Status: Proposed baseline for team review; hardware, browser compatibility, and capacity have not been benchmarked or approved.

## Scope and evidence

ACCS is a web prototype for the College of Computer Studies at Quezon City University Main Campus, covering Information Technology, Computer Science, and Information Systems. The latest paper specifies simulated enrollment lists, sample records, and frontend state management. It excludes biometrics, ID scanners, payment gateways, live university server deployment outside the approved capstone setup, connections to live enrollment or registrar systems, and automatic database synchronization [S1, paragraphs 55–59]. Use synthetic records in demonstrations; do not load sensitive institutional data.

The module inventory describes intended workflows for ten operational roles. These are requirements to demonstrate, not evidence that the workflows are implemented [S2]. The inspected code currently provides a React frontend shell that calls an Express health endpoint. No application database, institutional identity integration, or AI provider is configured in the inspected manifests and entry points [S4–S6].

The refreshed Notion proposal page contains placeholder sections and links to an older, broader proposal. Where that proposal or the inventory suggests persistent or institutional integrations, this document follows the latest paper's explicit prototype limits, pending team approval [S1–S3].

## Client devices

All figures below are recommended planning estimates, not measured minimum requirements. Existing devices should be evaluated before purchases.

| Item | Suggested baseline | Recommended for comfortable use | Evidence or qualification |
|---|---|---|---|
| Desktop or laptop processor | Dual core, approximately 2 GHz | Modern quad core | Estimate for browser use; clock speed alone does not establish suitability. |
| Desktop or laptop memory | 4 GB RAM | 8 GB RAM | Estimate; browser tabs and other applications affect available memory. |
| Desktop display | 1366 × 768 | 1920 × 1080 or greater | Proposed layout targets; not verified. |
| Mobile device | Existing smartphone with a maintained browser; 4 GB RAM suggested | Recent phone with enough free memory for browser use | Responsive browser access; no evidence supports a fixed Android or iOS version floor. |
| Mobile viewport | 360 CSS pixels wide | Test additional common widths | Proposed responsive target; not verified. |
| Client network | Stable 5 Mbps connection per active client suggested | Reliable connection with low latency | Planning estimate, not measured traffic demand or a service guarantee. |
| Input and graphics | Keyboard and pointing device on desktop; touchscreen on mobile; integrated graphics | Existing equipment | No biometric device, ID scanner, dedicated GPU, or printer purchase is required. |

Windows, Linux, and macOS are suitable desktop platform choices when they support a maintained browser. Target current maintained Chrome, Edge, Firefox, and Safari versions, including mobile browsers. ACCS compatibility across these platforms has not been verified; no exact browser version matrix is established.

## Development workstations and software

| Item | Minimum practical baseline (recommended estimate) | Recommended baseline |
|---|---|---|
| Processor | Modern quad core | Modern processor with at least six cores |
| Memory | 8 GB RAM | 16 GB RAM |
| Free SSD space | 20 GB | 50 GB |
| Network | Stable 10 Mbps connection suggested | Faster connection for dependency downloads and collaboration |
| Operating system | Maintained Windows, Linux, or macOS with compatible Node.js tooling | Maintained release familiar to the team |

Storage estimates cover source, dependencies, build output, and ordinary development tools. They exclude local AI model weights, large attachments, operating system storage, and additional virtual machines.

| Software | Current implementation | Requirement or recommendation |
|---|---|---|
| Frontend | React 19, Vite 7, TypeScript 5.9, TanStack Router 1, TanStack Query 5 declared in `frontend/package.json` | Use the repository's dependency ranges and reviewed resolved versions. These are manifest declarations, not an installed-package inventory. |
| Backend | Express 5 and TypeScript 5.9; backend manifest declares Node.js `>=22` | Backend currently exposes `GET /api/health`. It is needed to demonstrate the existing health indicator, not a requirement for simulated browser workflows. |
| Node.js | Backend engine range is broader than Vite's requirement | Use a patched Node.js 24 LTS release. For Node.js 22, use 22.12 or later: Vite documents Node.js 20.19+ or 22.12+, while this backend requires at least 22. See [Vite requirements](https://vite.dev/guide/) and [Node.js release status](https://nodejs.org/en/about/previous-releases). |
| Package and source tools | README files allow npm or pnpm | Use Git and npm as a proposed team baseline; agree on one package manager and retain its lockfile. No package lockfile was found during this review. |
| Editor | No required editor | Any editor with TypeScript support; an IDE is optional. |
| Spreadsheet editor | Module inventory is an `.xlsm` reference document | Needed only to review or maintain that document, not to run ACCS. |
| AI runtime | No AI provider or local model declared in the inspected manifests | Provider and model remain undecided. Mocked responses or a remote API require no local GPU. Local inference requires separate hardware sizing. |

Version support changes over time. Review maintained releases and dependency compatibility before deployment; these recommendations do not certify a tested environment.

## Prototype hosting

A static HTTPS host or CDN is sufficient for the proposed prototype's browser assets and simulated frontend state. Configure client route fallback where the deployed routes require it. Browser state is not a shared institutional database and must not be presented as durable university records.

The existing frontend also requests `/api/health`. If that indicator is demonstrated, provide the Express service and route `/api` to it. Otherwise, a static deployment alone will not satisfy the current health request [S5–S6].

| Component | Proposed allocation | Qualification |
|---|---|---|
| Prototype frontend | Static hosting or CDN with HTTPS | No dedicated application server or database is required for the paper's simulated workflows. Provider limits must be checked separately. |
| Optional health API | 2 vCPU, 2 GB RAM, 20 GB SSD | Unbenchmarked starting estimate only if the backend is demonstrated; no production capacity claim. |
| Database | None required for the stated prototype scope | No database dependency is declared in the inspected manifests. |

## Optional future production planning

These estimates apply only if the team later approves a persistent deployment. They are new recommendations, not implemented infrastructure, institutional approval, or commitments within the current prototype scope.

| Component | Initial planning estimate | Conditions |
|---|---|---|
| Application service | 2 vCPU, 4 GB RAM | Measure real workflows and concurrent sessions before choosing capacity. |
| Separate database service | 2 vCPU, 4 GB RAM, 50 GB SSD | Size storage from record counts, indexes, attachment policy, retention, and backup needs. |
| Database software | Patched PostgreSQL 17 or 18 | New recommendation; not installed or configured by this work. Both versions are listed in the [PostgreSQL support policy](https://www.postgresql.org/support/versioning/). |
| Attachments | Private storage with authorized access | Required only if attachments enter approved scope; capacity and retention remain undecided. |

Before handling institutional records, establish HTTPS, server-side authentication and authorization for every protected action, audit records, encrypted backups with tested restoration, and approved privacy and retention rules. Frontend role visibility alone does not enforce access to server data. AI data handling and provider policy also require approval before confidential content is sent to a remote service.

## Proposed acceptance checks and open decisions

Acceptance should cover keyboard use, desktop and mobile layouts, and role permissions using synthetic records. Exercise the shared modules and the ten roles listed in the inventory: Student, CCS Student Organization, Professor, CCS Staff, Program Chair, Dean, QCU Administrator / Moderator, Guidance, OSAS, and Office of the University President [S2, rows 7–64]. Include permitted actions and attempted access outside each role's authority.

Browser, device, accessibility, and performance checks remain proposed and were not run for this document. Set a concurrent-user target and response-time threshold with the team before load testing an implemented backend. Current evidence does not establish a supported number of users or production response-time guarantee.

Open sizing inputs include actual student and staff counts, peak concurrent sessions, record volumes, attachment sizes, retention periods, availability expectations, AI provider and policy, and approved role permissions. The inventory's `Open Decisions` sheet, rows 6–13, also records unresolved evaluation, escalation, restriction, threshold, and privacy rules. Resolve these before treating dependent production requirements as final.

## Sources and refresh provenance

Paths below are relative to the CAPSTONE workspace unless stated otherwise. Paragraph numbers count direct `w:p` elements under `w:body` in the DOCX XML, including empty paragraphs; sheet references use workbook row numbers.

| ID | Source | Evidence used |
|---|---|---|
| S1 | `capstone-drive/01_Documents/Final_Paper/FILE001: Draft-v1.docx` | `word/document.xml`, paragraphs 55–59: campus and program boundaries, exclusions, simulated records, frontend state; paragraphs 48–52: reporting, evaluation, offense records, AI support, and intended quality evaluation. |
| S2 | `capstone-drive/01_Documents/Final_Miscs/Module Inventory /DOC003_MODULE-INVENTORY.xlsm` | `Module Inventory`: row 3 identifies the earlier proposal source; rows 7–11 shared modules; rows 12–64 ten operational roles. `Open Decisions`: rows 6–13 unresolved policies. Read through ZIP/XML without executing macros. |
| S3 | `notion-context/rows/document-hub/final-proposal-accs.md` and `notion-context/gdocs/1xk6G0JGgVLQbiH8bFstpXCpIrmL_3HrQIwuZJpTpfaQ/SBIT4K_GROUP2_PROJECT-PROPOSAL.txt` | Refreshed proposal pointer, placeholder page sections, and linked older proposal. The page's recorded last update is 2026-10-08; refresh does not imply its content changed today. |
| S4 | `accs-system/frontend/package.json`, `accs-system/backend/package.json` | Declared stack, scripts, and Node.js engine constraint. |
| S5 | `accs-system/frontend/src/routes/index.tsx`, `accs-system/frontend/vite.config.ts` | Frontend shell, health request, and development `/api` proxy to port 3000. |
| S6 | `accs-system/backend/src/app.ts`, `accs-system/backend/src/routes/health.ts` | Express middleware and `GET /api/health` response. |

The 2026-10-18 refresh reported a successful Drive wrapper run: the paper was updated, a new `.xlsm` inventory arrived, and the old `.xlsx` inventory in that refreshed Drive location was removed. Live Notion inspection covered the Document Hub's seven entries; the linked proposal export was refreshed. This establishes the inspected sources' provenance, not freshness of the entire Notion workspace or every local export. Older cached inventory exports may remain elsewhere and were not used for this baseline.
