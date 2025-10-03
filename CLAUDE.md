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
- **Game constants** defined in `GAME_CONSTANTS` object - all magic numbers extracted into documented constants (player stats, physics values, combat parameters, spawning rules, AI behavior, etc.)
- **Difficulty modifiers** in `DIFFICULTY_MODIFIERS` object - multipliers that adjust gameplay balance (player health/ammo, enemy speed, invincibility duration)

### Constants Architecture

The codebase follows a **constants-first approach** for type safety and maintainability:

**Core Constants:**

- `RANDOM_CENTER` (0.5): Mathematical constant for centering bidirectional random ranges
- `GAME_STATES`: Game state machine values (MENU, PLAYING, LEVEL_COMPLETE, WON, LOST)
- `WEAPON_TYPES`: Weapon identifiers with icons (REGULAR/💥, TRANQUILIZER/💉)
- `OBSTACLE_TYPES`: Environment types (TREE, BUSH)
- `POWERUP_TYPES`: Collectible types (HEALTH, SPEED)
- `AI.STATES`: Dinosaur behavior states (PATROL, CHASE, FLEE, TERRITORY_RETURN)

**Utility Functions:**

- `randomCentered(range)`: Generates random values centered around zero (±range/2) using RANDOM_CENTER constant
- `randomizeBubbleOffset()`: Returns randomized {offsetX, offsetY} for tar pit bubble positioning using `randomCentered()`

**Entity Factory Functions:**

- `createObstacle()`, `createPowerup()`, `createAmmoPickup()`, `createTarPit()`, `createElectricFence()`, etc.
- Encapsulate entity creation logic and use constants throughout

**Benefits:**

- **Type safety**: String literal typos caught at usage sites
- **Single source of truth**: Constants defined once, used everywhere
- **Better IDE support**: Autocomplete for all game types
- **Easier refactoring**: Change constant value in one place
- **Self-documenting code**: `GAME_CONSTANTS.AI.STATES.TERRITORY_RETURN` is clearer than `'return'`

### Game State Management

The main `JurassicEscape` component manages:

- **Game states**: Defined in `GAME_CONSTANTS.GAME_STATES` - `MENU`, `PLAYING`, `LEVEL_COMPLETE`, `WON`, `LOST`
- **React state**: `gameState`, `score`, `playerHealth`, `playerAmmo`, `currentLevel`, `speedBoostActive`, `showPauseMenu`, `showSettingsMenu`, `soundEnabled`, `volume`, `viewportScale`, `difficulty`, `autoAim`, `canvasSize`, `isTouchDevice`
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
- **Aiming**:
  - Desktop: Mouse position tracked for manual aiming, or auto-aim toggle for trackpad users
  - Mobile: Auto-aim always enabled (targets nearest dinosaur)
- **Shooting**:
  - Desktop: Click to shoot (uses mouse position or auto-aim based on setting)
  - Mobile: FIRE button with auto-aim
- **Auto-aim**: Optional feature (enabled via Settings) that automatically targets nearest dinosaur
  - Mandatory on mobile devices (always enabled, setting is read-only)
  - Enabled by default on Easy difficulty for desktop
  - Helpful for trackpad users who struggle with precise aiming
  - Shows "🎯 AUTO-AIM" badge in HUD when active (both desktop and mobile)
  - Fallback to click position if no dinosaurs present
- Touch controls automatically hidden on desktop (using Tailwind `md:hidden`)
- ESC to pause (shows pause menu with Continue/Restart/Exit options)
- Camera follows player with map boundary constraints

#### Entities

- **Player**: Safari explorer character with gun that aims at mouse cursor
  - Properties: health, ammo, tranqAmmo, currentWeapon, speed (base + boost modifier), jump physics (isJumping, jumpVelocity, jumpHeight)
  - Procedurally drawn with safari hat, khaki clothing, and rotating gun
  - Jump physics: velocity starts at 8, gravity of 0.5 per frame
  - Weapon switching: Q key (desktop) or touch button (mobile) to toggle between regular gun and tranquilizer
- **Dinosaurs** (6 types): Raptor, T-Rex, Stegosaurus, Dilophosaurus (aggressive), Parasaurolophus, Triceratops (herbivores)
  - Each has unique stats: health, speed, size, points, aggressive flag, territorial flag
  - **AI states** defined in `GAME_CONSTANTS.AI.STATES`: `PATROL`, `CHASE`, `FLEE`, `TERRITORY_RETURN`
  - **Aggressive dinosaurs** (Raptor, T-Rex, Stego): AI states PATROL or CHASE (aggro within range), damage player on contact
  - **Territorial dinosaurs** (Dilophosaurus): AI states PATROL, CHASE, or TERRITORY_RETURN
    - Territory system: 200 pixel radius zone, marked by skull emoji and dashed red circle
    - Aggressive only when player enters territory (within 250px aggro range)
    - Spit attack: 180px range, 2 second cooldown, yellow-green projectiles dealing 1 damage
    - Returns to territory center when straying beyond 80% of radius (50% when already returning)
    - Territory spawning avoids obstacles and other territories (440px minimum spacing)
    - Territory marker removed when dinosaur dies
    - Hysteresis on state transitions prevents rapid flipping at boundaries
  - **Herbivores** (Parasaurolophus, Triceratops): AI states PATROL or FLEE (flee within range), no damage to player
    - Flee range: 150 pixels, flee speed: 1.5x base speed
    - Lower point values (50/75) vs predators (100-300)
  - Obstacle avoidance: when blocked, try steering left/right; if both blocked, reverse
  - Hand-drawn canvas art with facing direction (horizontal flip for left movement)
    - Facing direction stored in `facingLeft` property, only updates when horizontal velocity exceeds 0.3
  - Sleep mechanic: tranquilized dinosaurs become `isSleeping`, don't move or attack, show Z-Z-Z animation
