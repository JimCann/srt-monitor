# CannDo Transcription Tools — Product Spec
_Created: May 20, 2026_
_Owner: Jim Cannella, CannDo Productions_

---

## Vision

A professional caption workflow for the Inside Blackbird (IBB) series and similar music production interview content. Two complementary tools handle the full arc from raw audio/video to a final, human-verified SRT file ready for CMS ingest or direct video player use.

The goal is not perfect automation. The goal is getting automated transcripts to 95%+ accuracy, then giving editors a fast, low-friction path to the remaining 5%. Reducing the total time a video editor spends on captions — not eliminating human judgment.

---

## The Two Tools

### CannDo Transcribe
**URL:** https://charming-dodol-b0a1dc.netlify.app
**Repo:** JimCann/claude-projects (cando-transcription/)
**Role:** Submission, generation, macro editing, readable transcript review, initial SRT export.

### CannDo SRT Monitor
**URL:** https://clever-moxie-45f90b.netlify.app
**Repo:** JimCann/srt-monitor
**Role:** Frame-accurate caption QA, fine-grained per-caption editing synced to video playback, final SRT export.

---

## Intended Editor Workflow

```
1. Paste Dropbox URL into Transcribe
       ↓
2. Receive auto-generated transcript + SRT
       ↓
3. Quick scan in Readable view
   - Find/replace known difficult words, proper names, gear models
   - Spot-correct repeated misrecognitions
       ↓
4. Download SRT + readable transcript
       ↓
5. Load SRT + local video file into SRT Monitor
       ↓
6. Frame-accurate review with live caption overlay
   - Edit individual captions in sync with playback
   - Flag structural issues for future parseWords() improvement
       ↓
7. Export final corrected SRT (timestamped filename)
       ↓
8. Ingest into CMS or video player
```

**Current handoff (manual):** Editor downloads SRT from Transcribe, loads it manually into Monitor along with local video file.

**Near-future handoff:** "Open in Monitor" button in Transcribe launches Monitor in a new tab with SRT pre-loaded. Video file still loaded manually (constraint: video proxy not yet built).

**Long-term handoff:** Same-tab state change. Feasibility depends on video proxy landing. SRT/transcript handoff is fully feasible today — these are text files. Video file handoff requires the Netlify video proxy function.

---

## Constraints

- **Video proxy not built.** Monitor can only load local video files. Dropbox CDN serves video as `application/json` on the `/inline/` path — browsers reject it for `<video>` elements. Fix requires `netlify/functions/video-proxy.js`. This is the only blocker to a fully seamless in-browser workflow.
- **Both tools must be designed with eventual merge in mind**, but remain separate repos and separate deploys until the proxy lands and the editor is stable.
- **Glossary is multi-client in the future.** Each client context will need its own glossary instance. Authentication wall required before multi-client deployment.

---

## Shared Brand Tokens

| Token | Value |
|-------|-------|
| Font (mono) | DM Mono |
| Font (sans) | DM Sans |
| Background | `#0a0a0a` |
| Stripe | green `#27ae60` / yellow `#f1c40f` / blue `#2980b9` / red `#c0392b` / grey `#888888` |
| Active/input call-to-action | pulsing green border (shared animation pattern across both tools) |
| Unsaved changes indicator | red download/save button |
| Saved/locked state | solid green (no pulse) |

---

## Phase 1 — Cosmetic and Additive Pass (both tools, one build session)

### CannDo Transcribe

| Item | Description |
|------|-------------|
| Pulsing URL input | URL input box gets green pulsing border on page load. Pulse stops and border goes solid green when a valid URL is entered. Same animation pattern as Monitor file inputs. |
| Hide help card after delivery | "How to get your cloud link" card is hidden once transcription is delivered and editing state begins. It is only relevant on fresh page load. |
| Red download button on edit | Download button changes to red when any edit has been made to the transcript or SRT since last download. Returns to default state after download. |
| Edited filename | Initial download uses current auto-generated filename. Any subsequent download after an edit appends `EDIT_` + current date + time to the filename. Each new download gets a fresh timestamp. |

### CannDo SRT Monitor

| Item | Description |
|------|-------------|
| Font size | All fonts increased by 4pt across the tool. |
| Scrubber position | Scrubber bar moved down so it does not overlap the video frame. Full video frame visible at all times. |
| Playback controls | Add 5s back and 15s back buttons alongside existing controls. |
| Pulsing file inputs | Video File and SRT File input areas get green pulsing border on page load. Pulse stops, border goes solid green (no pulse) once a valid file is loaded. Same animation as Transcribe URL input. |
| Caption list width | Right-hand caption list column increased ~10% in pixel width. |
| Caption list scroll | Active caption always centered in the right-hand column during playback. Editor can see incoming and outgoing captions. |
| SAVE UPDATED SRT button | Appears in upper-right area (same row as file load inputs). Red when unsaved edits exist. Green after save. Filename appended with date + time stamp on every save. Reverts to red if further edits are made without refreshing. |

