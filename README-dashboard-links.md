# Dashboard Deep-Links

BPQ-Alt-Webmail accepts URL-hash parameters that pre-fill the compose
modal. Drop one of these links onto your dashboard, station status
page, or any other tool and clicking it opens a pre-populated message
ready to send.

## URL format

```
<bpq-host>/bpq-alt-webmail.html#compose?<params>
```

The fragment must start with `#compose?` followed by standard
`&`-separated query params. The hash auto-clears after the modal
opens, so refreshing the page won't re-fire the compose window.

## Parameters

| Param | Maps to | Notes |
|-------|---------|-------|
| `to` | **To** field | Uppercased automatically. Accepts a callsign (`K1AJD`), area routing (`04543@NTSME`), or Winlink address (`N4SFL@winlink.org`). |
| `subject` | **Subject** field | Free text. |
| `body` | **Message** textarea | Free text. Use `%0A` for newlines. |
| `type` | **Type** dropdown | One of `P` / `B` / `T`. Anything else (or omitted) falls through to `P`. |

Form-style types (`T/RRI` radiogram, `B/PKTNET` check-in) are **not**
supported via the hash — those need structured field data, not a
single pre-filled body string. Use the form by hand for those.

All values **must be URL-encoded**:

| Char | Encoding |
|------|----------|
| space | `%20` (or `+`) |
| `@`  | `%40` (or leave as-is inside a fragment) |
| newline | `%0A` |
| `&` (inside a value) | `%26` |
| `=` (inside a value) | `%3D` |
| `#` (inside a value) | `%23` |

## Examples

### Simple personal message to a callsign

```
http://127.0.0.1:8010/bpq-alt-webmail.html#compose?to=K1AJD&subject=Test&body=Hi%20there&type=P
```

### Winlink address with multi-line body

```
http://127.0.0.1:8010/bpq-alt-webmail.html#compose?to=K1AJD@winlink.org&subject=Net%20reminder&body=See%20you%20at%2019%3A00.%0A%0A73%20de%20N4SFL&type=P
```

### NTS routing destination, traffic-type plain

```
http://127.0.0.1:8010/bpq-alt-webmail.html#compose?to=04543@NTSME&subject=DAMARISCOTTA%20KY2D&type=T
```

### Bulletin to ALL

```
http://127.0.0.1:8010/bpq-alt-webmail.html#compose?to=ALL&subject=Net%20tonight%2019Z&body=See%20you%20on%20146.52&type=B
```

## Dashboard integration

### HTML

```html
<a href="http://127.0.0.1:8010/bpq-alt-webmail.html#compose?to=K1AJD&subject=Test&body=Hello&type=P"
   target="_blank">📧 Email K1AJD</a>
```

### JavaScript (recommended — handles encoding for you)

```js
function openComposeLink({ to, subject, body, type = 'P' }) {
  const params = new URLSearchParams({ to, subject, body, type });
  const url = `http://127.0.0.1:8010/bpq-alt-webmail.html#compose?${params}`;
  window.open(url, '_blank');
}

openComposeLink({
  to:      'K1AJD@winlink.org',
  subject: 'Quick note',
  body:    'Hello\n\n73 de N4SFL',
  type:    'P',
});
```

`URLSearchParams` encodes everything correctly — including newlines
inside `body` — without you having to think about percent-escaping.

### Building links from a form

```html
<form onsubmit="event.preventDefault(); openComposeLink({
  to:      this.to.value,
  subject: this.subject.value,
  body:    this.body.value,
});">
  <input name="to"      placeholder="callsign or winlink">
  <input name="subject" placeholder="subject">
  <textarea name="body"></textarea>
  <button>Compose in BPQ-Alt-Webmail</button>
</form>
```

## Behavior notes

- **The hash auto-clears** after the modal opens (`history.replaceState`),
  so reloading the page won't pop the compose window a second time.
- **The compose modal is reset** to its default state on each open —
  including switching the type to `P` if a prior session left it on
  `T/RRI` or `B/PKTNET`. Your hash-supplied `type` overrides this.
- **The browser must support `URLSearchParams`** for the
  recommended JS helper. Every browser made this decade supports it.
- **Already-open tabs don't react** to a new URL with the same path
  and a different hash — the browser doesn't reload. The hash
  listener fires only on initial page `load`. If you need existing
  tabs to react too (e.g. dashboard always opens in the same tab),
  request a `hashchange` listener as a follow-up — small change.

## BPQ host URL

Replace `http://127.0.0.1:8010` in the examples with whatever URL
serves your BPQ HTTP port. Common variations:

- LAN: `http://192.168.1.50:8080/bpq-alt-webmail.html`
- Cloudflare tunnel: `https://bpq.example.com/bpq-alt-webmail.html`
- Reverse proxy: `https://example.com/bpq/bpq-alt-webmail.html`
