# 5. Audio System

The game has **no bundled audio files**. All four sound effects are synthesised at startup as raw PCM data, wrapped in a minimal WAV container, and loaded into raylib's audio system from memory. This keeps the binary fully self-contained.

---

## WAV Construction Helpers

### `write_u16_le` / `write_u32_le`

```rust
fn write_u16_le(buf: &mut Vec<u8>, v: u16) { … }
fn write_u32_le(buf: &mut Vec<u8>, v: u32) { … }
```

Simple little-endian byte writers appended to a `Vec<u8>`. Used exclusively by `samples_to_wav` to build the WAV header.

---

### `gen_sine(freq, dur_s, vol) -> Vec<i16>`

Generates a **mono sine wave** with a **linear fade-out envelope**.

```
for each sample i:
    t   = i / SAMPLE_RATE          // time in seconds
    env = 1.0 − t / dur_s          // 1.0 → 0.0 linear fade
    s   = sin(2π × freq × t) × vol × env
    sample = clamp(s, −1, 1) × i16::MAX
```

| Parameter | Type | Meaning |
|-----------|------|---------|
| `freq` | Hz | Pitch of the tone |
| `dur_s` | seconds | Duration |
| `vol` | 0.0–1.0 | Peak amplitude |

---

### `gen_sweep(f0, f1, dur_s, vol) -> Vec<i16>`

Identical to `gen_sine` except the frequency **linearly interpolates** from `f0` to `f1` over the duration — a pitch sweep.

```
freq = f0 + (f1 − f0) × (t / dur_s)
```

Used for the scoring sound (rising sweep = satisfying "point scored" feel).

---

### `samples_to_wav(samples: &[i16]) -> Vec<u8>`

Wraps a PCM sample slice in a standard **RIFF/WAV** header:

```
RIFF  <file_size>  WAVE
fmt   16  1(PCM)  1(mono)  44100  88200  2  16
data  <data_size>  <raw_i16_samples_LE>
```

The resulting `Vec<u8>` is passed directly to raylib's `new_wave_from_memory(".wav", &bytes)`.

---

## Sound Effect Table

| Variable | Generator | Frequency | Duration | Volume | Trigger |
|----------|-----------|-----------|----------|--------|---------|
| `snd_hit` | sine | 480 Hz | 0.08 s | 0.65 | Ball hits a paddle |
| `snd_wall` | sine | 220 Hz | 0.09 s | 0.45 | Ball bounces off top/bottom wall |
| `snd_score` | sweep 280→900 Hz | — | 0.35 s | 0.70 | A player scores a point |
| `snd_menu` | sine | 360 Hz | 0.05 s | 0.30 | Menu navigation / game start |

---

## Lifetime & Loading

```rust
let audio = RaylibAudio::init_audio_device()?;

let wav_hit  = samples_to_wav(&gen_sine(480.0, 0.08, 0.65));
let wave_hit = audio.new_wave_from_memory(".wav", &wav_hit)?;
let snd_hit  = audio.new_sound_from_wave(&wave_hit)?;
```

`Wave` (decoded audio data) and `Sound` (GPU/driver audio buffer) are kept alive for the duration of `main`. Raylib requires the `Wave` to outlive the `Sound` construction call, so both are stored as local variables in `main`.

Playing a sound is non-blocking:
```rust
snd_hit.play();   // fire-and-forget
```
