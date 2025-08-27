# AGENTS.md — Coding agents guide for Tab Wrangler

## Project overview

- This repository contains a Chrome & Firefox extension that automatically closes idle tabs and keeps a "Tab Corral" for easy restore.
- Public store listings: Chrome Web Store and Firefox Add-ons pages reference the same feature set and privacy constraints.
- Do **not** add analytics, tracking, or remote calls. The extension must not read page content, per privacy policy and current permissions model.

## Setup & dev commands

- Prerequisites: Node 18+ and npm.
- Install dependencies:

  ```bash
  npm ci

  ```

  - Start a fast rebuild/dev mode (watch):

  ```bash
  npm run dev
  ```

- Build a production bundle to `dist/`:

  ```bash
  npm run build
  ```

## Run locally (Chrome, Firefox)

- Chrome: `chrome://extensions` → Enable "Developer mode" → "Load unpacked" → select `dist/`.
- Firefox (temporary add-on): `about:debugging#/runtime/this-firefox` → "Load Temporary Add-on..." → pick the built `manifest.json` from `dist/`.

## Tests & quality gates

- Unit tests (Jest):

  ```bash
  npm test
  ```

- Lint & format (ESLint + Prettier):

  ```bash
  npm run lint
  npm run format
  ```

- Agents: when modifying code, **add/update tests** and ensure `npm run build` passes before proposing changes.

## Code style & structure

- Language: TypeScript (strict), SCSS modules for styles when applicable.
- Entry points live under `app/` and are bundled by webpack into `dist/`.
- i18n: strings live under `_locales/<lang>/messages.json`. When adding UI text, reference message keys; do **not** hardcode.

## Browser permissions & invariants (must-keep)

- Manifest permissions currently include: `"alarms"`, `"contextMenus"`, `"sessions"`, `"storage"`, `"tabs"`.
- MUST NOT request host permissions or content-script access to page bodies.
- Keep behaviors:

  - Do not close pinned tabs.
  - Support "Tab Lock", "Exclude list / whitelist", "Min tabs", "Idle timeout".
  - Preserve "Tab Corral" semantics and export/import format.

## Backups / data format (for reference)

- Exported backup JSON structure:

  ```ts
  type TabWranglerExportFormat = {
    savedTabs: Array<chrome.tabs.Tab>;
    totalTabsRemoved: number;
    totalTabsUnwrangled: number;
    totalTabsWrangled: number;
  };
  ```

- Agents: changing this schema is a breaking change; avoid unless explicitly requested.

## PR guidance for agents

- Keep PRs small and focused; include tests and i18n updates.
- Do not introduce telemetry or network calls.
- Validate on both Chrome (MV?) and Firefox runs.
- Run:

  ```bash
  npm ci && npm run lint && npm test && npm run build
  ```

## Release checklist (for maintainers/agents)

- [ ] All tests pass, zero ESLint errors.
- [ ] Manual verification: idle close, pinned-tab protection, lock/unlock, whitelist, corral restore including history where applicable.
- [ ] Build artifacts load in Chrome and Firefox without warnings.
