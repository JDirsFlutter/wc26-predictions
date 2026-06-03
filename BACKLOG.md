# Backlog

Ranked priorities for World Cup 2026 Predictions. Re-rank as the situation changes. Add to "Parked" rather than fighting for priority on Ranked.

## Active

- **Cloudflare Web Analytics snippet** — waiting on the user to enable Web Analytics in Cloudflare dashboard and paste the `<script>` tag. Once provided, drop into `index.html` `<head>` and the integration is done.

## Ranked next

1. **Weekly insights digest** — Scheduled task that pulls the previous week's analytics, summarises into `INSIGHTS/YYYY-MM-DD.md`. Requires analytics fully in place first.
2. **Today tab pre-tournament copy** — Before June 11 the Today tab is empty. Replace with a countdown + featured upcoming fixtures.
3. **Slack post on full-time** — Incoming webhook into the colleague Slack: "Mexico 2 - 1 South Africa. James got it exactly. +3 pts." Triggered when admin or auto-sync marks a match final.
4. **Knockout team auto-population** — When the group stage finishes, populate R32 fixtures from group standings automatically rather than waiting on admin.

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

- 2026-06: Analytics — Microsoft Clarity (session recordings, heatmaps) + Firebase Analytics with seven custom events covering registration, drafting, saving, navigation, and the odds-anchoring evaluation
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
