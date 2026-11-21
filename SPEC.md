# Misc Utils: specification

This document describes the app in enough detail to rebuild it from scratch, or something very close to it. It always describes the current version. Whenever `index.html` changes, this file changes with it (see the rules in `CLAUDE.md`).

## 1. Overview

- A personal collection of everyday utilities in one web app, with one utility per page. Pages are switched with tabs.
- The whole app is a single static file, `index.html`, with all HTML, CSS and JavaScript inline.
- There is no backend, database, user login or build step, and no external dependencies (no frameworks, CDNs or web fonts). The app works offline, opened directly from disk (`file://`) or served from any static host.
- Target browsers: current Chrome, Safari, Firefox and Edge, on desktop and mobile. Plain modern JavaScript (ES2015+) is fine.

## 2. Structure of `index.html`

In order:

1. `<head>`: `charset` utf-8, viewport `width=device-width, initial-scale=1`, `<title>Misc Utils</title>`, and one `<style>` block with two parts: shared styles, then page-specific styles.
2. `<header class="app-header">` containing a `.container` with the app title (`<h1 class="app-title">Misc Utils</h1>`) and an empty `<nav class="tabs" id="tabs" aria-label="Pages">` that the shared script fills.
3. `<main class="container">` containing the pages, one `<section class="page">` each.
4. The shared navigation `<script>`.
5. Page scripts, one `<script>` element per page that needs one.

## 3. Page system

### 3.1 Page markup

Each page is a `<section>` that is a direct child of `<main>`:

```html
<section class="page" id="unit-converter" data-title="Unit converter">
  <h2>Unit converter</h2>
  ...
</section>
```

- `id`: a slug of lowercase letters, digits and dashes, unique in the file. It is the page's address in the URL.
- `data-title`: the tab label, also used in the browser tab title. If it is missing, the id is used instead.
- The content starts with an `<h2>` holding the page name.
- Tabs appear in the same order as the sections in the file. The first section is the default page.

### 3.2 Navigation

The shared script, wrapped in an IIFE, runs at the end of `<body>`:

1. It collects every `main > section.page`. If there are none, it stops.
2. For each page, it appends `<a href="#/<id>">`, with the page's title as its text, to `<nav id="tabs">`.
3. It shows the page named in the URL, and does so again on every `hashchange` event (tab clicks, back/forward, a typed URL).

Showing a page:
- The id is read from the hash by stripping a leading `#` or `#/`. Both `#/page-2` and `#page-2` work, but links always use `#/<id>`.
- If there is no hash, or no page has that id, the first page is shown. The URL is not rewritten.
- The shown section gets the class `is-active`. CSS hides every other page with `.page:not(.is-active) { display: none !important; }`. The `!important` stops a page's own `display` rule from un-hiding it.
- The matching tab link gets `aria-current="page"`, and the attribute is removed from the other links.
- The document title becomes `<page title> · Misc Utils`.

Links use `#/<id>` rather than `#<id>` because a fragment that matches an element id makes the browser scroll to that element. With the `/`, no element matches.

### 3.3 Adding a page

1. Copy an existing `<section class="page">` block and put it where its tab should appear.
2. Give it a new `id` and `data-title`, and update its `<h2>`.
3. Add any page-specific CSS in the "Page-specific styles" part of the `<style>` block, with every rule scoped under the page's id (for example `#unit-converter .result { ... }`).
4. Add any page-specific JavaScript in a new `<script>` element after the shared script, wrapped in `(() => { ... })();`. This keeps its variables private, and an error in it can't break other pages or the tab bar.
5. Document the page in section 5 of this file and in `README.md`.

The shared script needs no changes.

## 4. Visual style

- Font: `system-ui, -apple-system, "Segoe UI", Roboto, sans-serif`, line height 1.5. No web fonts.
- Light and dark themes follow the operating system setting through `prefers-color-scheme`. `:root` sets `color-scheme: light dark` so built-in form controls match the theme.
- Colors are CSS custom properties on `:root`, overridden inside a `prefers-color-scheme: dark` media query:

| Variable | Light | Dark | Used for |
|---|---|---|---|
| `--bg` | `#f6f7f9` | `#14171c` | Page background |
| `--surface` | `#ffffff` | `#1d2128` | Header and page cards |
| `--text` | `#1d2330` | `#e6e9ee` | Body text |
| `--muted` | `#5f6b7a` | `#9aa4b2` | Secondary text, placeholders |
| `--border` | `#dde1e7` | `#2e343d` | Borders |
| `--accent` | `#2f6fde` | `#5b8ef0` | Active tab, hover borders |
| `--accent-text` | `#ffffff` | `#0d1117` | Text on accent backgrounds |

- `--radius: 10px` for card corners. `box-sizing: border-box` on all elements. `body` has no margin.
- `.container`: max width 960px, centered, 16px padding left and right. Used by the header's inner wrapper and by `<main>`.
- Header: `--surface` background with a 1px `--border` bottom border. The title is 1.25rem with no margin, 12px padding above and 8px below. The tab bar is a wrapping flex row with 8px gaps and 12px padding below.
- Tabs: pill-shaped links with 6px × 14px padding, a 1px `--border` border, 999px radius, `--text` color, no underline, and no line breaks inside a label. On hover, the border turns `--accent`. The active tab has an `--accent` background and border, and `--accent-text` text.
- `<main>`: 24px padding above and 48px below.
- Page card: `--surface` background, 1px `--border` border, `--radius` corners, 20px padding. The page's `<h2>` has no top margin.
- `.placeholder`: text in `--muted`, used for "nothing here yet" notes.

## 5. Pages

| Tab | id | Purpose | Content |
|---|---|---|---|
| Page 1 | `page-1` | Not decided yet | Placeholder: `<h2>Page 1</h2>` and a `.placeholder` paragraph, "Nothing here yet." |
| Page 2 | `page-2` | Not decided yet | Same placeholder, titled Page 2 |
| Page 3 | `page-3` | Not decided yet | Same placeholder, titled Page 3 |
