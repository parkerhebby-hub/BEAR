# BEAR — Architecture & Development Guide

> Single-file HTML5 Canvas 2D stealth-survival game set in Manning Park, BC.
> Player controls a grizzly bear — stealth, combat, survival, Stardew Valley pixel art style.

---

## File Structure

```
BEAR GAME/
├── index.html              # ALL game code (8,250+ lines, ~324KB)
├── serve.js                # Node.js static file server (port 3001, URL-decoded paths)
├── extract-gif-frames.js   # GIF→PNG sprite strip extraction tool
├── package.json            # Dependencies: gifuct-js
├── CLAUDE.md               # This file
├── BEAR/                   # 24 source GIF animations (60x60 per frame)
│   ├── walk-6-frames_{dir}.gif     (8 directions)
│   ├── run-6-frames_{dir}.gif      (8 directions)
│   └── eating-11-frames_{dir}.gif  (8 directions)
├── BEAR_FRAMES/            # 24 extracted PNG sprite strips
│   ├── walk-6-frames_{dir}.png     (360x60, 6 frames)
│   ├── run-6-frames_{dir}.png      (360x60, 6 frames)
│   └── eating-11-frames_{dir}.png  (660x60, 11 frames)
├── HUMAN_FRAMES/           # Human animation PNG strips (8-direction walk/run)
│   ├── HIKER_BACKPACK_walking-6-frames_{dir}.png
│   ├── HIKER_BACKPACK_running-6-frames_{dir}.png
│   └── HIKER_BACKPACK_rotations_8dir.png
├── INSPIRATION/            # 102 sprite assets
│   ├── TREE_1-4.png, TREE_*_SMALL.png, TREE_*_STUMP*.png
│   ├── BUSH_LARGE/SMALL_1/SMALL_2, BUSH_BERRIES/NO_BERRIES
│   ├── BIRD_FLIGHT_LR/DOWN/UP.png, BIRD_IDLE.png
│   ├── SQUIRREL_RUN/IDLE.png, FISH.png, BAT_ANIMATION_4_FRAMES.png
│   ├── CABIN_STONE/LOG/ABANDONED.png (per-type sprites)
│   ├── TRAILER_1.png, TRAILER_2.png (79x74 each)
│   ├── LODGE.png (192x160)
│   ├── MANNING_TEMPLATE.png (256x256 color-coded map)
│   ├── FIRE/SPLASH/SMOKE_ANIMATION.png
│   ├── TENT_VERTICAL.png, CAMPFIRE.png, DUMPSTER.png
│   ├── GARBAGE.png, STONE_PATHWAY.png, LANTERN.png, BONES.png
│   ├── BUTTERFLY_ANIMATION.png, BUTTERFLY_ANIMATION_2.png
│   ├── BERRY_BUSH_2.png, LARGE_FERN.png, MUSHROOMS.png
│   ├── LOGS_SMALL.png, LOG_LARGE.png, STUMP_2X2.png
│   ├── WOOD_BUNDLE.png (16x24, campsite decor near campfires)
│   ├── PICNIC BENCH 1X2.png, HAMMOCKS.png, BOBBERS.png
│   ├── DIRT_1X1.png, DIRT_1X3.png, GRASSES.png, SLRG.png
│   ├── GRASS_CENTRE.png, GRASS_CENTRE_ALT.png, GRASS_GRASS.png, GRASS_WEED.png
│   ├── GRAVEL_CENTRE.png, GRAVEL_CENTRE_ALT.png, GRAVEL_EDGE*.png
│   ├── TRAIL_CENTRE.png, TRAIL_ROCKS.png, TRAIL_GRASS.png
│   ├── TRAIL_GRASS_ROCKS.png, TRAIL_GRASS_ROCKS_ALT.png
│   ├── GRASS-TRAIL transition tiles (edges, corners, outside corners)
│   ├── ROAD_CENTRE.png, ROAD TOP/SIDE/CORNER tiles
│   ├── WATER_EDGE_1-4.png
│   ├── CAVE_FLOOR.png, CAVE_WALL_TOP.png
│   ├── WALLS_FLOORS.png, FURNITURE.png, TOWN INTERIORS.png
│   ├── HEALTH_SPRITE.png, HUNGER SPRITE.png (HUD icons)
│   └── ISLAND.png, PELICAN TOWN.png (reference images)
├── TILE LAYOUTS/           # PNG-based structure templates
│   ├── CAMPSITES/
│   │   ├── CAMPSITE 12X12.png
│   │   ├── CAMPSITE 12X12_ALT_1.png
│   │   └── CAMPSITE 12X12_TRAILER_1.png
│   ├── CAVE_SPAWN.png
│   └── LODGE.png
├── MUSIC/                  # Audio assets
│   ├── AMBIENT FOREST/     # Zone-based ambient loops
│   │   ├── DAY CLEAR LOOP.mp3, DAY RAIN LOOP.mp3
│   │   └── NIGHT CLEAR LOOP.mp3, NIGHT RAIN LOOP.mp3
│   ├── SFX/                # Sound effects
│   │   ├── BEAR ATTACK HEAVY 1-3.mp3, BEAR SNARL DURING COMBAT.mp3
│   │   ├── BEAR EATING BERRIES.mp3, BEAR RUNNING.mp3
│   │   ├── CAMPFIRE LOOP.mp3, CAMPSITE PEOPLE TALKING LOOP.mp3
│   │   ├── BUSHES RUSTLING.mp3, HUMAN FOOTSTEPS.mp3
│   │   └── BEAR ATTACKING WOOD DOORS.mp3, BEAR ATTACK NEAR CAMPSITE.mp3
│   ├── WILDERNESS (SPAWN).mp3, WILDERNESS 2.mp3
│   ├── BACKCOUNTRY.mp3, BACKCOUNTRY 2.mp3
│   ├── CAMPGROUNDS.mp3, CAMPGROUNDS 2.mp3
│   ├── OUTPOSTS.mp3, OUTPOSTS 2.mp3
│   └── LODGE.mp3, LODGE 2.mp3
└── .claude/launch.json     # Dev server config (node serve.js, port 3001)
```

