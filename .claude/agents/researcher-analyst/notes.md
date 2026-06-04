# Researcher / Analyst — notes

Playbook for the Researcher / Analyst. This agent appends freely; entries are **not** gated.
Entry format: **situation → what happened → lesson → confidence**.
The Process Engineer periodically distills proven lessons here into `persona.md` (via reviewed PR).

---

## hearthflix (2026-06-04) — first outing

- **Spikes must VALIDATE, not ASSERT** — *Situation:* my first spike (HLS streaming for self-hosted video). I researched ffmpeg HLS and reported that `delete_segments` would bound on-disk segment growth per session. *What happened:* that claim was **never empirically tested** — and under `-hls_playlist_type event` (the mode the design used) `delete_segments` is a **no-op**. The Architect set a "disk is bounded" NFR on the strength of my report; the Engineer built to it; the Reviewer caught the resulting unbounded-`/data` bug (H1) at gate 5. My unvalidated assertion became a false NFR and a release-blocking defect. *Lesson:* a spike's job is to **reduce uncertainty with evidence**. Any claim that will become an NFR or a load-bearing design assumption must be demonstrated (a tiny repro / measured run), explicitly labelled **validated vs assumed**, and the validation conditions stated (here: which `hls_playlist_type`). Never hand an unqualified assertion to the Architect. *(confidence: high)*
- **State the conditions a finding depends on** — *Situation:* the segment-cleanup behavior was correct in some ffmpeg modes and not others. *What happened:* I reported the behavior without naming the mode it required. *Lesson:* findings about tool behavior are conditional — always pin the flags/version/config under which they hold, so downstream roles don't generalize them to a context where they're false. *(confidence: high)*
