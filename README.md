# ABC IT Service Desk — Training Demo

A single self-contained HTML page ([index.html](index.html)) that mocks up an internal
IT Support Service Desk for a fictional "ABC" organisation. It is a training/demo
artifact only — **not a real product, not a real bank, and it must not be presented
as one.**

**Live demo:** https://billnguyense.github.io/claude-demo/

## Preview

![Screenshot of the ABC IT Service Desk training demo page](docs/screenshot.png)

## Running it

Open the file directly — no server required:

```sh
open index.html      # macOS
```

It must work from `file://`. There is no build step, no framework, and no dependencies.

## What's inside

One file: four commented regions — Header/Hero, Ticket Form, FAQ, Footer — followed by a
single IIFE `<script>`. Notable pieces:

- A **support-ticket form** with data-driven, client-side validation (a `validators` map +
  `FIELD_ORDER`), grouped into *About you / The issue / Details* fieldsets.
- A generated ticket reference (`ABC-ITSD-YYYYMMDD-####`) and a priority-based SLA shown on
  the success panel. The reference is a display string, not a security token.
- A searchable, one-at-a-time **FAQ index** (custom `aria-expanded` accordion).
- An advisory heuristic that **warns but never blocks** if a password- or card-like string
  is typed into the Description field.
- Motion that stays out of the way: a scroll-progress rail, a hero load sequence,
  `IntersectionObserver` section reveals, a pulsing status dot, and a live character meter —
  all disabled under `prefers-reduced-motion`.

## Deliberate constraints

These are security/privacy properties of the demo. Breaking any of them is a bug, even if
it "works":

- **No network**: no `fetch`/XHR/beacon/WebSocket, no external scripts, styles, fonts, or
  images, no CDN. Enforced by a `Content-Security-Policy` `<meta>` tag
  (`default-src 'none'`; `form-action 'none'`; `frame-ancestors 'none'`). No inline event
  handlers — listeners are wired with `addEventListener`.
- **No persistence**: no `localStorage`, `sessionStorage`, cookies, or IndexedDB. Submitted
  tickets live only in an in-memory array and vanish on reload.
- **No `innerHTML` with dynamic data**: all runtime-set text goes through `textContent` or
  DOM node creation (XSS guard).
- **No credential capture**: no fields for passwords, OTPs, PINs, or card numbers. The
  Description heuristic is advisory only.
- **Keep the disclaimer** in the footer, and keep placeholder values (e.g. the
  `1800-XXX-XXXX` hotline) as placeholders.

The page names the Anthropic/Claude typefaces first in its font stack with full system
fallbacks; it does **not** load any web font (`@font-face` / remote / data-URI), consistent
with the no-network rule.

## Deployment

[.github/workflows/deploy-pages.yml](.github/workflows/deploy-pages.yml) publishes
`index.html` to GitHub Pages on every push to `main` (and via manual `workflow_dispatch`).

## Checks

No test suite exists. For an optional JS syntax check (requires Node), extract the
`<script>` block and run `node --check` on it.

---

Training demo — not affiliated with, endorsed by, or connected to any real financial
institution.