---

## Core Systems

### Sprite Pipeline
1. **SHEETS** object: maps key → `{src, img}`. `loadAllSprites()` loads all as `new Image()`
2. **ATLAS** object: defines frame regions `{sheet, x, y, w, h}` within each sheet
3. **drawSprite(ctx, atlas, sx, sy, ax, ay)**: draws with anchor + integer pixel enforcement
4. **spriteVar(wtx, wty, count, salt)**: deterministic hash for variant selection per tile

### Bear Animation (PNG Strip System)
- **Source**: 24 GIFs in `BEAR/` → extracted to PNG strips in `BEAR_FRAMES/` via `extract-gif-frames.js`
- **BEAR_STRIPS**: lookup table `{walk/run/eat}` → `{N/NE/E/.../NW}` → SHEETS key + frame count
- **getBearDir8(fx, fy)**: resolves facingX/facingY to 8 compass directions
- **Frame animation**: `ctx.drawImage(strip, frame*fw, 0, fw, fh, bx, by, dw, dh)` at `performance.now()`-based timing
- **Render size**: 64x64 pixels (scaled from 60x60 source frames), bottom-center anchored
- **Anchor offset**: `by = (sy - 16 - 64)` — 16px offset compensates for ~18px dead space below feet in source frames. Visual feet land at ~`sy - 34` on screen (row 43/60 of source)
- **CFG.BEAR_FOOT_OFS = -34**: all game logic (collision, tile detection, Y-sort, attacks, AI, interactions) uses `bear.y + BEAR_FOOT_OFS` to align with visual feet
- **Effect alignment**: splash/dust at `bear.y - 32`, eat particles at `bear.y - 38`, swim ripple at `sy - 34`, attack swipes at `sy - 32`
- **Speed**: walk=0.15s/frame, run=0.1s/frame, eat=0.12s/frame (doubled to 0.24s for human meat)
- **Idle**: freezes on frame 0 of walk animation in last-faced direction
- **Eating animation**: plays full 11-frame cycle once per eat; double duration for human meat via `eatStart` timestamp

