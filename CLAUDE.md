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
- **Difficulty modifiers** in `DIFFICULTY_MODIFIERS` object - multipliers that adjust gameplay balance (player health/ammo, enemy speed, invincibility duration, ammo bouncing physics)

### Constants Architecture

The codebase follows a **constants-first approach** for type safety and maintainability:

**Core Constants:**

- `RANDOM_CENTER` (0.5): Mathematical constant for centering bidirectional random ranges
- `GAME_STATES`: Unified game state configuration with all properties per state:
  - `GAME_STATES.MENU`: `{ id, isTerminal, runsGameLoop, canPause, soundEffect }`
  - `GAME_STATES.PLAYING`: `{ id, isTerminal, runsGameLoop, canPause, soundEffect }`
  - `GAME_STATES.LEVEL_COMPLETE`: `{ id, isTerminal, runsGameLoop, canPause, soundEffect }`
  - `GAME_STATES.WON`: `{ id, isTerminal, runsGameLoop, canPause, soundEffect }`
  - `GAME_STATES.LOST`: `{ id, isTerminal, runsGameLoop, canPause, soundEffect }`
  - Each state is a full object (not just a string ID)
  - React state stores full state object: `gameState = GAME_CONSTANTS.GAME_STATES.PLAYING`
  - Direct property access: `gameState.isTerminal`, `gameState.runsGameLoop`, `gameState.canPause`
  - `isTerminal`: Boolean indicating if game loop should stop (LOST, WON, LEVEL_COMPLETE are terminal)
  - `runsGameLoop`: Boolean indicating if game loop should execute (only PLAYING runs the loop)
  - `canPause`: Boolean indicating if pause menu is allowed (only PLAYING can be paused)
  - `soundEffect`: References SOUNDS object, assigned after SOUNDS is defined
  - State transitions handled by `transitionToState()` helper which automatically plays associated sound effects
- `WEAPONS`: Unified weapon configuration with all properties per weapon type:
  - `WEAPONS.REGULAR`: `{ id, icon, bulletColor, speed, radius, ammoConfig: { playerKey, stateKey }, soundEffect, bulletFactory }`
  - `WEAPONS.TRANQUILIZER`: `{ id, icon, bulletColor, speed, radius, startAmmo, scoreMultiplier, ammoConfig: { playerKey, stateKey }, soundEffect, bulletFactory }`
  - Each weapon type is a full object (not just a string ID)
  - Entities store full weapon object: `bullet.weaponType = WEAPONS.REGULAR`, `player.currentWeapon = WEAPONS.REGULAR`
  - Direct property access: `bullet.weaponType.bulletColor`, `player.currentWeapon.ammoConfig.playerKey`
  - All weapon properties (visual, mechanical, ammo, state management) in one place
  - `ammoConfig.playerKey`: Property name on player object (e.g., 'ammo', 'tranqAmmo')
  - `ammoConfig.stateKey`: React state variable name (e.g., 'playerAmmo', 'playerTranqAmmo') for setters and stateAccumulator
  - `soundEffect` references SOUNDS object, assigned after SOUNDS is defined
  - `bulletFactory` references (`createBullet`, `createTranquilizer`) assigned after functions are defined
  - Unified `fireWeapon()` helper uses weapon config for all shooting logic (no weapon-specific conditionals)
- `DINOSAURS`: Unified dinosaur configuration with all properties per dinosaur type:
  - `DINOSAURS.VELOCIRAPTOR`: `{ id, baseSize, health, speed, color, points, aggressive, territorial, tranqShots, sleepDuration, spitAttack }`
  - Similar structure for TYRANNOSAURUS_REX, STEGOSAURUS, PARASAUROLOPHUS, TRICERATOPS, DILOPHOSAURUS
  - Each dinosaur type is a full object with stats, AI flags, tranquilizer properties, and capabilities
  - Entities store full dinosaur object: `dino.type = GAME_CONSTANTS.DINOSAURS.VELOCIRAPTOR`
  - Direct property access: `dino.type.tranqShots`, `dino.type.sleepDuration`, `dino.type.baseSize`
  - Capability system: `spitAttack` is null for non-spitting dinos, object with properties for spitters
