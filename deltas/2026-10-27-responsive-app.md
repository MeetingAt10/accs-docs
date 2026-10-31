# Delta: responsive ACCS application

**Status:** merged 2026-10-27; frontend typecheck passed for the previously folded responsive work, and local browser verification covered the additional mobile layouts recorded below.  
**Date:** 2026-10-27

## Existing behavior touched

- [Design system §3](../design-system.md#3-layout-and-shape) defines responsive breakpoints and general mobile layout guidance; §4 and §7 cover navigation, data views, and accessibility.
- [App shell spec](../pages/01-shell-and-auth.md#app-shell-srcroutes_app_tsx) defines the desktop shell and the mobile navigation drawer.
- [Page conventions](../pages/README.md#shared-conventions) require labeled alternatives for tables on phones.

## Requested change

Make the full ACCS application usable across mobile, tablet, and desktop widths. Apply the existing responsive design-system guidance consistently to the shell and page workflows. Keep current routes, role-based access, page actions, validation, data, and privacy behavior intact; responsive presentation must not remove or change a supported workflow.

Use the existing 640, 768, 1024, and 1280 px layout thresholds as appropriate. Mobile layouts should prioritize readable single-column content and reachable navigation/actions. Tablet layouts should reflow multi-column regions and dense controls to fit the available width. Desktop layouts should retain the contextual navigation and use available space without making reading content uncomfortably wide. Data-heavy regions may scroll locally only when their labels and essential actions remain available.

## Success criteria

1. At representative mobile, tablet, desktop, and wide-desktop widths—and at a 320 CSS px viewport—content and controls remain visible and usable without page-level horizontal scrolling, overlap, clipping, or loss of information. Any necessary data-region scrolling is contained and has a usable narrow-screen presentation.
2. All currently supported role-allowed routes and workflows remain available at every viewport size, including navigation, search/filtering, opening records, and existing submit, review, and scheduling actions. Responsive navigation preserves current route and selection state.
3. Forms, dialogs, tables, calendars, charts, and message views reflow or provide an accessible narrow-screen presentation; primary actions and privacy/context cues remain perceivable. No interaction, field, status, or record detail is removed solely because of viewport width.
4. The interface supports keyboard operation with visible focus, accessible names and labels, logical reading/focus order, and clear state announcements. Navigation drawers and dialogs support expected focus entry, dismissal, and return behavior.
5. Common touch actions provide targets of at least 44 × 44 CSS px where practical. At 200% zoom and narrow reflow, essential content and functionality remain available; status and selection are not conveyed by color alone, and reduced-motion preferences are respected.
6. Responsive presentation leaves authorization, route URLs, data handling, form validation, and sensitive-record visibility unchanged.

## Fold-in record

Verified behavior is folded into the [design system](../design-system.md), [app-shell spec](../pages/01-shell-and-auth.md), [schedule spec](../pages/05-schedule.md), [evaluation spec](../pages/08-evaluations.md), [message spec](../pages/04-messages.md), [community spec](../pages/03-community.md), and [shared page conventions](../pages/README.md). The calendar keeps a month grid and week timetable at tablet and desktop widths; phones receive month and week event lists and native date selection. The phone header exposes the permission-filtered account menu. Evaluation cycles, question editing, and completion summaries use cards on phones. The P11 inbox and P12 thread are separate phone views, with a "Back to conversations" link in the thread. Chair and Dean section oversight and faculty consultation requests also use labeled phone cards while preserving their data and destinations. The previously folded responsive work passed frontend typecheck; local browser verification covered these additional mobile layouts.
