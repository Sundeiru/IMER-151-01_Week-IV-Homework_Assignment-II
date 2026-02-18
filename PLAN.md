# Interaction Plan — IMER-151-01 Week IV Assignment II

---

## Overview

A single HTML file (`index.html`) implementing a downward-scrolling space exploration interaction inside a phone-frame viewport. The user pilots a star character through astronomical space, collecting stardust and encountering celestial objects, culminating in an ending sequence.

---

## File Output

**`index.html`** — all HTML, CSS, and JavaScript in one self-contained file.
All SVG asset markup is extracted from `Assets_Updated/assets.svg` and `Assets_Updated/setup.svg` and inlined directly.

---

## Architecture

### Rendering Approach: DOM + Canvas Hybrid

- **Background layer**: `<canvas>` (402×874) for the space gradient and procedural star field.
- **World layer**: A single `<div id="world">` container, absolutely positioned, moved via `transform: translateY(-cameraY)`. All floating game objects live here as absolutely-positioned child divs containing their inline SVG markup.
- **Star layer**: The star element is positioned in **screen space** (not world space) — it does not scroll with the world container. Its screen position is updated directly from input.
- **UI layer**: Absolutely-positioned overlays for the text bubble, glow flash, sparkles, poem text, and restart prompt.

This keeps SVG filters (blur, turbulence, etc.) rendering correctly while giving full scroll/animation control.

### Coordinate Systems

- **Screen space**: 402×874, origin top-left of phone-frame.
- **World space**: Same width (402px), unbounded height downward. Y=0 is the start.
- **Camera**: `cameraY = star.worldY − 300`. The star appears ~300px from the top of the viewport. World objects are drawn at `screenY = worldY − cameraY`.
- **Star world position**: Updated each frame from key input. `worldX` is clamped to `[starHalfW, 402 − starHalfW]`. `worldY` has no lower bound (travels infinitely down). `worldY` has an upper bound of ~100 (cannot go above start).

---

## HTML Structure

```
<body>  (background: black)
  <div id="page">  (full viewport, flexbox-centered)
    <div id="phone-frame">  (402×874, overflow:hidden, position:relative)

      <!-- Layer 0: background -->
      <canvas id="bg-canvas" width="402" height="874"></canvas>

      <!-- Layer 1: world (scrolls via translateY) -->
      <div id="world">
        <div class="game-obj" id="obj-asteroid2-A"> … SVG … </div>
        <div class="game-obj" id="obj-asteroid1-A"> … SVG … </div>
        <div class="game-obj" id="obj-stardust-1"> … SVG … </div>
        <div class="game-obj" id="obj-asteroid2-B"> … SVG … </div>
        <div class="game-obj" id="obj-planet">      … SVG … </div>
        <div class="game-obj" id="obj-asteroid1-B"> … SVG … </div>
        <div class="game-obj" id="obj-stardust-2"> … SVG … </div>
        <div class="game-obj" id="obj-asteroid2-C"> … SVG … </div>
        <div class="game-obj" id="obj-dying-star">  … SVG … </div>
        <div class="game-obj" id="obj-asteroid1-C"> … SVG … </div>
        <div class="game-obj" id="obj-black-hole">  … SVG … </div>
        <div class="game-obj" id="obj-asteroid2-D"> … SVG … </div>
        <div class="game-obj" id="obj-stardust-3"> … SVG … </div>  ← LAST
      </div>

      <!-- Layer 2: star (screen-space) -->
      <div id="star-el">   … star SVG …   </div>
      <div id="starpass-el" style="display:none"> … star-pass SVG … </div>

      <!-- Layer 3: text bubble -->
      <div id="text-bubble" style="display:none">
        <p id="bubble-text"></p>
      </div>

      <!-- Layer 4: effects -->
      <div id="flash-overlay"></div>       <!-- full-frame white flash -->
      <div id="sparkles-container"></div>  <!-- sparkle particles -->

      <!-- Layer 5: poem text (ending) -->
      <div id="ending-poem"></div>

      <!-- Layer 6: restart prompt -->
      <div id="restart-overlay" style="display:none">
        PRESS ANYWHERE TO RESTART
      </div>

    </div>
  </div>
</body>
```

---

## Asset Extraction

