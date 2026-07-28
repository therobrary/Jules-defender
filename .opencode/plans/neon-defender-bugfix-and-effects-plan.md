# Neon Defender — Bugfix & Effects Hardening Plan

## Goal

Resolve the **9 bugs** identified in the code review of `docs/index.html` and add the **trails, expanding-explosion, screen-shake, and object-pool** subsystems borrowed from `../jules-missiledefense-game/docs/index.html`, **without regressing** the Neon Noir brand alignment defined in the sibling plan `neon-defender-brand-alignment-plan.md`. The shipped artifact remains a single self-contained `docs/index.html`.

## Context and Constraints

- The game ships as one file. No bundler, no external runtime deps, GitHub Pages serves `/docs`.
- `cameraY` is referenced by `Game.draw()` grid rendering (line 501) but only assigned inside `Game.start()` / `resetGame()`. Init order matters.
- `Player.update()` calls `Game.updateHUD()` every frame — keep that; new effect systems must not duplicate work.
- Existing classes (`Entity`, `Player`, `Lander`, `Bullet`, `Particle`, `PowerUp`) use a mix of per-frame and `dt`-scaled physics. Effect additions should normalize the pattern where cheap.
- `CONFIG.SPAWN_RATE` is declared but never consulted — leave the constant or delete it; do not introduce logic that depends on it during this plan unless required.
- The existing plan `neon-defender-brand-alignment-plan.md` owns **design tokens, overlays, HUD copy, and Neon Noir styling**. This plan must reference those tokens (e.g. `--accent: #00e5ff`, `--accent-glow`) once that plan lands, and must not redefine them.
- Per the user's selected scope:
  - **Full Neon Noir migration** (from the brand plan) is in effect — `:root` tokens, Courier New, glassmorphism HUD/overlays, single cyan accent.
  - **Full Pool class extraction** for particles, bullets, and explosions — same pattern as `jules-missiledefense-game` `class Pool`.
- No new dependencies, no multi-file deliverable, no test framework required. Validation = static checks + browser smoke test.

## Assumptions

- The brand-alignment plan will land first (or in parallel, with this plan only depending on the `:root` token block).
- "Trails" means per-entity fading dot trails identical to `Missile.trail` in `jules-missiledefense-game` (array of `{x, y, age}`, alpha = `1 - age/maxAge`, radius scaled by alpha).
- "Expanding explosions" means the two-stage expand/retract ring (`Explosion` in `jules-missiledefense-game`), with white core + white shockwave + faint outer glow. The current `Particle`-only burst is replaced/supplemented by this.
- "Screen shake" means `Game.shakeStrength` + `shakeDecay = 0.9` applied as `ctx.translate(randomX, randomY)` before world draw, and applied as offset to the clear-rect, exactly as in `jules-missiledefense-game` lines 1857–1871.
- Object pools follow `jules-missiledefense-game` `class Pool` exactly: `acquire(factory)`, `release(obj)`, `reset()`.
- Touchscreen detection currently uses `'ontouchstart' in window || navigator.maxTouchPoints` — leave it; not a bug for this scope.

## Deliverables

- Updated `docs/index.html` with all 9 review bugs fixed and the 4 effect subsystems (trails, expanding explosions, screen shake, pools) integrated.
- Updated `README.md` if any player-facing language changes (e.g. power-up letters or HUD verbs).
- A short `BUGFIX_NOTES.md` (or appended section in this plan) documenting what changed for future reviewers.
- Concrete validation commands and a manual smoke-test checklist.

## Non-Goals

- Adding new gameplay features (new enemy types, scoring tweaks, balancing) beyond what's required for the visual/effect upgrade.
- Refactoring the single-file architecture into modules.
- Replacing the procedural audio system.
- Touch control UX overhaul beyond ensuring the existing d-pad/fire listeners still work after CSS changes.

## Todo Checklist

