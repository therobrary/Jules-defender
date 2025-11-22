# Technical Documentation

## Overview
**Neon Defender** is a single-page web game built without any external dependencies or assets. It relies entirely on browser-native APIs.

## Tech Stack
*   **HTML5 Canvas**: Used for all rendering (vectors, particles, text).
*   **Vanilla JavaScript (ES6)**: Game logic, physics, and state management.
*   **Web Audio API**: Real-time sound synthesis (Oscillators for lasers, Noise buffers for explosions).
*   **CSS3**: UI overlay styling and responsive layout.

## Code Structure (`docs/index.html`)
The entire application is contained in `docs/index.html` to simplify deployment and portability.

### Key Components
1.  **Game Engine (`Game` object)**:
    *   Manages the main loop (`requestAnimationFrame`).
    *   Handles the entity list, collision detection, and wave management.
    *   `draw()`: Renders the world, handling the "world wrap" logic where entities are drawn multiple times or offset to appear seamless.
2.  **Entity System**:
    *   Base `Entity` class.
    *   `Player`: Handles momentum physics (`vx`, `vy`) and input.
    *   `Lander`: Simple enemy AI with drift and bobbing.
    *   `Bullet` & `Particle`: Transient objects.
3.  **Input System**:
    *   Maps Keyboard events (`ArrowKeys`, `Space`) and Touch events (Virtual D-pad) to control states.
4.  **Audio System**:
    *   `AudioSys` creates an `AudioContext` on the first user interaction to comply with browser autoplay policies.

## Deployment Instructions (GitHub Pages)

To deploy this game on GitHub Pages:

1.  Push the repository to GitHub.
2.  Go to the repository **Settings**.
3.  Navigate to the **Pages** section (on the left sidebar).
4.  Under **Build and deployment** -> **Source**, select **Deploy from a branch**.
5.  Under **Branch**, select `main` (or your default branch) and select the **/docs** folder.
6.  Click **Save**.

The game will be live at `https://<username>.github.io/<repo-name>/`.
