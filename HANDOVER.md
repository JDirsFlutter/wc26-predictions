# Handover

If you're a fresh Claude session (or a future James, or a colleague picking this up) — read this first. Then `README.md`, `CLAUDE.md`, `BACKLOG.md`, `ANALYTICS.md` in that order.

## What this is, in one paragraph

A score-prediction game for the 2026 FIFA World Cup, built for Flutter UK&I colleagues. Single-page vanilla JS, hosted on Cloudflare Workers Assets at `https://wc26-predictions.jamesdirs90.workers.dev/`. Firebase Anonymous Auth + Firestore for shared state. ESPN unofficial API for live fixtures, scores and DraftKings odds. PokerStars Fusion design system (dark, teal primary, condensed display heads). Open registration with first-name-only display names, no email or password collected.

## Strategic context (the bit that's not in the other docs)

**Audience**: 20 to 50 PokerStars / Flutter UK&I colleagues for the duration of the 2026 World Cup (June 11 to July 19, 2026). Not a public product. URL distributed via Slack.

**Why it exists**:
1. It's a colleague trial / social engagement project
2. It's a credible artefact for James's AI Enablement pillar (built end-to-end with Claude Code, deployed to real infra, properly measured)
3. It's a demo for the Director-level case James is building

**Success looks like**:
- High proportion of invited colleagues register (sign that the friction is low)
- People come back after MD1 (sign that the loop holds)
- Visible Slack engagement around results (sign that the *social* part works)
- A clean story to tell leadership about what Claude Code lets a non-engineer ship

**Build philosophy**: ship-fast, iterate-on-data, no premature scale. Two agents (this Claude session and the user) shipped this together over a handful of working sessions.

## Key decisions and why they're the way they are

These are decisions that *could* go the other way, with the reasoning so you don't relitigate them blindly.

**Anonymous auth, not email/password**. Two iterations in we briefly added Firebase Auth with email + password. James rightly flagged that colleagues won't enter credentials into a "vibe-coded" prototype. We reverted to anonymous auth. Trade-off: device switching is admin-mediated (manual UID swap in Firestore) rather than self-service. Acceptable at this audience size.

**Cloudflare Workers Assets, not GitHub Pages**. Original deploy was on `pokerstars-prototypes` GitHub Pages. Tech colleagues called it amateur. Moved to a dedicated repo + Cloudflare. Same Firebase backend, no data migration. The old GitHub Pages URL leaves a meta-refresh redirect to the new home.

**ESPN unofficial API, not paid odds/data**. ESPN bundles DraftKings odds + 104 fixtures + venues + team colours + scores in one free endpoint. The whole data pipeline is one URL. Trade-off: unofficial means it could break without warning. Admin manual entry is the fallback. The Odds API would be the upgrade if we ever wanted sharper odds (Pinnacle / Bet365) but we don't right now.

**Round batching with a 14-day open window**. Group Stage MD1 is open from launch, each subsequent round opens 14 days before its first match. Trade-off: stops the page being 104 cards on day one. Configurable via `WC_ROUND_OPEN_DAYS` in `data.js`. Drop to 7 for stricter drip-feed, raise to 30+ if people complain they can't plan ahead.

**Visible odds with the favourite teal-tinted**. Adds context for non-football colleagues. Anchoring concern is real: if everyone picks the favourite, leaderboard variance collapses. We added `odds_visible_first_pick` event specifically to measure this after MD1. Three dials if anchoring is too strong: collapsible odds row, drop the favourite-highlight, delay odds for 24h after registration. Decide based on data, not vibes.

**Dark Fusion design**. We rebuilt the visual layer in the PokerStars Fusion design system once James shared the spec. The original Flutter UK&I navy-and-blue light theme is gone. Dark-first surfaces (`#1C1C1C` bg, `#3B3B3B` raised cards, `#2E2E2E` sub-surfaces, `#141414` recessed inputs), teal primary (`#01866F` rest / `#02BD9C` text), brand-red active tab underline, condensed display heads via Oswald (free fallback for licensed Druk Text). Don't pull this thread without reading `~/Downloads/PokerStars-Fusion-Design-System.md`.

**No JS framework, no build step**. The whole app is `index.html` + `data.js`. ~1700 lines. Single file by design. Don't try to "improve" this with React or a bundler. The lack of toolchain IS the productivity story.

**No em dashes, no decorative emoji**. Project author voice rule. Country flag emoji are OK as functional fixture data (with `aria-hidden`). Status indicators / decorations: no emoji. CLAUDE.md restates this.

