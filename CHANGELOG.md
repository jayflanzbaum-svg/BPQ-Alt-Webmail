# Changelog

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
