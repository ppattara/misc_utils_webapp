# misc_utils_webapp

A web app that collects miscellaneous utilities for everyday use. It is a static HTML file, with no backend, database or user login.

## Project files
- `index.html`: the app itself.
- `CLAUDE.md`: ground rules and working notes for Claude (this file).
- `SPEC.md`: the full description of the app, detailed enough to rebuild it (or something very close) from that file alone.
- `README.md`: the project summary, which doubles as the user manual.

## Ground rules (always follow)

### 1. No local memory: everything lives in the repo
This project is worked on from multiple machines through a GitHub repo, so anything a future session needs to know must be in a file in this repo.
- Never use Claude Code's per-machine memory (`~/.claude/projects/<project>/memory/`), `~/.claude/CLAUDE.md`, or `CLAUDE.local.md` for this project.
- Record decisions, requirements, conventions, progress and open TODOs in this file, or in a repo file linked from here.

### 2. No git commands
The owner is the only one who interacts with git.
- Never run `git` in any form, including read-only commands like `git status`, `git diff` or `git log`. The same goes for `gh`.
- Watch for tools that run git under the hood: scaffolders that call `git init` (use their option to skip it), `npm version`, git-hook installers like husky, and Claude Code features built on git (worktrees, branch/diff-based reviews).
- When a change is ready, say so. Committing and pushing are up to the owner.

### 3. Keep the app and its docs in sync
Every change to the webapp (a new feature, changed behavior, a removal, a UI change) must update the docs in the same piece of work:
- `SPEC.md`, so it still describes the current app in enough detail to rebuild it from that file alone.
- `README.md`, so the summary and user manual match what the app does now.
- `CLAUDE.md`, if the change affects conventions, decisions or the status notes below.

A change isn't finished until the code and all three files agree. When reporting a change, list which of these files were updated. If the code and docs are found to disagree (for example after a manual edit), point it out and ask the owner which is correct before changing either.

## Working on the app
- Read `SPEC.md` before changing the app. It covers the structure, conventions and constraints (one self-contained file, no external dependencies), including how to add a page.
- To check a change, open `index.html` in a browser. There is no server or build step.

## Decisions
- **Saved settings** (owner, 2026-11-22): pages may remember their settings in the browser's `localStorage` only, never in files, a database or an account. If the browser's data is cleared, settings simply return to their defaults. Keep this simple. The convention is in `SPEC.md` section 3.4.
- **Browser tab title** (owner, 2026-11-22): the title is normally the page name, but while a Work Timer countdown runs it shows the time left instead (`SPEC.md` 3.2 and 5.1).
- **Floating window** (owner, 2026-11-22): ordinary browser windows can't be made narrower than about 400–500 px, and a page can't change that. So that the Work Timer can sit small in a corner of the screen, it can pop out into an always-on-top floating window using the Document Picture-in-Picture API (desktop Chrome, Edge and Firefox 151+; not Safari or phones). The owner tried a prototype in Firefox 160 before it was built in. Details in `SPEC.md` 5.1.

## Status
- v1: blank shell with three placeholder pages (Page 1, Page 2, Page 3) and the shared tab navigation.
- v2 (2026-11-22): the first page is now the Work Timer (`#/work-timer`, replacing Page 1). It has work and rest countdowns, random reminder dings during work, a settings sidebar with saved settings and sound previews, the countdown in the browser tab title, and a keep-screen-on button. Pages 2 and 3 are still placeholders.
- v3 (2026-11-22): the Work Timer can pop out into a floating window (the button next to the gear). The window adapts to its size, down to a one-line strip with the time and a single button.
- Next: the owner will describe the utilities for Pages 2 and 3.
