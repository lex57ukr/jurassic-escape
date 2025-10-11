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
gameState = GAME_CONSTANTS.CORE.GAME_STATES.PLAYING;
if (gameState.isTerminal) { ... }  // Direct property access

// Applies to ALL entity types:
bullet.weaponType = WEAPONS.REGULAR;               // Full weapon object
dino.type = DINOSAURS.VELOCIRAPTOR;                // Full dinosaur config
obstacle.obstacleType = OBSTACLES.TREE;            // Full obstacle config
powerup.powerupType = POWERUPS.HEALTH;             // Full powerup config
hazard.hazardType = HAZARDS.TAR_PIT;               // Full hazard config
exit.structureType = STATIC_STRUCTURES.EXIT;       // Full structure config
```

**Benefits:**

- Single source of truth - all properties (visual, behavioral, mechanical) in one place
- NO switch statements on types
- Better IDE autocomplete
- Direct property access: `dino.type.tranqShots`, `obstacle.obstacleType.colors.trunkBase`

**Type Checking Pattern:**

Use object identity comparison (===) NOT string ID comparison:

```javascript
// ✅ GOOD: Object identity check (fast, type-safe)
if (bullet.weaponType === GAME_CONSTANTS.ENTITIES.WEAPONS.TRANQUILIZER) { ... }
if (powerup.powerupType === GAME_CONSTANTS.ENTITIES.POWERUPS.HEALTH) { ... }

// ❌ BAD: String ID comparison (brittle, requires maintaining separate ID strings)
if (bullet.weaponType.id === 'tranquilizer') { ... }
```

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

const obstacle = createObstacle(game);
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
// Instead: if (jumpHeight >= GAME_CONSTANTS.CORE.PLAYER.JUMP_CLEAR_HEIGHT) ...

// All configuration defined as full objects in GAME_CONSTANTS:
// - CORE.GAME_STATES (id, isTerminal, runsGameLoop, canPause, soundEffect)
// - ENTITIES.WEAPONS (id, icon, bulletColor, ammoConfig, soundEffect, bulletFactory)
// - ENTITIES.DINOSAURS (id, baseSize, health, speed, points, aggressive, territorial, tranqShots, spitAttack, drawFunction)
// - ENTITIES.OBSTACLES (id, blocksMovement, zIndex, colors, drawFunction)
// - ENTITIES.POWERUPS (id, radius, icon, durationFrames, multiplier)
// - ENTITIES.HAZARDS (id, damage, slowMultiplier, colors, drawFunction)
// - AI.STATES (PATROL, CHASE, FLEE, TERRITORY_RETURN - each with id and state-specific properties)
// - AI (nested: STATES, TERRITORY, EXIT, FACING_DIRECTION_THRESHOLD)
// - WORLD.SPAWN (nested: PLAYER, DEPOTS, OBSTACLES, DINOSAURS, POWERUPS, HAZARDS)
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

**Spawning Utilities:**

- `spawnHazards({ count, hazardType, createFn, targetArray, existingArrays, game })` - Consolidated hazard spawning with collision avoidance
- `spawnDecorationPatches({ count, createFn, spread, targetArray, game, getExtraArgs })` - Consolidated decoration patch spawning with player clearance
- `spawnWithCollisionCheck(game, radiusGetter, factory)` - Generic spawn helper with infinite retries for simple cases (powerups, depots)
- `retryUntilValid(positionGenerator, validator, maxAttempts, allowFallback)` - Generic retry mechanism with configurable attempts/fallback
- `checkMinimumSpacing(x, y, entityArray, minDistance)` - Reusable spacing validator for same-type entities
- `checkPlayerClearance(x, y, radius, game)` - Validate spawn doesn't block player/structures

**Collision & Positioning:**

- `isInBounds(x, y, mapWidth, mapHeight)` - Check if position is within map bounds
- `isValidPosition(x, y, radius, game, clearance)` - Check if position has no collisions with existing entities
- `circleCollision(x1, y1, r1, x2, y2, r2)` - Circle-vs-circle collision
- `circleRectCollision(...)` - Circle-vs-rotated-rectangle (for electric fences)
- `distance(x1, y1, x2, y2)` - Euclidean distance
- `findValidSpawnPosition(...)` - Collision-free spawn positioning for dinosaurs

## Key Factory Functions

**Entity Factories** (all add `draw()` instance method):

- `createObstacle(game)` - Trees/bushes/boulders/rock clusters (uses `retryUntilValid` + `checkPlayerClearance`)
- `createMushroomPatch()` - Mushroom decorations (non-blocking)
- `createGrassPatch()` - Grass decorations (non-blocking)
- `createPowerup(game)` - Health packs, speed boosts (uses `spawnWithCollisionCheck`, guaranteed to succeed)
- `createTranqDepot(game)` - Tranquilizer ammo crates (uses `spawnWithCollisionCheck`, guaranteed to succeed)
- `createAmmoDepot()` - Regular ammo crates (fixed positions)
- `createAmmoPickup()` - Bouncing ammo drops from defeated dinosaurs
- `createTarPit()` - Movement-slowing tar hazards
- `createPond()` - Water hazards with ripples and vegetation (lily pads, cattails)
- `createElectricFence(game)` - Damage/stun hazards (uses `retryUntilValid` + `checkMinimumSpacing`, guaranteed to succeed)
- `createExit(x, y)` - Level exit (static structure with locked state)
- `createTerritoryForDino(dino, spawnPos, game)` - Territory for territorial dinosaurs (uses `retryUntilValid` with fallback)
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
    drawEntities(ctx, cameraX, cameraY,
      staticStructures, tarPits, electricFences, decorations, obstacles,
      powerups, depots, territories, ...);
    drawDinosaurs(); // Special rendering (transform, health bar, zap effect)
    drawPlayer();
    drawProjectiles();

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

6. **Use specialized arrays for entity categories**
   - `staticStructures` array: static map objects (exits, gates, signs, etc.)
   - `tarPits` array: tar pit hazards
   - `ponds` array: pond hazards
   - `electricFences` array: electric fence hazards
   - `territories` array: territory markers for dinosaurs
   - Enables clean batch rendering via `drawEntities()` with proper z-index layering

7. **Avoid magic numbers**
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

## Documentation Patterns

**When to document:**

- New functionality being added to the codebase
- Changes to existing functionality (update docs to match new behavior)
- Complex algorithms or non-obvious design decisions
- Performance optimizations that aren't immediately clear
- Anything a future maintainer might question "why did they do it this way?"

**JSDoc format:**

```javascript
/**
 * Brief one-line description of what the function does
 *
 * @param {Type} paramName - Parameter description
 * @returns {Type} Return value description
 *
 * @remarks
 * Why this implementation approach was chosen.
 *
 * Design decisions explained:
 * - Rationale for algorithm choices
 * - Performance considerations
 * - Trade-offs and alternatives considered
 * - Non-obvious behavior that needs context
 */
