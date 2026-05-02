# 3. Constants

All top-level constants are declared with `const` at module scope so they are inlined by the compiler and visible across every function without passing them as arguments.

```rust
const SCREEN_W:           f32   = 1280.0;
const SCREEN_H:           f32   = 720.0;
const TRAIL_LEN:          usize = 12;
const SPEED_BOOST_PER_HIT: f32  = 0.18;
const MAX_BALL_SPEED:     f32   = 22.0;
const SAMPLE_RATE:        u32   = 44100;
```

## Explanations

### `SCREEN_W` / `SCREEN_H`
Resolution of the window: **1280 × 720** (720p). The game is then toggled into fullscreen immediately, but all layout math still uses these pixel values as the logical coordinate space.

### `TRAIL_LEN`
The ball leaves a motion trail of up to **12** ghost positions. Each frame the current ball position is prepended to a `Vec`; once the vec exceeds `TRAIL_LEN` the oldest entry is removed. Older trail entries are drawn smaller and more transparent.

### `SPEED_BOOST_PER_HIT`
Each time the ball is hit by a paddle its speed is multiplied by:

```
new_speed = base_speed × (1.0 + 0.18 × hit_count)
```

So after 1 hit speed is +18 %, after 5 hits +90 %, and so on — until capped.

### `MAX_BALL_SPEED`
Hard cap on ball speed regardless of rally length. Without this the ball would eventually move faster than a paddle width per frame and tunnel through collisions.

### `SAMPLE_RATE`
Standard CD-quality audio sample rate (**44 100 Hz**), used when synthesising PCM sine/sweep waveforms for the in-game sound effects.
