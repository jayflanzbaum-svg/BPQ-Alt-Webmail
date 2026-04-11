# Changelog

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
