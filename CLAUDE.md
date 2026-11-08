# CLAUDE.md

Guidance for Claude (or any coding agent) working on **Slice Rush**, a browser game made by Alex, Emma, Claude, and Niklas.

## What this project is

A top-down, touch-first pizza-parlor game. The player races three computer "regulars" (Gino, Rosa, Sal) to eat slices that pop up on plates. Every slice makes the eater fatter; the first to reach 20 points wins. Extras: speed and jump coins, chewing gum, wind storms, cheese storms, a slice-stealing cat, tap-to-bop, a leader crown, and generated rock music.

The players are children. Keep the game silly, readable on a phone, and forgiving.

## Hard constraints

- **One file.** Everything lives in `slice-rush.html` (HTML, CSS, JS, graphics, audio). No build step, no frameworks, no external requests, no image or sound assets. All art is drawn on a canvas; all sound is synthesized with the Web Audio API.
- **Touch first.** Every feature must work with taps and drags on a phone. Keyboard support (arrows/WASD) is a bonus.
- **Portrait layout.** The world is a fixed 540 × 900 canvas scaled to fit the screen.
- **Storage is optional.** `localStorage` holds the selected difficulty and fastest win per difficulty; always wrap it in `try/catch`.
- Use U.S. English in all player-facing text.

## Code map (inside the `<script>` block)

| Section | What it holds |
|---|---|
| World geometry | `W`, `H`, `R` (base eater radius), `R_MAX`, `ROOM`, `TABLES`, `PLATES`, `GOAL` |
| Navigation | `navFor(r)` builds per-radius obstacle data (cached); `plan()` finds the next waypoint around tables via a visibility graph |
| Rules | `TYPES` (slice points/lifetimes), `DIFFS` (Easy/Normal/Hard), `FX` (power-up timings), `SLIP` (floor grip), wind/cheese/cat constants, `AIDEFS` (regular personalities) |
| Audio | `beep()` for effects; `startMusic()`/`stopMusic()` and `playStep()` for the 4-bar rock loop |
| Storms | `updateEvents()` runs the idle → warn (3-2-1) → active cycle for wind and cheese |
| The cat | `spawnCat()`, `updateCat()`, `catLand()` |
| Interactions | `grab()`, `tryGrab()`, `applyItem()`, `tryBop()`, `separate()` (bumping and bouncing) |
| AI | `aiThink()` scores slices and coins by points per estimated time; `updateAI()` handles stun, retargeting, and wandering |
| Rendering | `drawBackground()` is cached to an offscreen canvas; `render()` draws everything else each frame |
| Menus | `startGame()`, `endGame()`, overlay HTML at the top of the file |

## Game rules worth preserving

- The player runs 20% faster than the fastest regular (minimum 235 px/s).
- Regulars can never push the player. Once the player has eaten, regulars that touch the player bounce away and are stunned briefly.
- Eaters grow from radius 17 to 48 as they approach the goal, so near the end they no longer fit between the closest tables.
- Storms always give a visible 3-2-1 countdown with sound before they start.
- The cat comes about twice per game and shows a landing ring before it lands.

## Testing

`window.__slice` is a test hook that exposes game state and helpers (`startGame`, `forcePlay`, `update`, `setTarget`, `wind`, `cheese`, `summonCat`, `setDiff`, and getters). It has no effect on normal play.

A useful balance check: load the page in headless Chrome (Playwright), run `startGame(); forcePlay();`, then step `update(1/60)` in a loop with a simple bot that calls `setTarget()` on the nearest slice every 0.75 s. Twelve rounds per difficulty give a rough win rate. Recent results: Easy about 7/12, Normal about 4–7/12, Hard about 3–4/12.

Also take screenshots at a 390 × 844 viewport to check layout on a phone, and confirm there are no page errors.

## When changing things

- Tune numbers in the constants blocks before adding new mechanics.
- Keep the start-screen legend in sync with new items or events.
- After changes, re-run the balance check and look at a screenshot of the new feature.
