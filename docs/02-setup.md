# 2. Project Setup & Build

## Dependencies (`Cargo.toml`)

```toml
[package]
name    = "ping-pong"
version = "0.1.0"
edition = "2024"

[dependencies]
raylib = "5.5.1"   # raylib bindings — window, rendering, audio
rppal  = "0.22.1"  # Raspberry Pi GPIO
libc   = "0.2"     # low-level C types (used indirectly by rppal)
```

### Why these crates?

- **`raylib`** — wraps the C raylib library; handles the window, 2-D drawing primitives, keyboard input, and audio device management all in one.
- **`rppal`** — provides safe access to the Pi's GPIO pins so physical buttons can trigger the flap action without any extra hardware abstraction layer.
- **`libc`** — pulled in transitively; declared explicitly to satisfy build scripts.

## Nix Development Shell (`shell.nix`)

The shell provides all native libraries that `raylib-sys` (the C build) needs, plus Rust tooling:

```
nativeBuildInputs  →  cargo, rustc, pkg-config, cmake
buildInputs        →  raylib, alsa-lib, libx11, libxrandr,
                      libxinerama, libxcursor, libxi,
                      libglvnd, mesa, llvmPackages.libclang
```

Two environment variables are set:

| Variable | Purpose |
|----------|---------|
| `RUST_SRC_PATH` | Lets rust-analyzer resolve stdlib sources |
| `LIBCLANG_PATH` | Required by `bindgen` (used by `raylib-sys`) |
| `LD_LIBRARY_PATH` | Ensures the X11/GL libs are found at runtime |

## Nix Flake (`flake.nix`)

The flake exposes two dev shells per system:

| Shell | Command | Purpose |
|-------|---------|---------|
| `default` | `nix develop` | Native build for the current machine |
| `cross` | `nix develop .#cross` | Cross-compile from x86\_64 → aarch64 (Pi) |

Cross-compilation sets `CARGO_TARGET_<TRIPLE>_LINKER` and `PKG_CONFIG_ALLOW_CROSS` automatically via the `shellHook`.

## Building & Running

```bash
# Enter dev shell
nix develop

# Build (debug)
cargo build

# Build (release — recommended for Pi)
cargo build --release

# Run directly
cargo run --release
```

For Raspberry Pi, cross-compile on a faster machine then copy the binary:

```bash
nix develop .#cross
cargo build --release --target aarch64-unknown-linux-gnu
scp target/aarch64-unknown-linux-gnu/release/ping-pong pi@raspberrypi.local:~
```
