# Delta: responsive case-queue parity

**Status:** implemented; live viewport verification pending  
**Date:** 2026-10-27

## Existing behavior touched

- [P21 case queue](../pages/07-cases.md) defines sortable desktop columns and a phone `ReportRecord` alternative.
- [Responsive application delta](2026-10-27-responsive-app.md) requires responsive views to retain record details and workflows.

## Change

Use the compact report-record layout below 1024 CSS px for the eight-column case queue, and expose field and direction sorting there. Keep the desktop table at 1024px and wider. The compact view retains the table's reference, category, severity, status, received date, last-action date, waiting days, open action, and adviser-bypass indicator. Existing route and role permissions remain unchanged.

## Success criteria

1. Below 1024px, case records show every field and routing indicator available in the table, with the same `Open` destination.
2. Below 1024px, users can select any sortable table field and ascending or descending order; the current desktop table sorting remains available at 1024px and wider.
3. No route, authorization, data, or case action behavior changes.

## Fold-in record

Folded into the P21 case-queue layout in [07-cases.md](../pages/07-cases.md). The JSX and CSS parse successfully; live viewport verification remains for the parent session.