- `OBSTACLES`: Unified object-based obstacle definitions with full configuration per type:
  - `OBSTACLES.TREE`: `{ id, blocksMovement, zIndex, colors: {...}, foliageLayers }`
  - `OBSTACLES.BUSH`: `{ id, blocksMovement, zIndex, colors: {...}, clusters }`
  - `OBSTACLES.MUSHROOM`: `{ id, blocksMovement, zIndex, colors: {...}, sizes: {...} }`
  - `OBSTACLES.GRASS`: `{ id, blocksMovement, zIndex, colors: {...}, sizes: {...} }`
  - Each obstacle type is a full object (not just a string ID)
  - Entity objects store `obstacleType` field containing reference to constant: `{ x, y, radius, obstacleType: OBSTACLES.TREE }`
  - All visual properties (colors, sizes) and behavior properties (blocksMovement, zIndex) in one place
- `POWERUPS`: Unified powerup configuration with all properties per powerup type:
  - `POWERUPS.HEALTH`: `{ id, radius, icon, color, fillColor, strokeColor, lineWidth, drawIcon }`
  - `POWERUPS.SPEED`: `{ id, radius, icon, color, durationFrames, multiplier, fillColor, strokeColor, lineWidth, drawIcon }`
  - Each powerup type is a full object with visual and functional properties
  - Entities store full powerup object: `powerup.powerupType = POWERUPS.HEALTH`
  - Direct property access: `powerup.powerupType.fillColor`, `powerup.powerupType.durationFrames`
- `HAZARDS`: Unified hazard configuration with all properties per hazard type:
  - `HAZARDS.TAR_PIT`: `{ id, baseRadius, sizeVariance, slowMultiplier, minSpacing, colors: {...}, rimWidth, bubble: {...} }`
  - `HAZARDS.ELECTRIC_FENCE`: `{ id, width, height, damage, damageCooldown, pushbackForce, stunDurationMultiplier, minSpacing, obstacleClearance, colors: {...}, wireWidth, spark: {...}, zap: {...}, memory: {...} }`
  - Each hazard type is a full object with visual, mechanical, and behavioral properties
  - Entities store full hazard object: `tarPit.hazardType = GAME_CONSTANTS.HAZARDS.TAR_PIT`, `fence.hazardType = GAME_CONSTANTS.HAZARDS.ELECTRIC_FENCE`
  - Direct property access: `tarPit.hazardType.slowMultiplier`, `fence.hazardType.zap.duration`, `fence.hazardType.colors.wire`
- `AI.STATES`: Dinosaur behavior states (PATROL, CHASE, FLEE, TERRITORY_RETURN)
- `AI.EXIT_TERRITORY_RADIUS`: 300px radius for exit protection zones
- `AI.EXIT_GUARD_AGGRO_RANGE`: 350px chase range for exit guards
- `EXIT.LOCK_ICON`: 🔒 emoji for locked exits
- `EXIT.LOCK_ICON_SIZE`: 32px font size for lock icon
- `EXIT.LOCK_ICON_OFFSET_Y`: -40px position above exit
- `VIEWPORTS`: Canvas size presets (small, medium, large) with width/height/label
- `CANVAS_SIZING`: Responsive canvas layout constants (padding, spacing for desktop/mobile)
- `SOUNDS`: Unified audio configuration with `id` and `path` for each sound (SHOOT, HIT, DEATH, etc.)

**Utility Functions:**

- `randomCentered(range)`: Generates random values centered around zero (±range/2) using RANDOM_CENTER constant
- `randomizeBubbleOffset()`: Returns randomized {offsetX, offsetY} for tar pit bubble positioning using `randomCentered()`
- `transitionToState(newState, playSound, stateAccumulator, setGameState)`: Unified state transition handler
  - Automatically sets new game state via stateAccumulator (game loop) or setGameState (event handlers)
  - Automatically plays associated sound effect if state has one
  - Eliminates manual `playSound()` calls after state changes
  - Usage: `transitionToState(GAME_CONSTANTS.GAME_STATES.LOST, playSound, stateAccumulator)` or `transitionToState(GAME_CONSTANTS.GAME_STATES.WON, playSound, null, setGameState)`

**Entity Factory Functions:**

