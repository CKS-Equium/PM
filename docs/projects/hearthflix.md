---
slug: hearthflix
repo: https://github.com/CKS-Equium/hearthflix
status: active
phase: design
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

**Gate 1 (PRD) approved 2026-06-04; entering design phase.** Repo created
([CKS-Equium/hearthflix](https://github.com/CKS-Equium/hearthflix)), PRD seeded, GitHub state set
up (labels, 6 milestones), seed issues #1 Architecture, #2 UX/UI, #3 Plan. Next: Software Architect
(transcoding pipeline, thumbnails, Docker layout) + UX/UI → gate 2. Run with `run-project hearthflix`.

## Decision log

- 2026-06-04 — Discovery decided: video-first (photos if cheap); **on-the-fly ffmpeg transcoding**
  (operator wants any format to play); **trusted-LAN, no auth**; **Docker** deploy on Arch with a
  read-only mounted media volume; YouTube-style grid + full-screen watch page.

## Gate waivers

- _none_
