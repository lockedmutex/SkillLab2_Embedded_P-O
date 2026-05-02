# 4. Data Structures

## `Difficulty` (enum)

```rust
#[derive(Clone, Copy, PartialEq)]
enum Difficulty {
    Easy,
    Medium,
    Hard,
}
```

Represents the three selectable difficulty levels. Derives `Clone`, `Copy` (cheap value semantics) and `PartialEq` (used for equality checks in the menu rendering loop).

---

## `DifficultySettings` (struct)

```rust
struct DifficultySettings {
    gravity:       f32,
    impulse:       f32,
    ball_speed:    f32,
    paddle_height: f32,
}
```

Holds all four physics parameters that vary per difficulty. Constructed via `DifficultySettings::from(d: Difficulty)`:

| Field | Easy | Medium | Hard | Effect |
|-------|------|--------|------|--------|
| `gravity` | 0.35 | 0.60 | 0.90 | Added to `vy` every frame — pulls paddles down |
| `impulse` | −9.0 | −12.0 | −15.0 | Applied to `vy` on flap — negative = upward |
| `ball_speed` | 4.0 | 5.5 | 8.0 | Initial ball speed and base for speed scaling |
| `paddle_height` | 120.0 | 90.0 | 60.0 | Pixel height of each paddle |

> **Sign convention:** positive `vy` moves downward (screen-space Y increases downward). Gravity is therefore positive; impulse is negative.

---

## `GameState` (enum)

```rust
#[derive(PartialEq)]
enum GameState {
    Menu,
    Playing,
}
```

Drives the top-level branch in both the update and draw sections of the game loop. Only two states exist; there is no pause, game-over, or credits screen — pressing `ESC` during play returns directly to `Menu`.

---

## `Paddle` (struct)

```rust
struct Paddle {
    x:       f32,   // left edge X (fixed per player)
    y:       f32,   // top edge Y (changes with physics)
    vy:      f32,   // vertical velocity
    width:   f32,   // always 20px
    height:  f32,   // varies with difficulty
    is_left: bool,  // true = P1 (left side), false = P2 (right side)
    swing:   f32,   // visual tilt offset decaying each frame
}
```

### Physics
Each frame (while `Playing`):
```
vy  += gravity          // apply gravity
y   += vy               // integrate position
y    = clamp(y, 0, SCREEN_H - height)  // floor/ceiling
vy   = 0  if y hit a boundary          // stop at walls
swing *= 0.86           // exponential decay toward 0
```

### `swing`
When a paddle hits the ball, `swing` is set to ±0.45 radians. It decays by 14 % each frame, giving the bat a brief animated "hit" tilt. Values below 0.01 are zeroed to stop the decay loop.

---

## `Ball` (struct)

```rust
struct Ball {
    x:         f32,
    y:         f32,
    vx:        f32,
    vy:        f32,
    radius:    f32,          // always 10px
    trail:     Vec<(f32, f32)>,
    hit_count: u32,
}
```

### Methods

#### `fn speed(&self) -> f32`
Returns the scalar speed: `√(vx² + vy²)`. Used to tint the ball glow from white → orange → red as rally speed increases.

#### `fn push_trail(&mut self)`
Inserts the current `(x, y)` at index 0 of `trail`, then pops the last element if `trail.len() > TRAIL_LEN`. This keeps the most recent position at index 0 and the oldest at the end, which the draw function uses to fade opacity from front to back.

### Collision response
When the ball hits a paddle:
```
hit_count += 1
offset     = (ball.y − paddle_centre_y) / (paddle_height / 2)   // −1..+1
new_speed  = min(base_speed × (1 + 0.18 × hit_count), MAX_BALL_SPEED)
vx         = ±new_speed            // direction reverses
vy         = offset × new_speed × 0.75
```
The `offset` factor deflects the ball toward the edge it struck, giving players angle control.
