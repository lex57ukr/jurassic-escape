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
- **Mouse**: Aim your weapon
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
- **💥 Ammo**: Regular ammunition, dropped by defeated dinosaurs
  - Bounces continuously with decreasing amplitude until motion stops
  - Physics simulation with gravity, damping, and friction
- **💉 Tranquilizer Depot**: Green crate with 5 tranquilizer darts

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

- **Three challenging levels** with increasing difficulty and hazards
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
  - Difficulty (Easy/Normal/Hard) - affects player health, ammo, enemy speed, and invincibility duration
- **Jump mechanic** to escape tight spots and stuck positions
- **Pause menu** with options to continue, restart, or exit
- **Speed boost powerup** with visual glow effect
- **Health and ammo management** for strategic gameplay
- **Auto-aim** for touch controls (mobile)
- **Invincibility frames** after taking damage for fair gameplay

## 🔊 Sound Files

Place the following `.wav` files in the `./assets/` folder for full audio experience:

- `shoot.wav` - Firing regular gun
- `tranq_shoot.wav` - Firing tranquilizer
- `hit.wav` - Bullet hits dinosaur
- `tranq_hit.wav` - Tranquilizer hits dinosaur
- `death.wav` - Dinosaur defeated
- `pickup_ammo.wav` - Collecting ammunition
- `pickup_tranq.wav` - Collecting tranquilizer depot
- `pickup_health.wav` - Collecting health pack
- `pickup_speed.wav` - Collecting speed boost
- `player_hurt.wav` - Player takes damage
- `level_complete.wav` - Level completed
- `game_over.wav` - Player dies
- `victory.wav` - Game completed / exit to menu
- `game_start.wav` - Game starts / unpause
- `jump.wav` - Player jumps
- `pause.wav` - Pause menu opened
- `electric_shock.wav` - Player or dinosaur touches electric fence

## 🎨 Technical Details

- **Framework**: React 18 (via CDN)
- **Rendering**: HTML5 Canvas 2D API
- **Styling**: Tailwind CSS (via CDN)
- **Architecture**: Single-file application with embedded JavaScript
- **Game Loop**: requestAnimationFrame-based
- **File Size**: ~40KB single HTML file

## 🏗️ Architecture

The game uses a component-based architecture with React hooks for state management:

- **Game State**: `menu`, `playing`, `levelComplete`, `won`, `lost`
- **Physics**: Velocity-based movement, gravity for jumping, circle collision detection
- **AI**: Dinosaurs patrol randomly and chase when player is in range
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

## 🐛 Known Issues

- Player may occasionally spawn between obstacles at level start (use jump to escape)
- Audio files must be available in the `./assets/` folder

---

**Enjoy your escape from the Jurassic period!** 🦕🦖
