# World Cup 2026 Predictions

A score-prediction game for the 2026 FIFA World Cup. Built for Flutter UK&I colleagues. Pick exact scores for every match, climb the leaderboard.

Live at: TBD (Cloudflare Pages deploy pending).

## How it works

- **Add yourself** with a display name. No password, no email. Your browser remembers you (anonymous Firebase auth under the hood).
- **Predict** scores match by match. Picks are batched: the sticky bar at the bottom commits them.
- **Rounds unlock progressively**. Group Stage MD1 is open from launch. Each subsequent round opens 14 days before its first match (configurable in `data.js`).
- **Scoring**: 3 pts for an exact score, 1 pt for the correct result (W/D/L), 0 otherwise. Ties broken by exact-score count.
- **Live scores** auto-sync from ESPN's unofficial scoreboard API. Admin can manually override if ESPN is wrong.

## Stack

- Single-page vanilla JS, no build step
- [Firebase Auth (anonymous) + Firestore](https://firebase.google.com/) for identity and shared state
- [ESPN unofficial scoreboard endpoint](https://site.api.espn.com/apis/site/v2/sports/soccer/fifa.world/scoreboard) for fixtures and live results
- [Cloudflare Pages](https://pages.cloudflare.com/) for hosting (auto-deploy on push to main)
- Flutter UK&I brand tokens (Poppins, navy + accent blue, diagonal beam treatment)

## File layout

- `index.html` — the entire app: UI shell, CSS, all rendering JS, Firebase + ESPN logic
- `data.js` — config + reference data: Firebase web config, admin passcode, tournament window, country flag map, round definitions

## Updating

1. Edit `index.html` or `data.js`
2. Commit and push to `main`
3. Cloudflare Pages auto-builds and deploys in ~30 seconds

## Admin

The `#admin` URL hash opens a passcode-gated panel (passcode is set in `data.js`). From there:

- Sync fixtures from ESPN (one-shot, runs automatically on first visit too)
- Refresh today's scores
- Override final scores for any match (covers ESPN lag or errors)
- Manually enter knockout teams as the bracket fills in

## Switching devices

The anonymous auth ties a name to one browser. If a player wants to switch devices, an admin can update the `participants/{name}.uid` field in Firestore to the new browser's anonymous UID. Roughly 30 seconds.

## Firestore security rules

See `firestore.rules` in the Firebase Console. Summary:

- `matches`, `participants`, `predictions`: publicly readable (display names are first names only, scores aren't sensitive)
- Writes to `participants` and `predictions` are gated to the anonymous UID that owns the participant doc, so only you can edit your own picks

## Privacy

- Page is served with `noindex, nofollow` so it doesn't surface in search
- No emails or PII collected
- Display names are first names by convention (the form accepts any letters/spaces/hyphens, kept short)

## Known limitations

- Knockout-stage team assignments come from ESPN automatically, but may lag the official draw. Admin can fix manually.
- Switching devices needs admin help (no self-service flow)
- ESPN is an unofficial endpoint and could change. Admin manual entry is the fallback.

## Roadmap (post-WC26)

- Cloudflare Access SSO gate for strict company-only access
- Mini-leagues / sub-groups
- Bonus markets (winning country, top scorer, golden ball)
- Reuse the framework for Euros 2028
