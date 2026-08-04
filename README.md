# NFL Stats — Talkin' Giants

A live NFL stat lookup that puts a Talkin' Giants lower third on screen. Browse all 32 teams, open
any player, pick a timeframe, and press **Send to OBS**.

Everything is one HTML file. No build step, no npm, no API key. Stats come from ESPN's public
endpoints at page load, so the numbers are whatever the league is showing right now.

Adapted from the Talkin' Baseball [Trade Snapshot](https://robsjomboy.github.io/mlb_stats/) — same
lower-third lockup and the same control/display split, rebuilt on NFL data.

---

## Running it

**Just looking things up?** Double-click `index.html`. That's the whole install.

**Putting it on screen?** See the OBS section below. For the two-computer setup the page has to be
served from GitHub Pages rather than opened off disk — Settings → Pages → Branch `main` / root.

---

## What's in it

**Team directory** — all 32 clubs by conference and division. The page opens on the Giants; the
**Giants** button in the header jumps back there from anywhere.

**Roster view** — ESPN's own grouping: Offense / Defense / Special Teams / Injured Reserve or Out /
Suspended / Practice Squad. Real injury designations show in the Status column (QUESTIONABLE, OUT)
rather than a generic badge. Click any column header to sort; jersey numbers sort numerically.

**Global search** — type a name and it filters every rostered player in the league. The index is
built in the background from all 32 rosters (the Giants land first, so the page is usable
immediately) and the header shows `indexing n/32` while it fills.

**Player card** — click a name. Eight timeframes:

| Pill | What it covers |
|---|---|
| Full Season | ESPN's season totals |
| Last 3 / Last 5 | the most recent 3 or 5 games played |
| Home / Away | that season, split by venue |
| Vs Division | games against the other three teams in their division |
| Vs Opponent | one specific opponent this season, picked from a dropdown of teams they actually faced |
| Career Total | ESPN's career totals (regular season) |
| Career Vs Team | every meeting with one club across their whole career |

Stats are position-aware, because ESPN's response is: a quarterback comes back with
CMP/ATT/YDS/TD/INT/RTG, a linebacker with TOT/SOLO/SACK/TFL/INT/PD, a kicker with FG/FG%/LNG/XP/PTS.
The card shows everything the position returns; the cells outlined in red are the ones that go on
the bar.

**Season picker** goes back to 2000.

---

## Getting it into OBS

Both pages talk over a named room on Firebase Realtime Database — the same shared Jomboy project
(`pinpoint-abf21`) that Prompter and both Pinpoints use. The two machines don't have to be on the
same network, or in the same building. Nothing to install and no server to run.

1. In **Connection**, take the suggested room name (or type your own) and hit **Connect**. The pill
   goes green.
2. Hit **Copy Display URL**.
3. On the OBS computer, paste that into a **Browser Source**, 1920×1080.

```
https://chrisgrimm-jm.github.io/nfl_stats/index.html?display=1&topic=tg-snapshot-2kb024
```

The room is remembered, so next show you just open the page and it reconnects itself.

**This needs the page on a real host** — GitHub Pages. The OBS machine can't open a `file://` path
off your laptop, and the control page will warn you if you're running it that way.

**Room names are effectively public.** That's why the suggested one has a random tail — a guessable
room is a stranger's write access to your lower third. Don't shorten it to something tidy like
`talkingiants`.

If you only need a second window on the same computer (window capture rather than a Browser
Source), **Open Display ↗** works with no room at all — those two tabs talk over BroadcastChannel.

### Driving it during a show

Open a player, pick a timeframe, press **▶ Send to OBS**. The button turns green and reads **● On
Air** while that exact line is the one on screen.

Nothing reaches OBS until you press Send. Browsing players and flipping through timeframes mid-show
is therefore always safe — the bar holds whatever was last sent, so you can look something up on air
without it appearing on screen behind you.

**✕ Exit Lower Third** (in the header or on the player card) takes it off screen. Player-to-player
swaps fade the old bar out before the new one fades in, never a hard cut.

### If OBS shows nothing

1. **Room mismatch.** The control pill says connected but the display URL has an older room in it —
   re-copy it.
