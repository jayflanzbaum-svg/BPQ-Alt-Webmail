# Changelog

## v1.7.0 — 2026-09-17

A substantial address-book release: contact groups, sending to a group, bulk
import, a full contact card, and the ability to unsubscribe from bulletin
categories you do not want to see.

### Added

**Address book**
- **Rebuilt as a two-pane workspace** — a groups rail beside the contact list, sized `min(1600px, 96vw)` by `min(1000px, 92vh)` instead of the old 640px column. The groups rail has its own filter, the contact list has a select-all header and an inline count, and very large books render up to 400 rows at a time with a “narrow the filter” note rather than rebuilding thousands of rows on every keystroke. On a phone the two panes take turns behind a Groups / Contacts switcher.
- **A full contact card**, opened as a dialog over the address book: callsign, full name, street, city, state, ZIP, phone, email, website, free-text notes, and group membership. Reached with **+ Add Contact**, by clicking any contact, or straight off a message. Contacts saved before this release simply have the new fields blank.
- **Contact groups.** A group is a named list of callsigns (`bpq_groups` in localStorage) that *references* contacts rather than copying them, so editing a contact once updates it everywhere and a contact can belong to any number of groups — the Gmail-label model rather than Outlook's copy-into-the-list one. Renaming or deleting a contact follows through into every group; deleting a group never deletes contacts. Right-click a group to compose to it, rename it or delete it.
- **A chip + type-ahead group picker** on the contact card and in the importer. Groups show as removable chips; one input filters as you type, and a name that does not exist yet is offered as **+ Create “…”**, so creating a group is a side effect of typing it. Clicking the empty field drops down every group, so a handful still reads like a list while dozens are reached by typing. Keyboard: ↑↓ to move, Enter to take the highlight or create what you typed, Backspace on an empty field to drop the last chip, Esc to close.
- **Send to a group, or to a selection.** **✉ Compose** in the toolbar addresses the contacts you have ticked (the button carries the count) or, with nothing ticked and a group open, everyone in that group; right-clicking a group in the rail does the same. BPQ's `EMSave` takes one recipient per POST, so the app fans a comma-separated To field out into one send per recipient — behind a confirmation, with progress as it works and a single summary at the end. Groups also appear in the To field's autocomplete, expanded to their members.
- **Import contacts.** Paste callsigns separated by anything (spaces, commas, semicolons, newlines) or load — or drag and drop — a CSV, with or without a `callsign,name,city,state` header row, including quoted `"Surname, First"` fields. A list of nothing but callsigns is read as a list even when comma-separated; a row whose later fields are not callsigns is read as CSV columns. Routing suffixes are stripped, duplicates collapse, and anything not callsign-shaped is skipped and named in the preview rather than silently dropped. Before committing, the preview states how many are new versus already known and which groups they will land in. Existing contacts are never overwritten — an import only fills blank fields. Every imported contact can be filed into any number of groups, including one created on the spot.
- **Bulk actions** in the selection bar: add to group, remove from group, and delete.
- **📇+ Save Sender** in the reader toolbar, and a matching **📇+** beside compose's To field, file the station you are reading or replying to. Either opens the address book with the contact card over it, pre-filled with that callsign and — with QRZ connected — already looked up, so you can add a name, phone or group while the message is still in front of you. Both reduce an `@route` / `@winlink.org` / hierarchical BBS suffix to the bare callsign the address book is keyed on. A station already on file opens for editing rather than duplicating.
- **QRZ lookups now fill street, ZIP and email** as well as name, city and state, and only ever fill blank fields, so nothing you typed is overwritten. An optional post-import pass can look up newly imported contacts in the background, one per second.

