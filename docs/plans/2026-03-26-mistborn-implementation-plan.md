# Mistborn: Ash and Shadows — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a shareable browser-based side-scrolling platformer where you play as Vin from Mistborn Book 1, surviving the streets of Luthadel.

**Architecture:** Single HTML file containing all game code, sprites, and audio. HTML5 Canvas for rendering, vanilla JS for game logic, Web Audio API for sound. Tile-map based level with parallax scrolling background. Entity-component style: player, enemies, hazards, pickups all share update/render patterns.

**Tech Stack:** HTML5 Canvas, vanilla JavaScript, Web Audio API. No dependencies. No build tools.

**Testing approach:** Since this is a single-file browser game, we verify each task by opening `index.html` in a browser. Each task produces a visible, interactive result.

---

### Task 1: Canvas Boilerplate + Game Loop

**Files:**
- Create: `index.html`

**Step 1: Create the HTML shell with canvas**

Set up the HTML page with a full-viewport canvas, dark background (#1a1a2e), and the basic game loop using `requestAnimationFrame`. Include a simple FPS counter in the top corner for debugging.

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
  <title>Mistborn: Ash and Shadows</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { background: #0a0a12; overflow: hidden; }
    canvas { display: block; }
  </style>
</head>
<body>
  <canvas id="game"></canvas>
  <script>
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');
    function resize() {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
    }
    window.addEventListener('resize', resize);
    resize();

    let lastTime = 0;
    function gameLoop(timestamp) {
      const dt = (timestamp - lastTime) / 1000;
      lastTime = timestamp;
      // Clear
      ctx.fillStyle = '#1a1a2e';
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      // FPS debug
      ctx.fillStyle = '#666';
      ctx.font = '12px monospace';
      ctx.fillText(`FPS: ${Math.round(1/dt)}`, 10, 20);
      requestAnimationFrame(gameLoop);
    }
    requestAnimationFrame(gameLoop);
  </script>
</body>
</html>
```

**Step 2: Verify in browser**

Open `index.html`. Should see a dark screen with an FPS counter ticking.

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: canvas boilerplate with game loop"
```

---

### Task 2: Input System

**Files:**
- Modify: `index.html`

**Step 1: Add keyboard input tracking**

Add an input handler object that tracks which keys are currently pressed. Support both arrow keys and WASD. Track: left, right, jump (space), attack (X/J), luck (Z/K).

```javascript
const keys = {};
const input = {
  get left() { return keys['ArrowLeft'] || keys['KeyA']; },
  get right() { return keys['ArrowRight'] || keys['KeyD']; },
  get jump() { return keys['Space']; },
  get attack() { return keys['KeyX'] || keys['KeyJ']; },
  get luck() { return keys['KeyZ'] || keys['KeyK']; },
};
window.addEventListener('keydown', e => { keys[e.code] = true; e.preventDefault(); });
window.addEventListener('keyup', e => { keys[e.code] = false; });
```

**Step 2: Add visual input debug display**

Show which inputs are active on screen so we can verify they work.

**Step 3: Verify in browser**

Open in browser, press keys, see debug indicators light up for each input.

**Step 4: Commit**

```bash
git commit -am "feat: input system with keyboard tracking"
```

---

### Task 3: Player Physics + Movement

**Files:**
- Modify: `index.html`

**Step 1: Create player object with physics**

Add a player object with position, velocity, gravity, and ground collision. Player should:
- Move left/right at a set speed (200px/s)
- Jump with variable height (hold space = higher, up to a max)
- Fall with gravity (980px/s^2)
- Collide with a temporary ground plane at the bottom of the screen
- Have a small acceleration/deceleration for smoother movement

```javascript
const player = {
  x: 100, y: 0,
  vx: 0, vy: 0,
  width: 24, height: 36,
  onGround: false,
  jumpHeld: false,
  jumpTime: 0,
  facing: 1, // 1 = right, -1 = left
  speed: 200,
  jumpForce: -400,
  gravity: 980,
};
```

**Step 2: Add update and render for player**

Update: apply input to velocity, apply gravity, move position, check ground collision.
Render: draw as a colored rectangle for now (blue-grey, Vin's cloak color).

**Step 3: Verify in browser**

Open in browser. Arrow keys move a rectangle left/right. Space makes it jump with variable height. It falls back down and lands.

**Step 4: Commit**

```bash
git commit -am "feat: player physics with movement and jumping"
```

---

### Task 4: Camera System + Tile Map

**Files:**
- Modify: `index.html`

**Step 1: Define tile map constants and level data**

Create a tile-based level system. Tiles are 32x32 pixels. Define tile types:
- 0 = empty (air)
- 1 = solid ground (stone)
- 2 = crumbling platform
- 3 = toxic puddle (on top of ground)

Define Level 1 as a 2D array, at least 200 tiles wide (6400px). The level should have:
- Flat ground with gaps for the slums section
- Platforms at varying heights
- A few crumbling platforms
- Toxic puddles over gaps

**Step 2: Add camera that follows player**

Camera smoothly follows the player horizontally, with the player positioned roughly 1/3 from the left edge. Camera clamps to level bounds so it doesn't show empty space.

**Step 3: Render tile map with camera offset**

Only render tiles visible on screen (culling). Solid ground tiles rendered in dark grey/brown tones.

**Step 4: Update player to collide with tiles**

Replace the simple ground plane with proper tile collision. Check tiles around the player for solid collision on all four sides (not just below).

**Step 5: Verify in browser**

Open in browser. See a level made of tiles. Run right and the camera follows. Jump on platforms. Fall into gaps. Can't walk through walls.

**Step 6: Commit**

```bash
git commit -am "feat: tile map, camera system, and tile collision"
```

---

### Task 5: Parallax Background + Ash Particles

**Files:**
- Modify: `index.html`

**Step 1: Add parallax background layers**

Draw 3 background layers that scroll at different speeds relative to the camera:
- **Far layer (0.1x speed):** Kredik Shaw silhouette — a tall, spiky palace shape in dark purple/black against a hazy red-grey sky. Smoke stacks with faint orange glows.
- **Mid layer (0.3x speed):** Skaa shack rooftops — irregular low buildings, some with lit windows (warm amber rectangles). Steam rising.
- **Near layer (0.6x speed):** Foreground details — fence posts, barrels, crates. Drawn darker to feel closer.

All drawn procedurally using canvas shapes (rectangles, triangles, lines).

**Step 2: Add ash particle system**

Create a particle system with ~80 ash particles:
- Small circles/flakes (1-3px), varying grey/white tones
- Fall slowly at different speeds (20-60px/s)
- Drift horizontally with slight sine wave wobble
- Wrap around screen when they fall off bottom
- Some red ember particles mixed in (fewer, brighter, slightly larger)

**Step 3: Verify in browser**

Open in browser. See a moody layered background with Kredik Shaw in the distance. Ash falls constantly. Moving the player scrolls the parallax layers at different speeds. Looks atmospheric.

**Step 4: Commit**

```bash
git commit -am "feat: parallax background and ash particle system"
```

---

### Task 6: Vin's Pixel Art Sprite + Animation

**Files:**
- Modify: `index.html`

**Step 1: Create Vin's sprite data**

Define Vin as pixel art sprite data (24x36 pixels). She needs:
- **Face:** Skin tone, dark hair falling past shoulders, visible eyes (determined expression)
- **Body:** Blue-grey cloak over dark clothing, small frame
- **Animation frames:** idle (2 frames, subtle breathing), run (4-6 frames, cloak flowing), jump (1 frame, cloak billowing up), attack (2 frames, knife slash), fall (1 frame)

Store sprite data as 2D arrays of hex color values. Create a `drawSprite(spriteData, x, y, scale, flip)` function that renders pixel-by-pixel onto the canvas.

**Step 2: Add animation state machine**

Track animation state (idle, run, jump, fall, attack) and frame timing. Switch states based on player velocity and input. Attack animation plays on attack input and has priority over movement animations.

**Step 3: Replace player rectangle with animated sprite**

Render Vin's sprite instead of the colored rectangle. Flip sprite based on `player.facing`.

**Step 4: Verify in browser**

Open in browser. See Vin with a visible face and flowing cloak. She animates when idle (breathing), running (legs + cloak), jumping (cloak billows). Faces the direction she moves.

**Step 5: Commit**

```bash
git commit -am "feat: Vin pixel art sprite with animations"
```

---

### Task 7: Knife Attack Mechanic

**Files:**
- Modify: `index.html`

**Step 1: Add attack state to player**

When X/J is pressed:
- Player enters attack state for ~300ms
- A hitbox extends in front of Vin (about 20px beyond her sprite)
- Attack animation plays (knife slash)
- Can't attack again until current attack finishes
- Player can still move during attack

**Step 2: Add visual knife slash effect**

Draw a small arc/slash effect in front of Vin during attack frames. Use a light grey/silver color with slight transparency fade.

**Step 3: Verify in browser**

Open in browser. Press X to attack. See knife slash animation and effect. Can move while attacking. Can't spam attack (has recovery time).

**Step 4: Commit**

```bash
git commit -am "feat: knife attack mechanic with slash effect"
```

---

### Task 8: Street Thug Enemies

**Files:**
- Modify: `index.html`

**Step 1: Create street thug sprite data**

Define thug sprite (24x36 pixels):
- Bulkier than Vin
- Rough face with scowl, messy hair
- Ragged brown/grey clothing
- Animation frames: walk (4 frames), hit (1 frame, knocked back), death (fade out)

**Step 2: Add enemy entity system**

Create an enemy array and an `Enemy` class/object factory with:
- Position, velocity, width, height
- HP (thugs = 1)
- Patrol behavior: walk back and forth between two x-coordinates
- Collision with tiles (don't walk off platforms)
- Direction facing (for sprite flipping)

**Step 3: Add player-enemy interaction**

- If player touches enemy: player takes 1 damage, brief invincibility (1.5s with flashing)
- If player's attack hitbox overlaps enemy: enemy takes 1 damage
- Enemy death: small particle burst, enemy fades out and is removed

**Step 4: Place thugs in the Slums section of the level**

Add 4-5 thugs patrolling on platforms in the first third of the level.

**Step 5: Verify in browser**

Open in browser. See thugs walking back and forth. Run into one — take damage and flash. Attack one — it dies with a particle effect. They don't walk off platforms.

**Step 6: Commit**

```bash
git commit -am "feat: street thug enemies with patrol and combat"
```

---

### Task 9: Garrison Guard Enemies

**Files:**
- Modify: `index.html`

**Step 1: Create garrison guard sprite data**

Define guard sprite (26x40 pixels — slightly taller/wider than thugs):
- Steel helmet with visor
- Red tabard over dark armor
- Sword visible at side
- Animation frames: walk (4 frames), alert (turns toward player), attack (sword swing), hit, death

**Step 2: Add guard-specific AI**

Guards are an enemy subtype with:
- 2 HP
- Faster patrol speed (1.3x thug speed)
- **Detection:** If Vin is within ~150px horizontal range and on the same platform level, guard turns to face her and moves toward her
- **Sword attack:** When within ~40px of Vin, guard swings sword (slightly longer range than Vin's knife)
- Soothe effect is shorter on guards (1.5s instead of 2-3s)

**Step 3: Place guards in the Border section**

Add 3-4 guards in the middle third of the level. The transition from thugs to guards should feel noticeable.

**Step 4: Verify in browser**

Open in browser. Reach the Border section. Guards are taller, look different. They detect Vin and chase. Take 2 hits to kill. Their sword has longer reach. Noticeably harder than thugs.

**Step 5: Commit**

```bash
git commit -am "feat: garrison guard enemies with detection AI"
```

---

### Task 10: Luck/Soothe Ability

**Files:**
- Modify: `index.html`

**Step 1: Add Luck ability to player**

When Z/K is pressed:
- Check cooldown (5 seconds between uses)
- Find nearest enemy within ~120px radius
- That enemy freezes in place for 2.5s (thugs) or 1.5s (guards)
- Enemy gets a visual indicator: slight blue tint, confused expression/stance
- Cooldown bar starts recharging

**Step 2: Add visual Luck pulse effect**

When Luck is activated:
- A subtle circular pulse radiates outward from Vin (blue-purple, transparent, expands and fades)
- Affected enemy gets small "swirl" particles above head
- Brief screen vignette effect (edges darken for a moment)

**Step 3: Add cooldown bar to HUD area (bottom of screen)**

Draw a thin bar at the bottom showing Luck recharge. Blue-purple when ready, grey when cooling down, fills left to right.

**Step 4: Verify in browser**

Open in browser. Get near an enemy, press Z. See the pulse effect, enemy freezes. Try again — cooldown prevents it. Wait 5 seconds, works again. Guards freeze for less time than thugs.

**Step 5: Commit**

```bash
git commit -am "feat: Luck/Soothe ability with cooldown and visual effects"
```

---

### Task 11: Environmental Hazards

**Files:**
- Modify: `index.html`

**Step 1: Add toxic puddles**

Render puddles as green-tinted pools on the ground (placed via tile map type 3). Animated surface (slight shimmer). Touching one deals 1 damage to Vin and knocks her back slightly. Bubbling particle effect on the surface.

**Step 2: Add crumbling platforms**

Platforms with tile type 2 behave as:
- Solid when Vin first lands on them
- Start shaking after 0.5s of standing on them (visual vibration)
- Collapse after 1.2s (tiles fall away with particle effect, become non-solid)
- Respawn after 5s (fade back in)

Render crumbling platforms with a cracked/lighter texture to distinguish them from solid ground.

**Step 3: Add falling ash clumps**

Spawn hazardous ash clumps from above at certain x-positions in the level:
- Larger than ambient ash (8-12px)
- Fall faster (200px/s)
- Deal 1 damage on contact
- Shadow on ground below as warning (grows as clump falls)
- Spawn at intervals (every 2-3 seconds at each spawn point)
- Shatter into particles on hitting ground

**Step 4: Place hazards throughout the level**

- Toxic puddles in both Slums and Border sections
- Crumbling platforms more frequent in Border section
- Falling ash clumps scattered throughout, more frequent later

**Step 5: Verify in browser**

Open in browser. See toxic puddles bubbling. Step on a crumbling platform — it shakes then falls. See ash clump shadows, dodge or get hit. All hazards deal damage.

**Step 6: Commit**

```bash
git commit -am "feat: environmental hazards - puddles, crumbling platforms, falling ash"
```

---

### Task 12: HUD + Health + Pickups

**Files:**
- Modify: `index.html`

**Step 1: Add HUD rendering**

Draw the HUD layer (not affected by camera):
- **Top-left:** 3 metal vial icons. Full = bright silver/blue. Empty = dark outline. When hit, the current vial "drains" with a quick animation.
- **Top-right:** Boxing count with a small coin icon. Number in a clean font.
- **Bottom-center:** Luck cooldown bar (already partially done in Task 10, polish it here).

**Step 2: Add metal vial pickups**

Place small glowing vial sprites throughout the level:
- Float slightly (bob up and down)
- Glow effect (subtle light around them)
- On pickup: restore 1 HP, brief flash effect, satisfying sound later
- Only appear if player is not at full health (or always appear but give score bonus if full)

**Step 3: Add boxing (coin) pickups**

Place boxings throughout the level:
- Small gold coin sprites
- Bob animation like vials
- On pickup: increment counter, brief sparkle effect
- Place generously (20-30 throughout level) to encourage exploration

**Step 4: Add player death and respawn**

When HP reaches 0:
- Brief death animation (Vin collapses, screen darkens)
- "The mists claim another..." text
- Respawn at last section checkpoint (start of Slums, start of Border, start of Dead End)
- Reset HP to 3, keep score

**Step 5: Verify in browser**

Open in browser. See HUD with vials, coins, cooldown bar. Collect vials and boxings. Take damage — vials drain. Die — see death screen, respawn at checkpoint.

**Step 6: Commit**

```bash
git commit -am "feat: HUD, health system, pickups, death and respawn"
```

---

### Task 13: The Dead End + Cutscenes

**Files:**
- Modify: `index.html`

**Step 1: Build The Dead End level section**

The final section of the level:
- Narrow alley with walls closing in
- 3 garrison guards positioned to corner Vin
- A dead-end wall she can't pass
- Hitting the dead end triggers the ending sequence

**Step 2: Add opening cutscene**

When the game first loads, before gameplay:
- Black screen fades in
- Text appears line by line with typewriter effect:
  - "The ash falls. It always falls."
  - "Vin pulled her cloak tighter and kept to the shadows."
  - "The streets of Luthadel were no place for a girl alone."
- Press any key to start gameplay
- Screen fades to game

**Step 3: Add ending cutscene**

When Vin reaches the dead end:
- Gameplay pauses, screen darkens at edges
- Guards close in from behind
- "There was nowhere left to run."
- Screen pulses blue-purple (Luck activates automatically)
- Guards stop, look confused, wander away
- "She didn't understand what she'd done. Only that it worked."
- Camera pans up to rooftop — Kelsier's silhouette (recognizable hair, confident pose)
- "Someone was watching."
- Kelsier's silhouette smiles (subtle animation)
- Fade to black
- "Level 1 Complete" with boxing score
- "To be continued..."

**Step 4: Verify in browser**

Open in browser. See opening cutscene text. Press key to start. Play through entire level to the dead end. See ending cutscene play out with Kelsier silhouette. See completion screen.

**Step 5: Commit**

```bash
git commit -am "feat: Dead End section and cutscenes with Kelsier reveal"
```

---

### Task 14: Audio — Music + Sound Effects

**Files:**
- Modify: `index.html`

**Step 1: Create procedural background music**

Using Web Audio API, generate an ambient music loop:
- **Base layer:** Low drone/pad (dark minor chord, filtered oscillators)
- **Rhythm:** Subtle kick-like pulse, slow tempo (~70 BPM)
- **Melody:** Sparse high notes, minor pentatonic, echoing/delayed
- **Atmosphere:** Filtered noise layer for "wind/ash" ambiance
- **Border section:** When camera passes into the Border section, add a slightly faster rhythm layer and deeper bass to increase tension
- Music starts on first user interaction (browser autoplay policy)

**Step 2: Add sound effects**

Generate with Web Audio API:
- **Jump:** Short upward pitch sweep
- **Knife slash:** Quick noise burst with high-pass filter
- **Enemy hit:** Thud — low frequency burst
- **Enemy death:** Descending tone
- **Coin pickup:** Bright two-tone chime (ascending)
- **Vial pickup:** Liquid/shimmer sound (filtered noise + high tone)
- **Luck pulse:** Deep resonant "whooom" with reverb
- **Guard alert:** Short sharp tone (like a "!" sound)
- **Player hurt:** Dull impact
- **Crumble:** Rumbling noise burst

**Step 3: Wire up audio to game events**

Play appropriate sounds on each event. Keep volume balanced — music slightly quieter than SFX.

**Step 4: Verify in browser**

Open in browser. Click to start (enables audio). Hear brooding ambient music. Jump — hear jump sound. Attack enemy — hear slash and hit. Collect coin — hear chime. Use Luck — hear whooom. Music intensifies in Border section.

**Step 5: Commit**

```bash
git commit -am "feat: procedural audio - ambient music and sound effects"
```

---

### Task 15: Mobile Touch Controls

**Files:**
- Modify: `index.html`

**Step 1: Detect touch device and show controls**

On touch-capable devices, overlay semi-transparent touch controls:
- **Left side:** D-pad or left/right arrow buttons
- **Right side:** Jump button (large, bottom), Attack button (medium, middle), Luck button (small, top)
- Buttons are semi-transparent circles/rounded rects with icons
- Hide on desktop (mouse/keyboard users)

**Step 2: Wire touch events to input system**

Map touch start/end events on each button to the same input state the keyboard uses. Support multi-touch (e.g., hold right + press jump).

**Step 3: Verify on mobile**

Open on phone browser (or use Chrome DevTools device emulation). See touch controls. Can move, jump, attack, and use Luck with touch. Multi-touch works.

**Step 4: Commit**

```bash
git commit -am "feat: mobile touch controls overlay"
```

---

### Task 16: Polish + Title Screen

**Files:**
- Modify: `index.html`

**Step 1: Add title screen**

Before the opening cutscene:
- "MISTBORN" in large stylized text (drawn with canvas, not a font)
- "Ash and Shadows" subtitle
- Ash particles falling on the title screen
- Atmospheric background (Luthadel skyline silhouette)
- "Press any key to begin" / "Tap to begin"
- Fade transition to opening cutscene

**Step 2: Screen shake on impacts**

Add subtle screen shake when:
- Vin takes damage (small shake)
- Enemy dies (tiny shake)
- Crumbling platform collapses (medium shake)
- Luck pulse activates (brief shake)

**Step 3: Particle effects polish**

Add particles for:
- Vin's footsteps when running (tiny dust puffs)
- Landing after a jump (dust burst)
- Knife slash (small sparks)
- Collecting pickups (sparkle burst)

**Step 4: Add environmental details to tile rendering**

- Flickering lanterns on some tiles (warm glow circle that flickers)
- Lit windows in background buildings (amber rectangles that occasionally flicker)
- Rats that scurry away when Vin approaches (tiny dark sprites that run)
- Steam/mist rising from grates (semi-transparent particles rising)

**Step 5: Verify in browser**

Open in browser. See polished title screen. Start game. Feel the screen shakes, see dust particles, notice lanterns and rats. The world feels alive and moody.

**Step 6: Commit**

```bash
git commit -am "feat: title screen, screen shake, particles, environmental details"
```

---

### Task 17: Final Level Design Pass

**Files:**
- Modify: `index.html`

**Step 1: Review and refine the full level layout**

Play through the entire level and adjust:
- **Pacing:** Is the difficulty curve smooth? Slums should be easy, Border challenging, Dead End tense.
- **Platform spacing:** Are jumps fair? No pixel-perfect leaps.
- **Enemy placement:** Do thugs/guards feel well-positioned?
- **Pickup placement:** Are vials placed before hard sections? Are boxings rewarding to collect?
- **Hazard frequency:** Not too many falling ash clumps that feel unfair.

**Step 2: Add section transition markers**

Visual cues that you're entering a new section:
- **Slums → Border:** Architecture changes (shacks → taller stone buildings), a broken gate/fence
- **Border → Dead End:** Alley narrows, walls get taller, lighting gets darker

**Step 3: Final playtest and balance**

Play through 3 times. Adjust enemy speed, jump distances, hazard timing until it feels fun and fair. A competent player should be able to beat it in 3-5 minutes.

**Step 4: Remove FPS debug counter**

Remove or hide the FPS display (or make it toggle-able with a key like F3).

**Step 5: Verify in browser**

Full playthrough from title screen to "Level 1 Complete." The experience should feel complete, polished, and fun.

**Step 6: Commit**

```bash
git commit -am "feat: final level design, balance, and polish pass"
```

---

## Summary

| Task | What it adds | Key result |
|------|-------------|------------|
| 1 | Canvas + game loop | Dark screen with FPS counter |
| 2 | Input system | Keyboard controls tracked |
| 3 | Player physics | Moving/jumping rectangle |
| 4 | Camera + tile map | Scrolling level with platforms |
| 5 | Parallax + ash | Moody atmospheric background |
| 6 | Vin sprite | Animated character with face |
| 7 | Knife attack | Combat mechanic |
| 8 | Street thugs | First enemies to fight |
| 9 | Garrison guards | Tougher enemies with AI |
| 10 | Luck/Soothe | Signature Mistborn ability |
| 11 | Hazards | Puddles, crumbling, falling ash |
| 12 | HUD + pickups | Health, coins, death/respawn |
| 13 | Dead End + cutscenes | Story and level completion |
| 14 | Audio | Music and sound effects |
| 15 | Mobile controls | Touch support for sharing |
| 16 | Polish | Title screen, particles, juice |
| 17 | Level design pass | Final balance and feel |
