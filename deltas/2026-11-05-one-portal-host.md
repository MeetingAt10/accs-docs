# Delta: one portal host, no module subdomains

**Status:** implemented, 2026-11-05
**Replaces:** [`2026-11-01-subdomain-sites.md`](2026-11-01-subdomain-sites.md) for hosting. The seven module sites stay as navigation groups.

## Existing behavior touched

- [`../sitemap.md`](../sitemap.md) §4 serves one build on seven hosts, one per module site. A link to another module site loads that host as a full page.
- `architecture/README.md`, "Environments and hostnames", lists the hosts.
- The landing site has no dev host, so landing changes cannot be seen before they reach prod.

## Requested change

Team decision: the modules sit behind sign-in, so they gain nothing from their own hostnames, and moving between them should not leave the app.

| Site | Prod | Dev |
|---|---|---|
| Landing | `access-ccs.com`, `www.access-ccs.com` | `dev.access-ccs.com` |
| Portal, all modules | `portal.access-ccs.com` | `portal-dev.access-ccs.com` |
| Team | `team.access-ccs.com` | `team-dev.access-ccs.com` |
| Design system | `design.access-ccs.com` | `design-dev.access-ccs.com` |

- Modules become paths on the portal. Route names, permissions and role rules do not change.
- `forums`, `schedule`, `cases`, `evaluations`, `records` and `admin` hosts are deleted on both stacks, with no redirects. The domain was registered on 2026-11-03, so few links to them exist.
- `CORS_ORIGIN` lists only the portal.

## Success criteria

1. Moving between modules on `portal-dev` stays on one host with no full page load.
2. A deep link such as `/cases` opens on the portal, and sign-in returns to it.
3. Landing and team links lead to the portal paths for the same environment.
4. The six old hostnames no longer resolve on either stack, and the four kept sites still respond.
5. `dev.access-ccs.com` serves the landing site from the `dev` branch.
6. Frontend, landing and team builds and tests pass.

## Fold-in record

- Folded into `sitemap.md` §1 and §4, `architecture/README.md` and `frontend/README.md` on 2026-11-05.
