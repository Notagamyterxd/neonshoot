## Plan: Neon Arena – Mobile Controls, Fullscreen, Stats, High Score, Difficulty

All changes are scoped to `src/components/ArenaGame.tsx` (game logic + UI overlays) and `src/pages/Index.tsx` (high score badge + intro copy). No new dependencies.

### 1. Difficulty selector
- Add a `difficulty` state: `easy | normal | hard | insane`.
- Each level scales: bot count per wave, bot HP, bot fire cooldown, bot bullet damage, bot speed.
- Selector appears on the start/game-over overlay as 4 pill buttons (current selection highlighted with primary glow).
- Stored in `localStorage` so it persists between sessions.

### 2. On-screen virtual joystick + fire button (mobile/touch)
- Detect touch capability via `('ontouchstart' in window)` or `useIsMobile` hook + pointer type, and render an overlay layer only when touch is present (also toggleable for testing).
- **Joystick (bottom-left)**: a circular base (~120px) with a draggable knob. Tracks a single pointer; outputs a normalized vector (-1..1) that drives `player.vel` instead of WASD when active. Knob snaps back on release.
- **Fire button (bottom-right)**: a large circular button (~96px). While pressed (pointerdown→pointerup/cancel), sets `mouse.down = true`. Aim direction in touch mode auto-targets the nearest bot (since there's no mouse hover); falls back to movement direction when no bots in range.
- Both controls use `pointer-events` isolated from the canvas, use `touch-action: none`, and handle multi-touch (joystick and fire button can be pressed simultaneously via separate pointerIds).
- Controls are translucent with neon ring styling to match the theme.

### 3. Fullscreen
- Button in the HUD bar (icon: expand/compress from lucide-react).
- Uses the Fullscreen API on the game container (`requestFullscreen` / `exitFullscreen`) with webkit fallback for iOS Safari.
- Note: iOS Safari does not support true element fullscreen; fallback applies a `fixed inset-0 z-50` "pseudo-fullscreen" CSS class so it still fills the screen on iPad/iPhone.
- Listens to `fullscreenchange` to keep button state in sync.

### 4. Match stats
- Track during a run: shots fired, shots hit, accuracy %, kills, damage taken, time survived, highest wave.
- Displayed on the Game Over overlay in a compact stat grid.
- Live "time survived" shown in the HUD bar next to wave.

### 5. High score
- Per-difficulty high scores stored in `localStorage` (`neon-arena-highscores` JSON keyed by difficulty).
- Shown on start overlay ("Best on Normal: 240 · Wave 8") and updated on game over with a "NEW BEST!" flash if beaten.
- Also surfaced as a small badge on the landing page header in `Index.tsx`.

### 6. UI / layout adjustments
- HUD bar becomes responsive: on narrow viewports (<640px) it wraps to two rows and uses smaller padding so HP/Score/Wave/Time/Fullscreen all fit.
- Difficulty pill row sits under the title on the start overlay.
- Touch controls are hidden when overlay (start/game-over) is shown.

### Technical details
- New refs: `touchRef` for joystick state `{active, id, baseX, baseY, dx, dy}` and `fireBtnRef` for fire pointerId.
- New state: `difficulty`, `isFullscreen`, `isTouch`, `stats`, `highScores`.
- Difficulty config object:
  ```text
  easy:   {countBase:1, hpMul:0.7, fireMul:1.4, dmgMul:0.7, spdMul:0.85}
  normal: {countBase:2, hpMul:1.0, fireMul:1.0, dmgMul:1.0, spdMul:1.0}
  hard:   {countBase:3, hpMul:1.3, fireMul:0.75, dmgMul:1.3, spdMul:1.15}
  insane: {countBase:4, hpMul:1.7, fireMul:0.55, dmgMul:1.6, spdMul:1.3}
  ```
- Touch aim: each frame in touch mode, find nearest bot within 600px; if none, aim along last movement vector.
- `mouse` ref is reused as the unified "aim + trigger" input regardless of source (mouse or touch).
- All new elements respect existing design tokens (`primary`, `accent`, `card`, `shadow-glow-primary`).

No backend or new packages required.
