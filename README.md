# BEAR

**A stealth survival experience.**

Play as a grizzly bear navigating the wilderness of Manning Park, BC. Hunt, forage, and survive across five distinct zones while avoiding — or confronting — the humans who share the forest.

Built as a single-file HTML5 Canvas game with Stardew Valley-inspired pixel art.

## Play

```bash
git clone https://github.com/YOUR_USERNAME/bear-game.git
cd bear-game
npm install
node serve.js
```

Open [http://localhost:3001](http://localhost:3001) in your browser.

No build step required. No frameworks. Just a browser.

## Controls

| Action | Keyboard | Mobile |
|--------|----------|--------|
| Move | WASD / Arrows | - |
| Sprint | Shift (hold) | Paw pad |
| Sneak | Ctrl / C (toggle) | Toe button |
| Light attack | Space / Left click | Toe button |
| Heavy attack | Q / Right click | Toe button |
| Interact / Eat | E | Toe button |
| Climb tree | R | Toe button |
| Focus listen | F | - |
| Map | M | - |
| Zoom | `-` / `=` / Scroll wheel | - |
| Pan camera | Middle-click drag | - |
| Pause | Esc | - |
| Gamepad | Supported | - |

## Features

- **Procedural world** generated from a hand-painted 256x256 color map template, producing a 512x512 tile grid with trails, rivers, forests, campsites, and caves
- **5 zone rings** from the outer Wilderness to the inner Resort, each with increasing difficulty and human density
- **6 human classes** — Solo Hikers, Group Hikers, Casual Campers, Hunters, Rangers, and Tourists — each with unique stats, behavior, and threat level
- **Stealth system** with cone-based vision, hearing range, and detection states (Unaware > Suspicious > Investigating > Alert)
- **Sound propagation** — humans emit visible sound waves that attenuate through terrain; use Focus Listen (F) to amplify
- **Day/night cycle** with smooth lighting transitions, campfire glow, firefly spawning, and humans sheltering at night
- **Dynamic weather** — clear and rain states with ambient audio transitions
- **Combat** — light and heavy attacks with knockback, stamina management, and armed humans who fight back
- **Foraging** — eat berries, mushrooms, garbage, dropped food, or... the humans themselves
- **Cave system** — the bear spawns inside a cave and can return to caves throughout the world
- **Cabin interiors** — enter cabins and lodges with generated interior layouts
- **Full park map** (M) with zone boundaries and human positions
- **Chunk-based rendering** with pre-rendered offscreen canvases, frustum culling, and zoom-level LOD

## Tech

The entire game is a single `index.html` file (~8,250 lines). No dependencies at runtime — just vanilla JavaScript and the Canvas 2D API.

- **Sprite pipeline**: PNG sprite sheets with atlas definitions, 8-directional animation strips for bear and human characters
- **Chunk system**: 16x16 grid of 32x32 tile chunks, each pre-rendered to an offscreen canvas
- **Seeded RNG**: deterministic world generation from fixed seed 42
- **Pixel-perfect**: integer coordinates, `image-rendering: pixelated`, no sub-pixel rendering

The `serve.js` file is a minimal Node.js static file server for local development. The game itself has zero runtime dependencies.

## Project Structure

```
index.html          # All game code
serve.js            # Dev server (Node.js, port 3001)
BEAR/               # Source GIF animations (8 directions x 3 states)
BEAR_FRAMES/        # Extracted PNG sprite strips
HUMAN_FRAMES/       # Human walk/run/idle sprite strips
INSPIRATION/        # All sprite assets (terrain, structures, wildlife, UI)
TILE LAYOUTS/       # PNG templates for campsites, caves, lodge
MUSIC/              # Ambient loops, zone music, SFX
```

PARKER DANE HENDSBEE Licensed 2026

All rights reserved.
