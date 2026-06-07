# Config reference — `mhtri_vr_config.txt`

A plain `key=value` text file read **live** (~twice a second) — edit it while the game runs and
changes apply in about a second, **no restart**. It sits next to `Dolphin.exe`.

> **Note (current limitation):** the file path is presently hardcoded for the developer's machine
> and is being changed to "next to the exe" — see the [Roadmap](../ROADMAP.md). Until then, custom
> builds may read it from a fixed path.

All values have sensible built-in defaults; the file only *overrides* them.

## Presentation

| Key | Default | Meaning |
|---|---|---|
| `immersion` | `0` | `0` = flat **virtual screen** (recommended, fuses cleanly). `1` = experimental wrap-around projection layer — currently **doubles/can't fuse**, see Roadmap. |
| `quad_width` | `2.4` | Virtual-screen width in metres. Larger = bigger/more enveloping. |
| `quad_dist` | `1.6` | Virtual-screen distance in metres. Smaller = closer/bigger in view. |
| `fov_scale` | `0.6` | Game render FOV scale. **< 1 widens** the FOV (reduces the "zoomed-in" look); `1.0` = native. |
| `depth` | `0.02` | Stereo separation (3D pop). Higher = more depth; too high gets uncomfortable. |
| `convergence` | `20` | Stereo convergence plane. |

## First-person camera

| Key | Default | Meaning |
|---|---|---|
| `firstperson` | `1` | `1` = first-person (eye at the hunter's head). `0` = head-look on the normal 3rd-person camera. |
| `fp_follow_base` | `1` | `1` = **view follows the right-stick camera** + head (recommended). `0` = world-locked view + manual recenter. |
| `eye_height` | `150` | World units to raise the eye from the player position (find your comfortable head height). |
| `eye_forward` | `60` | Push the eye forward out of the hunter model (so you're not inside the head). |
| `look_dist` | `300` | Eye→target distance (direction only; rarely needs changing). |
| `recenter_flip` | `0` | `1` flips the recenter / follow-base heading 180° (if "forward" comes out backwards). |
| `fp_yaw_offset` | `0` | Constant yaw trim (radians) added to the view heading. |

## Advanced / sense knobs (usually leave alone)

| Key | Default | Meaning |
|---|---|---|
| `fp_yaw_sign` | `0` | Body-facing contribution to view yaw. Keep `0` — `±1` closes an unstable feedback loop (spin). |
| `fp_head_yaw_sign` | `-1` | Flips the head-look yaw sense (matches the HMD to the game). |
| `fp_head_pitch_sign` | `1` | Flips the head-look pitch (up/down) sense. |

## Example (the shipping defaults)

```text
fov_scale=0.6
depth=0.02
convergence=20
immersion=0
quad_width=2.4
quad_dist=1.6
firstperson=1
eye_height=150
eye_forward=60
look_dist=300
fp_yaw_offset=0
fp_yaw_sign=0
fp_head_yaw_sign=-1
fp_head_pitch_sign=1
recenter_flip=0
fp_follow_base=1
```

## Tuning tips

- **Too "zoomed in" / world feels small:** lower `fov_scale` (e.g. `0.5`, `0.45`).
- **Want a bigger screen:** raise `quad_width` and/or lower `quad_dist`.
- **Eye feels too high/low or inside the model:** adjust `eye_height` then `eye_forward`.
- **Forward is backwards after a load:** press Right Ctrl, or set `recenter_flip=1`.
- **Uncomfortable 3D:** lower `depth` toward `0.01`; `0` = mono (no depth, max comfort).