- `createObstacle()`, `createPowerup()`, `createAmmoPickup()`, `createTarPit()`, `createElectricFence()`, `createMushroomPatch()`, etc.
- `createBullet(fromX, fromY, toX, toY)`: Creates regular bullet projectile
- `createTranquilizer(fromX, fromY, toX, toY)`: Creates tranquilizer dart projectile
- `createTranqDepot(mapWidth, mapHeight)`: Creates tranquilizer depot with random position
- `createAmmoDepot(x, y, amount)`: Creates regular ammo depot with fixed position and amount
- `createExitTerritory(exitX, exitY, totalGuards)`: Creates exit protection territory with guard count tracking
- `createExitGuard(dinoType, angle, exitX, exitY, territoryIndex, difficulty)`: Creates guard dinosaur positioned around exit
- Encapsulate entity creation logic and use constants throughout

**Weapon System Functions:**

- `fireWeapon(weapon, player, targetX, targetY, game, playSound, stateAccumulator, setters)`: Unified weapon firing logic
  - Uses weapon config for all behavior (ammo checking, bullet creation, state updates, sound)
  - Works with both state accumulator (game loop) and direct setters (event handlers)
  - Eliminates weapon-specific conditionals throughout codebase
  - Returns boolean indicating if weapon was fired successfully

**Benefits:**

- **Type safety**: String literal typos caught at usage sites
- **Single source of truth**: All properties for each entity type in one place (like OBSTACLES pattern)
- **Better IDE support**: Autocomplete for all entity properties
- **Easier refactoring**: Change values in one location, no scattered constants
- **Self-documenting code**: `GAME_CONSTANTS.DINOSAURS.VELOCIRAPTOR.tranqShots` is clearer than separate constants
- **Eliminates lookup functions**: Direct property access instead of switch statements (e.g., removed `getSleepDurationForType()`, `getShotsNeededForType()`)
- **Consistent pattern**: All entity types (WEAPONS, DINOSAURS, OBSTACLES, POWERUPS, HAZARDS, GAME_STATES) follow the same unified object pattern - store full object references, access properties directly

### Game State Management

The main `JurassicEscape` component manages:

- **Game states**: Defined in `GAME_CONSTANTS.GAME_STATES` as full objects with properties
  - Each state object contains: `id`, `isTerminal`, `runsGameLoop`, `canPause`, `soundEffect`
  - React state stores the full state object (not a string ID)
  - State comparisons use object reference equality: `gameState === GAME_CONSTANTS.GAME_STATES.PLAYING`
  - State transitions use `transitionToState()` helper to automatically handle sound effects
  - Properties enable declarative logic: `if (!gameState.runsGameLoop) return;` instead of `if (gameState !== PLAYING)`
- **React state**: `gameState`, `score`, `playerHealth`, `playerAmmo`, `currentLevel`, `speedBoostActive`, `showPauseMenu`, `showSettingsMenu`, `soundEnabled`, `volume`, `viewportScale`, `difficulty`, `autoAim`, `canvasSize`, `isTouchDevice`
- **Game ref** (`gameRef`): Contains mutable game objects updated every frame
  - Player, bullets, dinosaurs, obstacles, decorations, powerups, ammoPickups, tranqDepots, ammoDepots
  - Camera position, map dimensions, input tracking
  - Touch controls state (joystick, shoot, jump)

### React Performance Patterns

The codebase follows React performance best practices to minimize re-renders and optimize the game loop:

**Component Extraction with React.memo:**

- `GameHUD` - Displays player stats, score, and control buttons (health, ammo, level, speed boost indicator)
- `TouchControls` - Virtual joystick and mobile buttons (weapon switch, jump, fire)
- `PauseMenu` - Pause overlay with Continue/Restart/Exit/Settings buttons
- `SettingsMenu` - Settings overlay with sound, volume, viewport, difficulty, and auto-aim controls

All UI components wrapped with `React.memo` to prevent unnecessary re-renders when parent state changes. Components only re-render when their props actually change.

**Handler Memoization with useCallback:**

All event handlers wrapped in `useCallback` and organized into logical sections within the JurassicEscape component:

- **Game Flow Handlers**: `startGame`, `nextLevel`
- **Desktop Input Handlers**: `handleKeyDown`, `handleKeyUp`, `handleMouseMove`, `handleClick`
- **Mobile Touch Handlers**: `handleJoystickStart`, `handleJoystickMove`, `handleJoystickEnd`, `handleWeaponSwitch`, `handleJumpStart`, `handleShootStart`, `handleShootEnd`
- **UI Interaction Handlers**: `handlePauseClick`, `handleSettingsClick`, `handleOpenSettings`, `handleCloseSettings`
- **Pause Menu Handlers**: `handleContinue`, `handleRestartLevel`, `handleExitToMenu`, `handleOpenSettingsFromPause`
- **Settings Menu Handlers**: `handleSoundToggle`, `handleVolumeChange`, `handleViewportScaleChange`, `handleDifficultyChange`, `handleAutoAimToggle`

