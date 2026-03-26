# Mistborn: Ash and Shadows — Game Design

## Overview

A side-scrolling platformer set in Luthadel from Mistborn Book 1 (The Final Empire). You play as Vin, a street urchin surviving the ashy slums. Built as a browser game (HTML5 Canvas + JavaScript), single HTML file, easy to share.

## Level 1: "Ash and Shadows"

Three sections with rising difficulty:

### The Slums
- Learn movement basics
- Street thugs patrol back and forth
- Falling ash clumps from above
- Crumbling platforms that give way
- Toxic puddles to jump over

### The Border
- Approaching the nobleman's district
- Garrison guards: tougher (2 HP, faster, longer reach with sword)
- Environmental hazards get nastier, bigger gaps
- Guards can't be Soothed as easily (shorter freeze duration)

### The Dead End
- Short final section
- Vin funneled into an alley, cornered by guards
- Player uses Luck to escape
- Cutscene: Kelsier silhouette on a rooftop
- Level complete

## Level 2 (future)
- The Camon/obligator con job scene

## Controls

- **Arrow keys / WASD** — Move left/right
- **Spacebar** — Jump (hold for higher)
- **X or J** — Knife attack (short range, quick)
- **Z or K** — Luck/Soothe (cooldown ~5s, freezes nearby enemy for 2-3s)

## Vin's Stats

- 3 hit points (metal vial icons)
- Health restored by metal vial pickups
- Boxings (coins) for score

## Enemies

### Street Thugs
- Walk back and forth on platforms
- 1 hit to kill
- Touch Vin to deal damage
- Rough faces, scowls, ragged clothes

### Garrison Guards
- Patrol, change direction if they spot Vin
- 2 hits to kill, slightly faster
- Carry a sword with longer reach
- Steel helmets with visors, red tabards over armor

## Environmental Hazards

- **Falling ash clumps** — Drop from above at intervals, deal damage
- **Crumbling platforms** — Shake then collapse after Vin stands briefly
- **Toxic puddles** — Instant damage if touched, must jump over

## Visual Design

- **Color palette:** Dark base with pops of contrast — warm orange/amber glow from forge fires and windows, Vin's blue-tinged cloak as signature color, red embers in the ash. Grim but visually alive.
- **Ash particles:** Constant slow-falling ash across the screen
- **Background:** Layered parallax — skaa shacks, smokestacks, Kredik Shaw silhouette in the distance
- **Lighting:** Flickering lanterns, glowing windows, steam from grates, rats scurrying

### Character Sprites
- **Vin** — Visible face, dark hair, determined expression. Blue-grey cloak, small frame. Hair and cloak flow on run/jump.
- **Street thugs** — Rough faces, ragged clothes. Each looks like a person.
- **Garrison guards** — Steel helmets, red tabards, intimidating.
- **Kelsier** — End silhouette, recognizable by half-up hair, confident stance, glint of a smile.

## HUD

- Top-left: 3 metal vial icons (health)
- Top-right: Boxing count
- Bottom: Luck cooldown bar

## Audio

- **Background music:** Procedurally generated ambient loop via Web Audio API. Low, brooding, subtle beat. Intensifies in garrison section.
- **Sound effects:** Jump, slash, enemy hit, coin pickup, Luck pulse, guard alert.

## Cutscenes

- Text boxes over darkened gameplay screen
- Opening: "The ash falls. It always falls. Vin pulled her cloak tighter..."
- Ending: Vin cornered, screen pulses with Luck, guards wander off. Shadow on rooftop. "Someone was watching."

## Technical Approach

- Single HTML file — HTML, CSS, JS, inline sprites
- HTML5 Canvas rendering
- Programmatic pixel art sprites (no external files)
- requestAnimationFrame game loop
- Simple gravity/velocity/collision physics
- Web Audio API for all audio
- Tile map array for level data (extensible for Level 2)
- Mobile touch controls overlay for shareability

## File Structure

```
mistborn-game/
  index.html    <- the entire game
```