### Map System
- **MANNING_TEMPLATE.png** (256x256): color-coded map, each pixel = 2x2 game tiles
- **parseMapTemplate()**: reads pixel colors → maps to tile types + zone markers; flood-fills contiguous campsite regions to place single anchor per region
- **mapTiles**: `Uint8Array[512x512]` — base terrain tiles
- **mapMarkers**: `Uint8Array[512x512]` — zone markers (0=none, 1=dense_forest, 2=campsite, 3=outpost, 4=resort)
- **512x512 tile grid** = 16x16 chunks of 32x32 tiles each, tile size = 16px → 8192px world
- **Fixed seed 42** for deterministic generation. Cached chunks persist across restarts
- **5 zone rings**: zone 1 (outer/easy) → zone 5 (center/hard), thresholds at norm 0.78/0.58/0.38/0.18
- **Zone names**: Wilderness, Backcountry, Campgrounds, Outpost, Resort

### Color Map Legend (MANNING_TEMPLATE.png)
| Color | RGB Range | Tile Type | Marker |
|-------|-----------|-----------|--------|
| Black | (0,3,11) | GRASS | none |
| Green | (41,255,31) | GRASS | dense_forest (1) |
| Red | (253,17,24) | DIRT (trail) | none |
| Light blue | (31,207,255) | WATER_S (shallow) | none |
| Dark blue | (36,31,255) | WATER_D (deep) | none |
| Yellow | (255,211,19) | PAVEMENT (road) | none |
| Orange | (255,141,0) | PAVEMENT | resort (4) |
| Pink | (R>180,B>100,G<130) | GRAVEL | outpost (3) |
| Purple/Magenta | (141,31,255) | DIRT | campsite (2) |
| Tan | (255,244,200) | GRAVEL (sand) | none |
| Gray | (144,144,144) | GRAVEL | none |
| Brown | (71,47,0) | BRIDGE | none |

### Zoom System
- **4 zoom levels**: toggled with `-`/`=` keys or scroll wheel
- **Level 0** (default): 426x240 internal resolution (~27 tiles wide, close-up)
- **Level 1**: 640x360 (~40 tiles wide)
- **Level 2**: 1280x720 (~80 tiles wide)
- **Level 3**: 2560x1440 (~160 tiles wide, ultra-wide)
- **Center-preserving zoom**: scroll/keyboard zoom preserves world point at viewport center (computes center world coord before zoom, re-centers after resize)
- **Ground cover despawn**: at zoom level 3+, small ground cover (bushes, logs, ferns, stumps) skipped — only structures (campfires, tents, trailers, wood bundles) rendered
- **HUD stays at base scale**: `renderHUD()` applies `ctx.scale(W/640, W/640)` so bars, minimap, and paw buttons remain constant size
- **Light map canvas** resizes to match zoom level
- **HUD_BUTTONS** positions scaled for click/touch hit detection at any zoom

### Entity Generation (per chunk)
- **Trees**: density scales by zone + dense_forest marker (12+ clusters in dense zones vs 1-7 elsewhere)
- **Bushes**: large (2x2), small (1x1), berry bushes — `bushOccupied` Set prevents 2x2 overlap
- **Ground cover**: all placed via seeded RNG (`r()`) with attempt-based random positioning to avoid patterns
  - Small trees, small logs, ferns, stumps, large logs — NO deterministic hash placement (prevents visible lines)
  - 2x2 items (stumps, logs) have minimum 5-tile spacing between each other
  - `occupied2x2` and `occupied1x1` Sets prevent all overlap
- **Mushrooms**: 50% shrunk, 50% reduced spawn, edible +8 hunger, 90s respawn
- **Campsites**: ONLY placed in map-marked zones (purple=campsite, pink=outpost, orange=resort); PNG layout templates define tile placement
- **Campsite decor**: 1-2 wood bundles per campfire on nearby GRASS tiles, avoiding tent entrances
- **Cabins**: placed in map-marked zones; types vary by zone (abandoned, log, stone, trailer)
- **Cave clearance**: 6-tile buffer around all cave entrances, cross-chunk boundary aware (iterates global `caveSpawnPoints`)
- **Humans**: spawn only on trail tiles (DIRT, GRAVEL, PAVEMENT, BRIDGE) via `hSpawnHumansOnTrails()`
- **Wildlife**: birds, squirrels, fish, butterflies, bats — spawned dynamically near player; half-size rendering