Handlers are grouped with clear section comments for easy navigation within the single-file architecture.

**State Batching Pattern:**

The game loop uses a `stateAccumulator` object to batch all React state updates into a single render cycle per frame:

```javascript
const stateAccumulator = {};
// Update functions populate stateAccumulator
updatePlayerSpeed(game, stateAccumulator);
updateHazards(game, difficulty, playSound, stateAccumulator);
updatePlayerMovement(game, canvas, stateAccumulator);
// ... more updates

// Batch apply all state updates at once (single re-render)
if (stateAccumulator.gameState !== undefined) setGameState(stateAccumulator.gameState);
if (stateAccumulator.playerHealth !== undefined) setPlayerHealth(stateAccumulator.playerHealth);
if (stateAccumulator.playerAmmo !== undefined) setPlayerAmmo(stateAccumulator.playerAmmo);
// ... etc
```

This prevents multiple re-renders per frame, which would cause performance issues at 60 FPS.

**Helper Function Extraction:**

Game loop logic extracted into focused helper functions to reduce complexity:

- `updatePlayerSpeed` - Speed modifiers (tar pits, speed boost)
- `updatePlayerMovement` - Input handling, physics, camera tracking
- `updateHazards` - Electric fence collision and damage
- `updateCollectibles` - Powerup, ammo pickup, tranq depot, and ammo depot collection
- `updateBullets` - Bullet movement and dinosaur collision
- `updateSpitProjectiles` - Spit attack logic
- `updateDinosaurAI` - Consolidated AI for all dinosaur types
- `updateDinosaurs` - AI execution, movement, sleep state

**Benefits:**

- Reduced re-renders from 100s per second to only when state actually changes
- Stable event handler references prevent component prop changes
- Single-file architecture maintained while following React best practices
- Main component remains readable despite game complexity

### Game Loop Pattern

Uses `useEffect` with `requestAnimationFrame` for the main game loop:

1. **Update phase**: Physics, collision detection, AI (via helper functions)
2. **State batching**: Accumulate all state changes, apply once per frame
3. **Render phase**: Canvas drawing with camera offset
4. Loop continues until component unmounts or game state changes

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
    - **Spit attack capability**: Defined in `dino.type.spitAttack` object (null for non-spitters)
      - Dilophosaurus: `{ range: 180, cooldown: 120, projectileSpeed: 4, projectileRadius: 6, damage: 1 }`
      - Capability-based check: `if (dino.type.spitAttack && ...)` instead of hardcoded type check
      - Yellow-green projectiles, configurable per dinosaur type
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
- **Obstacles**: Defined in `GAME_CONSTANTS.OBSTACLES` as full object references
  - `TREE` and `BUSH`: blocking obstacles (blocksMovement: true, zIndex: 2) with circular collision
  - Entity objects store reference: `{ x, y, radius, obstacleType: OBSTACLES.TREE }`
- **Decorations**: Walkable vegetation with visual-only purpose
  - `MUSHROOM`: non-blocking (blocksMovement: false, zIndex: 1), spawn in colorful patches with shared cap colors
  - `GRASS`: non-blocking (blocksMovement: false, zIndex: 0), ground layer with individual blade rendering
  - Entity objects store reference: `{ x, y, radius, obstacleType: OBSTACLES.MUSHROOM }`
- **Powerups**: Types defined in `GAME_CONSTANTS.POWERUPS` - `HEALTH` (red cross, ❤️ icon) and `SPEED` (lightning bolt, ⚡ icon)
- **Ammo pickups**: Single pickup per defeated dinosaur, bounces vertically with progressive fading amplitude
  - Each pickup restores ammo equal to dinosaur's max health (1-4 shots depending on dinosaur type)
  - Displays "×N" badge showing ammo amount on the bouncing ball
  - Vertical-only bouncing (no horizontal movement) with progressive damping
  - Spawns at dinosaur's feet, launches upward with initial velocity of -7 and gravity of 0.3
  - Maximum 10 bounces before automatic removal (configurable via `MAX_BOUNCES`)
  - Progressive damping: each bounce loses progressively more energy (30% reduction by final bounce)
  - Can only be collected after first bounce (prevents instant collection)
  - Bouncing duration affected by difficulty: Easy bounces 40% longer, Normal is baseline, Hard bounces 20% shorter
  - Controlled by damping multipliers in `DIFFICULTY_MODIFIERS`
  - Floating text shows "+N" when collected
