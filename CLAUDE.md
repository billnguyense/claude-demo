# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single self-contained page: [index.html](index.html) — a training/demo mockup of an internal
IT Support Service Desk, themed as "UOB Bank". It is not a real product and must not be presented
as one. There is no backend, no framework, no build step, and no dependencies.

## Commands

- **Run / preview**: open the file directly — `open index.html` (macOS). It must work from
  `file://` with no server.
- **Build / lint / test**: none exist. There is no package manager, no toolchain, no test suite.
- **JS syntax check** (optional, requires Node): extract the `<script>` block and run
  `node --check` on it.

## Hard constraints (do not regress these)

These are deliberate security/privacy properties of the demo. A change that breaks any of them is
a bug, even if it "works":

- **No network**: no `fetch`/`XHR`/`sendBeacon`/WebSocket, no `<script src>`/`<link href>`, no CDN,
  no web fonts, no remote images. The page is enforced by a `Content-Security-Policy` `<meta>` tag
  (`default-src 'none'`, `form-action 'none'`, `frame-ancestors 'none'`); keep it intact and
  CSP-clean (no inline event handlers — use `addEventListener`).
- **No persistence**: no `localStorage`, `sessionStorage`, cookies, or IndexedDB. Submitted tickets
  live only in the in-memory `submittedTickets` array and are meant to vanish on reload.
- **No `innerHTML` with dynamic/user data**: all runtime-set text goes through `textContent` or
  DOM node creation. This is the XSS guard.
- **No credential capture**: never add fields for passwords, OTPs, PINs, or card numbers. The
  Description field's credential heuristic (`looksLikeCredential`) is intentionally advisory —
  it warns but never blocks submit.
- **Keep the disclaimer** in the footer (column + bottom bar): "Training demo — not affiliated
  with or endorsed by United Overseas Bank Limited." Keep placeholder values like the
  `1800-XXX-XXXX` hotline as placeholders.

## Architecture

`index.html` is organised as four commented sections — Header/Hero, Ticket Form, FAQ, Footer —
followed by one IIFE `<script>`. Design tokens are CSS custom properties on `:root`
(`--blue #00539B`, `--amber #E8A33D` accent-only/non-text, `--ink #2C3038`); layout is a 1120px
container with grids that collapse to one column at `max-width: 768px`.

Key script structures:

- **Form validation** is data-driven: a `validators` map (name → function returning `''` or an
  error string) plus a `FIELD_ORDER` array. `validateField` runs one; submit runs all in order,
  tracks the first failure, and moves focus there. Errors render into `#err-<name>` nodes and set
  `aria-invalid` on the control (`#priorityGroup` fieldset for the radio group). Blur/change
  listeners are wired from `FIELD_ORDER`, so **adding a field means: add the control + its
  `#err-<name>` node + a `validators` entry + a `FIELD_ORDER` entry.**
- **Ticket reference** `UOB-ITSD-YYYYMMDD-####` is built from local date + `crypto.getRandomValues`
  (a display string, not a security token). `SLA` map translates priority → response time for the
  success panel. On success the `<form>` is hidden and `#successPanel` (a sibling, outside the
  form) is shown; "Submit another ticket" does a full reset including clearing error/counter/note
  state.
- **FAQ accordion**: custom `<button aria-expanded>` triggers, one-open-at-a-time enforced in JS
  by clearing `.open` on all items before opening the clicked one; chevron rotates via CSS,
  panels animate `max-height` (disabled under `prefers-reduced-motion`).
- **FAQ search** filters items by `textContent` substring match and toggles `#faqEmpty`.

## Other agent configs

An OpenAI Codex config exists at `~/.codex/config.toml`. To import shareable items (MCP servers,
slash commands, subagents, skills, instructions), reply `/import` to scan and list what's
importable, then `/import --yes=<digest>` to apply. (If `/import` isn't available here, run
`claude import` from a terminal.)
