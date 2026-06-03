# Analytics

How user behaviour is measured on wc26-predictions. Three tools layered, each answering a different question.

## Microsoft Clarity

**What it answers**: "What are people actually doing?" Session recordings, heatmaps, rage clicks, dead clicks, scroll depth.

**Where it lives**: `index.html` `<head>`. Loaded asynchronously, project ID `x18dsv1z5y`.

**Dashboard**: https://clarity.microsoft.com → wc26-predictions project.

**Use it for**:
- Watching colleagues use the app for the first time, end to end
- Spotting moments of confusion (a button no one finds, a panel everyone scrolls past)
- Sanity-checking that the round chip pattern is discoverable
- Mobile vs desktop interaction differences

**Privacy**: Clarity does not record passwords (we don't have any) or sensitive form fields. Display names ARE captured in session recordings; that's fine because they're first names by convention and the page is unindexed.

## Cloudflare Web Analytics

**What it answers**: "Who's coming to the site?" Page views, sessions, country, device, top referrers.

**Where it lives**: `index.html` `<head>`. Beacon token `51a4ff02e8be421d9146fa51b2c9b051`.

**Dashboard**: Cloudflare dashboard → Analytics & Logs → Web Analytics → wc26-predictions.

**Use it for**:
- Confirming traffic shape (mostly office-hours UK?)
- Device mix (mobile-first colleagues vs laptops)
- Referrer split (Slack vs direct vs email)

**Privacy**: No cookies, no fingerprinting, no PII. The most privacy-friendly of the three tools.

## Firebase Analytics with custom events

**What it answers**: "What's the user journey?" Funnel data, retention, custom event counts.

**Where it lives**: `index.html` — `getAnalytics` initialised in `initFirebase`, custom events logged at key flow points via the `logE` helper.

**Dashboard**: Firebase Console → wc26-predictions Firebase project → Analytics. Real-time view shows events as they fire.

### Custom events tracked

| Event | Fires when | Params | Question it answers |
|---|---|---|---|
| `claim_name_completed` | A user successfully claims their display name (writes participant doc) | `mode` (`"firebase"` or `"local"`) | What fraction of visitors register? |
| `pick_drafted` | First time a score input is touched for a match in a session | `match_id`, `stage` | Are people exploring picks before committing? Which matches get drafts but no save? |
| `picks_saved` | `saveAllPending` successfully writes the batch | `count`, `mode` | How many picks does the typical save batch include? Big batches vs single saves? |
| `round_chip_clicked` | A non-locked round chip is clicked | `round_id` | Which rounds get the most attention? Do people stay on GS1 or roam? |
| `tab_viewed` | `setView` is called for any tab | `view_id` | Tab popularity. Do people use Players or just Leaderboard? |
| `admin_opened` | `#admin` passcode is accepted | `{}` | Admin activity ledger. |
| `odds_visible_first_pick` | First save of a match that has odds available | `match_id`, `stage`, `favourite_picked` (bool) | **The anchoring evaluation**. What fraction of saved picks back the implied favourite? Compare to a no-odds baseline to measure the anchoring effect of showing odds. |

### The anchoring evaluation

`odds_visible_first_pick` is the key event. After MD1, query it in GA4:

- Count where `favourite_picked = true` vs `favourite_picked = false`
- Expected baseline (no anchoring): roughly the implied probability of the favourite, averaged across matches
- If `favourite_picked = true` rate is materially higher than baseline, the odds row is anchoring picks

Three dials we can turn if anchoring is too strong:
- Make the odds row collapsible (default hidden)
- Remove the favourite teal-tint
- Delay odds appearance until 24h after registration

See `BACKLOG.md → Anchoring trade-off on odds`.

### What's NOT logged

- Display names (already public on the leaderboard, but no point logging them as event params, GA4 user IDs handle identity)
- Score values (not interesting; aggregates are)
- Anything that would survive log retention as PII (we have none anyway)

## Operational

- All three tools fail open: if Clarity is blocked by an ad blocker, the site still works. If GA fails to initialise, the rest of the app works. If Cloudflare Analytics is off, no impact.
- The `logE` helper wraps `firebase-analytics.logEvent` in a try/catch and a null-check. Safe to call even when Analytics isn't initialised.
- Events are per-session (`SESSION_DRAFTED`, `SESSION_SAVED` Sets in module scope) so a page refresh resets the dedup. Aim for "first touch per session" granularity, not "first touch ever".
