# LinBPQ WebMail thread-safety report (for John Wiseman G8BPQ)

Draft — ready to post to the bpq32 groups.io group or send directly. Not yet sent.

## Summary

LinBPQ has segfaulted (SIGSEGV) more than once while its WebMail HTTP interface is in
use. The first occurrences correlated with killing held mail and were mitigated
client-side in BPQ-Alt-Webmail v1.5.10 (see below), but a fresh report on v1.6.1 shows
the same class of crash with a *different* stack, in code that has nothing to do with
killing a message. That points to a broader thread-safety issue in LinBPQ's shared
WebMail/message-list state, not something a client can fully work around.

## Report 1 — kill-held-mail crash (fixed client-side, root cause still open)

- **When:** reported ~2026-07-06, against BPQ-Alt-Webmail pre-v1.5.10.
- **Trigger:** bulk-killing held mail from the WebMail interface fired multiple kill
  requests concurrently.
- **Cause (as best diagnosed):** LinBPQ services every HTTP request on its own thread,
  reading/writing shared, unlocked WebMail globals. Overlapping kill requests raced on
  that shared state (symbols implicated: `KillWebMailMessage`, a NULL `Msg` pointer
  dereference, and a `CheckUserMsg` argument mismatch).
- **Client-side mitigation (BPQ-Alt-Webmail v1.5.10):** all requests to the node now go
  through a single-flight queue (`queuedFetch`) so this client never has more than one
  HTTP request in flight, and bulk-kill sends its kill requests one at a time with a
  short gap instead of firing them all at once.
- **Result:** reduces exposure for this one client, but does not fix the underlying
  race — any other concurrent access to the same WebMail globals (another browser tab,
  another WebMail client, or LinBPQ's own background threads) can still trigger it.

## Report 2 — new crash on v1.6.1, different stack, not tied to killing mail

- **Reporter:** Bill, PY2BIL / LU7ECX.
- **When:** 2026-07-17, running BPQ v6.0.25.32 (precompiled beta build) on a 32-bit
  Raspberry Pi.
- **Circumstances:** killed some held mail, then the crash happened **hours later** —
  not while any kill action was in flight. He had previously stopped using
  BPQ-Alt-Webmail for several days with no crashes, then resumed and hit this on the
  first day back.
- **addr2line output** (`addr2line -f -C -e linbpq 0xe744f 0x12bbd6 0x12bc6c 0x12bdfc`):
  ```
  CreateMessage
  /mnt/Source/bpq32/CommonSource/BBSUtilities.c:5594
  GetMsg
  /mnt/Source/bpq32/CommonSource/CommonCode.c:1819
  RXCount
  /mnt/Source/bpq32/CommonSource/CommonCode.c:1836
  MONCount
  /mnt/Source/bpq32/CommonSource/CommonCode.c:1904
  ```
- **Why this matters:** this stack is in message creation / RX / monitor counting —
  the live packet-receive path — not the kill-message path the v1.5.10 fix targeted.
  Since the crash happened hours after any client-initiated kill request, it looks like
  it's the RX thread creating/counting a newly received message while something else
  (plausibly a WebMail HTTP handler thread doing a routine folder-list GET, which
  BPQ-Alt-Webmail's auto-refresh issues every 5 minutes) is concurrently reading the
  same shared message list/counters — the same unlocked-globals pattern as Report 1,
  just a different pair of threads racing on it.

## Ask

Could the WebMail HTTP handlers and the RX/monitor code share a lock (or otherwise
synchronize) around the message list and its counters? From the outside it looks like
any two threads touching that shared state concurrently — regardless of which HTTP
endpoint or internal path triggered them — can race. A client-side request queue (as
added in v1.5.10) only prevents one client's own requests from overlapping each other;
it can't prevent a race against LinBPQ's own background RX activity.

Happy to help gather more addr2line traces or test candidate fixes against a live node
if useful.

## Additional reports gathered while asking users for diagnostics

- **Bill, PY2BIL / LU7ECX** — see Report 2 above.
- **Lee, K5DAT** — separately reported losing an in-progress WebMail compose draft,
  apparently tied to a page refresh. This is understood to be a client-side issue (the
  draft lived only in browser memory with nothing to survive a reload) and has been
  addressed independently in BPQ-Alt-Webmail with draft autosave — not part of this
  LinBPQ report.
