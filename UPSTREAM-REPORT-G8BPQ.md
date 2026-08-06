# LinBPQ WebMail segfault reports (for John Wiseman G8BPQ)

Send-ready — post to the bpq32 groups.io group or email John directly. Not yet sent.
Reworded 2026-08-05: facts stay firm, interpretations are owned as guesses, and the
"add a lock" prescription became questions — John knows his own architecture.

---

**Subject: Two LinBPQ segfaults while WebMail was in use — traces and a question**

Hi John,

Two users of BPQ-Alt-Webmail have reported LinBPQ segfaults while using it, and I
wanted to pass along what we've collected in case it's useful. I'll say up front: the
crash data below is solid, but my guesses about the cause are exactly that — guesses
from the outside. You know the internals, so please read the interpretation loosely
and correct me where I'm off.

**Report 1 (~6 July):** bulk-killing held mail from WebMail fired several kill
requests in quick succession, and LinBPQ segfaulted. The trace at the time implicated
`KillWebMailMessage` with what looked like a NULL `Msg` dereference. Since v1.5.10 my
client only ever has one HTTP request in flight (a single-flight queue) and spaces
bulk-kill requests out, and no kill-related crashes have been reported since — though
I don't know whether that actually addressed the cause or just made it less likely to
trigger.

**Report 2 (17 July):** Bill PY2BIL/LU7ECX, running the precompiled v6.0.25.32 beta
on a 32-bit Raspberry Pi, got a segfault *hours* after any kill activity — nothing the
client did was in flight at the time beyond its routine folder-list refresh (every 5
minutes). His addr2line output
(`addr2line -f -C -e linbpq 0xe744f 0x12bbd6 0x12bc6c 0x12bdfc`):

```
CreateMessage   BBSUtilities.c:5594
GetMsg          CommonCode.c:1819
RXCount         CommonCode.c:1836
MONCount        CommonCode.c:1904
```

One caveat: since that's a precompiled build, I can't be certain the addresses
resolved against exactly matching symbols.

What made me wonder whether the two are related is that this stack is in the
receive/monitor path — nothing my client touches directly — which made me guess at a
WebMail HTTP request arriving while a message was being received. But that's
speculation, and if the WebMail handlers are already synchronized with the BBS side
then my theory is simply wrong and I'd be glad to know what else to look at.

So rather than propose anything, a few questions:

- Is it expected to be safe for a WebMail HTTP request to arrive while a message is
  being received, or is that something a client should try to avoid?
- Is there anything a client like mine should do differently — pacing, ordering,
  endpoints to avoid — to be gentler on the node?
- Bill is willing to help reproduce, and I'm happy to gather more traces or test
  anything against a live node.

Thanks for all your work on BPQ — this client only exists because the WebMail
interface is there to build on.

73,
Jason

---

## Internal notes (not part of the message)

- Additional report gathered while asking users for diagnostics: **Lee, K5DAT** —
  separately reported losing an in-progress WebMail compose draft, apparently tied to
  a page refresh. Client-side issue (the draft lived only in browser memory); fixed
  independently in BPQ-Alt-Webmail with draft autosave — intentionally excluded from
  the message to John.
- Dropped from the earlier draft as unverified assumptions: that LinBPQ services each
  HTTP request on its own thread over unlocked shared globals, the `CheckUserMsg`
  argument-mismatch claim, and the assertion that Reports 1 and 2 share the same
  root-cause pattern. The facts (triggers, timing, traces) are retained above; the
  theories are now framed as questions.
