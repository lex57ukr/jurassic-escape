# Jurassic Escape

A thrilling single-player browser-based survival game where you navigate through a dangerous prehistoric park, avoiding dinosaurs and obstacles while trying to reach the exit.

*Inspired by the classic [Jurassic Park NES game](https://en.wikipedia.org/wiki/Jurassic_Park_(NES_video_game)).*

## 🎮 Game Overview

Jurassic Escape is a top-down action game built entirely in a single HTML file using React, Canvas API, and Tailwind CSS. Battle your way through three increasingly difficult levels filled with velociraptors, T-Rexes, and stegosauruses while managing your health and ammunition.

## 🚀 How to Play

1. Open [index.html](index.html) directly in any modern web browser
2. No installation, build process, or dependencies required!
3. Works on **desktop and mobile** devices (iOS Safari, Android Chrome, etc.)

### Controls

**Desktop:**

- **WASD** or **Arrow Keys**: Move your character
- **Spacebar**: Jump over obstacles
- **Q**: Switch between regular gun (💥) and tranquilizer (💉)
- **Mouse**: Aim your weapon (or enable auto-aim 🎯 in Settings for trackpad users)
- **Click**: Shoot (consumes ammo)
- **ESC**: Pause game menu
- **⚙️ Settings Button**: Access game settings

**Mobile/Touch:**

- **🕹️ Virtual Joystick** (bottom left): Move in any direction
- **Weapon Switch Button** (top right): Toggle between gun types
- **JUMP Button** (bottom right, yellow): Jump over obstacles
- **FIRE Button** (bottom right, red): Auto-aim at nearest dinosaur
- **⏸ PAUSE Button**: Access pause menu
- **⚙️ Settings Button**: Access game settings
- Touch controls automatically appear on mobile devices

### Objective

- Navigate through the prehistoric park
- Avoid or eliminate dinosaurs
- Collect powerups and ammo
- **Defeat exit guards** protecting each level's exit (🔒 locked until guards are defeated)
- Reach the golden EXIT zone to complete each level
- Survive all 3 levels to win!

## 🦖 Dinosaur Types

**Aggressive Predators** (chase and attack player):

- **Velociraptor**: Fast and agile hunters (2 HP, 100 points, 1 tranq shot)
- **Stegosaurus**: Slow but tough (3 HP, 200 points, 2 tranq shots)
- **T-Rex**: The ultimate predator (4 HP, 300 points, 3 tranq shots)

**Territorial Predators** (defend marked zones with 💀 skull marker):

- **Dilophosaurus**: Lime green with expandable neck frill (3 HP, 150 points, 2 tranq shots)
  - Guards 200-pixel radius territory (shown with dashed red circle)
  - Aggressive only when player enters territory
  - Spit attack: ranged projectile dealing 1 damage
  - Returns to territory center when straying too far

**Peaceful Herbivores** (flee from player, no damage):

- **Parasaurolophus**: Duck-billed with head crest (2 HP, 50 points, 1 tranq shot)
- **Triceratops**: Three-horned with frill (3 HP, 75 points, 2 tranq shots)

## 💎 Collectibles & Powerups

- **❤️ Health Pack**: Restores 1 heart (red cross)
- **⚡ Speed Boost**: Temporary speed increase (lightning bolt)
- **💥 Ammo Pickup**: Single bouncing ball per defeated dinosaur
  - Restores ammo equal to dinosaur's max health (1-4 shots per pickup)
  - Shows "×N" badge indicating ammo amount on the ball
  - Bounces vertically 10 times with decreasing amplitude
  - Physics simulation with gravity and damping (no horizontal movement)
  - **Difficulty scaling**: Affects bounce height/duration (Easy: taller/slower bounces, Hard: shorter/faster bounces) - all difficulties have 10 bounces total
- **💉 Tranquilizer Depot**: Green crate with 5 tranquilizer darts (randomly placed)
- **💥 Regular Ammo Depot**: Blue crate with minimal regular ammunition (fixed locations)
  - 2 depots per level to prevent softlocks at exit guards
  - One near spawn point (visible from start)
  - One in furthest corner of map (rewards exploration)
  - Provides just enough ammo for strongest guard + 1 buffer
  - Total ammo insufficient without collecting drops - resource management is key!

## ⚠️ Environmental Hazards

- **Tar Pits**: Dark, bubbling pools that slow down movement to 50% speed
  - Affects both player and dinosaurs
  - No damage, but makes it harder to escape from predators
  - Can be identified by dark brown/black coloring with bubbling animation (3 bubbles per pit)
- **Electric Fences**: High-voltage barriers that shock on contact
  - Player: Takes 1 damage (can jump over to avoid)
  - Dinosaurs: Knocked back and stunned temporarily
  - **Smart AI**: Dinosaurs learn to avoid fences after getting zapped, but forget after ~10 seconds
  - Yellow electric wires with sparking effects and electric zap animation
  - Strategically placed throughout levels

## 🎯 Features

- **Three challenging levels** with progressively larger worlds and increasing difficulty
  - Level 1: 2400x1800 (compact starting area)
  - Level 2: 3600x2700 (50% larger exploration zone)
  - Level 3: 4800x3600 (epic 2x sized final challenge)
- **Exit guard boss fights**: Each level's exit is protected by guard dinosaurs
  - Level 1: 3 Velociraptors
  - Level 2: 3 Velociraptors + 1 Dilophosaurus
  - Level 3: 3 Velociraptors + 1 Dilophosaurus + 1 T-Rex (ultimate boss fight)
  - Exit shows 🔒 lock icon when guards are active
  - Guards patrol 300-pixel territory around exit (larger than normal territories)
  - Defeat all guards to unlock the golden exit
- **Environmental hazards**: Tar pits that slow movement and electric fences that damage/stun
- **Dual weapon system**: Regular gun and tranquilizer dart
  - **Tranquilizer**: Non-lethal option that puts dinosaurs to sleep
  - Multi-shot mechanic: larger dinosaurs need more shots
  - Awards 50% points when tranquilizing vs 100% for killing
  - Sleeping dinosaurs display Z-Z-Z animation and don't attack
- **Six dinosaur species** with distinct behaviors:
  - Aggressive predators that chase and attack
  - Territorial predators that defend marked zones with spit attacks
  - Peaceful herbivores that flee when approached
- **Mobile and desktop support** with responsive canvas and touch controls
- **Hand-drawn canvas graphics** for all game entities
- **Dynamic camera system** that follows the player
- **Floating text system** for score popups and feedback
- **Sound effects** for all major game events (requires .wav files in `./assets/`)
- **Settings menu** with customizable options - persists across sessions:
  - Sound control (mute/unmute)
  - Volume control (0-100% slider)
  - Viewport size (Small: 800x600, Medium: 1000x750, Large: 1200x900)
  - Difficulty (Easy/Normal/Hard) - affects player health, ammo, enemy speed, invincibility duration, and ammo bouncing physics
  - Auto-aim toggle (🎯) - helpful for trackpad users, enabled by default on Easy difficulty, always ON for mobile (read-only)
- **Jump mechanic** to escape tight spots and stuck positions
- **Pause menu** with options to continue, restart, or exit
- **Speed boost powerup** with visual glow effect
- **Health and ammo management** for strategic gameplay
- **Auto-aim system**:
  - Always enabled on mobile/touch devices for easier targeting
  - Optional on desktop (toggle in Settings) - helpful for trackpad users
  - Automatically targets nearest dinosaur when enabled
  - Shows 🎯 AUTO-AIM badge in HUD when active (both desktop and mobile)
  - Mobile setting is read-only (button disabled, shows "Always enabled on mobile")
- **Invincibility frames** after taking damage for fair gameplay

## 🔊 Sound Files

Place the following `.wav` files in the `./assets/` folder for full audio experience:

**Combat Sounds:**

- `death.wav` - Dinosaur defeated
- `hit.wav` - Regular bullet hits dinosaur
- `shoot.wav` - Firing regular gun
- `tranq_hit.wav` - Tranquilizer dart hits dinosaur
- `tranq_shoot.wav` - Firing tranquilizer dart

**Pickup Sounds:**

- `pickup_ammo.wav` - Collecting regular ammunition
- `pickup_health.wav` - Collecting health pack
- `pickup_speed.wav` - Collecting speed boost
- `pickup_tranq.wav` - Collecting tranquilizer depot

**Player Sounds:**

- `electric_shock.wav` - Player or dinosaur touches electric fence
- `jump.wav` - Player jumps
- `player_hurt.wav` - Player takes damage

**Game State Sounds:**

- `game_over.wav` - Player dies
- `game_start.wav` - Game starts / unpause
- `level_complete.wav` - Level completed
- `pause.wav` - Pause menu opened
- `victory.wav` - Game completed / exit to menu

## 🎨 Technical Details

- **Framework**: React 18 (via CDN)
- **Rendering**: HTML5 Canvas 2D API
- **Styling**: Tailwind CSS (via CDN)
- **Architecture**: Single-file application with embedded JavaScript
- **Game Loop**: requestAnimationFrame-based
- **File Size**: ~142KB single HTML file

## 🏗️ Architecture

The game uses a component-based architecture with React hooks for state management:

- **Constants-First Design**: All game types, states, and magic numbers extracted into `GAME_CONSTANTS` for type safety and maintainability
  - Game states, AI states, weapon types, obstacle types, powerup types defined as constants
  - Utility functions for common patterns (random generation, entity creation)
  - Single source of truth prevents typos and improves IDE autocomplete
- **Game State**: Managed via `GAME_CONSTANTS.CORE.GAME_STATES` (MENU, PLAYING, LEVEL_COMPLETE, WON, LOST)
- **Physics**: Velocity-based movement, gravity for jumping, circle collision detection
- **AI**: Dinosaurs use state machine (PATROL, CHASE, FLEE, TERRITORY_RETURN) defined in `GAME_CONSTANTS.AI.STATES`
- **Sound System**: Audio pooling via `cloneNode()` for simultaneous playback

For detailed technical documentation, see [CLAUDE.md](CLAUDE.md).

## 🎓 Learning Resource

This project demonstrates:

- Single-file application architecture
- React with Canvas integration
- Game loop implementation with `requestAnimationFrame`
- Entity-component patterns
- Collision detection algorithms
- Camera systems for viewport management
- Audio management in browser games
- State machines for game states

## 📝 License

This is a personal project created for educational and entertainment purposes.

## 🎵 Audio Requirements

When playing the game offline locally, audio files (`.wav` format) must be available in the `./assets/` folder for sound effects to work. See the **🔊 Sound Files** section above for the complete list.

## 🎤 Developer Demo Notes

This project was also presented as a short demo at our developer onsite.  
Here’s the talk outline I used (~5 minutes):

### 1. How I Picked the Idea

- Wanted something nostalgic — hadn’t built a game in decades.  
- Our day job is solving real-world problems → this was a playful *escape*.  
- Chose a 2D game because I knew the challenges & scope from past experience.  
- Key: scope something I could **finish** in 5–6 hours.  

### 2. Process & Tools

- HTML + JS → nothing beats simplicity for fast iteration.  
- Source control: every commit = a feature; easy to track or rewind.  
- AI tools:  
  - Rough prototype in Claude (sandbox wasn’t fun).  
  - Shifted to VS Code + Claude Code + ChatGPT for serious iteration.  
  - Lesson: know your tools — use chat for planning, IDE for execution.  
- Workflow mantra: **plan → execute → review → refactor → repeat**.  

### 3. Demo (the game!)

- Quick playthrough: movement, weapons, hazards, escape zone.  
- Emphasize: all in one HTML file, hosted on GitHub Pages.  

### 4. What I Learned / Would Do Differently

- AI accelerates, but structure + planning make it efficient.  
- Fast iteration is fun, but not sustainable without **automation**.  
  - (Our backend has ~30k tests; that’s how we move fast and stay sane.)  
  - Same principles apply to frontend — testing frameworks + automation matter.  
- Big takeaway: AI + scope discipline = fast results; automation makes it repeatable.  

---

**Enjoy your escape from the Jurassic period!** 🦕🦖
