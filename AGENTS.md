# AGENTS.md

Single-file browser prototype for **Foldable Reader**: a text reader where passages can be collapsed into alternative "foldable" phrases, and those phrases themselves fold into higher-level ones (paragraph → section → chapter → whole book).

## Run / verify

- No package manager, build step, bundler, tests, lint, or formatter. Do not add `npm`/`package.json` workflows or claim they exist.
- `public/index.html` is the whole app. Open it directly in a browser, or serve the `public/` directory statically (`python3 -m http.server --directory public`) if a feature needs `http://` (e.g. `fetch` of a local text file).
- Requires network access at runtime: Vue 3 and Matcha.css load from CDN.
- No automated tests: verify in a real browser. For Playwright, drive the hidden input with `page.setInputFiles('input.file-input', path)` (clicking the button opens the native chooser); `page.goto('file://...')` inside `browser_run_code_unsafe` verifies the no-server case, since the navigation tool blocks `file:`.

## Implementation constraints

- Everything lives in `public/index.html`: markup in `#app`, styles in the inline `<style>`, logic in the inline `<script>`. New CSS/JS stays there unless the user asks to split files.
- `public/` is the **deploy root and must contain only static, browser-ready assets** (no build step, no sources to compile). Keep `AGENTS.md`, `README.md`, and `LESSONS.md` at the repo root, outside `public/`, so they are never published.
- Vue 3 **global build** only. No SFCs, no `import`/`export`, no `<script setup>`. Use the destructured globals (`const { createApp, ref, computed } = Vue;`).
- New dependencies must be CDN `<script>`/`<link>` tags in `<head>`/before the app script — there is no dependency installation path.
- Matcha.css styles `input[type=file]` and overrides the `hidden` attribute; hide non-visual inputs with a class (`.file-input { display: none }`) instead.
- UI language is Spanish (`<html lang="es">`); keep user-facing copy in Spanish.

## Domain model (from `README.md`)

- The original text and its alternative phrases live in **separate** data: an auxiliary file (JSON) that the reader consolidates with the source text to produce the foldable view. Editing alternate phrases must never mutate the source text.
- Input sources planned: pasted text, `.txt`, `.json`, `.epub`, `.pdf`, or a URL.
- Folding is recursive: any rendered level can be replaced by its alternative phrase, with the original expandable underneath.

## Working style

- The project advances **incrementally**, simplest thing first, one small verifiable step at a time (Spec Driven Development). Avoid speculative abstraction, frameworks, or scaffolding ahead of need.
- Specs live in `specs/NNN-slug.md`, written in Spanish, and follow `.agents/skills/spec/template.md` (header blockquote, Scope, Data model, Implementation plan, Acceptance criteria, Decisions, optional Risks, “not in”).
- Spec states are `Borrador` → `Aprobado` → `Implementado`. **Only a human marks a spec `Aprobado`.** `/spec-impl` refuses anything else, so never claim approval or implement an unapproved spec.
- Skills are vendored in `.agents/skills/` (`spec`, `spec-impl`) and pinned by `skills-lock.json`; branch behavior comes from `specs/.spec-config.yml` (`AutoCreateBranch`, default `true`).
- `LESSONS.md` is a required, agent-maintained deliverable written **for the developer** in Spanish: record implementation ideas, strategies, and lessons learned as features land. Keep it current with each feature.