- **Obstacles**: Types defined in `GAME_CONSTANTS.OBSTACLE_TYPES` - `TREE` and `BUSH` with circular collision
- **Powerups**: Types defined in `GAME_CONSTANTS.POWERUP_TYPES` - `HEALTH` (red cross) and `SPEED` (lightning bolt)
- **Ammo pickups**: Dropped when dinosaurs die, with physics-based bouncing animation that continues until motion stops
- **Tranquilizer depots**: Green crates with syringe icons, provide 5 tranq ammo each

#### Collision Detection

All collision uses circle-vs-circle distance checks:

- Player vs obstacles (movement blocking, bypassed when jump height >= 25)
- Dinosaurs vs obstacles (movement blocking with steering avoidance)
- Player vs aggressive dinosaurs (damage, only when not sleeping)
- Bullets vs dinosaurs (regular bullets: damage/kill, tranquilizer: track hits and put to sleep)
- Spit projectiles vs player (1 damage, triggers invincibility)
- Player vs powerups/ammo/tranq depots (collection)

#### Level Progression

Levels defined in `levelConfigs`:

- Each level specifies dinosaur types/counts, obstacle count, powerup count, hazard counts (tar pits, electric fences)
- Win condition: Reach exit zone (golden EXIT rectangle at map coordinates 1800, 1400)
- Player stats reset between levels, score persists
- Hazard counts increase with difficulty: Level 1 (3 tar, 2 fence), Level 2 (5 tar, 4 fence), Level 3 (7 tar, 6 fence)

#### Sound System

- Uses `soundsRef` with `useCallback` for audio management
- Audio pooling via `cloneNode()` for simultaneous playback (prevents freezing and enables mixing)
- 17 sound effects: shoot, hit, death, pickup_ammo, pickup_health, pickup_speed, player_hurt, level_complete, game_over, victory, game_start, jump, pause, tranq_shoot, tranq_hit, pickup_tranq, electric_shock
- `unpause` mapped to `game_start` for reuse
- All sounds are .wav files in `./assets/` folder
- **Volume control**: `volume` state (0.0-1.0, default 0.3) controls audio volume; applied to all sounds in `playSound` function
- **Sound control**: `soundEnabled` state controls whether audio plays; checked at start of `playSound` function
- Settings persisted in localStorage as `jurassicEscapeSoundEnabled` (boolean), `jurassicEscapeVolume` (number), `jurassicEscapeAutoAim` (boolean), `jurassicEscapeViewportScale` (string), and `jurassicEscapeDifficulty` (string)

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
- Layered drawing order: background → exit → tar pits → electric fences → obstacles → powerups → tranq depots → territories → dinosaurs → player → bullets → spit projectiles → ammo pickups → floating texts
- Health bars drawn above dinosaurs
- Sleeping Z-Z-Z animation drawn above sleeping dinosaurs (3 "Z"s of increasing size with bobbing animation)
- Floating text system: score popups (+N), tranq hit counters (1/2, 2/3), ammo pickups with fade-out over 1 second
- Jump shadow drawn below player when airborne
- Speed boost glow effect (cyan aura) around player
- Invincibility flashing effect (semi-transparent on alternating frames)
- Bullet coloring: yellow for regular, green for tranquilizer
- UI overlay in React (hearts, 💥 regular ammo, 💉 tranq ammo with weapon highlighting, score, speed boost indicator, settings gear button)
- Touch controls overlay (virtual joystick, weapon switch button, JUMP button, FIRE button) positioned absolutely over canvas
- Pause menu overlay with semi-transparent backdrop when ESC pressed
- Settings menu overlay accessible from main menu, pause menu, and gameplay HUD (gear button)

## Key Implementation Details