Each object's SVG markup is pulled from `assets.svg`, given its own `<svg>` with a trimmed `viewBox` matching the object's bounding box, and embedded inline. IDs in the extracted SVGs are namespaced (e.g., `star-filter0`) to avoid collisions when multiple copies exist.

| Object | Source group ID | Approx display size |
|---|---|---|
| star | `#star` | 120×115 px |
| star-pass | `#star-pass` | 120×115 px |
| stardust | `#stardust` | 80×80 px |
| planet | `#planet` | 180×180 px |
| dying-star | `#dying-star` | 160×180 px |
| black-hole | `#black-hole` | 150×150 px |
| asteroid-1 | `#asteroid-1` | 140×140 px |
| asteroid-2 | `#asteroid-2` | 160×165 px |

The `text-block` styling from `setup.svg` is replicated in CSS for the text bubble (see below).

---

## Background (Canvas)

**Gradient (redrawn each frame):**
- A vertical `linearGradient` drawn top-to-bottom across the full canvas.
- Top color interpolates from `#1e2050` (dark indigo, near top of world) to `#000000` (pure black at and beyond world-depth 6000px).
- Formula: `darknessFactor = clamp(cameraY / 6000, 0, 1)`. Top stop color fades toward `#000000` as factor rises. Once the factor reaches 1 it stays at full black.

**Procedural star field:**
- ~300 stars pre-generated with a seeded pseudo-random function.
- Each star: `{ wx, wy, r, baseBrightness }` — world X and world Y, radius 0.5–2px, brightness 0.4–1.0.
- `wy` values distributed across world height 0–15000 (extending well past the last object).
- Each frame: draw only stars whose `wy − cameraY` is within −50 to 924 (canvas height + margin).
- Twinkle: `opacity = baseBrightness + 0.15 * sin(time * speed + phase)`.

---

## Object World Positions

Objects are placed at these approximate `worldY` values (center of object). `worldX` is varied to feel organic but always within phone frame. Objects drift horizontally via a gentle sine offset applied each frame.

| Object | worldY | worldX (center) | Drift amplitude |
|---|---|---|---|
| asteroid-2 A | 900 | 300 | 20px |
| asteroid-1 A | 1400 | 80 | 15px |
| **stardust 1** | 1900 | 200 | 10px |
| asteroid-2 B | 2500 | 120 | 25px |
| planet | 3100 | 270 | 8px |
| asteroid-1 B | 3700 | 60 | 18px |
| **stardust 2** | 4400 | 310 | 10px |
| asteroid-2 C | 5000 | 180 | 22px |
| dying-star | 5700 | 100 | 5px |
| asteroid-1 C | 6300 | 320 | 16px |
| black-hole | 7000 | 160 | 3px |
| asteroid-2 D | 7600 | 80 | 20px |
| **stardust 3** *(LAST)* | 8400 | 220 | 10px |

Beyond worldY 8400 + stardust height: nothing. Pure darkening void.

**Drift formula:** `screenX = obj.baseX + obj.driftAmp * sin(time * obj.driftSpeed + obj.driftPhase)`

---

## Star Movement

- Controlled by **arrow keys** (keydown/keyup tracking, smooth per-frame movement).
- Speed: 4px/frame (can be tuned).
- `worldX` clamped to `[starHalfW, 402 − starHalfW]`.
- `worldY` clamped: minimum `startWorldY` (~150), no maximum.
- Star's screen position: `screenX = star.worldX`, `screenY = star.worldY − cameraY`.
- Screen Y is also clamped: star cannot go above `starHalfH` or below `874 − starHalfH` on screen (boundary of phone frame). This clamp adjusts `worldY` accordingly so the star never exits the viewport.
- Movement is **paused** during `TEXT_BUBBLE` and all `ENDING_*` states.

---

## Collision Detection

- Simple circle-circle distance check each frame (during `PLAYING` state only).
- Each collectible object has a `collisionRadius` (roughly half of its display size).
- Star collision radius: ~40px.
- On collision: trigger that object's interaction (text bubble sequence).

---

## Text Bubble

