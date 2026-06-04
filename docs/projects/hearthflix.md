---
slug: hearthflix
repo: https://github.com/CKS-Equium/hearthflix
status: active
phase: build
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

**Gate 2 (architecture & design) approved 2026-06-04; entering build phase.** Design merged to
`main` (#1, #2 closed): FastAPI/Python + SQLite, HLS+hls.js transcoding, ffprobe pass-through,
eager thumbnails, Docker (RO `/media`, RW `/data`), SW-transcode + VAAPI opt-in, photos IN, LAN
no-auth (operator-accepted), cinema-dark UI. Next: Project Manager breaks into build tickets →
Senior Engineer builds → gate 3.

## Decision log

- 2026-06-04 — Discovery decided: video-first (photos if cheap); **on-the-fly ffmpeg transcoding**
  (operator wants any format to play); **trusted-LAN, no auth**; **Docker** deploy on Arch with a
  read-only mounted media volume; YouTube-style grid + full-screen watch page.
- 2026-06-04 — **Gate 2 approved.** Researcher spike → Architect chose **HLS+hls.js** (not
  progressive), FastAPI/Python/SQLite, ffprobe pass-through (+remux tier), eager 10%-frame
  thumbnails, Docker (RO `/media`, RW `/data`), **SW transcode for v1 + VAAPI opt-in seam**,
  per-session ffmpeg w/ 90s watchdog + cap 2. **Photos IN for v1** (cheap). **LAN/no-auth
  explicitly accepted by the operator** (README to document risk).

## Gate waivers

- _none_
