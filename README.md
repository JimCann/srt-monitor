# CannDo SRT Monitor

A browser-based QA tool for verifying SRT caption files against source video. Used immediately after the CannDo Transcribe tool as step 2 in the captioning workflow.

**Live:** https://clever-moxie-45f90b.netlify.app  
**GitHub:** https://github.com/JimCann/srt-monitor  
**Netlify site ID:** `c6cfc095-c3a5-47d3-b4c7-38af39c62ec4`

---

## What It Does

- Load a video file + SRT file in-browser (no upload, no server)
- Video plays with live caption overlay synced to SRT timecodes
- Diagnostics bar shows current caption index, start/end, duration, gap to next, line count — all color-coded during playback
- Flag system lets user mark problem captions during or after playback with a type (Early / Late / Too Long / Read Ahead / Other) and optional note
- Export a structured flag report as a plain text file

---

## Design History

**Built with no design brief.** All original color, font, and layout decisions were made at build time with no brand context (JetBrains Mono + Outfit, amber `#f0a020` accent, `#0b0b0e` background).

**Design pass v1.0 — 2026-05-19, 6:23 PM PDT**  
Brief: `CannDo-SRTMonitor-Design-Brief-v1.0.docx`  
Applied via Claude Code. Five priorities completed in sequence:

1. **Brand alignment** — DM Mono + DM Sans fonts, blue `#2980b9` accent (replaces amber), five-color stripe bar, `#0a0a0a` background, static clapper icon, logo restructured
2. **Typography** — diagnostics labels/values scaled up, sidebar and flag log text more readable, status message promoted to 13px DM Sans
3. **Header restructure** — keyboard hints removed from header → relocated below flag bar; load-order badges (1/2) on buttons; status message higher contrast
4. **Flag system** — label shortening, Other pre-selected, live "Flagging: Caption #N" indicator, flag button disabled until SRT loaded, Enter key triggers flag, export button activates on first flag, structured export header block
5. **Spacebar** — play/pause keyboard shortcut added

---

## Shared Brand Language (with CannDo Transcribe)

Both tools must feel like siblings. Key shared decisions from Transcribe Redesign Brief v1.1:

| Token | Value |
|-------|-------|
| Font (mono) | DM Mono |
| Font (sans) | DM Sans |
| Background | `#0a0a0a` |
| Stripe | green `#27ae60` / yellow `#f1c40f` / blue `#2980b9` / red `#c0392b` / grey `#888888` |
| Diagnostic accent | `#2980b9` (blue — Monitor-specific) |
| Success | `#27ae60` (green) |

---

## Do Not Touch

- SRT parser (`parseSRT`, `tcToSec`)
- Playback and caption sync logic (rAF loop, caption overlay)
- Flag export logic (structure only — export format can gain headers, but flag data is unchanged)
- Right sidebar seek-on-click behavior
- Clickable timecodes in flag log

Flag a functional issue rather than fixing it — functional bugs have a separate workstream.

---

## Future State

This tool will eventually be **merged into CannDo Transcribe**. Intended merged layout:
- Video player with live caption overlay (top)
- Text editor with Readable/SRT toggle (below)
- Export final SRT on completion

Design decisions in v1.0 pass were made with this merge in mind:
- Caption overlay built as a self-contained module
- Flag system architected as a secondary/optional panel
- Right sidebar caption list will be absorbed by or merged with the Transcribe output block

---

## Versioning

Follows the same pattern as CannDo Transcribe.

- `html/index-vX_X.html` — versioned source of truth; filename reflects version
- `index.html` — deploy artifact at project root; always a copy of the current versioned file
- Version number is visible in the page header as a small badge (`ver-badge`)

**Version history:**
| Version | Date | Key change |
|---------|------|------------|
| v1.0 | 2026-05-19, 6:23 PM PDT | Design pass v1.0 — brand alignment, typography, header restructure, flag UX, spacebar |

End-of-session workflow:
```bash
# After making changes to html/index-vX_X.html:
cp html/index-vX_X.html index.html
git add .
git commit -m "vX.X -- description"
git push
netlify deploy --prod --site c6cfc095-c3a5-47d3-b4c7-38af39c62ec4 --dir .
```

Always update the `ver-badge` text in the HTML to match the new version number before copying to `index.html`.

---

## Deploy
