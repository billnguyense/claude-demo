# UOB IT Service Desk — Training Demo

A single self-contained HTML page ([index.html](index.html)) that mockups an internal
IT Support Service Desk, themed as "UOB Bank". It is a training/demo artifact only —
**not a real product, and it must not be presented as one.**

**Live demo:** https://billnguyense.github.io/claude-demo/

## Running it

Open the file directly — no server required:

```sh
open index.html      # macOS
```

It must work from `file://`. There is no build step, no framework, and no dependencies.

## What's inside

One file, organised as four sections — Header/Hero, Ticket Form, FAQ, Footer — followed by a
single IIFE `<script>`. Features:

- A ticket submission form with data-driven, client-side validation
- A generated ticket reference (`UOB-ITSD-YYYYMMDD-####`) and priority-based SLA on success
- A searchable, one-at-a-time FAQ accordion
- An advisory heuristic that warns (but never blocks) if a password-like string is typed into the
  Description field

## Deliberate constraints

These are security/privacy properties of the demo. Breaking any of them is a bug:

- **No network**: no `fetch`/XHR/beacon/WebSocket, no external scripts, styles, fonts, or images.
  Enforced by a `Content-Security-Policy` `<meta>` tag (`default-src 'none'`).
- **No persistence**: no `localStorage`, `sessionStorage`, cookies, or IndexedDB. Submitted
  tickets live only in an in-memory array and vanish on reload.
- **No `innerHTML` with dynamic data**: all runtime text goes through `textContent` / DOM nodes
  (XSS guard).
- **No credential capture**: no fields for passwords, OTPs, PINs, or card numbers.
- **Keep the disclaimer** in the footer and placeholder values (e.g. the `1800-XXX-XXXX` hotline).

## Checks

No test suite exists. For an optional JS syntax check (requires Node), extract the `<script>`
block and run `node --check` on it.

---

Training demo — not affiliated with or endorsed by United Overseas Bank Limited.