---

## Phase 2 — Transcribe: Find / Replace

**Scope:** Readable view only. Available after transcript delivery.

**Behavior:**
- Traditional Find Next flow: user types search term, steps through matches one at a time
- All instances highlighted simultaneously (if feasible without performance cost)
- Case-insensitive by default
- Replace field optional — user can find-only for review, or find + replace
- No auto-apply to all instances; each replacement is individually confirmed
- Any replacement counts as an edit and triggers the red download button + EDIT_ filename behavior

**Tracked corrections (placeholder — not built in Phase 2):**
- When a term is manually corrected 3 or more times within a session, it becomes a candidate for the glossary review list
- Does not auto-promote — goes to a pending review state
- Threshold of 3 is provisional; revisit after editor feedback
- Glossary management lives in a separate lightweight tool (see Glossary section below)

---

## Phase 2 — Monitor: Caption Text Editor

### Overview

A second operating mode for the Monitor, toggled from the default caption-edit state. The area below the video and scrubber (currently the flag log) changes state depending on mode.

### Two Modes

| Mode | Default? | What the lower panel shows |
|------|----------|---------------------------|
| Caption Edit | Yes | 3-caption edit view (prev / active / next) |
| Flag / Debug | No | Existing flag toolbar and flag log |

Mode toggle button lives near the current flag count and export report controls.

### Caption Edit Mode — Detail

**3-caption panel:**
- Previous caption (above, normal weight)
- Active caption (center, bold, visually highlighted/boxed — in sync with video timecode and right-column position)
- Next caption (below, normal weight)

**Editing rules:**
- Any text in any of the three captions is directly editable, including timestamps
- Timestamp edits are independent — no ripple to adjacent captions
- Video stops automatically when a text field receives focus
- Commit button finalizes the edit
- After commit: edits are reflected immediately in the right-column master caption list
- Video does not resume automatically — user must press Play or Spacebar
- Spacebar is guarded against firing while a text input has focus

**State management requirements (for build session spec):**
- In-memory SRT state, right-column display, video timecode sync, and edit buffer must all agree after every commit
- Right-column master list is the source of truth for the full SRT
- 3-caption panel reads from and writes back to that master list
- SAVE UPDATED SRT serializes the current in-memory master list to file

### Flag / Debug Mode

Identical to current flag system. No changes from v1.1 behavior. Only active when user explicitly toggles to this mode.

---

## Glossary — Placeholder

**Not built until Phase 3 or later.**

A separate lightweight tool (own page, own deploy). Holds:
- Approved terms (active glossary, passed to AssemblyAI as keyterms at transcription time)
- Pending terms (candidates from tracked corrections, awaiting human review)

Each client context gets its own glossary instance. Authentication required before multi-client deployment.

At build time: a visual placeholder for the glossary feature may appear in Transcribe UI to establish the workflow concept, but it is non-functional until the tool is built.

---

## Future Merge Plan

Intended final layout (single tool, post-proxy):

```
[ Video player with live caption overlay ]
[ Readable / SRT toggle text editor      ]
[ Export final SRT                       ]
```

Design decisions in both tools are being made with this merge in mind:
- Caption overlay is a self-contained module — liftable without refactoring
- Right-column caption list maps to the Transcribe output block
- Find/replace and tracked corrections carry forward to the merged tool
- Flag system becomes a secondary/optional panel or is deprecated in favor of direct editing

---

## Build Session Log

| Session | Date | Scope | Status |
|---------|------|-------|--------|
| Phase 1 cosmetic pass | TBD | Both tools — pulsing inputs, font sizes, scrubber, playback controls, SAVE button, download state | Pending |
| Phase 2 Monitor editor | TBD | In-memory SRT state, 3-caption edit panel, timecode sync, commit flow | Pending |
| Phase 2 Transcribe find/replace | TBD | Readable view, Find Next, tracked corrections placeholder | Pending |
| Phase 3 Glossary | TBD | Separate tool, pending/approved terms, multi-client scaffolding | Future |

---

## Open Questions

- Password/auth wall: when does this become necessary? Who is the intended user group at launch vs. 6 months out?
- "Open in Monitor" button: pre-load SRT only, or also pass transcript metadata?
- Bidirectional sync in Monitor (edit in 3-caption panel → right column reflects change): confirm this is the full scope, no sync back to Transcribe needed in Phase 2.

---

_End of PRODUCT_SPEC.md_
