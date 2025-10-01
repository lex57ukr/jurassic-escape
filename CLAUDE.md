# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Jurassic Escape** is a single-file HTML5 browser game built with React (via CDN), Canvas API, and Tailwind CSS. The player navigates through three increasingly difficult levels, avoiding dinosaurs and obstacles while collecting powerups to reach the exit.

## Running the Game

Open `index.html` directly in a web browser. No build process or dependencies required.

## Architecture

### Single-File Structure

The entire game exists in one HTML file with embedded JavaScript using Babel standalone for JSX transformation:

- React components and game logic in `<script type="text/babel">`
- External dependencies loaded via CDN (React 18, Babel standalone, Tailwind CSS)
- Canvas-based rendering with 2D context

### Game State Management

The main `JurassicEscape` component manages:

- **Game states**: `menu`, `playing`, `levelComplete`, `won`, `lost`
- **React state**: `gameState`, `score`, `playerHealth`, `playerAmmo`, `currentLevel`, `speedBoostActive`, `showPauseMenu`, `showSettingsMenu`, `soundEnabled`, `volume`, `viewportScale`, `canvasSize`, `isTouchDevice`
- **Game ref** (`gameRef`): Contains mutable game objects updated every frame
  - Player, bullets, dinosaurs, obstacles, powerups, ammoPickups
  - Camera position, map dimensions, input tracking
  - Touch controls state (joystick, shoot, jump)

### Game Loop Pattern

Uses `useEffect` with `requestAnimationFrame` for the main game loop (lines 141-889):

1. **Update phase**: Physics, collision detection, AI
2. **Render phase**: Canvas drawing with camera offset
3. Loop continues until component unmounts or game state changes

### Core Systems

#### Movement & Controls

- **Desktop**: WASD/Arrow keys for player movement
- **Mobile**: Virtual joystick (touch and drag) with variable speed based on joystick displacement
- Spacebar (desktop) or JUMP button (mobile) to jump (clears obstacles when jump height >= 25)
- Mouse position tracked for aiming (desktop)
- Click to shoot (desktop) or FIRE button with auto-aim (mobile)
- Touch controls automatically hidden on desktop (using Tailwind `md:hidden`)
- ESC to pause (shows pause menu with Continue/Restart/Exit options)
- Camera follows player with map boundary constraints

#### Entities

- **Player**: Safari explorer character with gun that aims at mouse cursor
  - Properties: health, ammo, speed (base + boost modifier), jump physics (isJumping, jumpVelocity, jumpHeight)
  - Procedurally drawn with safari hat, khaki clothing, and rotating gun
  - Jump physics: velocity starts at 8, gravity of 0.5 per frame
- **Dinosaurs** (3 types): Raptor, T-Rex, Stegosaurus
  - Each has unique stats: health, speed, size, points
  - AI states: `patrol` (random wandering) or `chase` (aggro within range)
  - Obstacle avoidance: when blocked, try steering left/right; if both blocked, reverse
  - Hand-drawn canvas art with facing direction (horizontal flip for left movement)
- **Obstacles**: Trees and bushes with circular collision
- **Powerups**: Health (red cross) and Speed (lightning bolt)
- **Ammo pickups**: Dropped when dinosaurs die, with physics (bouncing animation)

#### Collision Detection

All collision uses circle-vs-circle distance checks:

- Player vs obstacles (movement blocking, bypassed when jump height >= 25)
- Dinosaurs vs obstacles (movement blocking with steering avoidance)
- Player vs dinosaurs (damage)
- Bullets vs dinosaurs (damage)
- Player vs powerups/ammo (collection)

#### Level Progression

Levels defined in `levelConfigs` (lines 45-75):

- Each level specifies dinosaur types/counts, obstacle count, powerup count
- Win condition: Reach exit zone (golden EXIT rectangle at map coordinates 1800, 1400)
- Player stats reset between levels, score persists

#### Sound System

- Uses `soundsRef` with `useCallback` for audio management
- Audio pooling via `cloneNode()` for simultaneous playback (prevents freezing and enables mixing)
- 13 sound effects: shoot, hit, death, pickup_ammo, pickup_health, pickup_speed, player_hurt, level_complete, game_over, victory, game_start, jump, pause
- `unpause` mapped to `game_start` for reuse
- All sounds are .wav files in `./assets/` folder
- **Volume control**: `volume` state (0.0-1.0, default 0.3) controls audio volume; applied to all sounds in `playSound` function
- **Sound control**: `soundEnabled` state controls whether audio plays; checked at start of `playSound` function
- Settings persisted in localStorage as `jurassicEscapeSoundEnabled` (boolean) and `jurassicEscapeVolume` (number)

