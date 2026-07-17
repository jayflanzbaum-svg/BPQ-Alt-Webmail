# Local Dev Handoff — BPQ-Alt-Webmail Experimental Branch

## How to get this branch locally

```bash
git fetch origin
git checkout experimental/jay-dev
```

Or if you don't have the repo cloned yet:

```bash
git clone https://github.com/jayflanzbaum-svg/BPQ-Alt-Webmail
cd BPQ-Alt-Webmail
git checkout experimental/jay-dev
```

The only file you work on is `bpq-alt-webmail.html`. Everything is self-contained in that one file.

## Ground rules for this branch

- This is a personal experiment — **do NOT tag releases or update main**.
- Commit freely as you go so nothing is lost.
- If something turns out great and you want to ship it, cherry-pick it into main and do a proper release then.
- Version string stays at `v1.5.9` (or bump to `v1.5.9-exp` if you want to distinguish builds).

---

## Tasks to implement

### 1 — Classic Theme

Requested by Gary K7EK (N7EK?) who wants a tan/beige look matching stock BPQ32 WebMail HTML interface.

Add a new `body.classic` theme. CSS variable values:

```css
--bg:         #c8a87c   /* warm tan page background */
--bg2:        #d4b890   /* slightly lighter tan (topbar, sidebar, cards) */
--bg3:        #b89660   /* slightly darker tan (hover states) */
--border:     #9b7a40   /* medium brown border */
--border2:    #7a5c28   /* darker brown border */
--accent:     #0033aa   /* classic blue (links, active items) */
--accent-bg:  #e8dcc8   /* pale tan accent background */
--accent-border: #9b7a40
--text:       #1a1008   /* near-black primary text */
--text2:      #3d2b0a   /* dark brown secondary text */
--text3:      #6b5030   /* medium brown muted text */
--unread-dot: #cc0000   /* red unread indicator */
```

Add to the THEMES array: `{ key:'classic', label:'Classic', icon:'📻' }`

Cycle order: dark → dark-hc → light → light-hc → classic → (back to dark)

Reference mockup is in `mockup-classic-nts.html` in the repo root (look at it before coding).

---

### 2 — NTS Delivered Button

NTS traffic handlers need to mark messages delivered after phone-delivery to an addressee. Without this they must use a separate BBS terminal session.

- Add a `btn-nts-del` button in the `rv-tb` toolbar (after Kill/Reject buttons)
- Show it only when `msg.type === 'T'` (NTS Traffic) in `openMsg()`
- Style: green success button (not red/danger)
- On click: confirmation dialog → GET request to BPQ NTS delivery endpoint → toast success/failure

**BPQ endpoint (needs live test to confirm):**
Most likely `${base()}/WebMail/WMNDel/${n}?${CFG.key}` — pattern matches existing `WMDel`. If that 404s, fall back to a general BBS command POST.

Helper to add: `ntsDelUrl = n => \`${base()}/WebMail/WMNDel/${n}?${CFG.key}\``

---

### 3 — Questions to ask the user before implementing

Before writing any code, ask the user these questions:

**About `bpq-alt-webmail-hashtest.html`:**
- Is this file something you created? What was it testing?
- Should it be incorporated into the main file or was it a throwaway?
- Does it contain any features or changes you want to keep?

**About NTS message formatting:**
- How does NTS traffic currently display in the message reader? Is the raw NTS format hard to read?
- Do you want the message body to be reformatted/parsed into labeled fields (e.g., Check, Handling Precedence, Addressee, Text, Signature) when `msg.type === 'T'`?
- Should formatted display be optional (a toggle button) or always-on for NTS messages?
- Are there any other NTS-specific features besides Delivered that operators need? (e.g., reply templates, forwarding helpers)

**About other new features:**
- What else did you want to add in this experimental session?

---

## Files in repo

- `bpq-alt-webmail.html` — the entire app (single file)
- `mockup-classic-nts.html` — reference mockup for classic theme + NTS button layout
- `mobile-mockup.html` — mobile layout reference (probably ignore)
- `HANDOFF-local-dev.md` — this file (can delete once you're set up)
