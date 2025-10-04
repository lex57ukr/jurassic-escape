# CLAUDE.md

This file provides architectural guidance for working with this codebase.

## Project Overview

**Jurassic Escape** is a single-file HTML5 browser game built with React (via CDN), Canvas API, and Tailwind CSS. The entire game exists in one HTML file with embedded JavaScript using Babel standalone for JSX transformation.

**Run:** Open `index.html` directly in a web browser. No build process or dependencies required.

## File Structure & Organization

The codebase follows a strict 6-section organization pattern:

1. **Constants Section** (~lines 39-778)
   - `GAME_CONSTANTS` - All game configuration (player stats, AI behavior, entity types, etc.)
   - `DIFFICULTY_MODIFIERS` - Difficulty scaling multipliers
   - `LEVEL_CONFIGS` - Level definitions
   - `SOUNDS` - Audio file paths

2. **Static Data Bindings** (~lines 780-795)
   - Binds static data references (sounds) to constants
   - Weapon sound effects → WEAPONS types
   - Game state sound effects → GAME_STATES
   - **Rule**: Only data bindings here, NO function assignments

3. **Utility Functions** (~lines 797+)
   - Generic helper functions: math, collision detection, canvas helpers
   - See "Key Utility Functions" section below

4. **Factory Functions** (mid-file)
   - Entity creation functions (bullets, obstacles, powerups, dinosaurs, territories, hazards)
   - Each factory adds instance methods where appropriate
   - See "Key Factory Functions" section below

5. **Rendering Functions** (mid-file)
   - Type-specific draw functions (drawTree, drawBush, drawDinosaur, etc.)
   - All accept consistent parameters

6. **Function Bindings** (~line 3327, before React components)
   - Consolidated section for ALL function assignments
   - Bullet factory functions → WEAPONS types
   - Draw functions → DINOSAURS and OBSTACLES types
   - **Rule**: All function bindings together in ONE place
   - Clear boundary between game engine and UI layer

## Core Architectural Patterns

### 1. Unified Object Pattern

**All entity/state types are full objects with all properties in one place.**

```javascript
// Instead of string IDs and separate lookups:
// BAD: gameState = 'playing'; if (isTerminalState(gameState)) ...

// Store full object references:
gameState = GAME_CONSTANTS.GAME_STATES.PLAYING;
if (gameState.isTerminal) { ... }  // Direct property access

// Applies to ALL entity types:
bullet.weaponType = WEAPONS.REGULAR;          // Full weapon object
dino.type = DINOSAURS.VELOCIRAPTOR;           // Full dinosaur config
obstacle.obstacleType = OBSTACLES.TREE;       // Full obstacle config
powerup.powerupType = POWERUPS.HEALTH;        // Full powerup config
hazard.hazardType = HAZARDS.TAR_PIT;          // Full hazard config
```

**Benefits:**

- Single source of truth - all properties (visual, behavioral, mechanical) in one place
- NO switch statements on types
- Better IDE autocomplete
- Direct property access: `dino.type.tranqShots`, `obstacle.obstacleType.colors.trunkBase`

**Capability System:**
Use nullable objects for optional features:

```javascript
// Non-spitters: spitAttack: null
// Spitters: spitAttack: { range: 180, cooldown: 120, projectileSpeed: 4, ... }
if (dino.type.spitAttack && ...) { /* spit logic */ }
```

### 2. Instance Method Pattern

**Entities encapsulate their own behavior via instance methods.**

Factory functions add `update()` and `draw()` methods to entities:

```javascript
// Bullet factory adds instance methods:
bullet.update = function(mapWidth, mapHeight) {
  this.x += this.vx;
  this.y += this.vy;
  return !(this.x < 0 || this.x > mapWidth || this.y < 0 || this.y > mapHeight);
};

bullet.draw = function(ctx, cameraX, cameraY) {
  const screenPos = toScreenCoords(this.x, this.y, cameraX, cameraY);
  ctx.fillStyle = this.weaponType.bulletColor;
  ctx.beginPath();
  ctx.arc(screenPos.x, screenPos.y, this.radius, 0, Math.PI * 2);
  ctx.fill();
};

// Obstacle factory adds instance method:
obstacle.draw = function(ctx, cameraX, cameraY) {
  const screenX = this.x - cameraX;
  const screenY = this.y - cameraY;
  this.obstacleType.drawFunction(ctx, screenX, screenY, this.radius, this);
};

// Usage - simple, direct, OOP:
if (bullet.update(mapWidth, mapHeight)) {
  bullet.draw(ctx, cameraX, cameraY);
}
```