**Style (replicating `text-block` from setup.svg):**
```css
#text-bubble {
  position: absolute;
  width: 277px;
  background: black;
  border-radius: 35px;
  padding: 20px 24px;
  color: #F9DD99;          /* golden text */
  font-size: 13px;
  line-height: 1.7;
  letter-spacing: 0.02em;
  box-shadow: 0 0 40px rgba(94, 83, 244, 0.4);  /* purple glow */
  backdrop-filter: blur(11px);
  z-index: 10;
  white-space: pre-line;
}
```

**Positioning:** Centered horizontally in phone frame; above the star's current screen position. If star is too high, bubble appears below.

**Lifecycle:**
1. Collision detected → state = `TEXT_BUBBLE`.
2. Bubble fades in (0.3s).
3. User presses **any key** or **clicks/taps** → bubble fades out (0.3s).
4. Object fades out (0.4s) and is marked as collected/removed.
5. State returns to `PLAYING`.

---

## Object Texts

### stardust 1
```
Coral feelings emerging from the surface
With the falling sand persevering evermore

Here, where the blades did not pierce
Where the bubbles magnified vibrant frequencies
Where the ice was not so resentful

Here, where the switch was turned off
Where the envelopes contained handwritten letters
Where the bud showed signs of bloom

The flame flickered away at the slightest breeze
With the falling sand concluding its cycle
```

### stardust 2
```
Sunsets over the lake
While the chiming grew louder

Warmth from the cardinal
It was almost like this second was the only worry
Was it folly for such a thought to occur?

The vase was full then
Now those glimpses wilt and hang over the edge
But miracles drifted around us

Waddling over noticeable cracks
The chiming announced its last toll
```

### stardust 3 (LAST — triggers ending)
```
Gazing into a fictional sea
Ethereal moments did not last forever

Somewhere within, there was a suspense amidst the saturation
It brought daffodil bouquets to urban owls
Overflowing with wonders, the forest slowed its chatter

From red storm clouds, the lyre's song reached the roots
As space made its toll
Uncertainty was the sole figure who remained

Lyrics left unsaid
Ethereal moments were but a dream
```

### black-hole
```
Leaves of embers, descending into ashes
As the waves shatter against the glass rafters

The tug of the snapdragons remained alluring
Your perception succumbed to the absence of vision
High volume obscured those vital signals

Light's rays crumbled to the dust
As the wise sought answers in the mirrored corridor
Declining into an eternal spiral

Navigating through breathing pages with veiled eyes
As the waves finally shattered the glass rafters
```

### dying-star
```
Steel vines dispersed in motion
Crafting silently behind vacant conversations

The pedestals steadily ascended
Contributors disregarding the atmospheric reality
Merely lavishing the pools of rhodium

A rapid quicksand of sentiments
With unknown pauses left undeciphered
Yet the microphone was placed in our grasp

Childish automations in a clay jar
Scheming behind vacant conversations
```

### planet
```
Blank expressions across the canvas
Minutia enduring severed harmonies

Aquatic implosions disturb the horizon line
Suddenly the puzzle is reversed
For the threads tremble from the exposure

Noise covers the cracked eggshells
Fleeting whispers fade farther in the hallway
While the host is pulled away from the party

Five crayons scratch the pure tile
Insignificantly enduring severed harmonies
```

---

## Ending Sequence (triggered by stardust 3 interaction completing)

State machine progresses through these phases:

### Phase 1 — `ENDING_FADE` (duration ~1.5s)
- All remaining visible game objects fade out simultaneously via CSS `opacity` transition.
- Camera movement freezes (star stops responding to keys).

### Phase 2 — `ENDING_GLOW` (duration ~2s)
- The star element grows a bright radial glow: an expanding `box-shadow` / filter `drop-shadow` on the star SVG, pulsing from its normal golden color to a very bright white-gold.
- Implemented via CSS animation on the star's outer glow circles (SVG filter or CSS filter brightness increase).

### Phase 3 — `ENDING_POEM_IN` (duration ~1.5s fade-in)
- `#ending-poem` div appears above the star (screen-space, centered).
- Text:
  ```
  As night descends
  Rest befalls your eyes

  Sleep, my child
  To await tomorrow, whether sweet or dreadful
  ```
- Style: white/soft golden, small font (~14px), centered, `opacity` transitions from 0 → 1 over 1.5s.
- After full fade-in, hold for 2s.

### Phase 4 — `ENDING_POEM_OUT` (duration ~1s)
- Poem text fades out (`opacity` 1 → 0).
- Pause 1.5s.

