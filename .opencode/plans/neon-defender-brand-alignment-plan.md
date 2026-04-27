# Neon Defender Modernization and Brand Alignment Plan

## Goal
Modernize `Neon Defender` so it feels more polished, visually coherent, and mechanically deeper while preserving its mechanically distinct defender-style identity, aligning all player-facing UI and overlay language to the shared Robrary Neon Noir brand system, and keeping the entire shipped game self-contained in `docs/index.html` for GitHub Pages delivery.

## Context and Constraints
- The entire game currently lives in `docs/index.html`, including HTML, CSS, and JavaScript; single-file delivery must be preserved.
- The current build has no formal toolchain, package scripts, or automated tests in the repository root; validation must therefore rely on concrete static checks plus browser-based manual verification unless lightweight repo-local scripts are added later.
- Existing art direction only partially matches the requested Robrary aesthetic: the game currently uses `Inter` via Google Fonts, brighter neon accents, and mixed overlay/button treatments rather than a consistent grayscale-first glassmorphism system.
- The game already includes shared vocabulary candidates such as `SCORE`, `LIVES`, `PAUSED`, `MISSION FAILED`, `REBOOT`, and `INITIATE SYSTEM`, but menu hierarchy and label consistency need normalization.
- The current implementation includes defender-like wraparound movement, wave spawning, minimap/radar, smart bombs, hyperspace, touch controls, and synthesized audio. These are core identity features and should be preserved or improved rather than replaced with a generic shooter loop.
- Several implementation quirks should be accounted for during modernization because they may affect any visual/gameplay refactor: `cameraY` is referenced in grid rendering but not initialized, `PowerUp.update()` clamps against `Game.H` instead of `Game.height`, HUD status uses `BOMBS` instead of the requested `AMMO`/shared terminology, and typography still references `Inter` in canvas/UI text.
- The repo includes `README.md` and `documentation.md`; implementation may update them later if needed, but this plan assumes the primary deliverable remains the single playable file in `docs/index.html`.
- The requested brand language should be treated as authoritative: Neon Noir, grayscale foundation, restrained accent usage, `Courier New` monospace, uppercase labels, blurred glass overlays with white borders/soft glow, hover-lift buttons, terminal-style HUD copy, and consistent overlay verbs such as `INITIATE`, `DEPLOY`, `PAUSED`, `MISSION FAILED`, `REBOOT`, and `WAVE X` announcements.

## Assumptions
- `Neon Defender` is one of multiple "Jules" / Robrary games, and this plan should align its menu/overlay structure with a shared pattern even though the other repos are not present here for direct comparison.
- "Mechanically distinct" means preserving defender-specific movement, horizontal wrap, radar awareness, hyperspace/bomb risk-reward, and enemy pressure rather than simplifying the game into a stationary arena shooter.
- Because the user requested a build-ready plan artifact rather than implementation, no source changes beyond this plan file should be made now.
- Since there is no existing automated test harness, build mode may need to add lightweight verification scripts or reuse simple shell/Node-based assertions as long as the production game still ships as one HTML file.
- `docs/index.html` remains the canonical artifact for GitHub Pages, so any optional extraction/refactor work during implementation must end with an inlined final file, not a multi-file deployed bundle.
- Shared Robrary brand consistency likely matters more for menus, overlays, HUD language, and motion cues than for removing all gameplay-specific accent colors from enemy/projectile feedback.

## Deliverables
- A refreshed `docs/index.html` with upgraded visual system, gameplay tuning, and branded menu/HUD/overlay language while preserving single-file delivery.
- A unified start/pause/game-over/menu flow that mirrors the other Jules games structurally, using consistent Robrary copy and interaction patterns.
- Updated HUD language using terminal-style labels such as `SCORE`, `HI-SCORE`, `WAVE`, `LIVES`, and `AMMO` where applicable.
- Branded wave-transition and system-state announcements, including `WAVE X`, `PAUSED`, `MISSION FAILED`, `INITIATE`, `DEPLOY`, and `REBOOT` messaging.
- Concrete validation commands and manual test coverage suitable for a build agent to execute before handing off.
- Optional documentation touch-ups only if needed to reflect changed controls, menu wording, or deployment expectations.

