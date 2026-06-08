# Roadmap & status

Honest state of the project. It's **playable end-to-end** but rough and evolving.

## Working ✅

- OpenXR session sharing Dolphin's Vulkan device; per-eye swapchains; frame submit.
- **Stereoscopic 3D** on a head-locked **virtual screen** that fuses cleanly, with real depth.
- **Head tracking** drives the in-game camera (first-person, eye at the hunter's head).
- **Right-stick camera turn** + head-look (follow-base mode).
- **Quest Touch controllers** → emulated Classic Controller (sticks + buttons).
- Live-tunable comfort settings (no rebuild).
- Snap-recenter (Right Ctrl).
- **Multi-game architecture**: the VR core is game-agnostic; per-title support is a single profile
  row (camera hook + player-state offsets) in `VRGameProfiles.h`. MH Tri is the reference profile.
  See [docs/ADDING_A_GAME.md](docs/ADDING_A_GAME.md).

## Known issues / limitations ⚠️

- **No true 360° wrap-around immersion yet.** The image is a (large, comfortable) flat virtual
  screen. A real wrap-around **projection layer** doubles and won't fuse, because Dolphin's stereo
  is a clip-space shear (fake per-eye), not true dual-viewpoint rendering — the compositor can't
  reconcile the two images. Real immersion needs rendering the scene twice from two eye viewpoints
  (a large engine change). `immersion=1` exists but is experimental/broken.
- **Move-where-you-look:** movement is relative to your view (head + stick turn), not your body.
  This is an accepted design choice; a "body-relative" mode is possible but not implemented.
- **Camera jumps during scripted moments** (e.g. getting hit/knocked back, certain events): the
  game forces its own camera during these, which fights our hijack — the view snaps away and then
  back. Planned fix: only take over the camera in the normal camera mode and let the game's
  special/event cameras play through.
- **Areas can load facing backwards** (or otherwise rotated) on an area transition. Workaround:
  press **Right Ctrl** to recenter, or set `recenter_flip=1`. A proper auto-recenter on load is planned.
- **Config path is hardcoded** to a developer path — must be made relative to the exe before wide
  distribution (see Packaging).
- **Controller mapping is a first guess** — not yet fine-tuned; stick polarity / button roles may
  need adjusting per preference.
- **No controller motion** yet (sticks + buttons only).
- A frozen VR session can wedge `Dolphin.exe` (driver-level hang); a reboot clears it.
- Windows-only; tested on **Quest 3S**. Other runtimes likely work (standard OpenXR) but untested.

## Packaging TODO (before a public release) 📦

- [ ] **Make `mhtri_vr_config.txt` load relative to the exe** (currently a hardcoded absolute path).
- [ ] Ship a clean **release zip**: `Dolphin.exe` + `openxr_loader.dll` + default config + README.
- [ ] Strip temp diagnostic files written to the working dir (`mhtri_openxr_*.txt`).
- [ ] Verify a **clean-machine install** (no dev paths, fresh Dolphin user dir).
- [ ] Decide config defaults that are comfortable out-of-the-box.
- [ ] Confirm interaction-profile coverage (e.g. add `meta/touch_controller_plus` bindings if needed).

## Roadmap 🗺️

**Near-term**
- Relative config path + first public release zip.
- Controller fine-tuning (polarity, button roles, deadzone).
- Move recenter onto a Touch button.
- In-emulator toggle / simple UI instead of the text config.

**Medium-term**
- **Controller motion** (Phase B): use Touch controller pose for ranged-weapon aiming and a
  Wii-Remote-style menu pointer (the BetterVR-style gesture layer).
- Per-eye FOV / aspect cleanup for less distortion.
- Optional body-relative movement mode.

**Long-term / ambitious**
- **True wrap-around immersion** via real dual-viewpoint rendering (render each eye from its own
  position with the HMD frustum) — the big one.
- Positional 6DOF (lean/peek), world-scale calibration.

## Contributing / feedback

This is a fan project in active development. Issues and notes welcome. Source is the Dolphin fork
(`mhtri-vr` branch). GPL-2.0-or-later.
