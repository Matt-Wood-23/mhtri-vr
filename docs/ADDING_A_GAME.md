# Adding a game (VR profiles)

MH Tri VR is built on a **generic VR core** plus a small **per-game profile**. Most of the work —
OpenXR session, stereo rendering, head tracking, the compositor quad, Touch → Classic Controller
input — is game-agnostic and already done. To bring a *new* GameCube/Wii title into VR you don't
touch any of that. You add **one data row** describing where that game's camera and player state
live, then rebuild.

This document explains the split, the data you need to find, and how to find it.

> **Reality check.** The generic core gives you *stereo-on-a-virtual-screen + head tracking* on
> basically any Dolphin Vulkan title for free. The part that needs per-game reversing is **true
> head-tracked first-person** — the camera-commit hook and the player-state layout below. Games with
> a single, clean camera-commit function are easy; games that drive the camera from many places, or
> fight an external override during scripted events, are harder. Budget a reversing session per game.

---

## The two layers

| Layer | Where | Game-specific? |
|---|---|---|
| OpenXR session, per-eye swapchains, stereo quad, head pose, controller input | `Source/Core/VideoCommon/VROpenXR.*` | **No** |
| Camera injection driver (head-look, first-person rig, recenter, follow-base) | `Source/Core/Core/HW/MHTriVR.*` | No (reads the profile) |
| **Per-game addresses & offsets** | **`Source/Core/Core/HW/VRGameProfiles.h`** | **Yes — this is all you edit** |

`MHTriVR.cpp` is the generic driver. It used to hard-code MH Tri's addresses; now it reads them from
the **active profile**, chosen by Game ID at boot. Despite the historical `MHTriVR` name, it will
drive any title that has a profile.

---

## The profile struct

