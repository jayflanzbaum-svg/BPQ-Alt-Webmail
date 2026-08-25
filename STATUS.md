---
updated: 2026-08-25
summary: Single-file HTML webmail client for BPQ32/LinBPQ packet-radio BBS nodes — v1.6.3 released (address book + QRZ lookup, SMTP-reply fix).
---

## Recent work
- 2026-08-25: Released v1.6.3 — address book + QRZ lookup, SMTP-reply-to fix, compose-cursor fix, QRZ-credentials view/update UI, contact sort order. See CHANGELOG.md for full notes. Fixed a user-reported bug (credit: Chris AE7GE) — replying to an internet-mail-gated message (BPQ32 ISP email<->BPQ32 gateway) put the user's own callsign in the To field instead of `SMTP:sender@domain`. Root cause: `doReply()`/`doForward()` used the raw, unreliable list-row `from` column instead of the already-corrected `persFromKey(msg)`; separately, `baseCall()` truncated `SMTP:user@domain` down to `SMTP:user` on every message. Also fixed (same report): reply cursor landed at the bottom of quoted text instead of the top, and `doSend()` was force-uppercasing the To field (would mangle an `SMTP:` address's case). Still not tested against a live email-gated message — needs verification next time real ISP<->BPQ32 mail flows through the node.
- 2026-08-25: Added sort order (A–Z / Most used / Last used) to the address book contact list, and changed saved QRZ credentials to display as read-only text with an Update button instead of leaving password fields open with no visible save confirmation.
- 2026-08-16: Added a "📇 Add Contact" button to the reader toolbar — saves (or edits, if already saved) the open message's sender straight into the address book, auto-running the QRZ lookup if credentials are already on file. Credited the address book feature to N3MEL via a text credit line in the address book modal.
- 2026-08-14: Added an address book (callsign/name/city/state), reachable from the compose window's To field and a topbar "📇 Address Book" button. Includes an optional QRZ.com XML Data lookup (requires the user's own paid QRZ subscription login, stored only in that browser). All CRUD/QRZ-parsing logic verified in a headless jsdom harness; not yet clicked through in a real browser or with real QRZ credentials — see Open issues.
- 2026-08-11: Fixed a second cause of "[Could not extract body]" — BPQ rotates its WebMail session key very frequently. `bpqGet()` now compares the key it sent against the key BPQ embedded in the response and retries once on mismatch.

## Open issues
- [ ] **New:** v1.6.3's SMTP-reply fix needs a live-node check with a real ISP-gated message — verified by code reading only so far.
- [ ] Address book / QRZ lookup / "Add Contact from message" needs a real-browser click-through and a real paid-QRZ-subscription test — logic is verified in isolation (jsdom + mocked QRZ XML) but never exercised end-to-end with live credentials.
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
- Single-branch workflow since 2026-08-05: all work happens on `main`; releases are marked by tags + GitHub Releases (latest: `v1.6.3`, 2026-08-25). The old `experimental/jay-dev` dev branch was deleted once it had fully converged with `main`.
