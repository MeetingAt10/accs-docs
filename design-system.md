# ACCS Design System

**Status:** v0.1 · proposed foundation · **Product:** ACCS — Access for College of Computer Studies (“Access”) · **Primary surface:** responsive student and department portal · **Visual source:** [CAPSTONE (NAME): DESIGNS/BRANDING — Figma](https://www.figma.com/design/HhZBfgDKt4aMudftxOP4Jg/CAPSTONE--NAME---DESIGNS-BRANDING)

This document turns the Figma **Inspo/Concept** and **Brand Strategy and Core Identity** into a usable interface system. It is a starting specification, not a claim that every token below already exists in Figma. Palette values and component rules marked here are proposed choices to make the visual direction consistent and implementable.

## 1. Product and design intent

ACCS is a trusted campus workspace for student communication, academic support, department activity, project updates, and confidential grievance escalation. “Access” should feel welcoming for everyday participation and calm, private, and deliberate for sensitive matters.

The moodboard names the brand traits **professional, trustworthy, modern, accessible, structured**. Its concept explores deep blue, teal/mint, purple, soft gray, and near-black. The system makes deep indigo the stable identity, teal the supporting accent, and purple a limited secondary accent. This preserves the moodboard’s energy without making competing palettes look like separate brands.

### Design principles

1. **Trust before decoration.** Clear ownership, status, privacy, and next steps matter more than visual effects.
2. **Friendly in shared spaces.** Forum and student-support copy should sound human, encouraging, and plain.
3. **Restrained for sensitive work.** Grievance flows use neutral surfaces, explicit confidentiality cues, and no playful color or public-social patterns.
4. **Structure reduces effort.** Use consistent navigation, headings, labels, status patterns, and predictable page layouts.
5. **Color reinforces meaning.** Never communicate status, urgency, or privacy using color alone.

## 2. Visual identity

### Logo and naming

- Product name: **ACCS**; expanded name: **Access for College of Computer Studies**.
- Spoken name: **Access**.
- Use the approved Figma logo when available. Do not recreate, stretch, recolor, or place it over busy imagery.
- Until a final logo is approved, use a simple text wordmark, not an invented emblem.

### Color tokens

Proposed values; confirm against the final Figma palette and test rendered text/background pairs before release.

| Token | Value | Use |
|---|---|---|
| `brand-50` | `#EFF4FF` | Selected navigation and subtle brand fills |
| `brand-100` | `#DFE9FF` | Hover/selection background |
| `brand-300` | `#97B5FF` | Decorative or non-text accents |
| `brand-500` | `#426CE5` | Charts and non-critical highlights |
| `brand-600` | `#3157D5` | Primary actions, links, active navigation |
| `brand-700` | `#2546B7` | Hover/pressed primary action |
| `brand-900` | `#1C316E` | Strong brand text or dark-surface emphasis |
| `teal-50` | `#E8FAF7` | Supportive accent surface |
| `teal-700` | `#08746E` | Teal text/icons on light surfaces |
| `teal-800` | `#075D59` | Teal action hover or strong text |
| `purple-50` | `#F4EEFF` | Limited secondary accent surface |
| `purple-700` | `#7040C7` | Secondary category and data visualization |
| `ink-950` | `#172033` | Main text |
| `ink-700` | `#46516A` | Secondary text |
| `ink-500` | `#65718A` | Muted text; verify contrast per surface |
| `line-200` | `#DCE2EC` | Borders and dividers |
| `surface-50` | `#F5F7FB` | App canvas |
| `surface-0` | `#FFFFFF` | Cards, dialogs, form controls |
| `success-700` | `#166534` | Success text/icon |
| `warning-800` | `#854D0E` | Warning text/icon |
| `danger-700` | `#B42318` | Error/destructive text/icon |
| `info-700` | `#3157D5` | Informational text/icon |

#### Semantic color rules

- Primary action: `brand-600` fill with white label; hover `brand-700`; disabled uses a neutral fill and muted label.
- Links: `brand-700`, underlined in body text, with a visible focus style.
- Teal means supportive or secondary emphasis; never use it to imply success by itself.
- Purple is an accent for secondary categories or charts. Keep it out of primary action, error, and privacy semantics.
- Success, warning, danger, and info always include a text label or icon with accessible name.
- Use red only for errors, destructive actions, or genuine risk; do not use red as general brand decoration.
- Avoid gradients in core task UI. Reserve any approved gradient for small brand moments, not text-bearing controls or status.

#### Dark theme

Light theme is the launch default. If dark theme is implemented, map semantic roles to a separately tested palette: canvas `#111827`, raised surface `#1B2434`, elevated surface `#242F42`, main text `#F3F6FC`, secondary text `#C1CAD9`, border `#354258`, brand action `#8EACFF` with dark text. Recheck every text, control, chart, focus, and status pair; do not invert light values mechanically.

### Typography

- Display/headings: **Plus Jakarta Sans**, weight 600–700; geometric and confident, matching the Figma direction.
- Body and controls: **Inter**, weight 400–600; neutral, highly legible at compact UI sizes.
- Data/code identifiers: system monospace stack. Use only where fixed-width characters aid scanning.
- Load locally or from an approved font service; provide system sans-serif fallbacks.
- Use sentence case. Reserve all caps for short metadata labels, never paragraphs or primary navigation.

| Style | Size / line height | Weight | Use |
|---|---:|---:|---|
| Display | 36 / 44 px | 700 | Rare page-level welcome or campaign title |
| H1 | 30 / 38 px | 700 | Page title |
| H2 | 24 / 32 px | 650 | Major page section |
| H3 | 20 / 28 px | 650 | Card or form section |
| H4 | 16 / 24 px | 600 | Dense section heading |
| Body large | 16 / 26 px | 400 | Introductions and guidance |
| Body | 14 / 22 px | 400 | Default interface text |
| Body small | 13 / 20 px | 400 | Supporting detail; avoid for essential instructions |
| Label | 14 / 20 px | 600 | Input and control labels |
| Caption | 12 / 16 px | 500 | Timestamps and supplementary metadata |

Use 14–16 px body text by default. Do not use muted color and small type together for important information. Allow user zoom and text resizing without clipped controls or content.

## 3. Layout and shape

### Spacing scale

Use a 4 px base scale: `1=4`, `2=8`, `3=12`, `4=16`, `5=20`, `6=24`, `8=32`, `10=40`, `12=48`, `16=64` px. Prefer 8 px rhythm for page and component layout; use 4 px for fine alignment.

### Responsive layout

- Desktop app shell: persistent left navigation, top utility bar, scrollable content region.
- Navigation width: 248 px expanded; compact icon mode only when every item retains an accessible name and tooltip.
- Content: fluid width, maximum reading width 800 px; data-heavy pages may use full available width.
- Breakpoints: 640 px small/mobile, 768 px tablet, 1024 px desktop, 1280 px wide layout. Treat these as layout thresholds, not device categories.
- Mobile: replace the persistent sidebar with a labeled menu/drawer; keep the current page title and primary action visible; use stacked cards instead of horizontally overflowing tables. Calendar month and week views use date-grouped event lists on phones, with a native date input for date selection; retain the month grid and week timetable at tablet and desktop widths.
- Avoid fixed-height panels for content and forms. Use sticky headers only when they preserve usable viewport space.

### Surfaces, borders, and elevation

- Cards and inputs: white surface, 1 px `line-200` border, 8 px radius.
- Buttons and badges: 6 px radius. Dialogs and menus: 12 px radius.
- Use shadows sparingly for overlays only. Establish hierarchy first with spacing, headings, and surface contrast.
- Default card padding: 20–24 px; compact data row: 12–16 px vertical padding.
- Icon style: one consistent 1.75–2 px stroke family. Pair icons with labels where meaning is not universal.

## 4. Component standards

### Buttons

- **Primary:** one per task region; verb-first label such as “Submit request” or “Create update”.
- **Secondary:** outlined or neutral filled; complementary actions.
- **Tertiary:** text button for low-emphasis actions.
- **Destructive:** danger treatment plus specific label; require confirmation when action cannot be undone.
- Heights: 40 px default, 48 px touch-friendly. Provide 36 px compact only in dense desktop tables.
- Include default, hover, pressed, focus-visible, disabled, and loading states. Preserve button width while loading when practical.
- Avoid ambiguous labels such as “OK”, “Submit” without context, or icon-only actions.

### Forms

- Put a persistent visible label above each field. Placeholder text is an example, never the only label.
- Mark required fields in text and explain optional fields where needed.
- Put help text and validation next to the field; connect programmatically to its control.
- Validate on submit and after correction; preserve entered data after errors.
- Use native input semantics where possible. Group related controls with a legend and logical keyboard order.
- For sensitive submissions, explain who can read the information and what happens after submission before the final action.

### Navigation and wayfinding

- ACCS is seven module sites with a stable top-level order: Home, Forums, Schedule, Cases, Evaluations, Records, Admin. Show a site only when the current session has an allowed destination in it. Each site has its own host, landing page, hero, and lane color ([sitemap §4](sitemap.md#4-primary-navigation-and-contextual-sidebar)).
- Use the 72 px site rail (one icon per site the role may open) for switching sites and the contextual sidebar for destinations within the current site. On mobile, put a **Sites** section first in the navigation drawer, followed by the current site's destinations.
- Filter contextual destinations by the signed-in person's allowed pages and role-specific labels. Navigation helps people find pages; it does not replace route or API authorization.
- Show the current site and page with text plus a non-color indicator: the switcher marks the current site with `aria-current` and a sliding fill, and the sidebar marks the current page.
- Signed-in pages use no illustration art: no heroes, floating art, or art tiles. The crest on the sign-in and registration brand panel is the only brand graphic. App foundation tokens (`--accs-*`, per-site `--site-accent`) and primitives live in `frontend/src/styles.css`, `frontend/src/components/app/page-shell.tsx`, and the motion layer in `frontend/src/motion.css` (see [the 2026-11-01 delta](deltas/2026-11-01-real-auth-and-visual-refactor.md)).
- Show the site in the document title as `<Page> · ACCS <Site>`, with generic page names only.
- Use breadcrumbs for deep document/department paths. Do not use breadcrumbs as the only way back.
- Keep global actions separate from navigation; do not hide essential items in an unlabeled icon rail.

#### Site lanes

Each site carries one lane tint, set as `data-lane` on the shell and used for the switcher indicator, the hero wash, and the Home directory tile hover. A lane reinforces where the person is; it never carries meaning alone.

| Site | Lane | Tint |
|---|---|---|
| Home, Evaluations, Admin | navy (default) | `--ds-action-default`, wash mixed from it at 9% |
| Forums | community, violet | `--ds-accent-default`, wash `--ds-accent-bg` |
| Schedule | consult, teal | `--ds-secondary-700`, wash `--ds-secondary-bg` |
| Cases, Records | neutral | `--ds-text-secondary`, wash mixed from it at 7% |

- Violet and teal keep their existing roles: community and secondary emphasis. Do not use them for status.
- Schedule uses `secondary-700`, not a lighter teal, for the solid switcher indicator so the white label on the current item stays readable.
- Cases and Records stay neutral to follow the grievance mode above: no playful or brand color on confidential work.

#### Site art

Site art is decorative: `alt=""`, never the only way to identify a site. The files are transparent WebP converted from the `meeting-at-ten-assets` set and live in `frontend/public/brand/`. Each site shows its art on the hero (landing routes only) and on its Home directory tile.

| Public file | Used for | Source in `meeting-at-ten-assets` |
|---|---|---|
| `brand/sites/portal.webp` | Home | `brand-symbols/06-orbital-ring-crest` |
| `brand/sites/forums.webp` | Forums | `graphic-primitives-set8/06-nodal-constellation` |
| `brand/sites/schedule.webp` | Schedule | `floating-forms-set15/03-twisting-ribbon-torus` |
| `brand/sites/cases.webp` | Cases | `institutional-badges/05-faceted-c-shield` |
| `brand/sites/evaluations.webp` | Evaluations | `graphic-primitives-set8/04-frosted-faceted-prism` |
| `brand/sites/records.webp` | Records | `sculptural-elements/06-structural-grid-cube` |
| `brand/sites/admin.webp` | Admin | `functional-assets/05-cloud-server` |
| `brand/sites/crest.webp` | Sign-in brand panel | `brand-symbols/01-sapphire-crest` |
| `brand/topographic.webp` | Sign-in brand panel background | `accs-backgrounds-12/06-topographic` |

### Cards, lists, and tables

- Cards: clear title, concise context, status when applicable, one obvious click target/action.
- Announcements: category, author/owner, published date, priority where needed, and a short summary. Do not encode priority with color alone.
- Tasks: task name, owner, due date, status, and a clear open action; allow sorting/filtering when data volume grows.
- Documents: descriptive title, type/category, owner, last-updated date, and permission cue where restricted.
- Tables: semantic headers, row actions with labels, keyboard support, sorting state, and mobile alternative. Do not make an entire row behave as multiple conflicting links.
- Empty states: say what is empty, why it matters, and the next useful action. Distinguish empty, loading, failed, and filtered-to-zero states.

### Dialogs, alerts, and notifications

- Dialogs only for focused decisions or short forms. Title the dialog, provide explicit cancel/close, manage focus, and restore focus when closed.
- Use inline alerts for durable or task-critical information. Toasts are for brief confirmations, never the sole error message or only record of a sensitive submission.
- Confirm irreversible deletion or publication. Do not add confirmation steps to routine reversible actions.
- Loading states should explain the task when delay is noticeable. Keep layout stable with skeletons or progress indicators.

## 5. Product-specific patterns

### Student forum

Use warm, community-led language. Make author, date, topic, and moderation state easy to scan. Provide clear report/mute controls. Keep reactions secondary to readable content. Avoid public engagement mechanics on confidential or support records.

### Student support

Use empathetic, non-judgmental language; explain expected response time only when confirmed. Offer a clear request summary, contact preference if supported, and visible follow-up status. Provide a safe exit and preserve drafts where possible.

### Grievance escalation

Use a quieter visual mode: neutral background, limited brand color, clear confidentiality scope, structured steps, and calm confirmation. State visibility and recipient before submission; avoid promising anonymity or confidentiality beyond actual system policy. Do not reveal case title, status, or details in public feeds, notifications, or browser page titles. Record actionable errors without exposing sensitive details. Provide a receipt/reference and next step after successful submission.

### Department and project workspace

Use consistent department headers, owners, update dates, task states, and document categories. Keep policy announcements distinct from casual posts. Show deadlines and blockers in text; use status color as reinforcement only.

## 6. Content voice

- Forum: casual, community-driven, encouraging.
- Student support: empathetic, welcoming, clear.
- Grievance escalation: serious, secure, confidential, highly structured.
- Use plain language, short sentences, direct actions, and specific error recovery.
- Avoid blame, unexplained acronyms, punitive copy, fabricated response-time promises, and legal/security claims not approved by the project.

## 7. Accessibility and interaction

Target **WCAG 2.2 AA**. Text contrast: at least 4.5:1 for normal text and 3:1 for large text. Recheck against actual rendered colors, including hover, disabled, selected, error, and dark-theme states. See the [W3C contrast guidance](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html).

- All functionality works by keyboard; focus order follows reading order; focus indicator remains visible and unobscured.
- Use semantic HTML, accessible names, correct headings, and announced status changes. Color never carries meaning alone.
- Aim for 44 × 44 CSS px touch targets on common mobile actions; separate adjacent targets.
- Support 200% zoom and narrow reflow without loss of content or function.
- Respect `prefers-reduced-motion`; use transitions only to clarify state or spatial change, normally 120–200 ms. Page-entry reveals may take up to 320 ms (see Motion below).
- Provide text alternatives for meaningful images and accessible names for icon buttons.
- Do not rely on tooltips as the only source of essential instructions.

### Motion

Motion in the signed-in shell uses the design system tokens (`--ds-motion-duration-*`, `--ds-motion-ease-*`, `--ds-motion-distance-rise`).

- **Route changes:** View Transitions where the browser supports them. The old page fades out in 120 ms and the new page fades in over 200 ms.
- **Reveals:** the sections of a new page, the module chips, and the Home directory tiles rise 8 px over 320 ms (chips 480 ms), staggered by short delays of up to 160 ms (250 ms for directory tiles).
- **Switcher:** the current-site indicator slides between items.
- **Hero art:** enters once (640 ms), then floats 8 px with a slight tilt on a 7 s loop. It is the only looping animation and is decorative.
- **Tiles:** lift 2 px on hover; the tile art tilts and scales slightly.
- **Reduced motion:** with `prefers-reduced-motion: reduce`, the reveals, float, view transitions, and hover movement are removed, and the switcher indicator follows the provider's Motion setting. Content and the current-site marker stay visible without motion.

## 8. Implementation contract

Current frontend: React 19, TypeScript, Vite, TanStack Router, TanStack Query. Recommended UI foundation: **shadcn/ui components built on Radix primitives, styled with Tailwind CSS**. Treat shadcn/ui as editable project-owned component code, not a remotely themed black box. Radix supplies accessible interaction behavior for supported primitives; the application still owns labels, content, color contrast, and end-to-end accessibility checks. Official references: [shadcn/ui](https://ui.shadcn.com/docs), [Radix accessibility](https://www.radix-ui.com/primitives/docs/overview/accessibility), [Tailwind theme variables](https://tailwindcss.com/docs/theme).

Implementation rules:

1. Define semantic CSS variables first; components consume semantic roles, not palette literals.
2. Keep brand ramps separate from semantic aliases such as `--primary`, `--muted`, `--destructive`, and `--ring`.
3. Use one shared component per interaction pattern. Extend variants deliberately; avoid page-specific copies.
4. Keep business rules and privacy decisions outside presentational components.
5. Use TanStack Router for navigation and TanStack Query for server state; design loading, empty, error, and success states in every data-backed view.
6. Add components only as product screens need them; do not install a second component library alongside the chosen foundation.

Suggested token mapping:

| Semantic token | Light value |
|---|---|
| `--background` | `#F5F7FB` |
| `--foreground` | `#172033` |
| `--card` | `#FFFFFF` |
| `--card-foreground` | `#172033` |
| `--primary` | `#3157D5` |
| `--primary-foreground` | `#FFFFFF` |
| `--secondary` | `#E8FAF7` |
| `--secondary-foreground` | `#075D59` |
| `--muted` | `#EDF1F7` |
| `--muted-foreground` | `#46516A` |
| `--accent` | `#F4EEFF` |
| `--accent-foreground` | `#55309B` |
| `--destructive` | `#B42318` |
| `--border` | `#DCE2EC` |
| `--input` | `#DCE2EC` |
| `--ring` | `#3157D5` |
| `--radius` | `0.5rem` |

## 9. Governance and open decisions

- This file is the frontend design reference; update it when an approved brand or interaction decision changes.
- Treat token values as **proposed** until checked against source Figma swatches and contrast tests.
- Keep changes traceable: record substantial decisions with rationale and date in this section or project decision log.
- Before production, confirm the approved logo, final palette, typography availability, exact privacy policy, roles/permissions, and notification behavior with the project team.
- Review new components for keyboard behavior, contrast, narrow screens, long text, empty/loading/error states, and sensitive-data exposure.
