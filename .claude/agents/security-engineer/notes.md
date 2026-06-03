# Security Engineer — notes

Playbook for the Security Engineer. This agent appends freely; entries are **not** gated.
Entry format: **situation → what happened → lesson → confidence**.
The Process Engineer periodically distills proven lessons here into `persona.md` (via reviewed PR).

---

## team-pulse-dashboard (2026-06-03)

- **Scheme-check user-derived URLs (XSS sink)** — *Situation:* the dashboard renders links built from GitHub data (issue/PR URLs). *What happened:* I BLOCKed gate 5 on a HIGH — a `javascript:` URL was rendered without scheme validation (a classic XSS sink), plus missing CSP/headers and a non-loopback bind. Engineer fixed with a `safeUrl` https-allowlist, CSP/security headers, and loopback-only binding; re-review PASS. *Lesson:* any URL/href derived from external data must pass an explicit scheme allowlist (http/https only) before render; pair it with CSP and bind local-only tools to loopback. Add these to the standard review checklist for any data-driven web UI. *(confidence: high)*
- **Gate 5 earns its keep** — *Situation:* first dogfood through the review gate. *What happened:* the gate caught 1 HIGH + several MED before release; nothing security-relevant escaped to v0.1.0. *Lesson:* the security re-review loop (BLOCK → fix → PASS) is working as designed; keep blocking on HIGH rather than deferring. *(confidence: high)*
