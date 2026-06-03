# Backlog

Ranked priorities for World Cup 2026 Predictions. Re-rank as the situation changes. Add to "Parked" rather than fighting for priority on Ranked.

## Active

(In flight. Empty most of the time.)

## Ranked next

1. **Analytics setup** — Clarity, Cloudflare Web Analytics, Firebase Analytics with custom events. Detailed brief below.
2. **Weekly insights digest** — Scheduled task that pulls the previous week's analytics, summarises into `INSIGHTS/YYYY-MM-DD.md`. Requires analytics in place first.
3. **Today tab pre-tournament copy** — Before June 11 the Today tab is empty. Replace with a countdown + featured upcoming fixtures.
4. **Slack post on full-time** — Incoming webhook into the colleague Slack: "Mexico 2 - 1 South Africa. James got it exactly. +3 pts." Triggered when admin or auto-sync marks a match final.
5. **Knockout team auto-population** — When the group stage finishes, populate R32 fixtures from group standings automatically rather than waiting on admin.

## Task brief: Analytics setup

**Goal**: Baseline traffic, behavioural insight, and a conversion funnel. Zero UI change. No PII.

**Touch only**:
- `index.html` — three small initialisation blocks
- New file `ANALYTICS.md` — what's tracked and why

**Don't touch**:
- Anything visual
- Firestore data shape
- `data.js` other than docstring updates

**Three integrations, in order**:

### 1. Cloudflare Web Analytics
Zero code. User toggles it on in the Cloudflare dashboard for the `wc26-predictions` Workers project, pastes the auto-generated `<script>` snippet into `index.html` just before `</head>`. Free, no cookies. Captures page views, country, device, top URLs.

### 2. Microsoft Clarity
Sign up at clarity.microsoft.com (the project author already uses Clarity on other prototypes, ask for the existing tenant). Create project for `wc26-predictions.jamesdirs90.workers.dev`. Drop the project's `<script>` tag in `<head>`. Free, gives session recordings, heatmaps, scroll depth, rage clicks, dead clicks.

### 3. Firebase Analytics with custom events
- Add `firebase-analytics` to the existing modular SDK imports (alongside `firebase-app`, `firebase-auth`, `firebase-firestore`)
- Call `getAnalytics(App)` once after `initializeApp`
- Log events at these flow points (use `logEvent(analytics, name, params)`):

| Event | Fires when | Params |
|---|---|---|
| `claim_name_completed` | `claimName` writes the participant doc successfully | `{ mode: "firebase" \| "local" }` |
| `pick_drafted` | First time a score input is touched for a match in a session | `{ match_id, round_id }` |
| `picks_saved` | Batch commit in `saveAllPending` succeeds | `{ count }` |
| `round_chip_clicked` | Round chip onclick (unlocked path only) | `{ round_id }` |
| `tab_viewed` | `setView` is called | `{ view_id }` |
| `admin_opened` | Passcode accepted | `{}` |
| `odds_visible_first_pick` | First saved pick after the user has seen the odds row | `{ match_id, favourite_picked: bool }` |

The `odds_visible_first_pick` event answers the anchoring question (see below).

**Acceptance**:
- App behaves exactly as before. No UI / copy / layout change anywhere.
- Cloudflare Analytics dashboard shows traffic within an hour
- Clarity shows recordings within an hour
- GA4 real-time view shows custom events as you click around
- `ANALYTICS.md` lists every event + the question it answers

**Rough size**: 30 to 45 minutes. One commit per integration is fine.

## Anchoring trade-off on odds

Visible odds nudge picks toward the favourite. We need data before we decide whether to dial that down.

After MD1, look at:
- Pick distribution: how many picked the implied favourite vs draw vs underdog
- Standard deviation of leaderboard scores (lower = everyone clustered)
- Whether "I don't follow football" colleagues engage more confidently with odds visible

Dials we can turn if needed:
- Make the odds row collapsible (default hidden, "show odds" toggle)
- Drop the teal favourite-highlight (show three equal pills)
- Delay odds appearance until 24h after a user registers

Pick one based on what the data says, not guesses.

## Parked

- **Custom domain** (`wc26.something` via Cloudflare). Cosmetic. Do once trial sticks.
- **Cloudflare Access SSO** — gate to `@flutterint.com` (or wider Flutter brand emails). Adds friction. Probably overkill for the v1 audience.
- **Mini-leagues** — sub-groups within the main league. Useful at 50+ players.
- **Bonus markets** — winning country, top scorer, golden ball. Worth adding late group stage if engagement holds.
- **Push notifications** — PWA install + push for kickoff reminders. Heavy work for marginal benefit at this scale.
- **"Pick like X"** — copy a friend's picks. Cute but undermines the personal-pick loop.
- **Streak indicator** — "N picks in a row correct". Engagement nudge, low effort.
- **Compare with a friend** — head-to-head accuracy. Easy post-launch add.
- **Reuse framework for Euros 2028** — extract tournament config from app code. Real refactor, not a quick job.
- **Audit log of pick changes** — `submittedAt` already captures last edit; full history would need a `pick_history` collection.
- **Match-level insight per user** — "You picked correctly in 4 of last 5 group stage finals." Add to a profile page if profiles ever exist.
- **First-time tour** — interactive walkthrough on first visit. Probably not needed if the app is clear enough.

## Done (rolling)

- 2026-06: Live odds (1X2 implied probabilities, ESPN / DraftKings, favourite tinted teal)
- 2026-06: Card v2 (team colour stripes, bigger flags, VS pill, filled-input glow, Submitted/Unsaved status copy, AA contrast fix)
- 2026-06: Surface layering fix (cards properly raised from page bg)
- 2026-06: Rebuild visual layer in PokerStars Fusion design system (dark, teal, Druk-style condensed display heads)
- 2026-06: Strip email auth, restore anonymous (browser-bound, no PII)
- 2026-06: Pending picks + sticky save bar (browse anonymously, auth at save-time)
- 2026-06: Round batching (group stage MD1/2/3 + R32 + R16 etc, 14-day open window)
- 2026-06: Open registration + Players tab
- 2026-06: Country flags + venues + auto-sync from ESPN, all 104 fixtures
- 2026-06: Migration from `pokerstars-prototypes/` to dedicated repo + Cloudflare Workers hosting