2. **`file://`.** The OBS machine can't open a path on your laptop. Host it on Pages.
3. **Blocked network.** Some corporate networks block the Firebase socket. The Link chip in the
   header goes grey when that happens.

---

## Look

The whole colour scheme lives in the `:root` block at the top of `index.html` — five values:

```css
--brand:#003981;   /* Talkin' Giants blue */
--accent:#c9243f;  /* red — top rule, active pills, Send button */
--label:#8fbdf0;   /* small-caps stat labels */
--lt-from / --lt-mid / --lt-to   /* the bar's gradient sweep */
```

Change a hex there and the app and the on-air bar both follow. Bebas Neue for names and numbers,
DM Sans for body copy, DM Mono for labels.

---

## Notes

- **The bar never wraps to two rows.** Timeframes carry different stat counts, so cell width and
  font shrink to fit whatever is on screen. A single wide value (`216/339`, `2,272`) shrinks on its
  own without dragging its neighbours down with it.
- **Full Season uses ESPN's own totals; the other windows are aggregated from the game log.** One
  gamelog request drives Last 3, Last 5, Home, Away, Vs Division and Vs Opponent — it already
  carries week, opponent, venue and result per game.
- **Rate stats are recomputed, not summed.** Completion %, yards per attempt, yards per reception,
  FG% and passer rating are derived from the counting stats after they're totalled. Verified against
  ESPN: aggregating all 14 of Jaxson Dart's 2025 games reproduces their season line exactly,
  including a 91.7 passer rating, and Home + Away adds back up to the season total.
- **QBR is missing from the aggregated windows.** It can't be reconstructed from per-game rows, so
  those views drop it rather than show a number that's quietly wrong. Full Season and Career still
  have it, because those come from ESPN directly. Same for a punter's NET average.
- **Pro Bowls are excluded.** ESPN files the Pro Bowl in the game log as a *Postseason* game with
  the opponent listed as `AFC` or `NFC`. Left in, an all-star game lands inside "Last 3", inflates
  career totals, and turns up as a selectable opponent called AFC. Any game whose opponent isn't one
  of the 32 clubs is dropped, which catches that without touching real playoff games. Verified on
  Brian Burns: 114 games kept, 2 Pro Bowls dropped, and the career total then lands on ESPN's own
  384 tackles / 71 sacks exactly.
- **Career Total is regular season; Career Vs Team includes playoff meetings.** ESPN's career totals
  exclude the postseason, but a career head-to-head is more useful with playoff games in it, so the
  two pills can differ for a player with a postseason history.
- **Career Vs Team costs one request per season played** — a ten-year veteran means ten game-log
  fetches, run in parallel and cached for the session, so it's paid once per player. Stats are
  matched by name rather than column position there, since a player's stat columns can differ
  between seasons.
- **Longest-of stats take the max, not the sum** — LNG PASS over three games is the longest of the
  three, not their total.
- **Offensive linemen have no card.** The NFL doesn't track individual stats for the position, and
  the modal says so rather than showing an empty grid.
- **A player can legitimately be all zeros.** A special-teams-only back with four appearances and no
  touches will send a bar of zeros — that's the real stat line, not a bug.
- **The 32 teams are a static table in the file, not a fetch — and they have to be.** ESPN's
  `/teams` endpoint sends no `Access-Control-Allow-Origin` header, on either of its hosts, while the
  `/teams/{id}/roster` endpoint right next to it does. A browser hard-fails a cross-origin request
  when that header is missing, so a page can't read `/teams` at all; it isn't something you can work
  around client-side. Ids, abbreviations and division alignment are frozen anyway (last change was
  the Commanders rename in 2022), and every id in the table is verified against ESPN. If a team ever
  relocates or rebrands, edit `ALIGNMENT` at the top of the script.
- **Responses are cached per URL for the session.** Clicking back to a timeframe you already looked
  at is instant; reload the page to force fresh numbers. Changing the season clears the cache.
- **The 2026 season doesn't exist until September.** ESPN's "season 2025" runs Sept 2025 → Feb 2026,
  so before September the picker defaults to last calendar year — the newest season with games in it.
- **The overlay holds its last frame if the network drops**, and Firebase stores the current graphic
  server-side, so a Browser Source that OBS refreshes mid-show comes straight back up instead of
  sitting blank.
