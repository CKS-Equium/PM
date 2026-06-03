# Senior Software Engineer — notes

Playbook for the Senior Software Engineer. This agent appends freely; entries are **not** gated.
Entry format: **situation → what happened → lesson → confidence**.
The Process Engineer periodically distills proven lessons here into `persona.md` (via reviewed PR).

---

## team-pulse-dashboard (2026-06-03)

- **Sanitize external-data hrefs at render** — *Situation:* I built the dashboard rendering links from GitHub data; junior was not spawned, I built directly. *What happened:* gate 5 found a HIGH `javascript:` XSS sink (no scheme check), missing CSP/headers, and a non-loopback bind. I fixed with a `safeUrl` https-allowlist, CSP/security headers, and loopback binding. *Lesson:* default to a scheme allowlist for any href/URL built from external data, add CSP, and bind local tools to loopback — do this on first write, not after review. *(confidence: high)*
- **Resolve derived state from the right subset** — *Situation:* milestone/phase was derived from issues. *What happened:* I derived it from an arbitrary/closed issue, producing a false phase-conflict badge; also active-role was hardcoded to "orchestrator" and an in-progress event class was silently empty. *Lesson:* when deriving state from a collection, filter to the meaningful subset (open issues only), drive role from the role label, and never let an event class render silently empty — assert it has content or surface "none". *(confidence: high)*
- **Normalize shared identifiers at the boundary** — *Situation:* registry gave `repo` as a URL; architecture examples used `owner/name`. *What happened:* I normalized at the ingestion boundary, which worked. *Lesson:* when contract and source artifact disagree on identifier shape, normalize once at the boundary and keep one canonical form internally. *(confidence: med)*