- **Tranquilizer depots**: Green crates with syringe icons (💉), provide 5 tranq ammo each, randomly placed
- **Regular ammo depots**: Blue crates with bullet icons (💥), provide minimal regular ammo to prevent softlocks
  - 2 depots per level at fixed positions: one near spawn (250, 150), one in far corner
  - Level 1: 3 ammo per depot (strongest guard: Raptor=2 health +1)
  - Level 2: 4 ammo per depot (strongest guard: Dilo=3 health +1)
  - Level 3: 5 ammo per depot (strongest guard: T-Rex=4 health +1)
  - Total ammo from depots alone is insufficient; players must collect dropped ammo

#### Collision Detection

All collision uses circle-vs-circle distance checks:

- Player vs obstacles (movement blocking, bypassed when jump height >= 25)
- Dinosaurs vs obstacles (movement blocking with steering avoidance)
- Player vs aggressive dinosaurs (damage, only when not sleeping)
- Bullets vs dinosaurs (regular bullets: damage/kill, tranquilizer: track hits and put to sleep)
- Spit projectiles vs player (1 damage, triggers invincibility)
- Player vs powerups/ammo pickups/tranq depots/ammo depots (collection)

#### Level Progression

Levels defined in `LEVEL_CONFIGS`:

- **Per-level world sizes**: Each level has its own map dimensions, creating progressively larger worlds
  - Level 1: 2400x1800 (compact, manageable starting area)
  - Level 2: 3600x2700 (50% larger, more exploration required)
  - Level 3: 4800x3600 (2x larger, epic final challenge with room for all 22 dinosaurs)
  - Viewport size remains constant regardless of world size (player sees same amount)
- Each level specifies: map dimensions, exit position, exit guards, dinosaur types/counts, obstacle count, powerup count, hazard counts
- **Exit position**: Placed near far edge of each level's map
  - Level 1: (1800, 1400)
  - Level 2: (3000, 2200)
  - Level 3: (4200, 3000)
- **Exit guard system**: Exit is protected by guard dinosaurs that must be defeated to unlock
  - Level 1: 3 Velociraptors
  - Level 2: 3 Velociraptors + 1 Dilophosaurus
  - Level 3: 3 Velociraptors + 1 Dilophosaurus + 1 T-Rex (boss fight)
  - Guards patrol a 300-pixel radius territory around exit (larger than normal 200px territories)
  - Exit territory has no skull marker (only dashed red circle)
  - Exit shows 🔒 lock icon and appears gray when locked
  - When all guards are defeated, exit unlocks (turns golden) and territory is removed
  - Guards use existing territorial AI (PATROL, CHASE, TERRITORY_RETURN states)
  - Factory functions: `createExitTerritory()`, `createExitGuard()`
- Win condition: Reach exit zone (golden EXIT rectangle) when unlocked
- Player stats reset between levels, score persists
- Entity counts scale naturally with larger worlds: Level 1 (40 obstacles, 3 tar, 2 fence), Level 2 (50 obstacles, 5 tar, 4 fence), Level 3 (60 obstacles, 7 tar, 6 fence)

#### Sound System

- Uses `soundsRef` with `useCallback` for audio management
- Audio pooling via `cloneNode()` for simultaneous playback (prevents freezing and enables mixing)
- **Unified SOUNDS constant**: Each sound defined with `id` (runtime lookup key) and `path` (audio file location)
  - Prevents implicit coupling between sound names and file paths
  - Single source of truth - can't add sound without path or vice versa
  - Usage: `playSound(SOUNDS.SHOOT)` - function automatically extracts `.id` property
  - Example: `SOUNDS.SHOOT = { id: 'shoot', path: './assets/shoot.wav' }`