### Campsite Layout System
- **parseCampsiteLayouts()**: loads 3 PNG templates from SHEETS (campsiteLayout1/2, campsiteTrailer1)
- **Color-to-tile mapping**: pink→TENT/TRAILER, other colors→DIRT/CAMPFIRE/ENTRANCE/GARBAGE/PICNIC_TABLE
- **Trailer metadata**: during parsing, scans for contiguous T.TRAILER block, stores `trailerBlock = {lx, ly, w, h}`
- **Layout stamping in generateChunk()**: for each campsite anchor, selects layout deterministically, stamps tiles into chunk, registers trailer positions via `_layoutTrailers` array
- **Cross-chunk support**: trailers registered during stamping (not tile-scanning) so chunk-boundary trailers always render
- **campsiteFootprint Set**: tracks all tiles belonging to stamped layouts to prevent tree/decoration overlap

### Human AI (6 classes)
| Class | HP | Armed | Behavior | Spawn Location |
|-------|-----|-------|----------|----------------|
| SOLO_HIKER | 30 | No | PATROL | Trails |
| CASUAL_CAMPER | 25 | No | IDLE | Campsites |
| GROUP_HIKER | 35 | No | PATROL | Trails, Campsites |
| HUNTER | 60 | Yes | PATROL | Trails (zone 3+) |
| RANGER | 80 | Yes | PATROL | Outpost zones |
| TOURIST | 20 | No | IDLE | Resort zones |

**Detection states**: IDLE → SUSPICIOUS (0.25) → INVESTIGATE (0.55) → ALERT (0.85)
**Vision**: cone-based (range + angle), hearing range separate, noise level affects detection rate
**Patrol AI**: generates patrol points from nearby trail/path tiles (not random offsets)
**Movement restrictions**: humans cannot walk on water (WATER_S/WATER_D) or through 2x2 obstacles (stumps, large logs) via `humanBlocked` Set
**Night shelter**: at darkness > 0.5, unaware humans near shelter tiles (TENT/CABIN/TRAILER) become `_sheltered` — invisible, silent, skip AI. Return to position at dawn (darkness < 0.5)

### Sound Propagation System
- **SOUND_EMIT profiles**: per AI state — UNAWARE (8s), SUSPICIOUS (6s), INVESTIGATING (4.5s), PANICKED (2.5s), DEFENSIVE (3s)
- **Wall-clock timing**: uses `performance.now() * 0.001` for frame-rate-independent intervals (same on mobile and desktop)
- **Initial stagger**: first call sets random future emit time (prevents all humans pulsing simultaneously on load)
- **Group proximity**: humans within 80px reduce each other's interval by 15% per neighbor (min 70% of base)
- **Random jitter**: ±15% variation on each interval (`0.85 + Math.random() * 0.3`)
- **Distance culling**: only emits if human is within 2× max radius of bear
- **Sheltered silence**: sheltered humans emit no sound waves
- **Ring rendering**: expanding circles with terrain attenuation, fade starting at 70% of max radius
- **Pool limit**: max 40 concurrent waves; oldest shifted when full

### Combat
- **Light attack**: 20 dmg, 18 range, 0.4s CD, 12 stamina, 4px knockback
- **Heavy attack**: 55 dmg, 22 range, 1.0s CD, 30 stamina, 8px knockback
- **Knockback**: normalized bear→human direction vector, low magnitude so enemies die in place
- **Human damage**: 15 per hit (armed humans only)
- **On-screen buttons**: Bear paw HUD (see HUD section)

### Death & Drops
- **Bones**: 1-2 bone sprites dropped at exact death location (`deathX`, `deathY`)
- **Blood stains**: pixelated fillRect pool, grows from 16x16 to 32x32 over 8s, persists 1-2 in-game days (270-540s)
- **Human meat**: dropped at death location, +60 hunger
- **No inventory drops**: humans don't drop 'S' or 'C' letter items on flee