### Rendering

- **Viewport scaling system**: Configurable canvas resolution via `VIEWPORT_SCALES` constant (lines 119-123)
  - Small: 800x600 (default)
  - Medium: 1000x750
  - Large: 1200x900
  - All scales maintain 4:3 aspect ratio
  - Game world remains 2400x1800 regardless of viewport size
  - Larger viewports show more of the game world at once
- **Responsive canvas sizing**: Scales to fit screen on mobile while maintaining aspect ratio of selected scale
- Canvas styled with CSS to match `canvasSize` state (width/height in pixels)
- Camera offset (`cameraX`, `cameraY`) applied to all draw calls
- Layered drawing order: background → exit → obstacles → powerups → dinosaurs → player → bullets → ammo pickups
- Health bars drawn above dinosaurs
- Jump shadow drawn below player when airborne
- Speed boost glow effect (cyan aura) around player
- Invincibility flashing effect (semi-transparent on alternating frames)
- UI overlay in React (hearts, 💥 ammo count, score, speed boost indicator, settings gear button)
- Touch controls overlay (virtual joystick, JUMP button, FIRE button) positioned absolutely over canvas
- Pause menu overlay with semi-transparent backdrop when ESC pressed
- Settings menu overlay accessible from main menu, pause menu, and gameplay HUD (gear button)

## Key Implementation Details

- **Collision system**: Player and dinosaurs collide with obstacles; dinosaurs don't collide with each other
- **Jump mechanic**: Helps escape stuck spawn positions between bushes; visual feedback with shadow
- **Speed boost**: Multiplies player speed by 1.8x for 300 frames (5 seconds at 60fps)
- **Ammo economy**: Start with 10, dinosaurs drop ammo equal to their max health when killed
- **Dinosaur art**: Procedurally drawn on canvas with context transformations (scale, rotate, flip)
- **Player art**: Safari explorer with rotating gun using ctx.save/translate/rotate/restore pattern
- **Input handling**:
  - Keyboard state stored in `game.keys` object, mouse position relative to canvas
  - Touch state in `game.touch` object with joystick (active, startX/Y, currentX/Y) and button states (shoot, jump)
  - Virtual joystick calculates normalized direction vector and speed multiplier from displacement
- **Mobile support**:
  - Viewport meta tag prevents zoom and enables proper mobile rendering
  - Responsive canvas sizing effect updates on window resize
  - Touch controls overlay with `pointer-events-auto` on interactive areas
  - Auto-aim on mobile: FIRE button targets nearest dinosaur, falls back to shooting right
  - Touch shoot has cooldown to prevent rapid fire (sets `game.touch.shoot = false` after firing)
- **Pause system**: Game loop continues but returns early when `showPauseMenu` is true; ESC key toggles pause
- **Settings system**:
  - Modal overlay with configurable options
  - Sound toggle (🔊/🔇 icon, ON/OFF button)
  - Volume slider (🔊 icon, 0-100% range with visual fill and percentage display)
  - Viewport size selector (📐 icon, Small/Medium/Large buttons with active state)
  - Accessible from: main menu (below START GAME), pause menu (4th option), gameplay HUD (⚙️ gear button)
  - Settings persist via localStorage (`jurassicEscapeSoundEnabled`, `jurassicEscapeVolume`, `jurassicEscapeViewportScale`)
  - When sound is enabled, plays test sound (`game_start`) for immediate feedback
  - Clicking settings from gameplay also triggers pause menu state
  - Viewport scale changes trigger canvas resize via useEffect dependency
- **Sound on menu actions**:
  - Continue/Restart: plays `unpause` (mapped to `game_start`)
  - Exit to Menu: plays `victory`
  - Pause: plays `pause`
- **When adding or updating features**: Consider how the feature works on both desktop and mobile
- Keep in mind reusability and separation of concerns. Look out for opportunities to write and use more expressive functions
- When adding new features and patterns, keep this file and the main README.md file up to date with relevant details