```

**What to document:**

- **Complex collision math** - Inverse rotation transforms, clamping algorithms
- **Factory patterns** - Why instance methods, how they enable polymorphism
- **State management** - Dual-mode operation (accumulator vs direct setState)
- **AI behavior** - Hysteresis patterns, obstacle avoidance algorithms, learning systems
- **Physics systems** - Progressive damping, bounce mechanics, gravity
- **Performance optimizations** - Canvas state batching, entity rendering order
- **Non-obvious game mechanics** - Jump-over-obstacles, collection delays, territory ownership

**What NOT to document:**

- Obvious code patterns: `entity.draw = function() { ... }` doesn't need "Add instance draw method" comment
- Self-explanatory loops: `for (const tarPit of game.tarPits)` doesn't need "Loop through tar pits"
- Simple calculations: `x + vx` doesn't need "Update position"
- Function calls with clear names: `playSound(SOUNDS.PICKUP)` doesn't need "Play pickup sound"

**Documentation maintenance:**

- When changing a function's behavior, update its JSDoc immediately
- When refactoring code, remove orphaned/duplicate documentation blocks
- When extracting utilities, add JSDoc explaining the abstraction's purpose
- When removing features, remove associated documentation
- Review inline comments periodically to remove obsolete ones

**Focus on "why" not "what":**

```javascript
// ❌ Bad (obvious from code):
// Loop through obstacles
for (const obstacle of obstacles) { ... }

// ✅ Good (explains rationale):
// Check if initial position is valid (fast path optimization)
if (!checkObstacleCollision(x, y, radius, obstacles)) {
  return { x, y };
}

// ❌ Bad (restates code):
// Apply gravity to vertical velocity
this.vy += GAME_CONSTANTS.CORE.PHYSICS.AMMO_GRAVITY;

// ✅ Good (explains design decision):
// Progressive damping: bounces get weaker over time
// Later bounces have stronger damping for natural decay
const progressiveFactor = bounceCount / maxBounces;
const damping = adjustedDamping * (1 - bounceProgress * progressiveFactor);
```

## Common Pitfalls

❌ **Creating entities inline without factory functions** → Missing draw() methods, crashes at render time
❌ **Using string IDs instead of full object references** → Loses type safety, requires switch statements
❌ **Adding function bindings before function definitions** → Undefined reference errors
❌ **Forgetting state batching in game loop** → 100s of re-renders per second, performance issues
❌ **Magic numbers scattered in code** → Hard to maintain, inconsistent behavior
❌ **Leaving orphaned comments after refactoring** → Confusing documentation, maintenance burden
❌ **Over-documenting obvious code** → Noise that obscures valuable explanations

## Additional Resources

For gameplay mechanics, controls, dinosaur stats, and user-facing features, see [README.md](README.md).
