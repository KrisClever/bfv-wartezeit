# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A static web application for **Post SV Nürnberg e.V.** (club #6256) with two tools for football player administration:

- **`index.html`** — Passantrag-Generator: fills PDF pass request forms and generates WhatsApp forwarding messages
- **`wartezeit.html`** — Wartezeit-Rechner: calculates player transfer eligibility (Privatspielrecht / Verbandsspielrecht) based on BFV rules

## Running the App

No build step required. Open files directly in a browser or serve with any static file server:

```bash
python3 -m http.server 8080
# then open http://localhost:8080
```

There are no npm dependencies, no package.json, no build tools, no tests, and no linting configuration.

## Architecture

Both pages are self-contained single HTML files with embedded CSS and JavaScript. There is no shared JS module — common patterns are duplicated across files.

**PDF handling** uses `pdf-lib` v1.17.1 loaded via CDN. Templates (`mitgliedsantrag.pdf`, `passantrag_jugend.pdf`, `passantrag_senioren.pdf`) are fetched at runtime, filled programmatically, and merged into a single download.

**State** is managed through a plain object (`state = { ... }`) local to each page's IIFE.

**Date picker** is a custom scroll-snap wheel component (iOS-style) used on both pages for birth date and deregistration date input.

## Key Domain Logic

### index.html — Passantrag

- Age category (Jugend vs. Senioren) is determined from birth year at runtime to select the correct PDF template
- Two modes: *Ausfüllen* (player fills form) and *Weiterleiten* (manager forwards blank forms via WhatsApp)
- Nationality dropdown contains ~567 entries mapped from display names to PDF field values
- PDF fields are mapped by name using `pdf-lib`'s `getForm().getField()` / `getTextField()` / `getRadioGroup()` / `getCheckBox()` APIs
- The final PDF combines the pass form with the membership form (`mitgliedsantrag.pdf`) appended

### wartezeit.html — Wartezeit-Rechner

- BFV season runs **1 July – 30 June**; transfer windows are *Wechselperiode I* (summer) and *Wechselperiode II* (winter)
- Age categories follow BFV gender+birth-year rules (A/B/C/D-Jugend, Herren, Frauen, etc.)
- `calcJ()` handles youth (Jugend) eligibility; `calcH()` handles adult (Herren/Frauen) eligibility
- Key inputs: birth date, gender, Abmeldedatum (deregistration date), letztes Pflichtspiel (last competitive match), and release status (Freigabe / Einverständnis)
- Output is a color-coded result banner (green/yellow/red) with detailed legal notes referencing specific BFV § rules

## Conventions

- **Language:** All UI text, comments, and variable names are in German
- **Versioning:** Version string and date are hardcoded in the HTML header (e.g. `v2.12.0 · 05.06.2026`)
- **Styling:** Uses CSS custom properties for the color scheme; primary brand color is `#0062AD`
- **Mobile-first:** Viewport is locked (`user-scalable=no`), safe-area insets used, touch targets sized for mobile
- **No external state or backend:** Everything runs client-side; no API calls, no localStorage persistence
