# Security Engineer — notes

Playbook for the Security Engineer. This agent appends freely; entries are **not** gated.
Entry format: **situation → what happened → lesson → confidence**.
The Process Engineer periodically distills proven lessons here into `persona.md` (via reviewed PR).

---

## team-pulse-dashboard (2026-06-03)

- **Scheme-check user-derived URLs (XSS sink)** — *Situation:* the dashboard renders links built from GitHub data (issue/PR URLs). *What happened:* I BLOCKed gate 5 on a HIGH — a `javascript:` URL was rendered without scheme validation (a classic XSS sink), plus missing CSP/headers and a non-loopback bind. Engineer fixed with a `safeUrl` https-allowlist, CSP/security headers, and loopback-only binding; re-review PASS. *Lesson:* any URL/href derived from external data must pass an explicit scheme allowlist (http/https only) before render; pair it with CSP and bind local-only tools to loopback. Add these to the standard review checklist for any data-driven web UI. *(confidence: high)*
- **Gate 5 earns its keep** — *Situation:* first dogfood through the review gate. *What happened:* the gate caught 1 HIGH + several MED before release; nothing security-relevant escaped to v0.1.0. *Lesson:* the security re-review loop (BLOCK → fix → PASS) is working as designed; keep blocking on HIGH rather than deferring. *(confidence: high)*

## hearthflix (2026-06-04)

- **Same finding classes RECURRED — push them upstream of review** — *Situation:* second project, FastAPI/Docker. *What happened:* gate 5 again surfaced the predictable basics: missing security headers/CSP (FIND-01), CVE-bearing deps (FIND-02), root container (FIND-03), `0.0.0.0` bind (FIND-04). My data-driven-UI checklist (now in my contract) caught them at review — but they were still first-write defects, the same class as team-pulse. *Lesson:* my review checklist is working, but the cheaper win is a **build-time secure-by-default checklist** the Engineer runs *before* requesting review (headers/CSP, dependency audit, non-root container, loopback default). Across two projects this is proven; route it as a contract/build-DoD item for the senior engineer so I'm the safety net, not the first detector. *(confidence: high)*
- **Container & dependency hygiene belong in the standard pass** — *Situation:* root container + CVE-bearing pinned deps reached review. *What happened:* these weren't web-UI-specific (my existing checklist focus) — they're deployment/supply-chain basics. *Lesson:* extend my standard review to always include: non-root container user, a dependency CVE audit (pin to patched versions), and least-privilege bind/network — not just app-layer XSS/CSP. *(confidence: high)*