**Bulletins**
- **Subscribe / unsubscribe to bulletin TO categories.** Right-click (or long-press, on touch) a TO entry in the Bulletins tree, or any bulletin in the message list, to stop seeing that category. `WX`, `PKTNET` and the like drop out of the tree's *ALL* roll-up, out of the message list, and out of the folder's unread badge. This is a purely local view filter — nothing is killed or rejected on the BBS, unlike **Reject**, which asks the node itself to stop accepting the traffic. Unsubscribed categories stay listed at the bottom of the tree, dimmed and marked ⊘, so they can be reviewed and resubscribed at any time; picking one explicitly still shows its mail. Stored in `bpq_bull_unsub`.
- **Right-click any message** for quick actions: unsubscribe from that bulletin category, save the sender to the address book, or reply.
- **Long-press is the touch equivalent of right-click** throughout (iOS Safari never fires `contextmenu` on a long press). A press that moves is treated as a scroll and abandoned.

### Changed
- **QRZ is no longer a section of the address book.** It was a full-width block at the top of the modal despite being one-time setup. The only entry point now is a small **🔍 look up on QRZ** link on the contact card: with credentials saved it simply looks the callsign up, and without them it opens a compact “Connect QRZ.com” prompt naming the callsign, then carries straight on with the lookup once you sign in. The only always-visible trace is one dim `QRZ: <user>` / `QRZ not connected` link in the modal footer, which reopens the same prompt to change or disconnect the account.
- The per-contact **✉ compose button** in the contact list is much larger (0.77rem → 1.7rem) — it is the control reached for most often there and was previously a pinprick beside the delete ✕.

### Fixed
- **Sending to a group popped one blocking OK dialog per recipient.** BPQ's `EMSave` reply page runs its own `alert()` (e.g. “N3HYM.#FRDK.MD.USA.NOAM added from WP”), and because that reply loads in a same-origin hidden iframe, every one of them blocked the whole window — a 23-recipient group meant 23 OK presses before the send finished. The send iframe is now sandboxed with `allow-forms allow-same-origin`: without `allow-scripts`/`allow-modals` it cannot open a dialog at all, while `allow-same-origin` keeps its body readable, so BPQ's message is recovered from the reply and folded into the single end-of-send toast (and logged in full to the console) rather than lost. One send now produces exactly one confirmation, whatever the recipient count.
- **Saving a new contact whose callsign already existed silently overwrote the existing one.** `saveContact()` decided “new versus edit” from the *typed* callsign rather than from how the form was opened, so a fresh entry for a callsign already on file fell through to the update path and wrote its blank fields over the saved record — losing a name or city with no warning and no undo. It now keys off how the card was opened, refuses a duplicate, and also refuses a rename onto a callsign that already exists (which would have merged two contacts into one).
- **Reply and Forward windows were titled “New Message”.** `openCompose()` set the window title and then called `onComposeTypeChange()`, which unconditionally rewrites the title for the selected compose type — so the title passed in was always immediately overwritten. It is now applied after that call.

## v1.6.8 — 2026-09-10

### Fixed
- **SMTP/RMS messages dropped the first word of the subject in the reading-pane header** (credit: Chris AE7GE). Directory listing rows for internet-mail messages have no `via`/BBS column, so the sender's `SMTP:user@domain` address lands in that column instead, shifting everything else left by one. `parseList()` already had a heuristic to detect and correct this shift, but its `looksLikeSender()` check rejected the `SMTP:` pattern (the colon/dot fail the plain callsign regex), so the correction never fired — the first subject word was misread as the `from` field and silently dropped (e.g. "This is a test message." rendered as "is a test message."). `looksLikeSender()` now also recognizes the `SMTP:` prefix, matching the convention `baseCall()` already uses elsewhere for the same header format.

## v1.6.7 — 2026-09-02

