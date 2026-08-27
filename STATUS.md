---
updated: 2026-08-27
summary: Single-file HTML webmail client for BPQ32/LinBPQ packet-radio BBS nodes — v1.6.5 released; Chris AE7GE actively live-testing each release and reporting real bugs same-day.
---

## Recent work
- 2026-08-27: Chris AE7GE reported two more issues while live-testing v1.6.4: the "My Received (#)" sidebar badge didn't update when a message was read or killed — stayed stale until a full page reload (this confirms v1.6.4's readSet-persistence fix itself is working correctly; the bug was the *live* badge never recomputing without a server round-trip) — and separately asked what the topbar ↻ button does, since clicking it gave no visible feedback. Released v1.6.5: added `updateActiveFolderBadge()`, a local (no-fetch) recompute called immediately after marking a message read and after killing one; made the manual ↻ refresh show a toast ("No new messages" or the error) and update the "Xm ago" label immediately instead of only on the silent background timer; added a hover tooltip ("Refresh messages") to the ↻ button. Not yet live-tested by Chris.
- 2026-08-26: Chris AE7GE confirmed v1.6.3's SMTP-reply-to and reply-cursor fixes work live ("addressed my reported issues") and reported three new things while testing: read messages showing unread again after every reload (and a matching blue-badge/list-count mismatch), the compose cursor still jumping to the end of the signature on a brand-new message (only Reply/Forward were fixed in v1.6.3), and a request for oldest-first message ordering. Released v1.6.4 fixing all three: `readSet` (read-message tracking) is now persisted to localStorage like the killed-message list, the cursor-reset in `openCompose()` is now applied unconditionally instead of only when pre-filling quoted text, and a sort-order toggle (↓/↑ in the list header) switches between BPQ's default newest-first and oldest-first.
- 2026-08-25: Released v1.6.3 — address book + QRZ lookup (concept: N3MEL), SMTP-reply-to fix, compose-cursor fix for Reply/Forward, QRZ-credentials view/update UI, contact sort order. Fixed a user-reported bug (credit: Chris AE7GE) — replying to an internet-mail-gated message put the user's own callsign in the To field instead of `SMTP:sender@domain`; root cause was `doReply()`/`doForward()` reading the raw, unreliable list-row `from` column, plus `baseCall()` truncating `SMTP:user@domain` down to `SMTP:user`.
- 2026-08-16: Added a "📇 Add Contact" button to the reader toolbar — saves (or edits, if already saved) the open message's sender straight into the address book, auto-running the QRZ lookup if credentials are already on file.
- 2026-08-14: Added an address book (callsign/name/city/state), reachable from the compose window's To field and a topbar "📇 Address Book" button. Includes an optional QRZ.com XML Data lookup (requires the user's own paid QRZ subscription login, stored only in that browser).

## Open issues
- [ ] **New:** v1.6.5's two fixes (local badge recompute on read/kill, manual-refresh feedback) need live-browser verification by Chris — released same-day from his bug report, verified by code reading only so far.
- [ ] v1.6.4's compose-cursor and sort-order fixes still awaiting explicit live confirmation from Chris (the readSet-persistence half is now indirectly confirmed via the v1.6.5 report above).
- [ ] Address book / QRZ lookup / "Add Contact from message" still needs a real-browser click-through and a real paid-QRZ-subscription test — logic is verified in isolation (jsdom + mocked QRZ XML) but not yet exercised live.
- [ ] Live-browser check of the stale-key retry fix (2026-08-11) — curl confirmed the key-mismatch/retry *logic* against the live node, but the actual `bpqGet()` retry path needs exercising in the real app.
- [ ] The div#main HTML-body fallback in `parseBody()` (2026-08-05 fix) is still unverified against a live example — need to find/produce an actual message whose body contains `</html>`.
- [ ] Post `UPSTREAM-REPORT-G8BPQ.md` to the bpq32 group / to John Wiseman directly — content is ready, just needs sending.
- [ ] Compose-draft autosave is untested against a live node/browser.
- [ ] NTS Delivered endpoint (`WMNDel`) still needs a live-node check.

## Future ideas
- [ ] Optional toggle to reformat raw NTS message bodies into labeled fields in the reader (raised as an open question in the handoff).

## Decisions & blockers
- Intentionally a single self-contained HTML file: no build step, no toolchain, no runtime dependencies (fonts are local woff2 files). Editing means editing the one file and testing against a real BPQ node.
- Jason's actual node HTTPPORT is **8010**, not 8080 — 8080 is only the app's first-run setup-screen placeholder default. Confirmed 2026-08-25 via curl (8080 connection-refused, 8010 serves the app). Don't assume 8080 when reasoning about his live setup.
- BPQ32's built-in web server sends no CORS headers at all (confirmed 2026-08-25 via curl — no `Access-Control-Allow-Origin` even with `Origin: null`). The app must be loaded from BPQ's own server (same-origin, e.g. `http://127.0.0.1:8010/bpq-alt-webmail.html`) — opening the local file directly (`file://`, e.g. double-clicking it in Explorer) breaks every fetch to the node and also lands in a different, mostly-empty localStorage bucket (Chromium/Edge share one storage bucket across all `file://` pages, unrelated to the real one used at the `http://127.0.0.1:8010` origin).
- This dev environment can reach Jason's live BPQ node directly at `127.0.0.1:8010` (confirmed 2026-08-11 via curl) — raw endpoint checks no longer need to wait for Jason.
- Killed messages are hidden client-side and remembered in localStorage because BPQ marks-but-doesn't-purge; body cache capped at 30 (FIFO), killed list at 500.
- Single-branch workflow since 2026-08-05: all work happens on `main`; releases are marked by tags + GitHub Releases (latest: `v1.6.5`, 2026-08-27). The old `experimental/jay-dev` dev branch was deleted once it had fully converged with `main`.
