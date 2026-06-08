# Building from source

The code lives in the **Dolphin fork**: <https://github.com/Matt-Wood-23/dolphin>, branch
**`mhtri-vr`**. MH Tri VR is gated behind a compile flag, so the default build is a stock Dolphin;
only the flagged build pulls in OpenXR.

## Prerequisites

- The standard [Dolphin Windows build prerequisites](https://github.com/dolphin-emu/dolphin)
  (Visual Studio 2022 with the C++ / Windows SDK workloads).
- The OpenXR loader is **vendored** in the repo at `Externals/OpenXR/` (headers + `openxr_loader.lib`
  + `openxr_loader.dll`), so no extra SDK install is needed.

## Build (VR-enabled)

From the repo root, the VR build is a normal MSBuild with one extra property — `MHTriVROpenXR=true`:

```bash
MSBuild.exe "Source/Core/DolphinQt/DolphinQt.vcxproj" \
  -p:Configuration=Release -p:Platform=x64 -p:MHTriVROpenXR=true -m
```

That flag (via `Source/VSProps/Base.Dolphin.props`) defines `MHTRI_VR_OPENXR`, adds the OpenXR
include path, and links `openxr_loader.lib`. **Without it**, every VR code path compiles to inert
no-ops and the result is byte-identical to upstream Dolphin (no OpenXR dependency).

After building, copy the loader next to the exe:

```bash
cp Externals/OpenXR/bin/x64/openxr_loader.dll Binary/x64/openxr_loader.dll
```

Run `Binary/x64/Dolphin.exe`. (You must still select the **Vulkan** backend and a **Classic
Controller** extension — see [INSTALL.md](INSTALL.md).)

## Where the VR code lives

| Area | Files |
|---|---|
| OpenXR session, swapchains, frame submit, **controller input** | `Source/Core/VideoCommon/VROpenXR.{h,cpp}`, `VROpenXR_Vulkan.h` |
| Camera injection driver (head-look, first-person, recenter, follow-base) | `Source/Core/Core/HW/MHTriVR.{h,cpp}` |
| **Per-game profiles** (camera hook + player-state layout, keyed by Game ID) | `Source/Core/Core/HW/VRGameProfiles.h` |
| Stereo override | `Source/Core/VideoCommon/VRStereo.{h,cpp}` |
| Vulkan backend hooks (blit XFB → eye swapchains, single-thread submit) | `Source/Core/VideoBackends/Vulkan/VKMain.cpp`, `VulkanContext.cpp` |
| Per-eye FOV feedback | `Source/Core/VideoCommon/VertexShaderManager.cpp` |
| Touch → Classic Controller injection | `Source/Core/Core/HW/WiimoteEmu/Extension/Classic.cpp` |
| Present-time hooks | `Source/Core/VideoCommon/Present.cpp` |

## Architecture in one paragraph

`MHTriVR` (Core) hooks the camera commit and rewrites the eye/target from the HMD pose. The hook
address and player-state offsets are **not** hard-coded — they come from a per-game **profile**
(`VRGameProfiles.h`) selected by Game ID at boot, so the driver is multi-game. For MH Tri the hook is
`cam_commit_to_g3d` @ `0x802BE0F4` (RMHE08). To add another title, see
**[ADDING_A_GAME.md](ADDING_A_GAME.md)** — it's one data row, no driver changes.
`VROpenXR` (VideoCommon) owns the OpenXR instance, a Vulkan-shared
session, per-eye swapchains, the frame loop, and the Touch action set; the Vulkan backend blits
Dolphin's XFB into the eye swapchains and forces single-threaded queue submission while a VR session
is up (so Dolphin's submit thread can't race the XR compositor). Core sits above VideoCommon, so the
camera hook *pulls* the head pose and controller state via plain getters. The flag keeps it all
optional.

## License

GPL-2.0-or-later (inherited from Dolphin). Any binary distribution must offer this source.