### Survival
- **HP**: 100 max, regenerates slowly
- **Stamina**: 100 max, 12/s regen, sprint drains 25/s
- **Hunger**: 100 max, drains at 0.5/s, eat food to replenish
- **GOD_MODE** (`CFG.GOD_MODE = true`): infinite HP/stamina/hunger for playtesting

### Food Sources
| Food | Hunger | Source |
|------|--------|--------|
| Wild Berries | +12 | Berry bushes (E to interact, respawn 60s) |
| Granola Bar | +15 | Hiker drops |
| Sandwich | +25 | Hiker/camper drops |
| Cooler Food | +35 | Camper/tourist drops |
| Salmon | +40 | Fish (not yet catchable) |
| Meat | +60 | Human kills |
| Mushroom | +8 | Mushroom sprites (E to interact, respawn 90s) |
| Garbage | +/- 0 | Garbage tiles (random effect) |

### Day/Night Cycle
- **4.5 min total cycle**: 135s day → 25s dusk → 85s night → 25s dawn
- **dayTime** variable accumulates in `updateDayNight(dt)`, wraps at 270s
- **dayNightDarkness**: 0 (day) to 1 (night), smoothstep transitions during dusk/dawn
- **Light map**: offscreen canvas (resizes with zoom) filled with `rgba(10,10,40,alpha)`, light sources punched via `destination-out` compositing
- **Light sources**: campfires (55px + flicker), tents (28px faint), trailers (faint glow like tents) — no bear ambient glow
- **Campfire/lantern glow**: existing radial gradients scale by `nightBoost = 1 + darkness * 1.5`
- **Fireflies**: separate array from particles, spawn when darkness > 0.3, max 40, sine-wave drift, pulsing yellow-green glow (no aura), doubled lifespan (8-20s), rendered ON TOP of darkness overlay
- **HUD indicator**: sun/moon icon with progress arc, top-right above minimap

### Camera
- **Lerp-based smooth follow**: `lerpFactor = Math.min(1, 3.0 * dt)`
- **Bear-centered**: target = `bear.y - H/2 - 48` (offsets for sprite anchor so visual bear body is centered)
- **Snap threshold**: when within 0.5px of target, snaps directly to prevent float oscillation/jitter
- **Clamped to map boundaries**: camera never shows outside the map
- **Integer-rounded**: `Math.round()` on final position to prevent sub-pixel jitter
- **Middle-click pan** (desktop only): scroll wheel press + drag detaches camera from bear; pans freely with pixel-accurate scaling; camera snaps back to bear on WASD/arrow movement input
- **Pan state**: `panActive` (dragging), `panDetached` (free-roaming), stored start positions for delta calculation
- **Spawn positioning**: bear spawns inside cave interior; camera centers on interior map

### HUD — Bear Paw Design
- **Layout**: bottom-right corner, pixelated bear paw shape
- **Main pad** (sprint): large oval, 26x16 px drawn via fillRect scanlines
- **5 toe pads** (left to right): Interact, Light Attack, Heavy Attack, Climb, Sneak
- **Triangular claws**: 5px tall, tapered from 3px base to 1px tip above each toe
- **Scale-independent**: HUD always renders at 640x360 base resolution regardless of zoom level
- **Input**: R key for climb (separated from E interact), sneak button toggles `sneakHeld`
- **Hit detection**: `HUD_BUTTONS` stores scaled positions for click/touch at any zoom