From [`VRGameProfiles.h`](https://github.com/Matt-Wood-23/dolphin/blob/mhtri-vr/Source/Core/Core/HW/VRGameProfiles.h):

```cpp
struct GameProfile
{
  std::string_view game_id;  // 6-char Game ID from the disc header, e.g. "RMHE08"
  std::string_view name;     // human-readable, for logs only

  // Camera commit hook
  u32 hook_addr;       // engine fn that pushes the final eye + look-at into the camera each frame
  u8  eye_ptr_reg;     // GPR index holding the eye    vec3 pointer at hook entry
  u8  target_ptr_reg;  // GPR index holding the target vec3 pointer at hook entry

  // Player state (first-person rig)
  u32 player_work_ptr; // fixed RAM word that POINTS at the live player struct
  u32 pos_off;         // offset of the {X,Y(up),Z} f32 position vec3 within that struct
  u32 facing_off;      // offset of the s16 BAMS facing yaw (deg = v * 360 / 65536)
};
```

The reference row, for MH Tri:

```cpp
{
    "RMHE08", "Monster Hunter Tri (NTSC-U)",
    0x802be0f4, 4, 5,        // cam_commit_to_g3d; r4 = eye, r5 = target
    0x806bbc74, 0x48, 0xd8,  // player_work ptr; pos vec3; facing BAMS
},
```

You need to find six numbers for your game: the **hook address**, the **two register indices**, the
**player_work pointer address**, the **position offset**, and the **facing offset**.

---

## Finding the six values

You'll want a disassembler (Ghidra) and a live debugger attached to Dolphin. This project uses a
[Ghidra MCP](https://github.com/Matt-Wood-23/dolphin-re-mcp) and a
[Dolphin RE MCP](https://github.com/Matt-Wood-23/dolphin-re-mcp), but Dolphin's **built-in debugger**
(Config → enable Debugging UI: breakpoints, memory view, register view) does everything you need.

### 1. The camera-commit hook (`hook_addr`, `eye_ptr_reg`, `target_ptr_reg`)

The "commit" is the function that takes a final eye position and look-at target and pushes them into
the GPU / scene-graph camera each frame. Hooking the *commit* (rather than the camera-update logic)
means the game's own camera code runs untouched — you just overwrite what it's about to submit.

How to find it:

- Most Wii titles use **nw4r g3d** (`Camera::SetPosture` / `SetCameraPosition` / `LookAt`-style
  calls). Search the symbol map / strings for `Camera`, `Posture`, `LookAt`, `g3d`. If you have a
  `.map`/symbol file, that's the fastest route.
- Otherwise: set a **write watchpoint** on the view matrix or the camera's eye vector in memory and
  see what function writes it every frame. The last writer before the draw is your commit.
- Confirm by **breaking** at the candidate function and checking it runs **once per frame** during
  normal gameplay.

Once you have the function's entry address, that's `hook_addr`. Now find which **registers** hold the
eye and target pointers at entry:

- Break at the entry, then inspect the GPRs. The eye and target are usually passed as `vec3*`
  arguments → `r3`/`r4`/`r5`/`r6` on PowerPC.
- Dump memory at each candidate pointer: a camera eye/target is **three contiguous big-endian
  floats** with sane world-scale values that **change smoothly** as you move the camera. The eye and
  target differ by the view direction.
- Record the register **indices** (e.g. `r4` → `4`, `r5` → `5`). Those are `eye_ptr_reg` /
  `target_ptr_reg`.

> If the game passes eye/target by value on the stack instead of by pointer, this struct's
> pointer-register model doesn't fit as-is — note it in your PR; we'll extend the profile format.

### 2. The player struct (`player_work_ptr`, `pos_off`, `facing_off`)

This is for the **first-person rig** (eye placed at the player's head, aimed by body facing + HMD).
You can ship a profile *without* it (head-look on the follow cam still works), but first-person is the
part that actually feels like VR.

- **`player_work_ptr`** is a *fixed* RAM address whose **contents** are a pointer to the live player
  struct. Find it by locating the player object in memory (e.g. search for your known world position
  as three floats while standing still, then move and re-search), then find a stable global that
  holds its address. On MH Tri this is `0x806BBC74` (the camera caches it there). It may point into
  **MEM1 (`0x80xxxxxx`)** or **MEM2 (`0x90xxxxxx`)** — the driver accepts both.
- **`pos_off`** — from the player struct base, the offset of the `{X, Y(up), Z}` float vec3. Find it
  the same way you found the object: it's the three floats that track your movement.
- **`facing_off`** — the offset of the **signed 16-bit BAMS** yaw (degrees = `value * 360 / 65536`).
  Turn in place and watch a `s16` near the position that sweeps `0..0x7FFF..0x8000..0xFFFF`. If your
  game stores facing as a **float radian** or a **matrix** instead, note it in your PR — we'll add a
  facing-format field.

---

## Add the row and rebuild

1. Add your `GameProfile` to `kProfiles[]` in
   [`VRGameProfiles.h`](https://github.com/Matt-Wood-23/dolphin/blob/mhtri-vr/Source/Core/Core/HW/VRGameProfiles.h):

   ```cpp
   inline constexpr GameProfile kProfiles[] = {
       { "RMHE08", "Monster Hunter Tri (NTSC-U)", 0x802be0f4, 4, 5, 0x806bbc74, 0x48, 0xd8 },
       { "RXXE01", "Your Game (NTSC-U)",          0x80abcdef, 4, 5, 0x80123456, 0x10, 0x40 },
   };
   ```

   That's the **only** code change. No edits to `MHTriVR.cpp` or anywhere else.

2. Rebuild exactly as in **[BUILDING.md](BUILDING.md)** (the VR build, `-p:MHTriVROpenXR=true`).
   `VRGameProfiles.h` is header-only and already listed in both build systems
   (`Source/Core/DolphinLib.props` for the VS solution, `Source/Core/Core/CMakeLists.txt` for the
   CMake/Ninja tree), so there's nothing else to wire up.

3. Boot your game. The log (`Core` category) prints
   `MHTriVR: installed camera VR hook at 0x… (Your Game …)` when the profile is matched. Any title
   **without** a profile is a silent no-op — stock Dolphin behavior.

---

## Tuning & comfort

Everything in [`mhtri_vr_config.txt`](CONFIG.md) is game-agnostic and applies to your title too:
`firstperson`, `eye_height`, `eye_forward`, `look_dist`, the sign flips, `fp_follow_base`, and the
stereo/quad comfort knobs. Expect to re-tune **eye_height / eye_forward / look_dist** per game
(different world scale) and the **sign knobs** (`fp_yaw_sign`, `fp_head_yaw_sign`,
`fp_head_pitch_sign`, `recenter_flip`) until movement and head-look go the right way. These are all
live-reloaded — no rebuild to tune.

---

## Known per-game pitfalls

- **Scripted/event cameras.** Many games force their own camera during cutscenes, hits, or
  transitions, which fights the override (view snaps away and back). MH Tri has this too — see the
  ROADMAP. A clean fix is to only take over when the game is in its *normal* camera mode; that gate is
  itself game-specific.
- **Stereo is a clip-space shear**, not true dual-viewpoint rendering, so a real wrap-around
  projection layer won't fuse on any title (the image is a comfortable flat virtual screen). This is a
  core limitation, not a per-game one — see the ROADMAP.
- **Facing/eye formats vary.** Some engines use float radians, matrices, or quaternions, or pass the
  camera by value. The current profile assumes pointer args + BAMS facing (the nw4r-on-Wii common
  case). If yours differs, open an issue/PR — the format is easy to extend with an enum field.

---

## Submitting a profile

PRs welcome. Include: the Game ID and region, how you found each value (function name/address,
register evidence, the memory offsets), and a note on anything that needed a non-default config or
that doesn't work yet (event cameras, etc.). A short clip helps.