### Phase 5 — `ENDING_TRANSFORM`
- `#star-el` hides (display:none).
- `#starpass-el` shows at same screen position.
- Immediately begin `ENDING_FLASH`.

### Phase 6 — `ENDING_FLASH` (duration ~1.5s)
- `#flash-overlay` (full 402×874 white div) animates from `opacity:0` → `opacity:1` over 0.6s, then fades back `opacity:1` → `opacity:0` over 0.9s.
- The star-pass element is hidden once flash peaks.

### Phase 7 — `ENDING_SPARKLES` (duration ~2s)
- `#sparkles-container` is populated with ~20 small sparkle `<div>` elements (tiny 4-pointed star shapes or `✦` characters) at the star's last screen position.
- Each sparkle has a random upward velocity and slight horizontal drift.
- They fade out as they rise (`opacity` + `translateY` animation via JS each frame or CSS keyframes).
- After sparkles complete (~2s), transition to next phase.

### Phase 8 — `ENDED`
- `#restart-overlay` fades in over 1.5s (`opacity:0` → `1`).
- Style: centered white text, slightly large font, letter-spaced. Background: `rgba(0,0,0,0.0)` (no separate background — the dark space background is visible behind it).
- Interaction becomes active only after fade completes + 1s additional pause.
- Any click/tap anywhere on the phone-frame triggers restart.

---

## Restart

- All state reset to initial values.
- All game objects restored (opacity:1, display:block).
- Camera reset to cameraY=0.
- Star position reset to start.
- Background gradient reset.
- All ending-phase elements hidden.
- `requestAnimationFrame` loop continues uninterrupted (no page reload).

---

## Game Loop (pseudocode)

```js
function gameLoop(timestamp) {
  dt = timestamp - lastTime;
  lastTime = timestamp;

  if (state === PLAYING) {
    updateStarPosition(dt);
    updateObjectDrift(timestamp);
    checkCollisions();
  }

  updateCamera();
  drawBackground(ctx, cameraY);   // canvas gradient + stars
  updateWorldTransform(cameraY);  // move #world div
  updateStarScreenPos();

  requestAnimationFrame(gameLoop);
}
```

---

## CSS Highlights

```css
#phone-frame {
  width: 402px;
  height: 874px;
  position: relative;
  overflow: hidden;
  /* subtle noise edge from setup.svg filter replicated via CSS border/box-shadow */
  box-shadow: 0 0 80px 20px rgba(53, 57, 102, 0.6);
}

#world {
  position: absolute;
  top: 0; left: 0;
  width: 402px;
  /* height is effectively infinite — no explicit height set */
  will-change: transform;
}

.game-obj {
  position: absolute;
  /* left/top set in JS from world coordinates */
  transition: opacity 0.4s ease;
}

#text-bubble {
  /* see Text Bubble section above */
  transition: opacity 0.3s ease;
}

#flash-overlay {
  position: absolute;
  inset: 0;
  background: white;
  opacity: 0;
  pointer-events: none;
  z-index: 20;
}

#restart-overlay {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-size: 15px;
  letter-spacing: 0.2em;
  opacity: 0;
  z-index: 30;
  cursor: pointer;
}
```

---

## Implementation Steps (in order)

1. **HTML skeleton** — phone-frame, layer divs, canvas.
2. **Inline SVG assets** — extract each object from `assets.svg` into trimmed `<svg>` strings; create helper function to clone/namespace IDs for duplicated asteroids.
3. **Background canvas** — gradient draw function; procedural star generation and draw.
4. **World object placement** — create `.game-obj` divs at world positions from the placement table; apply initial `left` CSS.
5. **Star and camera** — star screen positioning logic; camera formula; world transform update.
6. **Input handling** — keydown/keyup for arrow keys; velocity-based movement.
7. **Object drift** — sine-wave horizontal drift per frame.
8. **Collision detection** — per-frame distance checks.
9. **Text bubble system** — show/hide with correct text per object; dismiss on key/click.
10. **Object removal on collect** — fade out and disable collision for collected objects.
11. **Ending sequence** — state machine driving all ending phases with timing.
12. **Restart logic** — full state reset.
13. **Polish** — star glow effect, sparkle particles, font selection (system sans-serif or Google Fonts `serif` for poem text).