## Pre-launch checklist (do once before sharing with colleagues)

- [ ] Confirm Anonymous Auth is enabled: Firebase Console → Authentication → Sign-in method → Anonymous = enabled
- [ ] Confirm Firestore rules are deployed (see CLAUDE.md or read the Firestore Rules tab — should have `allow read: if true` on participants / matches / predictions, write gated by anonymous UID)
- [ ] Confirm three analytics tools are live: visit the site, then within an hour check Cloudflare Web Analytics, Microsoft Clarity, Firebase Analytics Realtime view all show your visit
- [ ] Hit `#admin` with the passcode (in `data.js`, currently `dirsy-goat`) → "Sync fixtures from ESPN" once to ensure all 104 matches + odds + colours are in Firestore
- [ ] Smoke-test with a second account in incognito: register a second display name, predict on one tab, confirm they show up on Players + Leaderboard tabs
- [ ] Share the URL on Slack 2 to 3 days before kickoff (June 11) with a one-liner

## Things to watch in the first week of real use

1. **Anchoring rate**: in Firebase Analytics, count of `odds_visible_first_pick` where `favourite_picked = true` vs `false`. Expected baseline (no anchoring) is roughly the average implied favourite probability across matches. Materially higher = anchoring is real, turn a dial.

2. **Drop-off**: `tab_viewed` (registration) → `pick_drafted` → `picks_saved`. The two transitions tell you where the friction is. Flat-lined funnel = problem.

3. **Mobile vs desktop**: Cloudflare Analytics. If colleagues are mostly on mobile, prioritise mobile fixes; if mostly desktop, the bigger-flag-bigger-VS choices we made for desktop pay off.

4. **Session recordings of confused users**: Microsoft Clarity. Watch the first 3 to 5 first-time sessions. Look for hesitation on the chip bar, the score input, the save bar. Most of the v2 backlog priorities should come from these.

5. **Active player count vs registered count**: Players tab vs people actually saving picks. If 30 registered but only 12 saving by MD2, the loop is broken somewhere.

## Failure modes and what to do

- **ESPN endpoint returns 4xx/5xx or stale data** → admin manual score entry via `#admin`. Picks still scored. App keeps working.
- **Cloudflare deploy fails** → check the wc26-predictions repo Actions tab on GitHub. Usually a wrangler config issue if it does.
- **Firebase quota exceeded** → very unlikely at this audience size (free tier covers ~50k reads/day, ~20k writes/day; we're nowhere near). If it ever happens, the auto-sync check stops re-uploading 104 docs per new visitor, so most quota usage is single-doc writes from picks.
- **Someone registers a name they shouldn't** (rude word, real surname of a colleague who doesn't want to play, etc.) → Firestore Database → Data → `participants` → delete the offending doc. Their browser will be allowed to re-register a different name.
- **A colleague wants to play on a new device** → Firestore → `participants/{their_name}` → update the `uid` field to the new browser's anonymous UID. They tell you the UID by opening the live URL in dev console and running `firebase.auth().currentUser.uid` (or you ask them to register a new placeholder name first and copy the UID from the new doc).

## How to read the analytics data (quick guide)

- **Want to know: did anyone visit today?** → Cloudflare Analytics
- **Want to know: what are people *doing* on the page?** → Clarity session recordings (pick 3-5, watch them)
- **Want to know: are they converting through the funnel?** → Firebase Analytics → Events
- **Want to know: is the odds row anchoring picks?** → Firebase Analytics → Events → `odds_visible_first_pick` filtered by `favourite_picked = true`

For each one, the dashboard URL is in `ANALYTICS.md`.

## Out of scope, parked deliberately

See `BACKLOG.md → Parked`. The big ones: Cloudflare Access SSO gate, custom domain, mini-leagues, push notifications, framework reuse for Euros 2028. Don't pick any of these up unless James explicitly says so.

## Author context

James Dirs is Product Director for Transformation at PokerStars (Flutter Entertainment UKI). This project sits inside his **AI Enablement** strategic pillar. The wider workspace is at `~/Library/CloudStorage/OneDrive-TheStarsGroup/Desktop/Claude C JD/`. His operating rules (no em dashes, leadership tone, push back rather than just execute, etc.) live in `about-me.md` at that root and in his MEMORY index.

When picking work back up here, default to: ship the next obvious thing in BACKLOG.md → Ranked next, push directly to main, no review process, no over-engineering. The bar is "credible production app that 20 colleagues use happily for 6 weeks" — not "scales to a million users".
