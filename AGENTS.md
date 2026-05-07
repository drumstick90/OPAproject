# OPA Workspace

An AI-powered, browser-based note-writing tool for a UK consultant psychiatrist. Used to draft Outpatient Assessment (OPA) notes faster, with optional AI auditing.

## First principles

Cleanliness, accuracy, efficiency, modularity. Lead with the simplest thing that works.

## Stack

Single self-contained HTML file. Vanilla JS. No build step, no bundler, no framework, no npm. Loads fonts from Google Fonts; everything else is local. State lives in `localStorage` under `opa.*` keys.

Do **not** introduce React, Vue, Tailwind, a bundler, TypeScript, or a backend. If the file gets too long, split CSS and JS into sibling files (`opa.css`, `opa.js`) — that's the only refactor on the table.

## File map

`opa.html` is the only product file. Inside it, named regions in this order:

1. `<style>` — CSS custom properties at top (`--accent`, `--r-card`, `--ink`, etc.), then chrome → toolbar → main → drawer → mobile polish.
2. `<body>` — head, toolbar, AI panel, main, scrim/drawer, toast.
3. `<script>` — clearly-bordered regions: `STATE & CONSTANTS`, `STORAGE`, `STATE`, `UTIL`, `RENDER`, `FIELD OPS`, `REDACT`, `ASSEMBLE NOTE`, `DRAWER`, `SETTINGS`, `FRASARIO`, `QUESTIONNAIRES`, `AI`, `COPY MARKDOWN`, `RESET`, `INIT`.

When working on a specific feature, find its region by the comment header.

## Aesthetic

Cobalt-inspired direction 01.

- Palette: warm light grey (`#f4f4f4` base, `#fafafa` surfaces, `#ffffff` insets), deep ink (`#19191c`), single cobalt accent (`#1d4ed8`).
- Type: IBM Plex Sans for body and writing surfaces, IBM Plex Mono for chrome (labels, buttons, monospace UI).
- Radii: plump (18px cards, 14px inputs, 999px pills).
- Lowercase chrome — applied via CSS `text-transform: lowercase` only. The data underneath stays canonical proper case.

If asked to change the visual direction, change the CSS custom properties at the top of `<style>`. Don't search-and-replace colors throughout the file.

## Critical conventions

**Data is canonical, display is transformed.** TEMPLATE section labels are stored in proper case (`'Mental State Examination'`). The UI shows them lowercase via CSS. The markdown export uses canonical proper case so pasted notes look professional. Never lowercase the data itself — break this and clinical exports look bad.

**Clinical content is sacred.** Default field text (e.g. `'WK, good eye contact, pleasant manners, good rapport.'`) is real clinical phrasing. Don't normalize, autocorrect, or "improve" it without being asked.

**Anything sent to OpenAI is redacted first.** The `REDACT` module strips NHS numbers, postcodes, emails, UK phone numbers, DOB labels, dates, honorific names, and addresses. Every AI call goes through `showRedactPreview()` — a split-view drawer that shows original vs redacted, user confirms before sending. Never bypass this gate.

**`localStorage` keys are namespaced.** All under `opa.*` (`opa.draft.v1`, `opa.settings.v1`, `opa.frasario.v1`, `opa.questionnaires.v1`, `opa.cache.v1`). Don't pollute the namespace.

**The OpenAI API key lives in the user's browser.** Settings → API key. Stored in `localStorage`. Never hardcode, never log, never send anywhere except `api.openai.com`.

## Features

- **Sectioned writing surface** — auto-resizing textareas following the OPA template, with smart defaults.
- **Frasario** — JSON-driven phrasebook. Click a phrase, it inserts at the cursor of the focused field.
- **Questionnaires** — registry of `{id, label, url}` opened in an iframe drawer. Pages must `postMessage({type: 'opa-result', prose: '...'}, '*')` on completion; the parent shows a section picker for insertion.
- **AI functions** — defined in `DEFAULT_SETTINGS.functions`. Each has a `label`, `target` (`'note' | 'section'`), `mode` (`'review' | 'apply'`), and `systemPrompt`. v1 ships only `check_validity`. The architecture supports adding more (e.g. `polish`, `expand_bullets`) without code changes — just a new entry in settings.
- **Per-section undo cache** — when an `apply`-mode AI function overwrites a field, the pre-AI value is cached. A revert button appears. Currently dormant since the only function is `review`-mode.
- **Markdown export** — assembles all sections into a markdown document, copy-to-clipboard.

## Working style

The user prefers:

- Direct, concise responses. No exhaustive multi-section breakdowns with pros/cons lists or historical precedents.
- Honest pushback. Don't capitulate to bad ideas just because the user asked. Maintain a position with reasons.
- Complete file outputs when asked for an update.
- Simplest technical solution that fits the stated goal — not the solution optimized for growth or distribution.

## Things to ask before doing

- Adding a new dependency or library
- Restructuring the file (splitting, modularizing)
- Changing the data model of TEMPLATE, settings, or storage keys
- Adding new AI providers (currently OpenAI only by deliberate choice)
- Sending data anywhere other than `api.openai.com`

Things you can just do: tweak CSS, add new sections to TEMPLATE, add new functions to settings, fix bugs, improve the REDACT regexes, add new questionnaires to the registry.