- 18 sound effects: SHOOT, HIT, DEATH, PICKUP_AMMO, PICKUP_HEALTH, PICKUP_SPEED, PLAYER_HURT, LEVEL_COMPLETE, GAME_OVER, VICTORY, GAME_START, JUMP, PAUSE, TRANQ_SHOOT, TRANQ_HIT, PICKUP_TRANQ, ELECTRIC_SHOCK, UNPAUSE
- `UNPAUSE` is an alias that points to `game_start.wav` for reuse
- All sounds are .wav files in `./assets/` folder
- **Volume control**: `volume` state (0.0-1.0, default 0.3) controls audio volume; applied to all sounds in `playSound` function
- **Sound control**: `soundEnabled` state controls whether audio plays; checked at start of `playSound` function
- Settings persisted in localStorage as `jurassicEscapeSoundEnabled` (boolean), `jurassicEscapeVolume` (number), `jurassicEscapeAutoAim` (boolean), `jurassicEscapeViewportScale` (string), and `jurassicEscapeDifficulty` (string)

### Rendering

- **Viewport scaling system**: Configurable canvas resolution via `GAME_CONSTANTS.VIEWPORTS`
  - Small: 800x600 (default)
  - Medium: 1000x750
  - Large: 1200x900
  - All scales maintain 4:3 aspect ratio
  - Game world size varies per level (Level 1: 2400x1800, Level 2: 3600x2700, Level 3: 4800x3600)
  - Viewport size is independent of world size - larger viewports show more of the current level at once
  - Canvas sizing constants in `GAME_CONSTANTS.CANVAS_SIZING` handle responsive layout
- **Responsive canvas sizing**: Scales to fit screen on mobile while maintaining aspect ratio of selected scale
- Canvas styled with CSS to match `canvasSize` state (width/height in pixels)
- Camera offset (`cameraX`, `cameraY`) applied to all draw calls
- Layered drawing order: background → exit → tar pits → electric fences → obstacles → decorations → powerups → tranq depots → ammo depots → territories → dinosaurs → sleeping Z-Z-Z → player → bullets → spit projectiles → ammo pickups → floating texts
- Health bars drawn above dinosaurs (constants in `HEALTH_BAR`)
- **Sleeping Z-Z-Z animation**: Batched rendering after dinosaur loop (3 "Z"s of increasing size with bobbing animation)
  - Optimized with batched canvas state, viewport culling, shadowBlur instead of strokeText
  - 50% fewer draw calls (3 fillText vs 6 strokeText+fillText per sleeping dino)
  - Visual constants in `Z_ANIMATION` (fonts, colors, offsets, animation params)
- **Floating text system**: Batched rendering with optimization (score popups, tranq hit counters, ammo pickups)
  - Viewport culling with 50px buffer skips off-screen texts
  - Batched canvas state setup (font, shadow set once before loop)
  - shadowBlur replaces strokeText for 50% fewer draw calls
  - Visual constants in `FLOATING_TEXT`
- **Canvas rendering performance patterns**:
  - Batch canvas state changes (set once before loop, reset once after)
  - Use viewport culling to skip off-screen entities
  - Prefer shadowBlur over strokeText+fillText (50% fewer draw calls)
  - Extract magic numbers to constants for maintainability
  - Exit visual constants in `EXIT`, spit projectile colors in `DILOPHOSAURUS.spitAttack`, ammo pickup color in `AMMO_PICKUP`
- Jump shadow drawn below player when airborne
- Speed boost glow effect (cyan aura) around player
- Invincibility flashing effect (semi-transparent on alternating frames)
- Bullet coloring: yellow for regular, green for tranquilizer
- UI overlay in React (hearts, 💥 regular ammo, 💉 tranq ammo with weapon highlighting, score, speed boost indicator, settings gear button)
- Touch controls overlay (virtual joystick, weapon switch button, JUMP button, FIRE button) positioned absolutely over canvas
- Pause menu overlay with semi-transparent backdrop when ESC pressed
- Settings menu overlay accessible from main menu, pause menu, and gameplay HUD (gear button)

## Key Implementation Details

