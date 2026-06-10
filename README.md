# The Perfect XI

Draft an all-time cricket XI and find out if it could go a whole campaign **unbeaten** —
then invite your mates and play a tournament between your teams.
A cricket take on the viral [82-0](https://www.82-0.com) (NBA) and [38-0](https://38-0.app)
(football) draft-and-simulate games.

It's a **single self-contained `index.html`** — no build, no server, no dependencies, no sign-up.
Everything runs in the browser; nothing is uploaded.

## Two modes

- **World Cup (ODI)** — build a one-day XI: top a **10-team group** (single round-robin, 9 games),
  reach the **top 4** for the semi-final, then the final. Win all 11 → **11–0**.
- **World Test Championship (Test)** — build a Test XI and survive a real WTC cycle:
  **6 series (3 home, 3 away), 2–5 Tests each**, ranked on a points-percentage table;
  **top 2 of 9** reach the one-off final. Go the **whole cycle unbeaten** for the mace.

## Run it

```bash
open index.html        # macOS — or just double-click the file
```

To play multiplayer across devices (so the invite **links** work), host it anywhere static —
GitHub Pages, Vercel, Netlify, or `python3 -m http.server`.

## How it plays

1. **Pick a mode**, then choose your **team make-up** (see below) and **Classic**
   (ratings shown) or **Expert** (ratings hidden — draft on pure knowledge).
2. Each round a wheel spins a random **Nation × Era** (e.g. *West Indies · 1980s*).
   Draft one player from that group — they slot into the best open position in your XI.
   Three rerolls if you don't like the spin. Fill all 11.
3. The simulator **plays your campaign out match by match** — each scoreline lands, the
   record ticks up, then the verdict and awards are revealed (a **skip** button jumps to the end).
4. The result screen shows your record, every **game's scoreline + margin**, the
   **tournament awards**, your final XI, and a canvas **share image**.

## Choose your XI make-up

Like a football formation, you pick how your eleven is built — how many specialist
**batsmen**, **all-rounders**, **pace** bowlers and **spinners** (the wicketkeeper is always 1).
Pick a realistic **preset** or fine-tune with **+/- steppers**:

| Preset | Make-up | Who plays this way |
|---|---|---|
| Classic Balanced | 5 bat · 1 AR · 3 pace · 1 spin | the default international XI for 40 years |
| Subcontinent Spin | 5 · 1 · 2 · 2 | India / Pakistan / Sri Lanka at home |
| Galle Turner | 5 · 1 · 1 · 3 | a raging day-five turner, three spinners |
| Pace Battery | 5 · 1 · 4 · 0 | 1980s West Indies, modern Australia/SA |
| Batting Heavy | 6 · 1 · 2 · 1 | batting first for a big total |
| All-Rounder Rich | 4 · 2 · 3 · 1 | the Imran/Kapil/Botham era, modern India |

Custom picks are validated for realism (at least 1 pace bowler, at least 4 bowling
resources to take 20 wickets, 3–6 specialist batsmen, etc.), so you can't field a
nonsense side. The defaults reproduce the game's original fixed shapes exactly.

## Multiplayer — shareable codes, no backend

Each player drafts their own XI; a short **code** (about 20 characters) *is* the whole
transport — there is no server.

- **Invite a friend:** from the Tournament screen, **Copy invite link** (or copy any team's
  code). Send it on WhatsApp/anywhere.
- **They open the link** → they land in the tournament lobby with your team(s) already in,
  **draft their own XI to join**, then run the tournament or pass the link on.
- **The tournament:** up to **8 teams (88 players)**; empty seats are filled with AI sides.
  The host picks **World Cup** (group + knockout) or **WTC** (points table + final). It plays
  through the same match engine, and the results screen shows **standings, the knockout
  bracket, and tournament-wide awards** (a friend's player can win the Golden Bat).
- It's **deterministic**: the same set of codes always produces the same tournament, so
  everyone who runs them sees the same champion.

The code packs the mode, your make-up, and your 11 players into URL-safe text with a
checksum; a bad paste fails gracefully with a plain-English message. (A default make-up
emits the shorter v1 code; a custom make-up uses v2 — both round-trip exactly.)

## How the simulation works

Not a physics engine — a transparent ratings model:

- Every player has a **format-specific overall** (`test` / `odi`, 0–99) and **role tags**
  (`OPEN`, `TOP`, `MID`, `FIN`, `KEEP`, `AR`, `PACE`, `SPIN`).
- A player landing in a slot gets a **positional-fit multiplier**; disciplines never cross
  (a pace bowler can't fill a spin slot, a batter can't bowl), only a genuine all-rounder
  flexes, only a keeper keeps. See `pairFit()`.
- **Team strength** = mean of `overall × fit` across the XI.
- Each match outcome: `winProb = 1 / (1 + 10^((opp − team) / 12))` decides win/loss (and a
  draw chance in Tests). A realistic **scoreline + margin is then back-filled** to match the
  result — runs when batting first defends, wickets when the chase succeeds, an innings on a
  big gap, or a draw in Tests (MCC Law 16). Per-player **box scores** accumulate into the
  awards (most runs, most wickets, most 50s, most 100s, Player of the Tournament).
- Because the scoreline is generated *after* the result is decided, it never changes the odds.

Outcome distribution (Monte-Carlo, all-legend drafts per mode):

| World Cup | | World Test Championship | |
|---|---|---|---|
| Group-stage exit | 12% | Missed final | 41% |
| Semi-final exit | 49% | Runners-up | 28% |
| Runners-up | 24% | Champions | 31% |
| Champions | 14% | | |
| **11–0 perfect** | **2.2%** | **Champions · unbeaten** | **0.7%** |

A deliberate, optimised draft beats these averages; a perfect run is always rare, never impossible.

## Under the hood

Everything lives in `index.html` (322 players, vanilla JS, no libraries):

- **Players** → the `DB` array (`{n, c, e:[eras], r:[roles], test, odi}`).
  **The DB is append-only** — new players go on the END, existing entries are never reordered
  or removed, because a player's index is its stable id inside every shared code. Breaking
  that would silently invalidate codes; bump `CODEC_VERSION` instead.
- **Make-up** → `MK_PRESETS` (the presets), `MK` (the constraints), `buildShape(mode, makeup)`
  (turns counts into the ordered XI).
- **Difficulty / odds** → the `winProb` divisor in `simWC()` / `simWTC()`; WTC home advantage
  is the `HOME` constant.
- **Share code** → `encodeXI` / `decodeXI` (9-bit player ids + a make-up byte, base64url).
- **Draft axis** → `spin()` groups players by `Nation × Era`.

## Notes

Player ratings are deliberately subjective ballpark values for a fun draft game — **not
official ICC rankings**. This is an unofficial, fan-made game, not affiliated with the ICC,
any board, league, or player; player names are used only for identification. No data leaves
your device.

**Share image:** `navigator.share` with a file needs a *secure context* — it works over
`https://` or `http://localhost`. Straight from `file://` it falls back to copying the image
to the clipboard or downloading `perfect-xi.png`. Host it for the full one-tap
share-to-WhatsApp experience (and for the multiplayer invite links to carry across devices).
