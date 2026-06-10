# The Perfect XI

Draft an all-time cricket XI and find out if it could go a whole campaign **unbeaten**.
A cricket take on the viral [82-0](https://www.82-0.com) (NBA) and [38-0](https://38-0.app)
(football) draft-and-simulate games.

Two modes:

- **World Cup (ODI)** — build a one-day XI for the main tournament: top a **10-team group** (single round-robin, 9 games), reach the **top 4** for the semi-final, then the final. Win all 11 → **11–0**. Eleven players, eleven wins, zero losses.
- **World Test Championship (Test)** — build a Test XI and survive a real WTC cycle: **6 series (3 home, 3 away), 2–5 Tests each**, ranked on a points-percentage table; **top 2 of 9** reach the one-off final. Win it for the mace — go the **whole cycle unbeaten** for the perfect run.

## Run it

It's a single self-contained file — no build, no server, no dependencies.

```bash
open index.html        # macOS
# or just double-click index.html
```

Everything runs in the browser; no data leaves the page.

## How it plays

1. Pick a mode, then **Classic** (ratings shown) or **Expert** (ratings hidden — draft on pure knowledge).
2. Each round the wheel spins a random **Nation × Era** (e.g. *West Indies · 1980s*).
3. Draft one player from that group — they slot into the best open position in your XI.
4. Three rerolls if you don't like the spin. Fill all 11 positions.
5. The simulator plays your campaign and returns your record + a result card.
   **Share image** renders the card to a PNG (drawn on a `<canvas>`, no libraries) and opens
   your device's native share sheet; **Copy text** copies the emoji-grid summary.

## How the simulation works

Not a physics engine — a transparent ratings model, same idea as the originals:

- Every player has a **format-specific overall** (`test` / `odi`, 0–99) and **role tags**
  (`OPEN`, `TOP`, `MID`, `FIN`, `KEEP`, `AR`, `PACE`, `SPIN`).
- When a player lands in a slot they get a **positional-fit multiplier**. Disciplines never
  cross — a pace bowler can't fill a spin slot, a batter can't fill a bowling slot, only a
  genuine all-rounder flexes (bats at 0.75, bowls at 0.85), only a keeper keeps. The only
  sub-1.0 fits are batting-order shuffles (e.g. an opener batting at No.5). See `pairFit()`.
- **Team strength** = mean of `overall × fit` across the XI.
- Each match: `winProb = 1 / (1 + 10^((opp − team) / 12))`.
- **World Cup** — main tournament: a 10-team group (9 round-robin games), top 4 into the
  semi-final, then the final (up to 11 matches), W/L only.
- **WTC** — the real cycle: 6 series (3 home / 3 away), 2–5 Tests each (league capped at 21),
  with **home advantage** (±6 rating points). Results pay WTC points (**W = 12, D = 4**),
  ranked by points-percentage against 8 simulated rivals; the top 2 reach a one-off final.
  Test matches carry a draw probability (closer sides draw more).

Outcome distribution (Monte-Carlo, 6k random all-legend drafts per mode):

| World Cup | | World Test Championship | |
|---|---|---|---|
| Group-stage exit | 12% | Missed final | 41% |
| Semi-final exit | 49% | Runners-up | 28% |
| Runners-up | 24% | Champions | 31% |
| Champions | 14% | | |
| **11–0 perfect** | **2.2%** | **Champions · unbeaten** | **0.7%** |

A deliberate, optimised draft beats these averages; a perfect run is always rare, never impossible.

## Extending it

Everything lives in `index.html`:

- **Add players** → append to the `DB` array (`{n, c, e:[eras], r:[roles], test, odi}`).
  Use `odi:null` or `test:null` to make a player ineligible in that format
  (e.g. Bradman/Sobers have `odi:null`).
- **Change XI shape** → edit `SHAPES.wc` / `SHAPES.wtc` (ordered slot roles).
- **Retune difficulty** → opponent curves and the `winProb` divisor (`/12`) in `simWC()` /
  `simWTC()`; WTC home advantage is the `HOME` constant; the points-percentage qualifying bar
  is the rivals' distribution in `simWTC()`.
- **Swap the draft axis** → `spin()` groups players by `Nation × Era`; change the key to use
  IPL franchises, or any other tagging.

## Notes

Player ratings are deliberately subjective ballpark values for a fun draft game — not official
ICC rankings. Tweak them freely.

**Share image:** the native share sheet (`navigator.share` with a file) needs a *secure
context* — it works when the page is served over `https://` or `http://localhost`. Opened
straight from `file://`, the share button falls back to copying the image to the clipboard,
or downloading `perfect-xi.png`. Host it (GitHub Pages, Netlify, or `python3 -m http.server`)
for the full one-tap share-to-WhatsApp/X experience.
