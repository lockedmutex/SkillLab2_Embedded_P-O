# Flappy Pong — Rust Codebase Documentation

> A single-file Rust game built with [raylib](https://github.com/deltaphc/raylib-rs) and optionally [rppal](https://github.com/golemparts/rppal) for Raspberry Pi GPIO input.

---

## Table of Contents

1. [Project Overview](./01-overview.md)
2. [Project Setup & Build](./02-setup.md)
3. [Constants](./03-constants.md)
4. [Data Structures](./04-data-structures.md)
5. [Audio System](./05-audio.md)
6. [Rendering](./06-rendering.md)
7. [Game Logic](./07-game-logic.md)
8. [Main Function & Game Loop](./08-main-loop.md)

---

## Quick Reference

| File | Purpose |
|------|---------|
| `main.rs` | Entire game — single-file implementation |
| `Cargo.toml` | Dependencies: `raylib`, `rppal`, `libc` |
| `shell.nix` | Nix dev shell with all native libraries |
| `flake.nix` | Nix flake with cross-compilation support |

## Controls

| Key | Action |
|-----|--------|
| `TAB` | P1 (blue bat) — flap |
| `LEFT SHIFT` | P2 (green bat) — flap |
| `← / A` | Menu — select easier difficulty |
| `→ / D` | Menu — select harder difficulty |
| `ENTER / SPACE` | Menu — start game |
| `ESC` | In-game — return to menu; Menu — quit |

## GPIO (Raspberry Pi)

| GPIO Pin | Player |
|----------|--------|
| 17 | P1 |
| 27 | P2 |
