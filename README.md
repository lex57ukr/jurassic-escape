# Jurassic Escape

A thrilling single-player browser-based survival game where you navigate through a dangerous prehistoric park, avoiding dinosaurs and obstacles while trying to reach the exit.

*Inspired by the classic [Jurassic Park NES game](https://en.wikipedia.org/wiki/Jurassic_Park_(NES_video_game)).*

## 🎮 Game Overview

Jurassic Escape is a top-down action game built entirely in a single HTML file using React, Canvas API, and Tailwind CSS. Battle your way through three increasingly difficult levels filled with velociraptors, T-Rexes, and stegosauruses while managing your health and ammunition.

## 🚀 How to Play

1. Open `src/jurassic-escape-game.html` directly in any modern web browser
2. No installation, build process, or dependencies required!

### Controls

- **WASD** or **Arrow Keys**: Move your character
- **Spacebar**: Jump over obstacles
- **Mouse**: Aim your weapon
- **Click**: Shoot (consumes ammo)
- **ESC**: Pause game menu

### Objective

- Navigate through the prehistoric park
- Avoid or eliminate dinosaurs
- Collect powerups and ammo
- Reach the golden EXIT zone to complete each level
- Survive all 3 levels to win!

## 🦖 Enemy Types

- **Velociraptor**: Fast and agile hunters (2 HP, 100 points)
- **Stegosaurus**: Slow but tough (3 HP, 200 points)
- **T-Rex**: The ultimate predator (5 HP, 300 points)

## 💎 Powerups

- **❤️ Health Pack**: Restores 1 heart (red cross)
- **⚡ Speed Boost**: Temporary speed increase (lightning bolt)
- **💥 Ammo**: Dropped by defeated dinosaurs

## 🎯 Features

- **Three challenging levels** with increasing difficulty
- **Hand-drawn canvas graphics** for all game entities
- **Dynamic camera system** that follows the player
- **Sound effects** for all major game events (requires .wav files in `./assets/`)
- **Jump mechanic** to escape tight spots and stuck positions
- **Pause menu** with options to continue, restart, or exit
- **Speed boost powerup** with visual glow effect
- **Health and ammo management** for strategic gameplay

## 🔊 Sound Files

Place the following `.wav` files in the `./assets/` folder for full audio experience:

- `shoot.wav` - Firing weapon
- `hit.wav` - Bullet hits dinosaur
- `death.wav` - Dinosaur defeated
- `pickup_ammo.wav` - Collecting ammunition
- `pickup_health.wav` - Collecting health pack
- `pickup_speed.wav` - Collecting speed boost
- `player_hurt.wav` - Player takes damage
- `level_complete.wav` - Level completed
- `game_over.wav` - Player dies
- `victory.wav` - Game completed / exit to menu
- `game_start.wav` - Game starts / unpause
- `jump.wav` - Player jumps
- `pause.wav` - Pause menu opened

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
- Audio files must be manually added to `./assets/` folder

---

**Enjoy your escape from the Jurassic period!** 🦕🦖
