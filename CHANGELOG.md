# Changelog

## v1.5.2 — 2026-04-10

### Added
- **Auto-detect host and port from URL** — when served from the BPQ HTML folder, the setup screen pre-populates host and port from the browser URL. `completeSetup()` also uses detected values if the host field is empty or still the default. (credit: K5DAT Lee)
- Setup screen port hint clarifying HTTPPORT vs telnet port

### Changed
- Default port changed from 8010 to 8080 — 8010 is the telnet port, not the HTTP port (credit: K5DAT Lee)
- Settings button in topbar now labeled "⚙ Settings" and highlighted in amber for visibility (credit: K5DAT Lee)

## v1.5.1 — 2026-04-10

### Changed
- **Mobile settings is now a full-screen overlay** — tapping the gear icon opens a dedicated settings screen instead of toggling the config bar inline. Includes a close button header, version/callsign/status info row, and quick-action buttons (Refresh, Theme, Font, Rules) so all controls are accessible without the topbar.
- Mobile topbar simplified — only the logo and gear button are visible; version label, callsign, node, status pill, theme/font/rules buttons, and refresh label are all hidden (moved into the settings overlay).
- Mobile status pill in settings overlay stays in sync with the main status pill via MutationObserver.

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
