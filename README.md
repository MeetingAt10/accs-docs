# ACCS docs

Specifications for ACCS, the web portal for the College of Computer Studies. The repo holds the sitemap, the page specs, the change records (deltas) and the design and requirements documents. It has no code.

Start at [`sitemap.md`](sitemap.md). It is the index: sources, the eleven roles and access rules, the route list, navigation, case stages, module coverage, components, build order and open decisions. Page-level detail is in `pages/`.

## Contents

| Path | What it holds |
|---|---|
| [`sitemap.md`](sitemap.md) | The index and master spec for the whole app |
| [`pages/`](pages/README.md) | One spec per area (shell and auth, home, community, messages, schedule, reports, cases, evaluations, records, people and account, admin). `pages/README.md` lists the pages in each file and the conventions they share |
| [`deltas/`](deltas/) | One file per change, named `YYYY-MM-DD-topic.md`. Each states the status, the existing behavior it touches, the requested change and the success criteria |
| [`design-system.md`](design-system.md) | The ACCS design system foundation (v0.1, proposed) |
| [`hardware-software-requirements.md`](hardware-software-requirements.md) | Proposed hardware and software baseline for team review |
| [`hardware-software-requirements-final.md`](hardware-software-requirements-final.md) | The same requirements as short paper-ready text |

The infrastructure is documented in the `accs-architecture` repo. Its Mermaid sitemap is `diagrams/sitemap.md` there. It shows the pages by area; routes and access rules stay in `sitemap.md` here.

## How deltas fold back into the specs

`sitemap.md` and `pages/` describe the system as it is meant to work. A change does not edit them first.

1. Write a delta in `deltas/` that covers only the change.
2. Build against the delta and verify it against its success criteria.
3. Fold the result into `sitemap.md`, `pages/` and any other affected document, then add a "Fold-in record" to the delta saying what was folded and when.
4. When a later delta replaces an earlier one, mark the earlier file as superseded and link the new one. The old file stays as history.

A delta's text can be out of date once it is folded in or superseded. Read the specs for current behavior and the deltas for why it changed.