### Fixed
- **Reply to a Winlink-gated message dropped the `@winlink.org` off the sender's address**, leaving a bare callsign in the To field. BPQ then has no way to tell the reply is destined for the Winlink gateway — `KJ4XYZ` looks like an ordinary local/BBS callsign — so the reply routes by the BBS routing table (or sits locally) instead of going back out to Winlink. Same root cause as the v1.6.1 `SMTP:user@domain` bug: `baseCall()` strips everything after the `@` because a BBS *hierarchical* suffix is routing noise for display purposes, and `doReply()` was reusing that display value as the reply address. The full `From:` header value is now kept alongside the stripped one (`msg.hdrFromFull`), and a new `replyAddr()` puts the complete address in the To field whenever the header carried a domain — including a hierarchical home-BBS suffix such as `@KE4MOB.#CFL.FL.USA.NOAM`, which BPQ also needs in order to route a reply off-node. The reader's From line and the Personal-folder sender filters still show/group by the bare callsign.

## v1.6.6 — 2026-08-30

### Fixed
- **Sent messages appeared in "My Received" after a LinBPQ restart** (credit: Chris AE7GE, with a full step-by-step repro). Three bugs stacked up:
  1. **The login-recovery path in `bpqGet()` returned the wrong page.** A LinBPQ restart invalidates the saved WebMail session key; on the next page load BPQ answers the `WMtoMe` request with its login form, the app logs back in — and then returned BPQ's *post-signon landing page* as if it were the requested `WMtoMe` list. That landing page includes mail **from** the user, which is how sent messages entered the received list. Now, after a successful login, the originally requested URL is re-fetched under the fresh key (mirroring what the stale-key-rotation path already did).
  2. **The client-side "addressed to me" safety filter was a substring match**, so a sent message whose destination *email address contains the user's callsign* (e.g. `to:CHRIS.AE7GE@gmail.com` for AE7GE) slipped through — which is why only internet-bound sent mail leaked, never BBS-to-BBS sent mail. The filter (`filterPersonal()`) now token-matches the `to` field: exact callsign, `CALL@route`, or `CALL-SSID` only.
  3. **The topbar "● N msgs" status pill was only written by a full folder load**, so after auto/manual refresh pruned the leaked messages the pill kept showing the stale count until the folder was clicked again. `silentRefresh()` now updates it, applies the same Personal filter to refreshed lists, and `updateFolderBadges()` applies it to the sidebar unread badge too.

## v1.6.5 — 2026-08-27

### Fixed
- **"My Received" sidebar unread badge didn't update when a message was read or killed** — it stayed on the old count until the next auto-refresh or a full page reload. It only recomputed from a server round-trip (`updateFolderBadges()`), which nothing local triggered. Now a lightweight local recompute (`updateActiveFolderBadge()`) runs immediately after marking a message read and after killing a message. (credit: Chris AE7GE)
- **Manual refresh (↻) gave no feedback when there was nothing new** — it silently re-fetched and diffed the list but showed no toast unless new mail arrived, and the "Xm ago" label only updated on a lazy 60-second timer, so clicking it looked like it did nothing. It now shows "No new messages" or the error, and updates the "ago" label immediately, on manual clicks only (the background auto-refresh stays silent). Also added a hover tooltip to the ↻ button.

## v1.6.4 — 2026-08-26