### Rendering
- **Y-sorted depth**: all entities sorted by Y coordinate for proper overlap
- **isBearBehind()**: trees/bushes become semi-transparent (0.45 alpha) when bear walks behind them
- **Integer pixel enforcement**: all draw positions use `| 0` bitwise truncation
- **Odd-width centering**: uses `>> 1` (integer division) instead of `/ 2` to avoid 0.5px offsets
- **Grass tiles**: rendered using GRASS_CENTRE.png with doubled GRASS_GRASS frequency (spriteVar values 1 and 3) plus GRASS_CENTRE_ALT and GRASS_WEED variants
- **Trail tiles**: adjacency-based transition sprites (edges, corners, outside corners) + centre variants (TRAIL_ROCKS, TRAIL_GRASS, TRAIL_GRASS_ROCKS, TRAIL_GRASS_ROCKS_ALT); uses `getWorldTile()` for cross-chunk boundary checks
- **Bridge tiles**: horizontal wood plank pattern drawn with fillRect over water base color, with side rails
- **Cabin collision**: bottom 4 rows are solid (T.CABIN), top 2 rows are roof overhang (walkable)
- **Campfire alignment**: sprite and fire animation shifted down 6px from tile center for visual alignment on 1x1 tiles
- **2x2 sprite helper**: `draw2x2Sprite()` for stumps, large logs, ferns, berry bushes
- **2x2 overlap prevention**: `occupied2x2` Set + `bushOccupied` Set for berry bushes; 1x1 decorations skip these tiles
- **Wildlife rendering**: birds/squirrels at half size (`w >> 1`, `h >> 1`); butterflies at half size with center-aligned frames
- **Night overlay**: rendered after particles, before HUD — HUD stays fully visible
- **Minimap**: top-right corner, shows tiles color-coded, zone label, bear position dot
- **Full map view**: press M, incremental chunk rendering (16/frame to prevent freezing)

### Interior System
- **Cave interiors**: generated tile maps with wall border, floor, furniture
- **Cabin interiors**: generated tile maps (12-16 wide, 8-10 tall) with wall border, dirt floor, furniture
- **Transitions**: fade-to-black enter/exit with `transitionCooldown` to prevent flicker
- **Door detection**: `isNearCabinDoor()` checks adjacent tiles for cabin door position
- **Camera**: centers on interior map dimensions
- **Cave spawn**: bear starts inside a randomly selected cave on game start; exits to overworld
- **Cave trigger tracking**: `bear._wasOnCaveTrigger` prevents re-entry flicker when exiting

### Cave System
- **Cave spawn points**: parsed from CAVE_SPAWN.png template, placed at map-marked locations
- **6-tile clearance**: trees/decorations cleared around cave entrances, cross-chunk boundary aware
- **2x3 entrance**: cave entrance tiles on overworld, detected by proximity for E-key interaction
- **Interior generation**: `createCaveInterior(hash)` generates unique interior layout per cave

### Weather System
- **States**: CLEAR → RAIN (with transitions)
- **Timing**: configurable min/max durations per state
- **Effects**: cloud overlay alpha, rain particle density, ambient audio switching
- **Transitions**: smooth intensity ramp-up/down

---

## Performance Architecture

### Frame Loop
- **~83 FPS cap**: 12ms minimum between frames (`ts - lastTime < 12` guard)
- **Delta time clamped**: `Math.min(dt, 1/20)` prevents physics jumps on tab resume
- **Try/catch**: loop errors logged but don't crash the game

### Chunk System
- **Pre-rendered chunk canvases**: each 32x32 tile chunk (512x512px) rendered once to an offscreen canvas (`chunkCanvases{}`)
- **Frustum culling**: only chunks within viewport + 2-chunk margin are drawn
- **Background generation**: 4 chunks/frame during PLAYING state (bear starts in cave while world generates)
- **Lazy loading**: `ensureChunks()` generates 3x3 grid around bear for on-demand chunk creation
- **Persistent cache**: generated chunks never regenerated; `chunks{}` and `chunkCanvases{}` persist

### Tile Adjacency
- **getWorldTile()**: bypasses `interiorState` routing for cross-chunk boundary checks during background generation; falls back to `mapTiles` for ungenerated chunks
- **8-direction checks**: grass-trail transitions sample all 8 neighbors using `getWorldTile()`

### Render Optimizations
- **Zoom-level LOD**: at zoom 3+, small ground cover skipped (only structures rendered)
- **Viewport culling**: per-sprite screen-space bounds check before adding to renderList
- **Y-sort once**: single sort of renderList per frame, all entity types intermixed
- **Light map**: separate offscreen canvas composited once, resized with zoom level
- **Integer coordinates**: all draw positions truncated to prevent sub-pixel anti-aliasing overhead

### Memory
- **Typed arrays**: `mapTiles` and `mapMarkers` use `Uint8Array` for compact storage
- **Set-based lookups**: `occupied1x1`, `occupied2x2`, `humanBlocked`, `campsiteFootprint` for O(1) collision checks
- **Sound wave pool**: capped at 40; oldest waves shifted on overflow

