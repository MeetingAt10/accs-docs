# Delta: local demo-role launcher

Status: merged 2026-10-27; launcher API and all 16 account tabs verified in the local Playwright browser. Existing shell and sign-in behavior: [shell and auth](../pages/01-shell-and-auth.md).

Developers need to compare demo users concurrently. Browser tabs on the hosted app share its session cookie. A local-only Vite launcher will open a tab per chosen demo account on an isolated `*.localhost` host, sign in through the existing demo endpoint, and proxy API requests to the selected development API. Existing server-side permissions and signed cookies continue to apply.

Acceptance: launcher is available only when explicitly enabled on a loopback-bound development server; account choices come from the demo API; each tab can sign in independently and remain independent after reload; normal production builds and the default development proxy do not enable the launcher. Document startup and the shared API data implication.
