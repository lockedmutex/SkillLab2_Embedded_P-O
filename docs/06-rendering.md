# 6. Rendering

All drawing happens inside `rl.begin_drawing()` / `end_drawing()` at the bottom of the main loop. The background is cleared to solid black every frame.

---

## Helper: `rot(px, py, cx, cy, a) -> (f32, f32)`

Rotates point `(px, py)` around pivot `(cx, cy)` by angle `a` radians.

```rust
let (s, c) = a.sin_cos();
let dx = px - cx;
let dy = py - cy;
(cx + dx*c - dy*s, cy + dx*s + dy*c)
```

Used pervasively by the bat drawing code to orient every sub-part of the paddle around a shared centre.

---

## Helper: `draw_rotated_rect`

```rust
fn draw_rotated_rect(
    d: &mut RaylibDrawHandle,
    rx, ry, rw, rh,   // rectangle origin and size
    ox, oy,           // rotation pivot
    angle: f32,
    color: Color,
)
```

Raylib has no native rotated-rectangle primitive. This function decomposes the rect into two triangles, rotates all four corners with `rot()`, then calls `draw_triangle` twice.

```
TL ── TR          TL ── TR
│  ╲              ╲  │
│   ╲     →        ╲ │
BL   BR           BL ─ BR

Triangle 1: TL, BL, TR
Triangle 2: TR, BL, BR
```

---

## `draw_bat(d, paddle, color)`

Draws a stylised **table-tennis bat** centred on the paddle's bounding box. All parts are positioned in local space then rotated together around `(cx, cy)` by `base_tilt + swing`:

| `base_tilt` | Value |
|-------------|-------|
| Left paddle (P1) | +0.22 rad (~12.6°) |
| Right paddle (P2) | −0.22 rad |

### Parts drawn (back to front)

1. **Head outline** — dark circle slightly larger than the head, creating a border.
2. **Head** — filled circle in the player's colour (`color` parameter).
3. **Rubber face** — two `draw_circle_sector` calls (half-circles) on the leading side:
   - Outer sector: dark red, `head_r − 2`
   - Inner sector: lighter red, `head_r − 6`
4. **Glints** — two small white semi-transparent circles simulating specular highlight.
5. **Handle** — rotated rectangle, brown (`COL_HANDLE`).
6. **Grip wraps** — up to 3 thin darker strips drawn over the handle.
7. **Knob** — small filled circle at the butt of the handle.

---

## `draw_trail(d, ball)`

Iterates `ball.trail` from newest (index 0) to oldest (index N):

```rust
alpha = 255 × (1 − i/TRAIL_LEN) × 0.5   // fades out
r     = ball.radius × (1 − i/TRAIL_LEN × 0.6)  // shrinks
color = (255, 200, 60, alpha)            // warm yellow-orange
```

The result is a comet-like tail that fades and shrinks toward the rear.

---

## Ball Rendering

The ball is drawn as two concentric circles:

1. **Glow ring** — radius `ball.radius + 4`, colour interpolated from green (slow) to orange-red (fast) at 55 alpha:
   ```
   t  = speed / MAX_BALL_SPEED   // 0..1
   R  = 255 × t
   G  = 200 × (1 − t)
   ```
2. **Ball** — solid white circle, `ball.radius`.

---

## `draw_center_dashes()`

Draws a vertical dashed line down the screen centre (x = 640), alternating 18 px dark-grey rectangles with 12 px gaps, mimicking classic Pong's court divider.

---

## Menu Rendering

### Floating bubbles
Six large semi-transparent circles drift with independent sine/cosine offsets driven by `menu_tick` (incremented 0.04 per frame):

```rust
ox = sin(tick×0.7 + i×1.1) × 120 + SCREEN_W/2
oy = cos(tick×0.5 + i×0.9) ×  80 + SCREEN_H/2
```

### Difficulty buttons
Three fixed-position rectangles at x = 358, 553, 748. The selected one is filled blue with a white outline; others are dark grey. The label is horizontally centred inside each button with a simple `bx + 87 − label.len() × 7` approximation (works well for the monospaced raylib font).

### Pulsing "PRESS ENTER" text
Alpha oscillates between 185 and 255 using:
```rust
alpha = 185 + (sin(tick × 3.0) × 0.5 + 0.5) × 70
```

---

## Score & HUD

| Element | Position | Size | Colour |
|---------|----------|------|--------|
| P1 score | `SCREEN_W/4` | 52 px | Gray |
| P2 score | `SCREEN_W×0.75` | 52 px | Gray |
| Rally label (`RALLY xN`) | Centre-bottom | 18 px | Orange, shown when `hit_count ≥ 4` |
| ESC hint | Bottom-left | 14 px | Dark, 180 alpha |
