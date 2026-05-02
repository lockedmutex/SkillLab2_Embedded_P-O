# 1. Project Overview

Flappy Pong is a two-player local arcade game that combines the gravity-based flapping mechanic of *Flappy Bird* with the paddle-and-ball rallying of *Pong*.

## Concept

- Both paddles are subject to **gravity** and fall continuously.
- Each player taps a button (keyboard or GPIO) to apply an upward **impulse**, keeping their paddle from falling off screen.
- A ball bounces between the two paddles. Every successful hit **increases the ball's speed**, building rally pressure until someone misses.

## Technology Stack

| Layer | Library / Tool |
|-------|---------------|
| Window & rendering | `raylib` via `raylib-rs` crate |
| Audio | `raylib` audio API (WAV loaded from memory) |
| GPIO (Pi only) | `rppal` crate |
| Build system | Cargo + Nix |
| Target platforms | x86\_64-linux, aarch64-linux (Raspberry Pi) |

## File Structure

The entire game is contained in **one file** — `src/main.rs` — intentionally keeping the codebase small and self-contained. There are no modules, no external assets (sounds are synthesised at runtime), and no configuration files.

```
.
├── src/
│   └── main.rs       ← entire game
├── Cargo.toml
├── flake.nix
└── shell.nix
```

## Game Modes

There is no AI — this is strictly a **two-player local game**. The two players can be:

- Two people at the same keyboard.
- Two people each pressing a physical button wired to a Raspberry Pi GPIO pin.
- Any mix of the above.

## Difficulty Levels

| Level | Effect |
|-------|--------|
| Easy | Slower gravity, stronger impulse, larger paddle, slower ball |
| Medium | Balanced defaults |
| Hard | Fast gravity, weak impulse, small paddle, fast ball |

Difficulty is chosen from the main menu before each game and affects all four physics parameters simultaneously.