- **Collision system**: Player and aggressive dinosaurs collide with blocking obstacles (trees/bushes with `blocksMovement: true`); decorations (mushrooms/grass with `blocksMovement: false`) are walkable; dinosaurs don't collide with each other; herbivores don't damage player
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
- **Speed boost**: Multiplies player speed by `GAME_CONSTANTS.POWERUPS.SPEED.multiplier` (1.8x) for `GAME_CONSTANTS.POWERUPS.SPEED.durationFrames` (300 frames = 5 seconds at 60fps)
- **Ammo economy**: Start with 10 regular ammo (`GAME_CONSTANTS.PLAYER.START_AMMO`), tranquilizer ammo from `GAME_CONSTANTS.WEAPONS.TRANQUILIZER.startAmmo` (15); dinosaurs drop single ammo pickup equal to their max health when killed
  - Single ammo pickup per dinosaur with vertical-only bouncing animation
  - Physics: initial velocity -7, gravity 0.3, base damping -0.65
  - Exactly 10 bounces before removal (lifetime controlled by bounce count, not velocity/duration)
  - **Difficulty scaling affects bounce physics** (count remains 10 across all difficulties):
    - **Easy difficulty**: Higher/slower bounces (damping: -0.741, 40% more bouncy) - 10 bounces spread over longer duration
    - **Normal difficulty**: Baseline bounces (damping: -0.65) - 10 bounces at standard duration
    - **Hard difficulty**: Lower/faster bounces (damping: -0.559, 20% less bouncy) - 10 bounces complete more quickly
  - Progressive damping factor of 0.3 (30% energy reduction by final bounce)
  - Can only be collected after first bounce to prevent instant pickup
- **Tranquilizer mechanic**: Multi-shot system based on dinosaur size (properties stored in `GAME_CONSTANTS.DINOSAURS`)
  - Raptor/Para: 1 shot (`tranqShots: 1`), Dilo/Stego/Trike: 2 shots, T-Rex: 3 shots
  - Shows hit counter as floating text (1/2, 2/3, etc.)
  - Awards `GAME_CONSTANTS.WEAPONS.TRANQUILIZER.scoreMultiplier` (50%) of kill points when dinosaur is put to sleep
  - Sleep duration stored per dinosaur: Raptor 480 frames (8s), Para 420 (7s), Dilo/Stego/Trike 360 (6s), T-Rex 240 (4s)
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
  - Canvas focus management: `tabIndex={0}` on canvas for keyboard focusability, `canvas.focus()` called when game loop starts and after unpausing
  - `preventDefault()` on all game keys (WASD, arrows, spacebar, Q, ESC) to prevent browser shortcuts from interfering
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
  - **Unified object pattern**: Follow the OBSTACLES/WEAPONS/DINOSAURS/POWERUPS/HAZARDS/GAME_STATES pattern - each entity/state type is a full object with all properties
    - Store full config object reference: `obstacleType: OBSTACLES.TREE`, `weaponType: WEAPONS.REGULAR`, `dino.type: DINOSAURS.VELOCIRAPTOR`, `powerupType: POWERUPS.HEALTH`, `hazardType: HAZARDS.TAR_PIT`, `gameState: GAME_STATES.PLAYING`
    - All entity and state types (WEAPONS, DINOSAURS, OBSTACLES, POWERUPS, HAZARDS, GAME_STATES) store full object references
    - All properties (visual, behavioral, mechanical, functional) defined in one place
    - Direct property access: `bullet.weaponType.bulletColor`, `dino.type.tranqShots`, `powerup.powerupType.durationFrames`, `obstacle.obstacleType.colors`, `fence.hazardType.zap.duration`, `gameState.isTerminal`
    - Capability system: Use nullable objects (e.g., `spitAttack: null` vs `spitAttack: { ... }`) for optional features
  - **Separate generic spawn constants from type-specific properties**: Generic spawn logic (e.g., `DEFAULT_ENTITY_CLEARANCE`, `HAZARD_SPAWN_MAX_ATTEMPTS`) belongs in `SPAWN` namespace, not in type definitions
  - Avoid magic numbers - extract to `GAME_CONSTANTS` with descriptive names
  - Avoid string literals for types - use object constants (OBSTACLES.TREE not 'tree', GAME_STATES.PLAYING not 'playing')
  - Pass full object references when possible - entities store `obstacleType: OBSTACLES.TREE`, code accesses `obs.obstacleType.colors.trunkBase`; state stores `gameState: GAME_STATES.PLAYING`, code accesses `gameState.runsGameLoop`
  - Eliminate lookup/helper functions when consolidating - replace switch statements with direct property access (e.g., removed `isTerminalState()` in favor of `gameState.isTerminal`)
  - Extract repeated patterns into utility functions (see `randomCentered()`, `randomizeBubbleOffset()`, `transitionToState()`)
  - Use factory functions for entity creation (see `createObstacle()`, `createPowerup()`, `createTarPit()`, `createElectricFence()`, etc.)
  - Separate concerns: blocking entities (`obstacles` array) vs decorative (`decorations` array), determined by `blocksMovement` property
  - Keep constants at the top of the file before utility functions and components
