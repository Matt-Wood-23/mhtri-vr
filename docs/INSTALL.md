# Install & Setup

## 1. Prerequisites

- **Windows** + a **Vulkan-capable GPU** (most GPUs from the last ~8 years).
- A headset with an **OpenXR runtime**:
  - **Meta Quest** via **Quest Link** (cable), **Air Link** (Wi-Fi), or **Virtual Desktop**.
  - Any **SteamVR** headset (set SteamVR as the active OpenXR runtime).
- **Monster Hunter Tri**, NTSC-U, game ID **`RMHE08`** (your own dump).

## 2. Get the build

1. Download the latest [release](../../releases) zip and extract it.
2. Confirm the folder contains, side by side:
   - `Dolphin.exe`
   - `openxr_loader.dll`  ← **must be next to `Dolphin.exe`** or VR won't initialize
   - `mhtri_vr_config.txt` ← comfort/tuning settings (optional to edit)

## 3. Configure Dolphin (one time)

Launch `Dolphin.exe`, then:

1. **Graphics → General → Backend = `Vulkan`.**
   This is **required** — the VR path only runs on the Vulkan backend. (On D3D/OpenGL the game
   runs flat with no headset output.)
2. **Controllers → Wii Remotes → Wii Remote 1 = `Emulated Wii Remote`.**
   Click **Configure**, then set **Extension = `Classic Controller`**.
   The VR controller mapping injects into the Classic Controller, so this must be set.
3. *(Recommended)* Graphics → Hacks/Enhancements: leave defaults; raising internal resolution
   improves headset clarity at a performance cost.

## 4. Start VR

1. Put the headset on and start your OpenXR runtime **before** booting the game:
   - Quest Link / Air Link: enter Link from the headset.
   - Virtual Desktop: connect, then "Launch SteamVR"/OpenXR as configured.
   - SteamVR headset: start SteamVR.
2. In Dolphin, boot your MH Tri image.
3. You should see the game in the headset. Head movement drives the camera.

> **Tip:** if the headset is asleep or the runtime isn't ready when the game boots, the VR
> session may not attach. Wake the headset first, then boot.

## 5. First things to try

- **Look around** — your head drives the view.
- **Right stick** — turns your view (VR camera turn).
- **Left stick + buttons** — move and act (see [Controls](CONTROLS.md)).
- **Right Ctrl** (keyboard) — re-center the view to your hunter's facing.
- Tweak comfort live by editing `mhtri_vr_config.txt` — it's re-read about twice a second, no
  restart needed (see [Config](CONFIG.md)).

## Troubleshooting

| Symptom | Fix |
|---|---|
| No headset image, game runs on monitor only | Graphics backend isn't Vulkan; switch it. |
| VR never initializes | `openxr_loader.dll` not next to `Dolphin.exe`; or runtime not started. |
| Touch controllers do nothing | Wii Remote 1 not Emulated, or Extension isn't Classic Controller. |
| Image is doubled / can't fuse | Use the default flat-screen mode (`immersion=0`); see Config. |
| View faces the wrong way on load | Press **Right Ctrl** to recenter; or set `recenter_flip=1`. |
| Stuck/black headset, can't close Dolphin | A frozen VR session can wedge the process — **reboot** clears it. |
| Choppy / low framerate | Lower internal resolution; close background apps; try a wired Link. |