---

## Key Conventions

1. **All code in one file** — index.html contains HTML, CSS, and all JS
2. **No frameworks** — vanilla Canvas 2D API, no build tools
3. **Pixel-perfect rendering** — `image-rendering: pixelated`, `imageSmoothingEnabled = false`
4. **Integer coordinates** — always `| 0` on draw positions, `>> 1` for centering
5. **Bottom-center anchoring** — sprites positioned by their feet, not top-left
6. **Seeded RNG placement** — fixed seed 42 + `r()` calls for all ground cover/entity positioning (no deterministic hash modulus which creates visible patterns)
7. **Chunk caching** — generated chunks persist in `chunks{}` object
8. **GOD_MODE** — toggle at `CFG.GOD_MODE` for playtesting
9. **Map-driven structure placement** — campsites/cabins only where MANNING_TEMPLATE markers specify
10. **Trail-bound humans** — humans spawn and patrol only on trail/path tiles
11. **Wall-clock timing** — sound emission and other real-time systems use `performance.now()` for frame-rate independence
12. **`var` for large function locals** — use `var` (not `const`/`let`) for variables in `generateChunk()` to avoid V8 scoping issues in very large functions

---

## Running the Game

```bash
cd "BEAR GAME"
node serve.js
# Open http://localhost:3001
```

**Controls**:
- **Movement**: WASD/arrows
- **Sprint**: Shift (hold)
- **Sneak**: Ctrl/C (toggle)
- **Interact**: E (bushes, garbage, cabin doors, caves)
- **Climb**: R (trees)
- **Light attack**: Space / click
- **Heavy attack**: Q / right-click
- **Map**: M
- **Zoom**: `-` (out) / `=` (in) / scroll wheel
- **Pan camera**: middle-click drag (desktop only, snaps back on movement)
- **Focus listen**: F (hear sound waves better, drains stamina)
- **Pause**: Esc
- **Gamepad**: supported

---

## Unintegrated Assets (loaded but not fully placed)

These sprites exist in SHEETS/ATLAS but have limited or no in-world placement:
- **DUMPSTER** — atlas defined (2x2), meant for resort zone
- **STONE_PATHWAY (6 variants)** — atlas defined, not placed on trails
- **LANTERN** — atlas defined, not placed in world
- **HAMMOCKS** — loaded, not placed
- **BOBBERS** — loaded, not placed (fishing mechanic not implemented)
- **Fishing mechanic** — not implemented
- **Swimming animation** — uses walk GIF, no dedicated swim sprites

---

## Performance Improvement Opportunities

Potential optimizations that preserve visual quality:

1. **Object pooling for particles/soundWaves** — currently uses `push()`/`splice()` which creates GC pressure; pre-allocate fixed-size arrays with active/inactive flags
2. **Spatial hash for human AI** — group proximity checks in `emitSoundWave()` iterate all humans O(n²); a grid-based spatial hash would reduce to O(n)
3. **Chunk canvas recycling** — offscreen canvases are never freed; for very long sessions, implement LRU eviction for distant chunk canvases
4. **Batch similar draw calls** — group sprites by sheet to reduce texture switching overhead
5. **Web Worker for chunk generation** — move `generateChunk()` + `prerenderChunk()` to a worker thread to eliminate frame drops during background generation (requires OffscreenCanvas)
6. **Typed array for renderList** — replace array-of-objects with struct-of-arrays for better cache locality during Y-sort
7. **Reduce light map resolution** — render light map at half resolution and upscale, since it's a blurred overlay
8. **Visibility-based human updates** — skip full AI updates for humans far from camera (>2 chunks away); only update position/patrol at reduced frequency
9. **Terrain adjacency cache** — pre-compute grass-trail transition results per tile during chunk generation instead of re-checking neighbors each prerender

---

## GIF Frame Extraction

To re-extract bear animations from GIFs:
```bash
cd "BEAR GAME"
npm install gifuct-js  # if not installed
node extract-gif-frames.js
```
Reads `BEAR/*.gif` → outputs horizontal PNG strips to `BEAR_FRAMES/`. Each frame composited with GIF disposal handling. Uses minimal PNG encoder (no canvas dependency).