## Todo Checklist
- [ ] Audit the current UI flow in `docs/index.html` and map it to the shared Jules/Robrary menu structure (title, controls, start, pause, game over, restart, quit).
- [ ] Define a Robrary Neon Noir design token block in `docs/index.html` using grayscale-first variables, one restrained accent family, glow values, border opacity, and blur settings.
- [ ] Replace external `Inter` usage with `Courier New`, `Courier`, `monospace` across HTML/CSS and canvas-rendered text so the game stays visually consistent without external font dependency.
- [ ] Redesign overlays and buttons to use uppercase terminal labels, glassmorphism panels, white hairline borders, soft glow, and hover-lift motion while preserving mobile usability.
- [ ] Normalize menu and overlay copy to the approved vocabulary (`INITIATE`, `DEPLOY`, `PAUSED`, `MISSION FAILED`, `REBOOT`, `WAVE X`) and remove one-off phrasing such as `INITIATE SYSTEM` if it conflicts with the shared standard.
- [ ] Expand the HUD to show `HI-SCORE`, `WAVE`, `LIVES`, and `AMMO`/special inventory status in a terminal-style layout without obscuring gameplay.
- [ ] Add a branded wave announcement/state banner system so wave transitions feel deliberate and aligned with the Robrary interface language.
- [ ] Tune gameplay feel and spectacle without losing defender identity: improve enemy pacing, feedback readability, projectile clarity, power-up signaling, and risk-reward systems already present.
- [ ] Preserve and polish mechanically distinctive systems such as horizontal wrap, minimap/radar awareness, smart bomb, hyperspace, and touch controls rather than removing them.
- [ ] Fix known implementation inconsistencies discovered during inspection (`cameraY`, `Game.H`, font references, terminology drift) as part of the modernization pass.
- [ ] Ensure any refactor still produces a single deployable `docs/index.html` with inline CSS/JS and no new runtime asset dependencies.
- [ ] Add or document concrete automated validation steps that build mode can run locally, plus a manual gameplay verification checklist for desktop and mobile layouts.

## Implementation Plan
1. Establish the target brand system inside `docs/index.html`.
   - Replace the current `Inter` import and bright multi-accent defaults with a self-contained typography and token system based on `Courier New`.
   - Define CSS variables for background layers, panel fill, white border alpha, soft glow, text hierarchy, muted grayscale values, and one restrained accent color reserved for important states.
   - Introduce shared utility styles for overlay panels, HUD chips, button hover-lift, focus states, and state banners so every menu/panel uses the same visual grammar.

2. Rework menu architecture to match the shared Jules structure.
   - Convert the start screen into a Robrary-styled mission boot panel with title, concise control grid, and primary/secondary actions using consistent verbs.
   - Update pause and game-over overlays to use the same panel scaffolding, spacing, labels, and action order as the start screen.
   - Add any missing structural affordances expected by the shared menu model, such as a persistent subtitle/status line, unified action stack, or control legend panel.
   - Keep all overlays touch-friendly and legible on narrow screens.

3. Normalize terminal-style copy across HUD and overlays.
   - Replace mixed labels (`BOMBS`, ad hoc prompts) with the approved terminal vocabulary.
   - Add `HI-SCORE` persistence strategy; default to `localStorage` if permitted, with a safe fallback when storage is unavailable.
   - Surface `WAVE` explicitly in the HUD and ensure wave announcements appear both visually and, if appropriate, via subtle audio/flash cues.
   - Ensure all user-facing labels are uppercase and consistent between keyboard and touch contexts.

4. Modernize in-game visuals without flattening gameplay identity.
   - Refresh background treatment from a simple grid-and-stars layout to a more intentional Neon Noir scene: restrained parallax, grayscale atmospheric shapes, scanline/noise-like subtlety, and stronger depth separation.
   - Update player, enemy, projectile, explosion, radar, and power-up rendering so each element is easier to read in motion while staying mostly grayscale with selective accent emphasis.
   - Move the minimap/radar and HUD into cohesive glassmorphism containers rather than standalone boxes.
   - Add lightweight, meaningful motion such as wave-entry banners, overlay fades, and button lifts rather than excessive generic animation.

5. Deepen gameplay feel while preserving defender distinctiveness.
   - Review player acceleration/friction/max-speed tuning so movement remains momentum-based but more deliberate and responsive.
   - Improve wave composition and difficulty ramping so later waves change pressure, not just enemy count; examples include spawn cadence variation, enemy firing rhythm, or mixed behaviors while keeping the core loop readable.
   - Clarify the role of hyperspace, smart bomb, and power-ups through HUD language, effects, and pacing rather than removing them.
   - Consider adding branded feedback layers like hit flashes, warning pulses, and wave-clear celebrations that reinforce the noir terminal fantasy.