**Benefits:**

- Perfect encapsulation - each entity knows how to update and draw itself
- Eliminates wrapper functions and type checking at render time
- Enables batch rendering via `drawEntities()` utility
- Cleaner game loop - physics and rendering encapsulated in entities

### 3. Factory Function Pattern

**All entities created via factory functions to ensure consistency.**

```javascript
// Factories handle:
// 1. Object creation with all properties
// 2. Instance method assignment (draw, etc.)
// 3. Initialization logic (collision avoidance, positioning)

const obstacle = createObstacle(mapWidth, mapHeight, playerX, playerY, game);
const territory = createTerritoryForDino(dino, spawnPos, game);
const guard = createExitGuard(dinoType, angle, exitX, exitY, territoryIndex, difficulty);
```

**Rule:** NEVER create entities inline without factory functions - risks missing instance methods or initialization logic.

### 4. React Performance Pattern

**State batching prevents multiple re-renders per frame (60 FPS).**

```javascript
// Game loop uses stateAccumulator to batch updates:
const stateAccumulator = {};

// Update functions populate accumulator
updatePlayerSpeed(game, stateAccumulator);
updatePlayerMovement(game, canvas, stateAccumulator);
updateDinosaurs(game, difficulty, playSound, stateAccumulator);

// Batch apply all state updates at once (single re-render)
if (stateAccumulator.gameState !== undefined) setGameState(stateAccumulator.gameState);
if (stateAccumulator.playerHealth !== undefined) setPlayerHealth(stateAccumulator.playerHealth);
// ... etc
```

**Additional Performance:**

- All event handlers wrapped in `useCallback` with proper dependencies
- UI components (`GameHUD`, `TouchControls`, `PauseMenu`, `SettingsMenu`) wrapped with `React.memo`
- Handlers organized into logical sections (Game Flow, Desktop Input, Mobile Touch, UI Interaction, etc.)

### 5. Constants-First Approach

**All magic numbers extracted to `GAME_CONSTANTS` with descriptive names.**

```javascript
// Avoid: if (jumpHeight >= 25) ...
// Instead: if (jumpHeight >= GAME_CONSTANTS.PLAYER.JUMP_CLEAR_HEIGHT) ...

// All entity types defined as full objects in GAME_CONSTANTS:
// - GAME_STATES (id, isTerminal, runsGameLoop, canPause, soundEffect)
// - WEAPONS (id, icon, bulletColor, ammoConfig, soundEffect, bulletFactory)
// - DINOSAURS (id, baseSize, health, speed, points, aggressive, territorial, tranqShots, spitAttack, drawFunction)
// - OBSTACLES (id, blocksMovement, zIndex, colors, drawFunction)
// - POWERUPS (id, radius, icon, durationFrames, multiplier)
// - HAZARDS (id, damage, slowMultiplier, colors, drawFunction)
```

## Key Utility Functions

**Rendering Utilities:**

- `applyCanvasStyles(ctx, styles)` - Batch apply canvas state (fillStyle, lineWidth, shadowBlur, etc.)
- `toScreenCoords(worldX, worldY, cameraX, cameraY)` - Convert world coordinates to screen coordinates
- `drawEntities(ctx, cameraX, cameraY, ...entityArrays)` - Batch render entity arrays (argument order = z-index)

**Game Logic Utilities:**

- `randomCentered(range)` - Generate random values centered around zero (±range/2)
- `transitionToState(newState, playSound, stateAccumulator, setGameState)` - Unified state transition with auto sound playback
- `fireWeapon(weapon, player, targetX, targetY, game, playSound, stateAccumulator, setters)` - Unified weapon firing logic

**Collision & Positioning:**

- `isInBounds(x, y, mapWidth, mapHeight)` - Check if position is within map bounds
- `circleCollision(x1, y1, r1, x2, y2, r2)` - Circle-vs-circle collision
- `circleRectCollision(...)` - Circle-vs-rotated-rectangle (for electric fences)
- `distance(x1, y1, x2, y2)` - Euclidean distance
- `findValidSpawnPosition(...)` - Collision-free spawn positioning

## Key Factory Functions

**Entity Factories** (all add `draw()` instance method):

