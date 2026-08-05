---
updated: 2026-08-05
summary: Single-file HTML webmail client for BPQ32/LinBPQ packet-radio BBS nodes — v1.6.1 shipped; just fixed HTML-format (Winlink) messages showing "[Could not extract body]" in the reader.
---

## Recent work
- 2026-08-05: Consolidated to a single-branch workflow — `experimental/jay-dev` was byte-identical to `main` and only added merge ceremony, so it was deleted (local + origin). Day-to-day work now happens directly on `main`; tags + GitHub Releases define what's released. CLAUDE.md release checklist trimmed accordingly.
- 2026-08-05: Fixed HTML-format messages failing to display ("[Could not extract body]"). Root cause: BPQ's WebMail.c swaps the usual `<textarea>` for a raw `<div id='main'>` whenever a message body contains `</html>` (common for Winlink/RMS Express traffic like `//WL2K R/` messages), a shape `parseBody()` didn't know. Fix: `parseBody()` now extracts readable text from that div (stripping script/style/meta noise), and `openMsg()` falls back to BPQ's `/WebMail/DisplayText` endpoint (forces plain-text textarea rendering) if parsing still fails — covers RMS Express form messages too. Verified against mock pages in a real browser DOM; needs a live-node check on message #3181.
- 2026-07-25: Added compose-draft autosave (localStorage, debounced) to fix Lee K5DAT's lost-draft report — plain compose/reply/forward text now survives an unexpected reload and is offered back on next load with a toast. Scoped to the plain compose fields only; radiogram/PKTNET forms already persist their own sticky fields.
- 2026-07-25: Wrote `UPSTREAM-REPORT-G8BPQ.md` — a ready-to-send report covering both the original kill-mail SIGSEGV and Bill PY2BIL/LU7ECX's new v1.6.1 crash (addr2line through `CreateMessage`/`GetMsg`/`RXCount`/`MONCount`, the RX/monitor path). Conclusion: our client already serializes all its own requests (`queuedFetch`), so this looks like LinBPQ's WebMail handlers racing its own RX thread on shared globals — not something a client-side fix can close. Not yet posted.
- 2026-07-25: v1.6.0 announcement posted to the bpq32 groups.io group. Held-mail crash reporter's callsign question resolved by omission — CHANGELOG v1.5.10 entry stays attribution-free ("reported on the bpq32 group"), nothing to fill in.

## Open issues
- [ ] Live-node check of the HTML-message fix: open message #3181 (`//WL2K R/ M18 & M12` from KN4KSW) and confirm readable text appears instead of the debug dump.
- [ ] Post `UPSTREAM-REPORT-G8BPQ.md` to the bpq32 group / to John Wiseman directly — content is ready, just needs sending.
- [ ] Compose-draft autosave (above) is untested against a live node/browser — exercise it once before the next release: start a plain compose, reload the tab mid-draft, confirm it's offered back.
- [ ] NTS Delivered endpoint (`WMNDel`) still needs a live-node check — I can't reach a BPQ node from this environment, so this needs Jason directly. Steps: (1) open a T-type NTS message, click Delivered, confirm the success toast; (2) the risk is that `bpqGet()` only checks HTTP status, not response content, so a 200-OK error page would still look like success — reopen the app in a private/incognito window (no localStorage) and confirm the message *still* shows delivered, proving BPQ's own state changed and it wasn't just our client-side `deliveredSet`; (3) optionally hit `http://<host>:<port>/WebMail/WMNDel/<msgnum>?<sessionkey>` directly in a browser to see BPQ's raw response.

## Future ideas
- [ ] Optional toggle to reformat raw NTS message bodies into labeled fields in the reader (raised as an open question in the handoff).

## Decisions & blockers
- Intentionally a single self-contained HTML file: no build step, no toolchain, no runtime dependencies (fonts are local woff2 files). Editing means editing the one file and testing against a real BPQ node.
- Killed messages are hidden client-side and remembered in localStorage because BPQ marks-but-doesn't-purge; body cache capped at 30 (FIFO), killed list at 500.
- Single-branch workflow since 2026-08-05: all work happens on `main`; releases are marked by tags + GitHub Releases (latest: `v1.6.1`, 2026-07-18). The old `experimental/jay-dev` dev branch was deleted once it had fully converged with `main`.
