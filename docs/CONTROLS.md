# Controls

Your **head** drives the camera (first-person). The **Quest Touch controllers** are mapped to an
emulated **Classic Controller**, which MH Tri reads as a standard pad. What each face button *does*
in-game follows MH Tri's own Classic Controller scheme (and is remappable in the game's options).

## Head & view

| Input | Action |
|---|---|
| **Head movement** | Look around (drives the in-game camera, first-person) |
| **Right stick ← →** | Turn your view (VR camera turn) — layered with head-look |
| **Right Ctrl** *(keyboard)* | Re-center the view to your hunter's current facing |

> Movement is **"move-where-you-look"**: pushing the left stick moves you relative to your current
> view direction (head + stick turn).

## Touch → Classic Controller mapping

| Touch (Quest) | Classic Controller | Notes |
|---|---|---|
| **Left thumbstick** | Left stick | Move |
| **Right thumbstick** | (camera turn) | Turns the view in VR; also the Classic right stick |
| **A** (right hand) | A | |
| **B** (right hand) | B | |
| **X** (left hand) | X | |
| **Y** (left hand) | Y | |
| **Right trigger** | ZR | analog + digital past ~50% |
| **Left trigger** | ZL | analog + digital past ~50% |
| **Right grip** | R | shoulder |
| **Left grip** | L | shoulder |
| **Menu** (left hand) | + (Start) | pause / menus |
| **Left stick click** | − (Minus) | map / sub-menu |
| **Right stick click** | HOME | reserved |

The physical/host controller still works alongside the Touch controllers (buttons OR together;
sticks hand over to the Touch sticks past a small deadzone).

## Notes & current limitations

- This mapping is a sensible default and **not yet fine-tuned**. If a stick axis feels inverted or
  a button is on the "wrong" key for your taste, it can be changed (see the project roadmap / open an issue).
- **Controller motion** (using the Touch controllers' position/rotation for aiming or a menu
  pointer) is **not implemented yet** — planned. Today it's sticks + buttons only.
- The recenter is on a keyboard key for now; moving it to a Touch button is planned.