- `createObstacle()` - Trees/bushes (blocking obstacles)
- `createMushroomPatch()` - Mushroom decorations (non-blocking)
- `createGrassPatch()` - Grass decorations (non-blocking)
- `createPowerup()` - Health packs, speed boosts
- `createTranqDepot()` - Tranquilizer ammo crates
- `createAmmoDepot()` - Regular ammo crates
- `createAmmoPickup()` - Bouncing ammo drops from defeated dinosaurs
- `createTarPit()` - Movement-slowing hazards
- `createElectricFence()` - Damage/stun hazards
- `createTerritoryForDino(dino, spawnPos, game)` - Territory for territorial dinosaurs (handles collision avoidance)
- `createExitTerritory(exitX, exitY, totalGuards)` - Exit protection zone
- `createExitGuard(dinoType, angle, exitX, exitY, territoryIndex, difficulty)` - Guard dinosaurs around exit

**Projectile Factories** (all add `update()` and `draw()` instance methods):

- `createBullet(fromX, fromY, toX, toY)` - Regular bullets (physics + rendering)
- `createTranquilizer(fromX, fromY, toX, toY)` - Tranquilizer darts (physics + rendering)
- `createSpitProjectile(fromX, fromY, toX, toY, dinoType)` - Spit attacks (physics + rendering)

## Game Loop Structure

```javascript
useEffect(() => {
  const gameLoop = () => {
    const stateAccumulator = {};

    // Update phase (helper functions populate stateAccumulator)
    updatePlayerSpeed(game, stateAccumulator);
    updatePlayerMovement(game, canvas, stateAccumulator);
    updateHazards(game, difficulty, playSound, stateAccumulator);
    updateCollectibles(game, difficulty, playSound, stateAccumulator);
    updateBullets(game, playSound, stateAccumulator);
    updateSpitProjectiles(game, difficulty, playSound, stateAccumulator);
    updateDinosaurs(game, difficulty, playSound, stateAccumulator);

    // State batching (single re-render per frame)
    applyStateUpdates(stateAccumulator, setters);

    // Render phase (canvas drawing with camera offset)
    drawBackground();
    drawEntities(ctx, cameraX, cameraY, tarPits, obstacles, powerups, territories, ...);
    drawDinosaurs(); // Special rendering (transform, health bar, zap effect)
    drawPlayer();
    drawBullets();

    requestAnimationFrame(gameLoop);
  };
  gameLoop();
}, [dependencies]);
```

## Code Organization Rules

1. **Function bindings AFTER all definitions**
   - NEVER initialize function properties to `null` early
   - Only assign once when functions exist
   - Keep all bindings together in one section (~line 3327)

2. **NO switch statements on types**
   - Use `entity.type.drawFunction()` or `entity.draw()`
   - Use `gameState.isTerminal` not `switch(gameState) { case LOST: ... }`

3. **Extract repeated patterns to utilities**
   - See `randomCentered()`, `applyCanvasStyles()`, `drawEntities()`, `transitionToState()`

4. **Use factory functions for all entity creation**
   - Ensures instance methods are added consistently
   - Centralizes initialization logic

5. **Separate blocking vs decorative entities**
   - `obstacles` array: entities with `blocksMovement: true`
   - `decorations` array: entities with `blocksMovement: false`

6. **Avoid magic numbers**
   - Extract to `GAME_CONSTANTS` with descriptive names

## Adding New Features

When adding new entity types, weapons, or game mechanics:

1. **Define in GAME_CONSTANTS** as full object with all properties
2. **Create factory function** that:
   - Creates entity object
   - Adds `draw()` instance method
   - Handles initialization logic
3. **Create draw function** with consistent signature
4. **Bind draw function** in Function Bindings section (~line 3327)
5. **Update game loop** helper functions as needed
6. **Consider desktop + mobile UX** (touch controls, auto-aim, responsive sizing)

## Testing Changes

- Test level transitions (Level 1 → 2 → 3)
- Test all difficulty settings (Easy/Normal/Hard)
- Test both desktop (mouse/keyboard) and mobile (touch) controls
- Verify settings persistence (localStorage)
- Check canvas rendering at all viewport sizes (Small/Medium/Large)

## Common Pitfalls

❌ **Creating entities inline without factory functions** → Missing draw() methods, crashes at render time
❌ **Using string IDs instead of full object references** → Loses type safety, requires switch statements
❌ **Adding function bindings before function definitions** → Undefined reference errors
❌ **Forgetting state batching in game loop** → 100s of re-renders per second, performance issues
❌ **Magic numbers scattered in code** → Hard to maintain, inconsistent behavior

## Additional Resources

For gameplay mechanics, controls, dinosaur stats, and user-facing features, see [README.md](README.md).
