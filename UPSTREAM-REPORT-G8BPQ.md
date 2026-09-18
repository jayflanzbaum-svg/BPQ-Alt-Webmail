# LinBPQ / BPQ32 reports for John Wiseman G8BPQ

Send-ready — post to the bpq32 groups.io group or email John directly. Neither sent yet.
Reworded 2026-08-05: facts stay firm, interpretations are owned as guesses, and the
"add a lock" prescription became questions — John knows his own architecture.

Two separate items, each written to stand alone:

1. **WebMail segfaults** (below) — two crash reports with traces.
2. **Web server serves a stale `Last-Modified`** (further down, added 2026-09-18)
   — small, reproducible, and it bites every sysop who updates a file in their
   HTML directory.

---

## Report 1 — WebMail segfaults

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

## Report 2 — Web server reports a stale `Last-Modified`, so browsers serve old files

**Subject: BPQ web server sends a stale Last-Modified for files in the HTML directory**

Hi John,

Small one, with a clean reproduction. The BPQ web server appears to cache a file's
`Last-Modified` the first time it serves that path and then never re-stat the file,
so after a sysop updates a file in their HTML directory the server keeps advertising
the *original* date — while correctly serving the new bytes.

Reproduced on BPQ32 on Windows, `HTTPPORT=8010`, 2026-09-18:

| path | file mtime on disk | `Last-Modified` in the response |
|---|---|---|
| `bpq-alt-webmail.html` (first served in August) | 2026-09-17 14:52 | **Thu, 27 Aug 2026 00:45:22 GMT** |
| `bpq-alt-webmail-TEST.html` (first served that morning) | 2026-09-17 14:52 | Wed, 17 Sep 2026 03:27:41 GMT |
| a brand-new filename, copied from the same source | just now | correct, to the second |

The `Content-Length` and body are correct and current in every case — it is only the
date that is stale, and the staleness matches when that particular path was first
requested rather than anything about the file.

Two related things I noticed in the same responses:

- There is no `ETag` and no `Cache-Control` header.
- `If-Modified-Since` appears to be ignored: sending the exact date the server itself
  advertised still returns `200` with the full body rather than `304`.

Why it matters in practice: with no `Cache-Control` and no `ETag`, browsers fall back
to heuristic freshness, commonly around 10% of the age implied by `Last-Modified`. A
date three weeks in the past therefore tells the browser it may reuse its copy for
roughly two days **without revalidating**. The sysop updates a file, reloads, and sees
the old version with no indication why — and because Chrome and Firefox implement
that heuristic differently, it often presents as "it works in one browser but not the
other", which sends people looking in entirely the wrong place. It cost me a while
before I thought to compare `curl` against the browser.

I am guessing at the cause from the outside, so please read this loosely: it looks
like the stat is cached alongside the path on first use. If that is what is happening,
re-stat'ing per request would fix it. Sending an `ETag`, or simply a
`Cache-Control: no-cache` on files served from the HTML directory, would also stop
browsers caching heuristically and would make the `If-Modified-Since` question moot.

Happy to test any change against a live node, or to gather the same table on LinBPQ if
that would help — I only have BPQ32 on Windows in front of me.

73,
Jason

---

## Internal notes (not part of the message)

- Additional report gathered while asking users for diagnostics: **Lee, K5DAT** —
  separately reported losing an in-progress WebMail compose draft, apparently tied to
  a page refresh. Client-side issue (the draft lived only in browser memory); fixed
  independently in BPQ-Alt-Webmail with draft autosave — intentionally excluded from
  the message to John.
- **Report 2 evidence** (2026-09-18) was gathered with `curl` against the live node:
  response headers show only `Content-Length`, `Date`, `Last-Modified` and
  `Content-Type`; `curl -H "If-Modified-Since: <the advertised date>"` returned 200 with
  the full 275,180-byte body; a fresh filename reported the correct date, which is what
  isolates it to per-path metadata caching rather than a clock or timezone issue.
  Not verified on LinBPQ — deliberately said so in the message.
- Dropped from the earlier draft as unverified assumptions: that LinBPQ services each
  HTTP request on its own thread over unlocked shared globals, the `CheckUserMsg`
  argument-mismatch claim, and the assertion that Reports 1 and 2 share the same
  root-cause pattern. The facts (triggers, timing, traces) are retained above; the
  theories are now framed as questions.
