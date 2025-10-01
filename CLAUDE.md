# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Jurassic Escape** is a single-file HTML5 browser game built with React (via CDN), Canvas API, and Tailwind CSS. The player navigates through three increasingly difficult levels, avoiding dinosaurs and obstacles while collecting powerups to reach the exit.

## Running the Game

Open `src/jurassic-escape-game.html` directly in a web browser. No build process or dependencies required.

## Architecture

### Single-File Structure
The entire game exists in one HTML file with embedded JavaScript using Babel standalone for JSX transformation:
- React components and game logic in `<script type="text/babel">`
- External dependencies loaded via CDN (React 18, Babel standalone, Tailwind CSS)
- Canvas-based rendering with 2D context

### Game State Management
The main `JurassicEscape` component manages:
- **Game states**: `menu`, `playing`, `levelComplete`, `won`, `lost`
- **React state**: `gameState`, `score`, `playerHealth`, `playerAmmo`, `currentLevel`, `speedBoostActive`, `showPauseMenu`
- **Game ref** (`gameRef`): Contains mutable game objects updated every frame
  - Player, bullets, dinosaurs, obstacles, powerups, ammoPickups
  - Camera position, map dimensions, input tracking

### Game Loop Pattern
Uses `useEffect` with `requestAnimationFrame` for the main game loop (lines 141-889):
1. **Update phase**: Physics, collision detection, AI
2. **Render phase**: Canvas drawing with camera offset
3. Loop continues until component unmounts or game state changes

### Core Systems

**Movement & Controls**
- WASD/Arrow keys for player movement
- Spacebar to jump (clears obstacles when jump height >= 25)
- Mouse position tracked for aiming
- Click to shoot (consumes ammo)
- ESC to pause (shows pause menu with Continue/Restart/Exit options)
- Camera follows player with map boundary constraints

**Entities**
- **Player**: Safari explorer character with gun that aims at mouse cursor
  - Properties: health, ammo, speed (base + boost modifier), jump physics (isJumping, jumpVelocity, jumpHeight)
  - Procedurally drawn with safari hat, khaki clothing, and rotating gun
  - Jump physics: velocity starts at 8, gravity of 0.5 per frame
- **Dinosaurs** (3 types): Raptor, T-Rex, Stegosaurus
  - Each has unique stats: health, speed, size, points
  - AI states: `patrol` (random wandering) or `chase` (aggro within range)
  - Hand-drawn canvas art with facing direction (horizontal flip for left movement)
- **Obstacles**: Trees and bushes with circular collision
- **Powerups**: Health (red cross) and Speed (lightning bolt)
- **Ammo pickups**: Dropped when dinosaurs die, with physics (bouncing animation)

**Collision Detection**
All collision uses circle-vs-circle distance checks:
- Player vs obstacles (movement blocking, bypassed when jump height >= 25)
- Player vs dinosaurs (damage)
- Bullets vs dinosaurs (damage)
- Player vs powerups/ammo (collection)

**Level Progression**
Levels defined in `levelConfigs` (lines 45-75):
- Each level specifies dinosaur types/counts, obstacle count, powerup count
- Win condition: Reach exit zone (golden EXIT rectangle at map coordinates 1800, 1400)
- Player stats reset between levels, score persists

**Sound System**
- Uses `soundsRef` with `useCallback` for audio management
- Audio pooling via `cloneNode()` for simultaneous playback (prevents freezing and enables mixing)
- 13 sound effects: shoot, hit, death, pickup_ammo, pickup_health, pickup_speed, player_hurt, level_complete, game_over, victory, game_start, jump, pause
- `unpause` mapped to `game_start` for reuse
- All sounds are .wav files in `../assets/` folder
- Volume set to 0.3, with cleanup via 'ended' event listener

### Rendering
- 800x600 canvas viewport into 2400x1800 game world
- Camera offset (`cameraX`, `cameraY`) applied to all draw calls
- Layered drawing order: background → exit → obstacles → powerups → dinosaurs → player → bullets → ammo pickups
- Health bars drawn above dinosaurs
- Jump shadow drawn below player when airborne
- Speed boost glow effect (cyan aura) around player
- UI overlay in React (hearts, 💥 ammo count, score, speed boost indicator)
- Pause menu overlay with semi-transparent backdrop when ESC pressed

## Key Implementation Details

- **No collision between game entities** except player (dinosaurs don't collide with obstacles or each other)
- **Jump mechanic**: Helps escape stuck spawn positions between bushes; visual feedback with shadow
- **Speed boost**: Multiplies player speed by 1.8x for 300 frames (5 seconds at 60fps)
- **Ammo economy**: Start with 10, dinosaurs drop ammo equal to their max health when killed
- **Dinosaur art**: Procedurally drawn on canvas with context transformations (scale, rotate, flip)
- **Player art**: Safari explorer with rotating gun using ctx.save/translate/rotate/restore pattern
- **Input handling**: Keyboard state stored in `game.keys` object, mouse position relative to canvas
- **Pause system**: Game loop continues but returns early when `showPauseMenu` is true; ESC key toggles pause
- **Sound on menu actions**:
  - Continue/Restart: plays `unpause` (mapped to `game_start`)
  - Exit to Menu: plays `victory`
  - Pause: plays `pause`
