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
- **React state**: `gameState`, `score`, `playerHealth`, `playerAmmo`, `currentLevel`, `speedBoostActive`
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
- Mouse position tracked for aiming
- Click to shoot (consumes ammo)
- Camera follows player with map boundary constraints

**Entities**
- **Player**: Circle with health, ammo, speed (base + boost modifier)
- **Dinosaurs** (3 types): Raptor, T-Rex, Stegosaurus
  - Each has unique stats: health, speed, size, points
  - AI states: `patrol` (random wandering) or `chase` (aggro within range)
  - Hand-drawn canvas art with facing direction (horizontal flip for left movement)
- **Obstacles**: Trees and bushes with circular collision
- **Powerups**: Health (red cross) and Speed (lightning bolt)
- **Ammo pickups**: Dropped when dinosaurs die, with physics (bouncing animation)

**Collision Detection**
All collision uses circle-vs-circle distance checks:
- Player vs obstacles (movement blocking)
- Player vs dinosaurs (damage)
- Bullets vs dinosaurs (damage)
- Player vs powerups/ammo (collection)

**Level Progression**
Levels defined in `levelConfigs` (lines 45-75):
- Each level specifies dinosaur types/counts, obstacle count, powerup count
- Win condition: Reach exit zone (golden EXIT rectangle at map coordinates 1800, 1400)
- Player stats reset between levels, score persists

### Rendering
- 800x600 canvas viewport into 2400x1800 game world
- Camera offset (`cameraX`, `cameraY`) applied to all draw calls
- Layered drawing order: background → exit → obstacles → powerups → dinosaurs → player → bullets → ammo pickups
- Health bars drawn above dinosaurs
- UI overlay in React (hearts, ammo count, score, speed boost indicator)

## Key Implementation Details

- **No collision between game entities** except player (dinosaurs don't collide with obstacles or each other)
- **Speed boost**: Multiplies player speed by 1.8x for 300 frames (5 seconds at 60fps)
- **Ammo economy**: Start with 10, dinosaurs drop ammo equal to their max health when killed
- **Dinosaur art**: Procedurally drawn on canvas with context transformations (scale, rotate, flip)
- **Input handling**: Keyboard state stored in `game.keys` object, mouse position relative to canvas
