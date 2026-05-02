# 7. Game Logic

---

## `check_collision_circle_rec`

```rust
fn check_collision_circle_rec(center: Vector2, radius: f32, rec: Rectangle) -> bool
```

Standard AABB ↔ circle collision test using the "nearest point on rectangle" method:

```
test_x = clamp(center.x, rec.x, rec.x + rec.width)
test_y = clamp(center.y, rec.y, rec.y + rec.height)
dx = center.x − test_x
dy = center.y − test_y
return dx² + dy² ≤ radius²
```

This correctly handles all cases: circle fully inside, overlapping an edge, overlapping a corner.

---

## `reset_ball`

```rust
fn reset_ball(ball: &mut Ball, s: &DifficultySettings, go_left: bool)
```

Resets the ball to screen centre with:

- `vx = −ball_speed` if `go_left`, else `+ball_speed` — serves toward the player who just scored.
- `vy = ball_speed × 0.6` — slight downward angle on serve.
- `trail` cleared.
- `hit_count = 0` — rally counter resets.

Called whenever a point is scored (ball exits left or right edge).

---

## `reset_paddles`

```rust
fn reset_paddles(p1: &mut Paddle, p2: &mut Paddle, s: &DifficultySettings)
```

Resets both paddles to the vertical centre of the screen with zero velocity and zero swing. Also updates `paddle.height` from the current `DifficultySettings`, so a difficulty change mid-session is applied immediately on the next game start.

---

## Paddle Physics (per frame, `Playing` state)

```
for each paddle p:
    p.vy += gravity          // constant downward acceleration
    p.y  += p.vy             // Euler integration
    p.y   = clamp(p.y, 0, SCREEN_H − p.height)
    if p.y hit a boundary:
        p.vy = 0             // stop, don't accumulate velocity at wall
    p.swing *= 0.86          // 14% decay per frame ≈ half-life ~4.6 frames
    if |p.swing| < 0.01:
        p.swing = 0          // snap to zero to avoid floating-point drift
```

### Flap input
```
if KEY_TAB pressed  (or GPIO 17 rising edge):  p1.vy = impulse
if KEY_SHIFT pressed (or GPIO 27 rising edge): p2.vy = impulse
```

`impulse` is negative (upward). It **replaces** `vy` rather than adding to it, so mashing the button doesn't accumulate velocity.

---

## Ball Physics (per frame)

```
ball.push_trail()          // save position before moving
ball.x += ball.vx
ball.y += ball.vy
```

No gravity is applied to the ball — it travels in a straight line between bounces.

### Wall bounce (top / bottom)

```
if ball.y − radius ≤ 0:
    ball.vy = +|ball.vy|   // force downward
    ball.y  = radius        // push out of wall
    play snd_wall

if ball.y + radius ≥ SCREEN_H:
    ball.vy = −|ball.vy|   // force upward
    ball.y  = SCREEN_H − radius
    play snd_wall
```

Using `abs()` instead of negation prevents the ball getting stuck in a wall on repeated fast collisions.

### Paddle collision (P1 — left paddle)

```
if circle_rect_collision(ball, p1_rect) AND ball.vx < 0:
    hit_count += 1
    offset    = (ball.y − paddle_mid_y) / (paddle_height / 2)   // −1..+1
    new_speed = min(ball_speed × (1 + 0.18 × hit_count), MAX_BALL_SPEED)
    ball.vx   = +new_speed              // reflect rightward
    ball.vy   = offset × new_speed × 0.75
    ball.x    = p1.x + p1.width + radius + 1   // eject from paddle
    p1.swing  = −0.45                   // visual hit kick
    play snd_hit
```

The `ball.vx < 0` guard prevents double-counting a collision when the ball is already moving away (i.e., it only registers when the ball is approaching the paddle).

P2 (right paddle) is symmetric: `ball.vx > 0` guard, `ball.vx = −new_speed`, `p2.swing = +0.45`, ball ejected left.

---

## Scoring

```
if ball.x < 0:       score2 += 1;  reset_ball(go_left=false)  // serve to P2
if ball.x > SCREEN_W: score1 += 1; reset_ball(go_left=true)   // serve to P1
```

There is no win condition — the game runs until a player presses `ESC`.

---

## GPIO Input (Raspberry Pi)

```rust
let gpio_pins: Option<(InputPin, InputPin)> = Gpio::new().ok().and_then(|gpio| {
    let p1 = gpio.get(17).ok()?.into_input();
    let p2 = gpio.get(27).ok()?.into_input();
    Some((p1, p2))
});
```

The entire GPIO initialisation is wrapped in `Option` via `.ok().and_then(…)`. If `rppal` fails (not running on a Pi, or insufficient permissions), `gpio_pins` is `None` and the keyboard-only fallback is used transparently.

### Edge detection
```rust
let (p1_curr, p2_curr) = gpio_pins.as_ref()
    .map(|(a, b)| (a.is_high(), b.is_high()))
    .unwrap_or((false, false));

if p1_curr && !p1_prev { p1.vy = impulse; }   // rising edge only
```

`p1_prev` / `p2_prev` track the previous frame's pin state. The flap is only triggered on a **low → high transition** (button press), not while held down — matching the keyboard `is_key_pressed` (not `is_key_down`) behaviour.
