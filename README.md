# MH Tri VR

**Play Monster Hunter Tri (Wii) in VR** — head-tracked stereo first-person, with full Quest
Touch controller support — via a custom [Dolphin](https://dolphin-emu.org/) build.

> ⚠️ **Work in progress — actively being fine-tuned.** It plays end-to-end today (render + camera +
> controllers), but it is **not finished**: comfort settings, control mapping, and immersion are still
> being dialed in, and you should expect rough edges and frequent changes. Treat current builds as
> early/experimental. See the [Roadmap](ROADMAP.md) for what works, what doesn't, and what's next.

## What it does

- **Stereoscopic 3D** rendered to the headset (OpenXR).
- **Head tracking** — your head drives the in-game camera (first-person, eye at the hunter's head).
- **Right-stick camera turn** + head-look layered on top.
- **Quest Touch controllers** mapped to an emulated Classic Controller (sticks + buttons).
- Live-tunable comfort settings (FOV, screen size, eye height, …) via a text config — no rebuild.

This is the same three-layer approach as [BotW-BetterVR](https://github.com/Crementif/BotW-BetterVR),
applied to MH Tri: stereo rendering, **PowerPC camera injection from the HMD pose**, and
Vulkan↔OpenXR presentation.

## Requirements

- **Windows** PC with a **Vulkan-capable GPU**.
- A **PCVR-capable headset** with an OpenXR runtime — Meta Quest (Link / Air Link / Virtual
  Desktop), or any SteamVR headset. *Developed and tested on a Quest 3S.*
- **Monster Hunter Tri** disc image — **NTSC-U, game ID `RMHE08`** (you must provide your own).
- ~2 GB free disk + the usual Dolphin requirements.

## Quick start

1. **Download** the latest [release](../../releases) and unzip it somewhere.
2. Make sure **`openxr_loader.dll`** sits next to **`Dolphin.exe`** (it's in the zip).
3. Launch `Dolphin.exe`. In Dolphin:
   - **Graphics → Backend = Vulkan** *(required)*.
   - **Controllers → Wii Remote 1 = Emulated**, then **Configure → Extension = Classic Controller**.
4. Put your headset on, start your OpenXR runtime (Quest Link / Virtual Desktop / SteamVR).
5. Boot your MH Tri image. You should be in VR.

Full walkthrough: **[docs/INSTALL.md](docs/INSTALL.md)**.

## Documentation

- **[Install & setup](docs/INSTALL.md)** — step by step, including headset/runtime notes.
- **[Controls](docs/CONTROLS.md)** — Touch → Classic Controller mapping, recenter, head-look.
- **[Config reference](docs/CONFIG.md)** — every `mhtri_vr_config.txt` setting, tuned live.
- **[Building from source](docs/BUILDING.md)** — for developers.
- **[Roadmap & known issues](ROADMAP.md)** — what works, what doesn't, what's next.

## Source & license

This is a fork of Dolphin Emulator and is distributed under the **GPL-2.0-or-later**. Full
source for the build is the Dolphin fork at **<https://github.com/Matt-Wood-23/dolphin>**
(branch **`mhtri-vr`**). MH Tri VR is a fan project, not affiliated with or endorsed by Nintendo,
Capcom, or the Dolphin project. **No game files are distributed** — bring your own legally-obtained copy.

## Credits

Built by Matt Wood. Conceptually indebted to **BotW-BetterVR** (Crementif) and the **Dolphin** team.
