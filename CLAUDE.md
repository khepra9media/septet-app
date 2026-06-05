# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

**Septet** is a Kemetic Sidereal Calendar PWA by Khepra9Media. The entire application lives in a single file: `index.html`. There is no build system, no package manager, no test suite, and no CI/CD pipeline. Changes are made directly to `index.html` and deployed as static files.

## Development Workflow

Since there is no build step, development is:
1. Edit `index.html` directly
2. Open it in a browser (or use a local static server: `python3 -m http.server 8080`)
3. Commit and push — the deployed site is the file itself

There are no lint, test, or build commands to run.

## Architecture

The app is structured as a single HTML file with three sections:

- **`<style>` (lines ~17–278)**: All CSS using custom properties. Key layout classes: `.hdr` (header), `.cal` (calendar grid), `.mv` (scroll view panels), `.gate` (paywall overlay), `.dns` (10-column dekan strip), `.linen` (parchment texture).
- **`<body>` (lines ~281–295)**: Minimal — a splash screen div, a root `<div id="root">`, and a toast area. All UI is generated via JavaScript.
- **`<script>` (lines ~296–1061)**: All application logic inline.

### State Management

Global state lives in a single object `S` that is persisted to `localStorage`. Key fields:
- `S.view` — current view (`cal`, `today`, `dreams`, `neteru`, `festivals`, `year`)
- `S.sel` — selected Gregorian date string
- `S.dreams` — array of dream log entries
- `S.month` / `S.year` — currently displayed month/year

Unlock status is stored separately under the key `septet_unlocked_v1`. Used gift codes are tracked under `septet_used_codes`.

### Calendar Engine

The Kemetic calendar uses a custom epoch (EP = 1600) anchored to the heliacal rising of Sirius (Septet). Key functions:
- `gKY(date)` / `gNY(date)` — convert Gregorian date to Kemetic year
- `gKD(date)` — full Kemetic date object from a Gregorian date
- `mS(m, y)` / `mE(m, y)` — month start/end Gregorian dates
- `eS(y)` / `eE(y)` — Epagomenal (Nwt) period start/end dates
- `dayType(date)` — classify a day (holy/divine/rest/regular)
- `meduNum(n)` — render a number as ancient Egyptian Medu numerals

### Data Structures

All calendar data is declared as top-level constants:
- `DEKAN_DAYS` — 10 dekan periods with names and Neteru associations
- `MO` — 12 months with deity names, descriptions, sacred animals, dekan mappings
- `SS` — 3 seasons (Akhet, Peret, Shemu) with date ranges
- `NWT_EV` — 5 intercalary day events (births of deities)
- `FE` — festival/holy day map keyed by `MM-DD` date strings

### Rendering

All UI is generated imperatively via `render()`, which builds an HTML string and sets `document.getElementById('root').innerHTML`. There is no virtual DOM or framework. Views are switched by `sv(viewName)` which updates `S.view` and calls `render()`.

### Monetization

Premium features are gated by `isUnlocked()`, which checks `localStorage`. Unlock paths:
1. **Stripe payment** — client-side Stripe redirect, success sets unlock via URL param
2. **Gift codes** — base64-encoded codes validated by `validateGiftCode()` / `redeemGiftCode()`; founding member and single-use variants stored in `GIFT_FOUNDING` and `GIFT_SINGLE` constants

The `.gate` overlay and `.locked-card` elements render over premium content for non-subscribers. Premium views: dream logging, Neteru profiles, festivals/holy days, year view.

### PWA

A service worker is registered inline at the bottom of the script for cache-first offline support. `manifest.json` and `icon-192.png` / `icon-512.png` provide installability. The app is portrait-optimized with a 480px max-width container and safe-area insets for notched devices.

## Conventions

- **No framework abstractions** — use direct DOM manipulation and string concatenation for HTML generation, consistent with the existing codebase.
- **CSS variables** — all colors and fonts are defined as custom properties on `:root`; use those rather than hardcoding values.
- **Mobile-first** — the app targets portrait phone screens; avoid layouts that assume wide viewports.
- **Stripe is client-side only** — there is no backend; payment success is handled via redirect URL parameters on the front end.
