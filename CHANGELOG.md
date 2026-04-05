# Changelog

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
