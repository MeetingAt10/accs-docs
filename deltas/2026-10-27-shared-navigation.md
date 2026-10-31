# Delta: shared top-level navigation

**Status:** merged 2026-10-27; frontend build passed and browser validation covered student desktop Community navigation and mobile navigation.  
Superseded by [2026-11-01-subdomain-sites.md](2026-11-01-subdomain-sites.md).  
**Date:** 2026-10-27

## Existing behavior touched

- Before this delta, [`../sitemap.md` §4](../sitemap.md#4-primary-navigation-and-contextual-sidebar) described a role-specific sidebar as the primary navigation. Home, feed, schedule, and each role's operational pages were mixed into role-defined sections.
- Before this delta, [`../pages/01-shell-and-auth.md`](../pages/01-shell-and-auth.md) described that sidebar as the shell's main navigation and moved it into a drawer on phones.
- Before this delta, [`../design-system.md` §4, Navigation and wayfinding](../design-system.md#navigation-and-wayfinding) listed a generic primary navigation that did not match ACCS's current routes.

## Requested change

Use four shared top-level areas in the signed-in application shell: **Home**, **Community**, **Schedule**, and **Workspace**. Selecting an area reveals a contextual sidebar containing the destinations relevant to that area and the signed-in person's roles. Preserve the existing route permissions and record-level relationship checks; navigation visibility is not authorization. Keep the current page location identifiable, retain the existing global utility actions, and provide an equivalent labeled navigation path on mobile.

The implemented route grouping and fixed area order are:

| Top-level area | Destinations |
|---|---|
| Home | `/home`; `/admin` is the administrator's Home landing |
| Community | Feed, spaces, organizations, messages, directory |
| Schedule | Calendar, consultations |
| Workspace | Reports, cases, evaluations, records, nested administration routes, Help, Settings |

Areas appear in the fixed order **Home**, **Community**, **Schedule**, **Workspace**, and only when the session has an allowed destination in that area. The contextual sidebar contains only the selected area's allowed role entries. On mobile, the drawer presents the same area selector first, followed by links for the current area. Routes remain unchanged; `/admin` is Home while nested `/admin/*` routes are in Workspace. This change does not change page permissions or role capabilities.

## Domain note

`access-ccs.com` is a proposed domain name only. Treat it as an unconfigured product proposal; do not describe DNS, hosting, HTTPS, or deployment as configured on its basis.

## Success criteria

1. Top-level areas use the shared labels and fixed order Home, Community, Schedule, Workspace; each appears only when the session has an allowed destination in it.
2. The secondary navigation reflects the selected area and includes only destinations available to the current role assignments; direct route and API access remain independently authorized.
3. The current destination remains perceivable without color alone, and mobile users can reach the same permitted destinations through the area selector and labeled contextual links.
4. The verified sitemap and shell spec document the actual area-to-route mapping and contextual navigation behavior; route URLs remain unchanged, with `/admin` as the admin Home landing and nested admin routes in Workspace.
5. Documentation continues to identify `access-ccs.com` as proposed unless configuration is confirmed separately.

## Fold-in record

The verified behavior is folded into [sitemap §4](../sitemap.md#4-primary-navigation-and-contextual-sidebar), the [app-shell spec](../pages/01-shell-and-auth.md), and [design-system navigation guidance](../design-system.md#navigation-and-wayfinding). The frontend build passed; browser validation covered student desktop Community navigation and the mobile navigation drawer.