- **Collision system**: Player and aggressive dinosaurs collide with obstacles; dinosaurs don't collide with each other; herbivores don't damage player
- **Environmental hazards**: Two types of hazards affect gameplay:
  - **Tar pits**: Circular hazards (40px radius) that slow entities to 50% speed when inside
    - Dark brown/black visual with 3 bubbles per pit, spawning every 0.5 seconds with 40-frame animation
    - Bubbles grow from radius 2 to 6 pixels with fade-out, randomized positions within 20px spread
    - Affects both player and dinosaurs equally
    - No damage, purely movement penalty
    - Spawn with collision avoidance (obstacles, other tar pits)
  - **Electric fences**: Rectangular barriers (80x10px) that damage player and stun dinosaurs
    - Player: 1 damage on contact (with 1s cooldown), pushed back by 50px, plays electric_shock sound, can jump over if high enough
    - Dinosaurs: Bounced back 50px, stunned (sleep state) for half of tranq duration
    - Stun doesn't reset tranq hits or award points (pure mechanical stun)
    - **Learned avoidance**: Dinosaurs remember fences that zapped them for 10 seconds, steering around with 30px safety buffer
    - Memory decay: After 10 seconds, dinos forget and may hit the same fence again
    - Blue-gray base with yellow electric wires, spark animation every 0.5 seconds
    - Electric zap visual effect with yellow aura and rotating lightning bolts (30 frames)
    - Randomly rotated for varied placement
    - Spawn with collision avoidance (obstacles, other fences, tar pits)
- **Jump mechanic**: Helps escape stuck spawn positions between bushes; also bypasses electric fences; visual feedback with shadow
- **Speed boost**: Multiplies player speed by 1.8x for 300 frames (5 seconds at 60fps)
- **Ammo economy**: Start with 10 regular, 15 tranquilizer; dinosaurs drop regular ammo equal to their max health when killed
  - Ammo pickups use physics-based bouncing animation
  - Bounces continuously with decreasing amplitude (damping: -0.7, friction: 0.85)
  - Disappears when velocity drops below minimum threshold (0.05)
  - No fixed bounce count or lifetime limit
- **Tranquilizer mechanic**: Multi-shot system based on dinosaur size
  - Raptor/Para: 1 shot, Dilo/Stego/Trike: 2 shots, T-Rex: 3 shots
  - Shows hit counter as floating text (1/2, 2/3, etc.)
  - Awards 50% of kill points when dinosaur is put to sleep
  - Sleep duration: Raptor 8s, Para 7s, Dilo/Stego/Trike 6s, T-Rex 4s
  - Sleeping dinosaurs don't move or attack, display Z-Z-Z animation
  - Tranq hit counter resets when dinosaur wakes up
- **Territory system**: Territorial dinosaurs defend specific zones
  - Territory spawning uses collision avoidance (obstacles and other territories)
  - Visual markers: skull emoji at center, dashed red circle outline
  - Linked data structure: territories array tracks ownership, dinosaurs store territoryIndex
  - Territory cleanup: removed when dinosaur dies with index rebalancing
- **Dinosaur art**: Procedurally drawn on canvas with context transformations (scale, rotate, flip)
  - Parasaurolophus: Tan/brown with distinctive swept-back head crest
  - Triceratops: Tan/beige with three horns and prominent neck frill
  - Dilophosaurus: Lime green with expandable red neck frill (expands when chasing), red eyes
- **Player art**: Safari explorer with rotating gun using ctx.save/translate/rotate/restore pattern
- **Floating text system**: Reusable for score popups, hit counters, and ammo pickups with fade-out animation
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
  - Difficulty selector (⚔️ icon, Easy/Normal/Hard buttons with active state and stat preview)
  - Auto-aim toggle (🎯 icon, ON/OFF button, helpful for trackpad users)
    - Read-only on mobile (always ON, button disabled with visual feedback)
    - Helper text changes to "Always enabled on mobile" on touch devices
  - Accessible from: main menu (below START GAME), pause menu (4th option), gameplay HUD (⚙️ gear button)
  - Settings persist via localStorage (`jurassicEscapeSoundEnabled`, `jurassicEscapeVolume`, `jurassicEscapeViewportScale`, `jurassicEscapeDifficulty`, `jurassicEscapeAutoAim`)
  - When sound is enabled, plays test sound (`game_start`) for immediate feedback
  - When difficulty is set to Easy, auto-aim is automatically enabled (but can be manually toggled on desktop)
  - When touch device is detected, auto-aim is forced ON and persisted to localStorage
  - Clicking settings from gameplay also triggers pause menu state
  - Viewport scale and difficulty changes require restarting level to take effect
- **Sound on menu actions**:
  - Continue/Restart: plays `unpause` (mapped to `game_start`)
  - Exit to Menu: plays `victory`
  - Pause: plays `pause`
- **When adding or updating features**: Consider how the feature works on both desktop and mobile
- Keep in mind reusability and separation of concerns. Look out for opportunities to write and use more expressive functions
- When adding new features and patterns, keep this file and the main README.md file up to date with relevant details
- **Code organization principles**:
  - Avoid magic numbers - extract to `GAME_CONSTANTS` with descriptive names
  - Avoid string literals for types - use type constants (GAME_STATES, AI.STATES, WEAPON_TYPES, etc.)
  - Extract repeated patterns into utility functions (see `randomCentered()`, `randomizeBubbleOffset()`)
  - Use factory functions for entity creation (see `createObstacle()`, `createPowerup()`, etc.)
  - Keep constants at the top of the file before utility functions and components
