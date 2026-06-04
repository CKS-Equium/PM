---
slug: hearthflix
repo: https://github.com/CKS-Equium/hearthflix
status: active
phase: review
created: 2026-06-04
team: [orchestrator, product-manager, software-architect, ux-designer, ui-designer]
board: n/a
---

## Summary

Hearthflix — a self-hosted, YouTube-style web app for an Arch Linux server that browses an on-disk
media library as a thumbnail grid and plays any video full-screen in the browser via pass-through
or on-demand ffmpeg transcoding, on a trusted LAN with no auth, deployed via Docker with a
read-only mounted media volume. Photos are a conditional v1 extra (only if cheap).

## Current state

**Gates 1–5 passed; PAUSED before release (gate 6) per operator.** Built (PR #17), tested (59/59),
and reviewed — gate 5 cleared after a fix loop (security/correctness blockers all resolved). Code
on branch `feat/build-v1`. **To release:** the operator builds/runs the Docker image on the Arch
host (`docker compose up`, AC11) and confirms the in-browser ACs (full-screen, lightbox), then
gate 6 sign-off → merge `feat/build-v1` → tag → post-mortem (gate 7). Resume with
`run-project hearthflix`.

## Decision log

- 2026-06-04 — Discovery decided: video-first (photos if cheap); **on-the-fly ffmpeg transcoding**
  (operator wants any format to play); **trusted-LAN, no auth**; **Docker** deploy on Arch with a
  read-only mounted media volume; YouTube-style grid + full-screen watch page.
- 2026-06-04 — **Gate 2 approved.** Researcher spike → Architect chose **HLS+hls.js** (not
  progressive), FastAPI/Python/SQLite, ffprobe pass-through (+remux tier), eager 10%-frame
  thumbnails, Docker (RO `/media`, RW `/data`), **SW transcode for v1 + VAAPI opt-in seam**,
  per-session ffmpeg w/ 90s watchdog + cap 2. **Photos IN for v1** (cheap). **LAN/no-auth
  explicitly accepted by the operator** (README to document risk).
- 2026-06-04 — **Gate 3 (build) passed.** Tickets #4–#16 delivered as PR #17. 31 tests pass; live
  smoke test verified scan/thumbnail/pass-through(no-ffmpeg)/HLS-transcode-lifecycle/cap-503/clean
  shutdown. VAAPI = documented v1.1 seam. Docker-runtime + in-browser ACs flagged for QA (need a
  Docker host / real browser).
- 2026-06-04 — **Gate 4 (test) passed.** 49/49 tests (QA added 18: transcode ready-signal both
  branches, capacity reaping, watchdog, path-traversal rejection). AC6/AC11 documented manual.
- 2026-06-04 — **Gate 5 (review) BLOCKED — PAUSED per operator (pre-release).** Reviewer: **H1**
  `delete_segments` is a no-op under `-hls_playlist_type event` → unbounded `/data` per session
  (NFR false); **M1** "Preparing forever" when ffmpeg dies immediately (`ready()` ignores
  `process.poll()`); M2 sparse-keyframe remux stall; lows. Security: **FIND-01** no security
  headers (CSP/etc.); **FIND-02** CVE-bearing deps (jinja2 3.1.5→3.1.6, python-multipart bump);
  FIND-03 Docker runs as root; FIND-04 binds 0.0.0.0. Path-traversal/subprocess/output-encoding/
  read-only all PASSED. Fix loop + re-review needed before gate 5 clears.
- 2026-06-04 — **Gate 5 (review) PASSED after fix loop.** Engineer fixed all blockers (HLS
  session-dir teardown proven; transcode error-state vs "preparing forever"; security headers +
  strict CSP via data-attributes; jinja2 3.1.6 + python-multipart removed; non-root Docker; bind
  configurable + rescan cooldown). 59/59 tests; both reviewers re-verified PASS. **PAUSED before
  release per operator** — gate 6 needs the operator to build/run the Docker image on the Arch host
  (AC11) and confirm in-browser ACs (AC6 full-screen, lightbox).

## Gate waivers

- _none_
