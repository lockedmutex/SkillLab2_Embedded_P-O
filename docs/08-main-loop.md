# 8. Main Function & Game Loop

---

## Initialisation Sequence

```rust
fn main() {
    // 1. Window
    let (mut rl, thread) = raylib::init()
        .size(SCREEN_W as i32, SCREEN_H as i32)
        .title("Flappy Pong")
        .build();
    rl.set_target_fps(60);
    rl.set_exit_key(None);     // disable raylib's default ESC-to-quit
    rl.toggle_fullscreen();

    // 2. Audio device
    let audio = RaylibAudio::init_audio_device()?;

    // 3. Sound synthesis & loading (see Audio doc)
    …

    // 4. Initial game state
    let mut state         = GameState::Menu;
    let mut selected_diff = Difficulty::Medium;
    let mut settings      = DifficultySettings::from(selected_diff);

    // 5. Paddles
    let mut p1 = Paddle { x: 30.0,   … is_left: true  };
    let mut p2 = Paddle { x: 1230.0, … is_left: false };

    // 6. Ball
    let mut ball = Ball { x: SCREEN_W/2, y: SCREEN_H/2, … };

    // 7. Scores and animation state
    let mut score1: i32 = 0;
    let mut score2: i32 = 0;
    let mut menu_tick: f32 = 0.0;

    // 8. GPIO (optional)
    let gpio_pins: Option<(InputPin, InputPin)> = …;
    let mut p1_prev = false;
    let mut p2_prev = false;
```

### Notes

- `set_exit_key(None)` is important: without it raylib auto-exits on `ESC`, which would prevent using `ESC` for menu navigation.
- `toggle_fullscreen()` is called once after window creation. The window was created at 1280×720 so the logical coordinate space matches regardless of actual display resolution.

---

## Main Loop Structure

```
while !rl.window_should_close() {

    ┌─ UPDATE ─────────────────────────────────────────────┐
    │  match state {                                        │
    │    Menu    => { handle menu input }                   │
    │    Playing => { physics + collision + scoring }       │
    │  }                                                    │
    └───────────────────────────────────────────────────────┘

    ┌─ DRAW ────────────────────────────────────────────────┐
    │  let mut d = rl.begin_drawing(&thread);               │
    │  d.clear_background(BLACK);                           │
    │  match state {                                        │
    │    Menu    => { draw menu UI }                        │
    │    Playing => { draw game }                           │
    │  }                                                    │
    └───────────────────────────────────────────────────────┘
}
```

Update and draw are **not** separated into distinct functions — everything lives inline in `main`. This is a deliberate simplicity choice for a single-file game.

---

## State Transitions

```
        ┌──────────────────────────────────────────┐
        │                                          │
        ▼                                          │
    [ Menu ] ──ENTER/SPACE──► [ Playing ] ──ESC───┘
        │
       ESC
        │
       exit
```

| From | Input | To | Side effects |
|------|-------|----|-------------|
| Menu | ENTER or SPACE | Playing | Reset scores, paddles, ball; apply selected settings |
| Playing | ESC | Menu | None (scores preserved visually but reset on next start) |
| Menu | ESC | exit | `break` out of main loop |

---

## Frame Rate

`set_target_fps(60)` instructs raylib to sleep after each frame to target 60 FPS. All physics use **fixed per-frame deltas** (no `get_frame_time()` delta-time scaling), so the game speed is tied to frame rate. On slower hardware, increasing `TARGET_FPS` or using `get_frame_time()` to scale physics values would be needed.

---

## Shutdown

The loop exits when either:
- `rl.window_should_close()` returns `true` (window close button or OS signal).
- `ESC` is pressed on the menu (`break`).

Rust's RAII automatically cleans up all raylib handles (`RaylibHandle`, `RaylibAudio`, `Sound`, `Wave`) as they drop at end of scope. No explicit cleanup calls are needed.