- [ ] Read brand-alignment plan's `:root` token block; use `var(--accent)`, `var(--accent-glow)`, `var(--text-primary)`, `var(--text-dim)` for all new visual code. If the block isn't in `docs/index.html` yet, stub it temporarily and add a TODO marker.
- [ ] Fix Bug 1: initialise `Player.invulTimeoutId = null` in the `Player` constructor.
- [ ] Fix Bug 2: track the respawn `setTimeout` ID in `Player.die()` and clear any pending timeout on subsequent death.
- [ ] Fix Bug 3: in `Game.checkCollisions()`, `break` out of the enemy loop (or short-circuit on `player.dead`) after `player.die()` to prevent multi-kill per frame.
- [ ] Fix Bug 4: move fire-input check ahead of entity updates in `Game.update()` OR adjust `canFire()` to use a fixed-timestep-friendly accumulator; pick the cheaper fix and document why.
- [ ] Fix Bug 5: replace the `while`-loop world-wrap in `Game.draw()` with the modular form `((rx % WORLD_WIDTH) + WORLD_WIDTH) % WORLD_WIDTH` clamped to `[-WORLD_WIDTH/2, WORLD_WIDTH/2)` so a single high-velocity frame can't infinite-loop or wrap incorrectly.
- [ ] Fix Bug 6: in `Lander.fire()`, normalise camera-relative `rx` using the same modular helper before the on-screen test.
- [ ] Fix Bug 7: in `Bullet.update()`, replace the single-step world wrap with a modular wrap so future high-velocity bullets don't overshoot.
- [ ] Fix Bug 8: in `Game.togglePause()`, guard the unpause `requestAnimationFrame` behind `if (!this.paused && this.running)`.
- [ ] Fix Bug 9: same wrap helper applied to `Entity.update()` (defensive, even though current speeds are safe).
- [ ] Add `class Pool` mirroring `jules-missiledefense-game` lines 72–95; expose `acquire(factory)`, `release(obj)`, `get activeCount()`, `reset()`.
- [ ] Add `class Explosion` mirroring `jules-missiledefense-game` lines 594–663: two-stage expand/retract ring with white core + cyan shockwave when "flak"/power-driven, white otherwise; integrate with the existing explosion trigger sites (`Player.die`, `Lander.takeDamage`, `triggerSmartBomb`, `hyperspace`).
- [ ] Add per-entity `trail[]` arrays on `Bullet` (player and enemy) and on `Lander` engine trail; render via the missile-defense pattern (fading dots, `globalAlpha = 1 - age/MAX`, age capped at 20 frames).
- [ ] Add `Game.shakeStrength`, `Game.shakeDecay = 0.9`, `Game.shake(amount)`; call from `Player.die`, `Lander.takeDamage` (light), `triggerSmartBomb` (heavy), and from city/Player-shield-break events. Apply in `Game.draw()` as `ctx.translate(shakeX, shakeY)` and offset the clear-rect by the same delta (matching lines 1857–1871 of missile-defense).
- [ ] Wire object pools: replace direct `new Particle(...)` / `new Bullet(...)` / `new Explosion(...)` calls with `this.particlePool.acquire(...)` + `init(...)`; replace direct `dead = true` releases with `this.particlePool.release(...)` (and equivalents). Verify no leaked references after pool release.
- [ ] Pass the pools through `Bullet`/`Particle`/`Explosion` constructors OR via a `Game.pools` namespace picked up at `init()` time. Pick the cheaper pattern (constructor argument recommended for explicitness).
- [ ] Touch handlers: add `{passive: false}` to the three `touchend` listeners that call `e.preventDefault()` (dpad, fireBtn, bombBtn — note: only the dpad currently attaches `touchend`; fireBtn attaches touchstart+touchend). Move the `preventDefault` calls onto `touchstart` instead where possible to avoid the passive warning. (Note: the original brand plan may also touch mobile controls — coordinate so we don't double-edit.)
- [ ] Update HUD copy to use brand tokens: `SCORE`, `HI-SCORE`, `WAVE`, `LIVES`, `AMMO` rendered in Courier New, uppercase, letter-spacing, dim labels per the brand plan.
- [ ] Confirm `cameraY` is initialised in `Game.start()` AND `resetGame()` (and on `Player.die()` respawn so vertical camera tracks player cleanly). Currently `cameraY = 0` in `resetGame` — verify and add a one-line comment explaining the intended "no vertical camera" behaviour.
- [ ] Update `README.md` only if a player-facing string changed (e.g. if power-up letters move from S/R/F to anything else; current code uses S/R/F).
- [ ] Add `BUGFIX_NOTES.md` at repo root summarising what each bug was and how it was fixed (one line each).

## Implementation Plan

The plan is structured in **5 phases** that can each be reviewed and merged independently. Phases 1–4 are the brand-alignment phases (owned by the sibling plan); Phase 0 is this plan's bug-fix ground, and Phases 5–7 are the effect enhancements. Phase ordering is chosen so each phase leaves the game runnable.

### Phase 0 — Bug fixes (no visual change)

1. Open `docs/index.html` in a non-corrupting editor (`code -r` or `vi`; per repo memory, `read_file` interleaves lines on this file).
2. Locate `class Player` constructor (~line 707 of the inline script) and add `this.invulTimeoutId = null;` immediately after `this.speedTimer = 0;` (Bug 1).
3. In `Player.die()` (~line 850), introduce `this._respawnTimeoutId` tracking:
   - In constructor: `this._respawnTimeoutId = null;`
   - At top of `die()`: `if (this._respawnTimeoutId !== null) { clearTimeout(this._respawnTimeoutId); this._respawnTimeoutId = null; }`
   - Replace the bare `setTimeout(() => { this.invulnerable = false; }, 2000);` with `this._respawnTimeoutId = setTimeout(() => { this.invulnerable = false; this._respawnTimeoutId = null; }, 2000);` (Bug 2).
4. In `Game.checkCollisions()` (~line 401), restructure the player-vs-enemy loop:
   - Wrap the existing body in `if (!player.dead && !player.invulnerable) { ... }`.
   - Inside, set `if (player.shieldActive) { ... } else { player.die(); e.takeDamage(); break; }` so a single frame's ram kills only one enemy. (Bug 3.)
5. In `Game.update()` (~line 360), move the fire-input check above the `this.entities.forEach(e => e.update(dt))` block. This makes fire feel responsive on slow frames; keep the `canFire()` cooldown as-is. (Bug 4.)
6. Introduce a single helper, `wrapWorldX(rx)`, at the top of the script:
   ```js
   function wrapWorldX(x) {
       const W = CONFIG.WORLD_WIDTH;
       return ((x % W) + W) % W - W / 2;
   }
   ```
   - Use it in `Game.draw()` for both entities (lines 538–540) and particles (lines 550–552). Replace the two `while` loops with the helper call. (Bug 5.)
   - Use it in `Lander.fire()` (lines 959–963) for the on-screen check. (Bug 6.)
   - Use it in `Bullet.update()` (lines 1094–1095). (Bug 7.)
   - Use it in `Entity.update()` (lines 685–686) defensively. (Bug 9.)
7. In `Game.togglePause()` (~line 264), change the unpause branch to `} else { pauseScreen.classList.add('hidden'); this.lastTime = performance.now(); if (this.running) requestAnimationFrame(t => this.loop(t)); }`. (Bug 8.)
8. **Phase 0 verification:**
   - `node --check` on the extracted JS (strip `<script>` and `</script>`).
   - Manual smoke test: start, move, fire, hyperspace, bomb, pause, resume, die, restart. No regressions.

### Phase 5 — Object pools

1. Add `class Pool` mirroring missile-defense lines 72–95:
   ```js
   class Pool {
       constructor(factory, prewarp = 0) {
           this.factory = factory;
           this.free = [];
           this.active = new Set();
           for (let i = 0; i < prewarp; i++) this.free.push(factory());
       }
       acquire(...args) {
           const obj = this.free.pop() || this.factory();
           this.active.add(obj);
           return obj;
       }
       release(obj) {
           if (this.active.delete(obj)) this.free.push(obj);
       }
       get activeCount() { return this.active.size; }
       reset() { this.free.length = 0; this.active.clear(); }
   }
   ```
2. In `Game.init()`, add `this.particlePool = new Pool(() => new Particle(), 200); this.bulletPool = new Pool(() => new Bullet(0, 0, 0, 'player'), 50); this.explosionPool = new Pool(() => new Explosion(), 10);`
3. Refactor `Particle`, `Bullet`, `Explosion` to expose an `init(...)` method that resets state without re-constructing. Keep the constructors for the initial prewarm.
4. Replace spawn sites:
   - `Game.spawnExplosion(x, y, color)` (line 607) — now spawns an `Explosion` from the pool instead of (or in addition to) the particle burst. Particles become the trailing debris from the explosion, not the explosion itself.
   - `Player.fire()` — `this.bulletPool.acquire(bx, this.y, bvx, 'player').init(...)`. Same for `Lander.fire()` with `owner: 'enemy'`.
5. Replace release sites:
   - `this.entities = this.entities.filter(e => !e.dead)` in `Game.update()` (line 374) — when removing a `bullet`, `particle`, or `explosion`, call the appropriate `pool.release(obj)`.
6. **Phase 5 verification:**
   - Static: `grep -n 'new Particle\|new Bullet\|new Explosion' docs/index.html` should only return matches inside `class Pool`'s factory lambdas.
   - Smoke test: spam smart bombs, verify no console errors and stable FPS.

### Phase 6 — Trails and expanding explosions

1. Add `class Explosion` (if not added in Phase 5; this phase may own it if Pool is split out):
   - Mirrors missile-defense lines 594–663: `radius`, `maxRadius`, `expansionRate`, `isRetracting`, `isFlak` flag for cyan-accent variant.
   - `init(x, y, isFlak, maxRadius)`.
   - `update(dt)`: expand until `radius >= maxRadius`, then retract until `radius <= 0` → `dead = true` (or `active = false`).
   - `draw(ctx)`: white core (40% radius), white shockwave (full radius, alpha 0.6), faint outer glow (70% radius, alpha 0.3). Cyan accent if `isFlak`.
2. Wire explosion triggers:
   - `Player.die()`: large white explosion + screen shake 18.
   - `Lander.takeDamage()`: medium white explosion + screen shake 6.
   - `triggerSmartBomb()`: heavy screen shake 25 + multi-point white explosion at bomb center, secondary explosions chained to each killed enemy.
   - `Player.hyperspace()`: cyan explosion at origin (depart), cyan at destination (arrive), screen shake 4. Failure-case malfunction: bigger red-tinted explosion (use existing particle color override, no new palette).
3. Add `trail[]` to `Bullet`:
   - In `init()`: `this.trail = [];`
   - In `update(dt)`: push `{x, y, age: 0}`; in `draw(ctx)`, iterate, age++, draw `arc(point.x, point.y, 2 * alpha, ...)` with `globalAlpha = 1 - age/20`. After draw, filter `age <= 20`.
   - Player bullets: white trail. Enemy bullets: red-tinted (`#ff5555` already in palette).
4. Add `trail[]` to `Lander`:
   - Subtle grey/cyan short trail behind each Lander (engine glow), aged similarly but shorter (max 8 frames).
5. Add `trail[]` to `Player`:
   - Engine flame is already drawn (lines 815–821). Replace the random-flicker `Math.random() > 0.5` with a real particle-style trail: a short cyan trail when thrusting (inputX !== 0). This also fixes the existing flicker bug (engine flame drawing is non-deterministic and out of sync with input — see code review point).
6. **Phase 6 verification:**
   - Static: every explosion site calls the new system; no orphan `Game.spawnExplosion` calls.
   - Smoke test: confirm trails visible behind bullets and enemies; explosions expand-then-retract; cyan-on-flak events appear during smart bomb.

### Phase 7 — Screen shake

1. Add `Game.shakeStrength = 0; this.shakeDecay = 0.9;` in `Game.init()`.
2. Add `Game.shake(amount)`:
   ```js
   shake(amount) {
       this.shakeStrength = Math.min(Math.max(this.shakeStrength, amount), CONFIG.maxShake || 30);
   }
   ```
3. Apply in `Game.draw()` after `this.ctx.save()` and before the grid draw (mirrors missile-defense lines 1857–1871):
   ```js
   let shakeX = 0, shakeY = 0;
   if (this.shakeStrength > 0) {
       shakeX = (Math.random() - 0.5) * this.shakeStrength;
       shakeY = (Math.random() - 0.5) * this.shakeStrength;
       this.shakeStrength *= this.shakeDecay;
       if (this.shakeStrength < 0.5) this.shakeStrength = 0;
   }
   this.ctx.translate(shakeX, shakeY);
   ```
4. Update the background clear-rect: `this.ctx.fillRect(-shakeX, -shakeY, this.width, this.height);` (compensates so trails/shake look like a camera, not a viewport slide).
5. Wire shake amounts (from Phase 6 list).
6. **Phase 7 verification:**
   - Smoke test: take damage — visible shake; smart bomb — heavy shake; hyperspace malfunction — strong shake.

## Files and Areas to Touch

- `docs/index.html` — all bug fixes and effect additions happen here.
- `README.md` — update only if a player-facing string changes.
- `BUGFIX_NOTES.md` (new) — one-line summary per bug, one-line summary per effect added.
- `.opencode/plans/neon-defender-bugfix-and-effects-plan.md` (this file) — keep current; mark checkboxes as phases complete.

## Validation and Test Criteria

### Static checks (run from repo root)

```bash
# 1. JS syntax check (strip <script>/</script> for node --check)
python3 -c "
import re, pathlib
src = pathlib.Path('docs/index.html').read_text()
m = re.search(r'<script>(.*?)</script>', src, re.S)
pathlib.Path('/tmp/jd.js').write_text(m.group(1))
"
node --check /tmp/jd.js && echo "JS OK"

# 2. Bug-fix grep checks
test -z "$(grep -n 'while (drawnX < -100)' docs/index.html)" \
  && echo "Bug 5 fixed (modular wrap)"
test -z "$(grep -n 'while (rx < -100) rx += CONFIG.WORLD_WIDTH' docs/index.html | grep -v wrapWorldX)" \
  && echo "Bug 6/7 fixed"
grep -q 'invulTimeoutId = null' docs/index.html && echo "Bug 1 fixed"
grep -q '_respawnTimeoutId' docs/index.html && echo "Bug 2 fixed"
grep -q 'break;' docs/index.html && echo "Bug 3 break present"

# 3. Effect subsystems present
grep -q 'class Pool' docs/index.html && echo "Pool class added"
grep -q 'class Explosion' docs/index.html && echo "Explosion class added"
grep -q 'shakeStrength' docs/index.html && echo "Screen shake added"
grep -q 'this.trail' docs/index.html && echo "Trails added"

# 4. No external font deps remain (brand plan owns this)
! grep -q '@import url' docs/index.html && echo "No external font import"
```

### Manual smoke test

1. `python3 -m http.server 8000 -d docs` and open `http://localhost:8000/`.
2. Desktop browser:
   - Start, move (arrow keys), fire (space) — confirm trails behind bullets.
   - Pause (Esc), resume — confirm pause overlay matches brand system (delegated to brand plan).
   - Smart bomb (B) — confirm heavy screen shake, expanding white explosion, screen-flash overlay still appears (existing behavior preserved).
   - Hyperspace (H) — confirm cyan departure/arrival particles, screen shake; malfunction path (~1 in 10) shows stronger shake + bigger explosion.
   - Ramming into Landers — confirm only one death per frame even when overlapping multiple enemies (Bug 3 fix).
   - Lose all lives — confirm `MISSION FAILED` overlay matches brand system; high score persists across reload.
3. Mobile viewport (DevTools responsive mode, ~390×844):
   - Touch d-pad fires movement; fire button fires; bomb/warp buttons work; no passive-listener warnings in the console.
   - HUD legible, minimap not clipped.
4. Visual regression sweep:
   - Player bullets leave white trails; enemy bullets leave red-tinted trails.
   - Lander drift leaves a faint cyan/grey engine trail.
   - Explosion sequence: white core → shockwave ring → retract → fade.
   - Screen shake visible but not nauseating; resolves within ~0.5 s.

### Regression check

- `HighScore` still persists in localStorage across reloads.
- Wave progression (defeat all enemies → next wave) still works; `WAVE X` banner still appears.
- Smart bomb still only kills on-screen enemies (existing behavior preserved by Phase 6 wiring).
- Hyperspace malfunction still costs a life on a 10% roll.

## Risks and Open Questions

- **Phase ordering**: brand-alignment plan may not have landed when this plan is executed. Phases 5–7 read `var(--accent)` etc.; if the brand tokens are missing, the canvas will fall back to default colours. Mitigation: at the start of Phase 6, verify `:root` contains `--accent` and `--accent-glow`; if missing, add a minimal stub with a TODO pointer to the brand plan.
- **Pool + class extension**: `Bullet extends Entity` already exists. Adding an `init()` method that re-uses an existing instance requires resetting `dead`, `vx`, `vy`, `x`, `y`, `life`, `owner`, `trail` in one place — easy to miss fields. Mitigation: add a single `Entity.reset(x, y)` or rely on the constructor being called via `Pool.acquire(factory)` for new ones and the explicit `init()` for reused ones. Whichever is picked, document it.
- **Pool release on game restart**: pools must be cleared/reset on `Game.start()` so a restarted game doesn't render stale pooled entities from the previous run.
- **Trails across world wrap**: bullets that wrap around `WORLD_WIDTH` will leave a teleporting trail. Mitigation: clear `trail[]` whenever `Bullet.update()` detects a wrap event (or use the modular wrap and only push to the trail when the displacement is < half the world — simplest).
- **Screen shake + touch**: heavy shake on mobile could disorient aim. Mitigation: cap `CONFIG.maxShake = 20` on touch devices (use `matchMedia('(pointer: coarse)')`).
- **No test framework**: validation is manual + static grep. Acceptable for a single-file arcade game; documenting the smoke checklist in `BUGFIX_NOTES.md` is sufficient.

## Appendix — Mapping table (review → plan → file)

| Review # | Bug (one line) | Plan phase | Approx. location in `docs/index.html` |
|---|---|---|---|
| 1 | `invulTimeoutId` not initialised | Phase 0.2 | `class Player` constructor (~line 707) |
| 2 | Respawn `setTimeout` not tracked | Phase 0.3 | `Player.die()` (~line 850) |
| 3 | Multi-kill in one frame on ram | Phase 0.4 | `Game.checkCollisions()` (~line 401) |
| 4 | Fire input checked after entity update | Phase 0.5 | `Game.update()` (~line 360) |
| 5 | `while`-loop world wrap can desync | Phase 0.6 | `Game.draw()` (~lines 538–540, 550–552) |
| 6 | `Lander.fire()` on-screen test wrong with negative `cameraX` | Phase 0.6 | `Lander.fire()` (~lines 959–963) |
| 7 | `Bullet.update()` single-step wrap can overshoot | Phase 0.6 | `Bullet.update()` (~lines 1094–1095) |
| 8 | Unpause `rAF` not guarded by `running` | Phase 0.7 | `Game.togglePause()` (~line 264) |
| 9 | `Entity.update()` single-step wrap (defensive) | Phase 0.6 | `Entity.update()` (~lines 685–686) |
| — | Object pools for Particle/Bullet/Explosion | Phase 5 | new `class Pool`, all spawn sites |
| — | Expanding Explosion class | Phase 6 | new `class Explosion` |
| — | Per-entity trails | Phase 6 | Bullet, Lander, Player.draw/update |
| — | Screen shake | Phase 7 | `Game.init`, `Game.draw`, all damage sites |