6. Stabilize code paths that could undermine the modernization effort.
   - Correct rendering/state bugs found during inspection before or during the visual pass, especially undefined or inconsistent properties affecting layout and play feel.
   - Consolidate UI update logic so HUD, overlays, wave announcements, and inventory/status indicators update from a predictable set of helpers.
   - If implementation temporarily splits concerns for maintainability, end by re-inlining CSS/JS into `docs/index.html` so delivery remains single-file.

7. Validate across desktop, mobile, and deployment constraints.
   - Verify keyboard play, touch controls, pause/resume, restart/quit, wave progression, wraparound, minimap behavior, smart bomb, hyperspace, and game-over flow.
   - Confirm there are no external runtime dependencies beyond the single HTML file.
   - Recheck that layout, copy, and contrast remain consistent with the Robrary brand at common viewport sizes.

## Files and Areas to Touch
- `docs/index.html`
  - HTML overlay structure for start, pause, game-over, HUD, minimap, and wave/system announcements.
  - Inline CSS for design tokens, typography, panels, responsive layout, hover/focus states, and glassmorphism styling.
  - Inline JavaScript for HUD copy, menu flow, wave announcements, high-score persistence, gameplay tuning, bug fixes, and mobile/desktop state handling.
- `README.md`
  - Update only if implementation changes player-facing control names, menu wording, or feature descriptions.
- `documentation.md`
  - Update only if implementation materially changes technical architecture, validation workflow, or deployment assumptions.

## Validation and Test Criteria
- Automated checks
  - Run `python3 -m http.server 8000 -d docs` from the repository root and verify the server starts without missing-file errors.
  - Run `python3 - <<'PY'
from pathlib import Path
html = Path('docs/index.html').read_text()
required = ['COURIER NEW', 'HI-SCORE', 'WAVE', 'LIVES', 'PAUSED', 'MISSION FAILED', 'REBOOT']
missing = [token for token in required if token not in html.upper()]
assert not missing, f'Missing required UI tokens: {missing}'
assert '@import url(' not in html, 'External font import should be removed for single-file self-contained delivery'
print('static checks passed')
PY`
  - Run `python3 - <<'PY'
from pathlib import Path
html = Path('docs/index.html').read_text()
for needle in ['<style>', '<script>', 'id="start-screen"', 'id="pause-screen"', 'id="game-over-screen"']:
    assert needle in html, f'Missing expected single-file structure marker: {needle}'
print('structure checks passed')
PY`
  - If implementation adds npm-based validation helpers, run them from the repo root and document exact commands in the final implementation notes, but do not make them required unless they are committed.
- Manual verification
  - Desktop: open `http://localhost:8000/` in a desktop browser, confirm the start/menu layout uses Robrary styling, uppercase labels, glass panels, grayscale-first palette, and hover-lift buttons.
  - Desktop: start a run and verify HUD shows `SCORE`, `HI-SCORE`, `WAVE`, `LIVES`, and any ammo/special inventory labels without overlapping the playfield.
  - Desktop: confirm player movement remains momentum-based with horizontal wraparound, minimap/radar remains readable, and the game still feels defender-like rather than arena-bound.
  - Desktop: clear at least one wave and verify a visible `WAVE X` announcement appears, then pause and confirm the `PAUSED` overlay matches the same menu structure and language system.
  - Desktop: use smart bomb/hyperspace and verify both remain functional, visually legible, and represented with consistent HUD/menu terminology.
  - Desktop: lose all lives and confirm the `MISSION FAILED` screen appears with `REBOOT` and shared menu actions in the correct order.
  - Mobile/narrow viewport: test in browser responsive mode at approximately `390x844` and `768x1024`; confirm overlays fit on screen, touch controls remain reachable, and the HUD/minimap do not clip essential content.
  - Regression: refresh the page after setting a non-zero score and confirm `HI-SCORE` persists if local storage is implemented.

## Risks and Open Questions
- The shared Jules menu structure is referenced but not available in this repository, so implementation will need to infer the pattern from naming and the Robrary aesthetic unless another repo/spec is provided.
- Over-indexing on grayscale could reduce gameplay readability if enemy bullets, power-ups, and damage cues lose contrast; accent use should stay restrained but purposeful for threats and rewards.
- Deepening gameplay in a single-file script can make maintenance harder unless UI/state helpers are carefully organized before re-inlining.
- High-score persistence via `localStorage` is likely appropriate, but this should be confirmed if the shared Jules games use a different persistence pattern.
- If build mode chooses to add automated browser tests, it must avoid introducing a multi-file production dependency or changing deployment away from `docs/index.html`.
