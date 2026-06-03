# wc26-predictions — agent operating notes

Score-prediction game for the 2026 FIFA World Cup. Built for Flutter UK&I colleagues. Single-page vanilla JS, Firebase Anonymous Auth + Firestore for shared state, ESPN unofficial API for fixtures / scores / odds. Hosted on Cloudflare Workers Assets.

## Read these first

1. `README.md` — what this is, how it works, file layout
2. `BACKLOG.md` — ranked priorities. Pick from the top of `Ranked next` unless directed otherwise. Add ideas to `Parked`, not the ranked list.
3. The PokerStars Fusion Design System spec (`~/Downloads/PokerStars-Fusion-Design-System.md` or ask the human for the current path). All visual changes go through this.

## File layout

- `index.html` — UI shell, CSS, all rendering JS, Firebase + ESPN logic. ~1700 lines, single file by design.
- `data.js` — config + reference data: Firebase web config, admin passcode, tournament window, country flag map, round definitions.
- `README.md` — public-facing project description.
- `BACKLOG.md` — ranked work, parked ideas, rolling done log.
- `wrangler.jsonc` — Cloudflare Workers Assets config.

## Hard rules

- **No em dashes anywhere.** Use commas, periods, colons, semicolons, parentheses. (Project author voice rule across the whole workspace.)
- **No decorative emoji in UI.** Country flag emoji are OK as functional fixture data (with `aria-hidden`). Status indicators, decorations: no emoji.
- **Use Fusion semantic tokens, not raw hex.** `var(--fill-interactive-primary-default)` not `#01866F`. Exception: ESPN team colours and other dynamic data are hex by definition.
- **Dark theme is canonical.** Don't add light-theme overrides unless explicitly requested.
- **WCAG AA on all text.** Body text 4.5:1, large text 3:1. Muted text on dark cards should use `--font-interactive-medium` (`#BFBFBF`), not `--font-interactive-weak` (`#757575`).
- **No build step.** The file is served directly. Don't introduce bundlers, frameworks, or compilers.
- **One commit per discrete change.** Commit messages are imperative, no em dashes. Two-line format: short title, blank line, longer explanation paragraph.

## Deploying

Push to `main` on GitHub (`JDirsFlutter/wc26-predictions`). Cloudflare auto-deploys in about 30 seconds. No CI to wait on.

## Verifying locally

```
python3 -m http.server 8124 --directory wc26-predictions
```

or use the `wc26` config in `~/.../Claude C JD/.claude/launch.json` via `mcp__Claude_Preview__preview_start`. Test desktop + mobile (`preview_resize preset: mobile`). Hard-refresh after deploy to bust cache.

## Important state and surfaces

- **Firebase project**: `world-cup-predictor-9a331`. One project for dev + prod (intentional, no migration needed).
- **Live URL**: `https://wc26-predictions.jamesdirs90.workers.dev/`
- **Admin route**: `https://wc26-predictions.jamesdirs90.workers.dev/#admin`. Passcode in `data.js` (`WC_ADMIN_PASSCODE`).
- **Firestore collections**: `participants/{name}`, `matches/{matchId}`, `predictions/{name_matchId}`. All publicly readable. Writes gated by anonymous auth UID.

## What NOT to do

- Don't migrate the Firebase project. Same project across all surfaces.
- Don't change the shape of `predictions/{predId}` without a migration plan. Real picks may be in there.
- Don't add a JS framework or build step.
- Don't introduce light-mode overrides unless explicitly requested.
- Don't show colleagues' emails anywhere (we don't collect them, but reinforce the rule).
- Don't anchor pick UI to odds without giving the user a way to turn the anchoring off (see Anchoring section in BACKLOG.md).

## When you're done with a task

- Commit and push directly to `main`. There's no review process for this project.
- Move the line item from `Ranked next` to `Done` in `BACKLOG.md` in the same commit.
- If you spotted something out of scope worth doing, append it to `Parked` rather than fixing inline.
