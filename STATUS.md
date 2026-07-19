---
updated: 2026-07-18
summary: Single-file HTML webmail client for BPQ32/LinBPQ packet-radio BBS nodes — v1.6.1 patch improves the PKTNET Check-In form (default To, structured subject, Town/State fields).
---

## Recent work
- 2026-07-18: v1.6.1 committed (d2ee09f) and pushed to origin/experimental/jay-dev. Set up git CLI + GitHub auth (PAT via Git Credential Manager) on this machine, since it had no git install at all.
- 2026-07-18: v1.6.1 patch — PKTNET Check-In: To defaults to PKTNET@USA (sticky), subject now built as "Name, Call, Town, State", Town + State fields added to Location section.
- 2026-07-17: v1.6.0 released and pushed — main, experimental/jay-dev, and tag v1.6.0 all on GitHub; HANDOFF-local-dev.md and stray tmp files deleted first. groups.io announcement drafted with attributions and a call for crash diagnostics; GitHub release page + posting still to do.
- 2026-07-07: Committed v1.5.10 as four clean commits on experimental/jay-dev (hashchange deep links, request-serializer crash fix, remember-password option, release+CHANGELOG). File ready to hand to the two reporters for verification before the bigger v1.6.0 feature release.
- 2026-07-06: Added "remember password" checkbox (Settings, next to BBS Password) after KB1TAE's forum request — unchecked keeps the password in sessionStorage only and prompts each browser session; supersedes his hand-patched copy in the groups.io files area.
- 2026-07-06: Fixed reported linbpq SIGSEGV when killing held mail — added a global request serializer (`queuedFetch`) so only one HTTP request is ever in flight to the node, and made bulk kill sequential with a 250 ms gap; bumped to v1.5.10. Root cause is in linbpq itself (thread-per-request over unlocked WebMail globals); upstream report to G8BPQ drafted.
- 2026-05-25: Added dashboard deep-link docs (README-dashboard-links.md) for URL-hash `#compose?` pre-filled messages; uncommitted working-tree change adds a `hashchange` listener so already-open tabs react to new compose links.
- 2026-05-25: Added B/PKTNET PKTNET Check-In form-style message type and an attribution/version line at the bottom of each form.
- 2026-05-25: Added HX-code variables (HXA miles, HXB hours, HXF date) and the full ARRL extended-punctuation table for radiogram body/email.
- 2026-05-24: Built the NTS Radiogram (T/RRI) composer end-to-end — form drives BPQ To/Subject, REVIEW flow, HX-code meanings, auto-incrementing msg #, and parsing of incoming radiograms back into the form.

## Open issues
- [ ] Post the v1.6.0 announcement to the bpq32 group (GitHub release is live: https://github.com/jayflanzbaum-svg/BPQ-Alt-Webmail/releases/tag/v1.6.0).
- [ ] Fill in the held-mail crash reporter's callsign in the CHANGELOG v1.5.10 entry and the announcement before posting.
- [ ] Collect addr2line output from anyone still seeing the LinBPQ kill crash; then send the upstream report to G8BPQ (NULL-Msg deref in KillWebMailMessage, thread-safety of WebMail globals, CheckUserMsg arg mismatch).
- [ ] NTS Delivered endpoint (`WMNDel`) was flagged in the handoff as "needs live test to confirm" — verify against a live node.
- [ ] CHANGELOG.md and README still document only up to v1.5.9 (2026-04-29); none of the NTS/classic/PKTNET/deep-link work on this branch is reflected there.

## Future ideas
- [ ] Cherry-pick the strongest experimental features (classic theme, NTS tools, form composers) into main and ship a proper release.
- [ ] Optional toggle to reformat raw NTS message bodies into labeled fields in the reader (raised as an open question in the handoff).

## Decisions & blockers
- Intentionally a single self-contained HTML file: no build step, no toolchain, no runtime dependencies (fonts are local woff2 files). Editing means editing the one file and testing against a real BPQ node.
- `experimental/jay-dev` is explicitly a personal sandbox: HANDOFF-local-dev.md says do not tag releases or touch main from here; promotion to main is by cherry-pick only.
- Killed messages are hidden client-side and remembered in localStorage because BPQ marks-but-doesn't-purge; body cache capped at 30 (FIFO), killed list at 500.
- Released as `v1.6.0` (2026-07-17): experimental/jay-dev merged to main and tagged, ending the branch's cherry-pick-only rule with Jason's sign-off.