### Added
- **Sort order toggle** — the ↓/↑ button next to the message list title switches between newest-first (BPQ's default order) and oldest-first, persisted per browser.

### Fixed
- **Read messages showed as unread again after every page reload**, and the blue "My Received" sidebar badge (unread count) didn't match the total shown above the message list. Both were caused by the same bug: the read-message tracking (`readSet`) was intentionally never persisted, so a reload forgot everything you'd already opened — every message counted as unread again until re-clicked. Now persisted to `localStorage` like the killed-message list. (credit: Chris AE7GE)
- **Compose cursor jumped to the end of the signature line when tabbing into the message body** on a brand-new message (not just Reply/Forward, which was fixed in v1.6.3). Root cause: setting the textarea's `.value` moves the browser's caret to the end of the new text; the previous fix only corrected this for Reply/Forward. Now applied unconditionally on every compose open. (credit: Chris AE7GE)

## v1.6.3 — 2026-08-25

### Added
- **Address Book** — callsign/name/city/state contacts, reachable from the compose window's To field (📇 button + autocomplete) and a topbar "📇 Address Book" button. Includes an optional QRZ.com XML Data lookup (requires your own paid QRZ subscription login, stored only in that browser) to auto-fill name/city/state from a callsign, and a reader-toolbar "📇 Add Contact" button to save a message's sender directly. (concept: N3MEL)
- **Address Book — sort order** — Contacts list can be sorted A–Z, by most-used, or by last-used, persisted across sessions.

### Changed
- **QRZ credentials UI** — once saved, QRZ username/password now display as read-only text with an Update button instead of leaving password fields sitting open with no visible confirmation that Save worked.

### Fixed
- **Reply/Forward on internet-gated mail put your own callsign in the To field instead of the sender's `SMTP:user@domain` address.** `doReply()`/`doForward()` were reading the raw, unreliable message-list `from` column instead of the already-corrected header value the app computes elsewhere; separately, the address-cleanup helper was truncating `SMTP:user@domain` down to `SMTP:user` on every message, discarding the domain needed to route a reply back out. Also fixed Send force-uppercasing the To field, which could mangle an `SMTP:` address's case. (credit: Chris AE7GE)
- **Reply cursor landed at the bottom of the quoted text instead of the top.** Compose now explicitly places the cursor at the start of the message body on Reply/Forward. (credit: Chris AE7GE)

## v1.6.2 — 2026-08-11

### Fixed
- **"[Could not extract body]" when opening a message — stale session key.** BPQ's WebMail session key expires after a period of idle time; when the app sends a now-stale key to fetch a specific message, BPQ doesn't error, it silently serves the message-*list* page instead (rendered under its own current key), which the reader couldn't parse as a message body. `bpqGet()` now compares the key it sent against the key BPQ embedded in the response's own links and, on a mismatch, updates the stored key and retries the same request once, transparently.
- **"[Could not extract body]" for HTML-format (Winlink) messages.** BPQ swaps the usual `<textarea>` for a raw `<div id='main'>` whenever a message body contains `</html>` (common for Winlink/RMS Express traffic like `//WL2K R/` messages). `parseBody()` now extracts readable text from that div (stripping script/style/meta noise), and the reader falls back to BPQ's `/WebMail/DisplayText` endpoint (plain-text rendering) if parsing still fails — covers RMS Express form messages too.

### Added
- **Compose draft autosave** — plain compose/reply/forward text now survives an unexpected reload and is offered back on next load with a toast. Scoped to the plain compose fields; radiogram/PKTNET forms already persist their own sticky fields.

## v1.6.1 — 2026-07-18

### Changed
- **PKTNET Check-In — To field** now defaults to `PKTNET@USA` and persists across sessions (was cleared after each send).
- **PKTNET Check-In — Subject** is now built from the form as `Name, Call, Town, State` (e.g. `John Doe, W4ABC, Orlando, FL`) instead of the fixed `PKTNET CHECK-IN` string.
- **PKTNET Check-In — Location section** adds Town and State fields (saved as sticky defaults alongside call and grid square).

## v1.6.0 — 2026-07-17

### Added
- **NTS Radiogram (T/RRI) composer** — full ARRL radiogram form: drives BPQ's To/Subject automatically, REVIEW step before send, HX handling-code meanings shown inline, HX variables (HXA miles, HXB hours, HXF date), auto-incrementing message number with never-blank fallback, full ARRL extended-punctuation table (X-RAY, COMMA, QUERY, "R" as decimal point) applied on send and reversed when reading, and parsing of received radiograms back into the form.
- **PKTNET Check-In composer (B/PKTNET)** — form-style check-in message type. (form concept: N3MEL, form created: KN4LQN)
- **NTS Delivered button** — green button on any T-prefixed message marks it delivered via BPQ's `WMNDel`, with delivered-state tracking, NTS sub-folders, and list pills.
- **Classic theme** — recreates the stock BPQ32 look: wheat/tan background, cream content surfaces, tan sidebar, white reader body.
- **Compose deep links** — `#compose?to=CALL&subject=...` opens a pre-filled compose window, so external dashboards can link straight into the webmail (documented in README-dashboard-links.md).
- Attribution/version line at the bottom of each form.

### Also includes (from v1.5.10)
- The LinBPQ kill-crash fix (request serializer + sequential bulk kill) and the "remember password" option — see the v1.5.10 notes below.

## v1.5.10 — 2026-07-07

### Fixed
- **LinBPQ crash (SIGSEGV) when killing held mail** — LinBPQ handles every HTTP request on its own thread over shared, unlocked WebMail state, so overlapping requests from this interface (bulk kill fired all its kill requests at once) could segfault the node. All requests now go through a single-flight queue so only one is ever in flight, and bulk kill sends one request at a time with a short gap and live progress on the confirm button. The underlying thread-safety bug is in LinBPQ itself and is being reported upstream. (reported on the bpq32 group)

### Added
- **"remember" checkbox next to BBS Password** (⚙ Settings) — when unchecked, the password is kept only for the current browser session and you are prompted for it again on the next visit; any previously saved password is removed from browser storage. Default is checked, so existing setups are unchanged. Useful for public-facing reverse-proxy installs. (credit: KB1TAE)
- **Compose deep links now work in already-open tabs** — the app reacts to `#compose?…` URL changes without needing a reload.

## v1.5.9 — 2026-04-29

### Added
- **HTTPS / reverse-proxy support** — works with Cloudflare Tunnel, nginx, or any TLS reverse proxy in front of BPQ. New protocol selector (auto / http / https) on the setup screen and in the settings bar, persisted as `bpq_proto`. Standard ports 80/443 are omitted from URLs; port 443 infers https. The topbar displays the full resolved base URL. (credit: Ben Kuhn, K5DAT Lee)

### Changed
- **Session-key detection overhaul** — extracts the WebMail session key from post-login redirect URLs and follows `<meta http-equiv="refresh">` redirects that `fetch()` silently ignores. Traverses WebMail entry-point links on the node index page (with off-origin guard). Retries the original URL after form login when no key is found in the response.
- **Login-page detection** rewritten as a case-insensitive regex — correctly matches BPQ's unquoted uppercase `TYPE=PASSWORD` attribute, which broke detection in earlier versions.
- **Connection error message** updated to explain the Cloudflare/proxy root cause and provide the `LOCALNET=127.0.0.0/8` BPQ config workaround for the underlying BPQ `HTTPcode.c` bug.

### Fixed
- Reverse-proxy/HTTPS access (K5DAT Lee's original report) — now confirmed working through HTTPS reverse proxies.

## v1.5.8 — 2026-04-18

### Fixed
- **Compose modal no longer overflows small laptop screens** — the modal now caps at 92% of viewport height and the body scrolls internally, so the TO field and Send button always remain visible regardless of screen size. Previously, on smaller laptop viewports the top (TO line) and bottom (Send button) could be clipped with no way to scroll. (credit: G7TAJ Steve)

## v1.5.7 — 2026-04-18

### Changed
- **Compose modal widened to 1040px** — more comfortable width for writing longer messages
- **Compose modal is now user-resizable** — drag the edge to resize; width persists across sessions via localStorage

### Fixed
- **Font filenames match fontsource.org download names** — drop the woff2 files into `fonts/` as-downloaded; no manual renaming required (credit: KC2NJV Wayne Spivak)
- **Compose/reply dialog no longer closes on backdrop click** — prevents accidental loss of draft content

## v1.5.6 — 2026-04-13

### Changed
- **Renamed "Personal" to "My Received"** — clearer label matching actual behavior (received messages only)
- **Reverted Personal/My Received to WMtoMe endpoint** (received only, not sent+received)
- **Restored TO-callsign filter** for My Received
- **Removed redundant My Received folder** (was identical to Personal)
- **Reordered sidebar** — My Received, Bulletins, NTS Traffic, Mine, My Sent, All Messages

### Added
- **Unread badges on sidebar folders** — blue for My Received, green for Bulletins, purple for NTS Traffic — updated on load and every auto-refresh

## v1.5.5 — 2026-04-13

### Changed
- **Clarified WebMail login credentials** — all labels and hints now specify "BBS Callsign" and "BBS Password" instead of "BPQ Username / Password", making it clear that WebMail login uses your BBS callsign and password — not the sysop credentials from your BPQ config `USER=` lines (credit: John Wiseman G8BPQ for clarifying BPQ authentication)
- **Removed debug console.log statements** from `bpqFormLogin()`
- **Updated README and mobile mockup** to match v1.5.4–v1.5.5 changes (credit: K5DAT Lee)

## v1.5.4 — 2026-04-13

### Changed
- **Clarified WebMail login credentials** — all labels and hints now specify "BBS Callsign" and "BBS Password" instead of "BPQ Username / Password", making it clear that WebMail login uses your BBS callsign and password — not the sysop credentials from your BPQ config `USER=` lines (credit: John Wiseman G8BPQ for clarifying BPQ authentication)
- **Removed debug console.log statements** from `bpqFormLogin()`

## v1.5.3 — 2026-04-11

### Fixed
- **Login page detection handles quoted attributes** — `isLoginPage()` now matches `type="password"` and `type='password'` in addition to unquoted, fixing detection on BPQ nodes that emit quoted HTML (credit: N3MEL Glenn, KC2NJV Wayne)
- **Login not silently skipped when credentials missing** — if BPQ returns a login page but no username/password is configured, the app now throws a clear error directing users to Settings instead of silently failing
- **Improved connection error message** — the "could not connect" screen now walks users through three common scenarios: login required, remote access misconfiguration, and localhost/HTTPPORT issues

## v1.5.2 — 2026-04-11

### Fixed
- **Host/port auto-detect no longer overwrites user settings** — `detectHostFromUrl()` was running on every page load and silently overwriting any manually configured host/port. Now only applies on first run when no saved config exists.

## v1.5.1 — 2026-04-11

### Added
- **Line spacing setting** — cycle between Compact, Normal, and Relaxed line height in the message reader. Available in the topbar and mobile settings quick actions. Preference saved across sessions. (credit: HB9DHG)
- **Auto-detect host and port from URL** — when served from the BPQ HTML folder, host and port are automatically read from the browser URL on every page load. No more stale port settings. (credit: K5DAT Lee)
- Setup screen port hint clarifying HTTPPORT vs telnet port

### Fixed
- **Duplicate "ALL Bulletins" subfilter** — when a bulletin's TO field was literally "ALL", it appeared twice in the sidebar (credit: HB9DHG)
- **Settings bar stays open after Apply** — clicking Apply now automatically closes the settings panel (credit: HB9DHG)
- **Script crash on load** — duplicate `isMobile` declaration (function vs let) caused a SyntaxError that prevented the entire page from loading

### Changed
- **Mobile settings is now a full-screen overlay** — tapping the gear icon opens a dedicated settings screen with close button, version/callsign/status info, and quick-action buttons (Refresh, Theme, Font, Spacing, Rules)
- Mobile topbar simplified — only the logo and gear button are visible; all other controls moved into the settings overlay
- Settings button in topbar now labeled "⚙ Settings" and highlighted in amber for visibility (credit: K5DAT Lee)
- Default message reader line height changed from 1.8 to 1.6 (Normal)

## v1.5.0 — 2026-04-09

### Added
- **Mobile responsive layout** — the same single HTML file now adapts automatically to phones and tablets (breakpoint at 768px)
- Bottom navigation bar on mobile with Folders, Messages, and Settings tabs
- Full-screen stacked views: Folders → Message List → Message Reader with back navigation
- Floating compose button (FAB) on mobile replaces sidebar compose button
- Mobile back button in message reader header to return to message list
- Unread message count badge on mobile Messages tab
- Touch-friendly sizing — larger tap targets for folders, subfilters, and message rows
- Compose modal goes full-screen on mobile for easier typing
- All four themes (Dark, Dark Hi-Contrast, Light, Light Hi-Contrast) fully supported on mobile
- Config bar stacks vertically on mobile for usability
- Safe area inset support for phones with notches/dynamic islands

### Changed
- On mobile, opening a folder navigates to the message list instead of auto-opening the first message
- On mobile, kill with no next message returns to message list view
- Subfilter clicks on mobile navigate to message list automatically

## v1.4.0 — 2026-04-05

### Added
- Form-based login support — when BPQ has `USER=` password set and serves an HTML login page (common for remote/non-local access), the app now detects the login form automatically, submits credentials, and extracts the session key. This fixes the "Could not detect session key" error for remote users.
- Improved error message when key detection fails — now suggests checking username/password in settings
- All fetch calls now consistently include auth headers (fixes remote access for header fetch and silent refresh)

## v1.3.1 — 2026-04-05

### Fixed
- Reject filter now works — was silently failing because the WebMail session key is not valid for the Mail config page. Fixed by establishing a proper Mail management session via `/Mail/Header`, using CRLF line endings for textarea fields, and reading textarea content correctly from the parsed DOM.

## v1.3.0 — 2026-04-04

### Added
- Reject filter — block unwanted messages by FROM callsign or TO category directly from the message reader. Adds entries to BPQ's native Reject From / Reject To config fields.
- Larger compose textarea for new messages
- Shift-click range selection for multi-select

## v1.2.5 — 2026-04-01

### Fixed
- Auth URL fix — removed credentials from base URL, rely solely on Authorization header for Basic Auth (credit: N3MEL)
- Added auth method comment documenting Basic Auth and future form-based login support

### Added
- Column widths now persist across sessions and page reloads (credit: HB9DHG)

## v1.2.4 — 2026-03-30

### Fixed
- Sidebar subfilter scrolling for long bulletin/personal lists (credit: HB9DHG)

### Added
- QTH as a user setting in setup screen and ⚙ Settings

## v1.2.3 — 2026-03-30

### Added
- HTTP Basic Auth support — username/password fields in setup screen and settings bar
- `parseHostInput` helper — host field now accepts `192.168.1.1:8008` or `http://192.168.1.1` formats
- Authorization header sent with all BPQ fetch requests when credentials are configured

## v1.2.2 — 2026-03-29

### Fixed
- LinBPQ crash — header fetch now sequential (1 at a time, 600ms delay, max 50 messages)
- Personal folder mixing bulletin types on LinBPQ — added P-type safety filter
- README release download links had wrong repo name casing
- README credits had wrong node callsign (WB4MOZ → N4SFL)

### Added
- Four color themes: Dark, Dark Hi-Contrast, Light, Light Hi-Contrast
- Theme button cycles through all four, preference saved across sessions
- High-contrast themes fix pale grey-on-grey readability issue reported by users

## v1.2.1 — 2026-03-28

### Fixed
- Send crash — switched to native BPQ send window (POST endpoint unreliable on LinBPQ)
- Session key 404 retry in background header fetch
- Settings button renamed to "⚙ Settings" for discoverability

## v1.2.0 — 2026-03-28

### Added
- First public release
- Three-pane layout, light/dark themes, adjustable font size
- Personal inbox with FROM subfilters, Bulletins with TO subfilters
- Star rules with built-in New User notification starring
- Multi-select bulk kill, auto-advance after kill
- Auto-refresh with session key recovery
- First-run setup screen
