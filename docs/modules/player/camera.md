# Camera

Part of [Player](index.md).

## Camera

The scene holds one `CinemachineCamera` with:

- `CinemachineOrbitalFollow` for the third person orbit,
- `CinemachineInputAxisController` with the Look action on each axis.

Leave its **Tracking Target** empty. When the local player spawns, `PlayerNetworkSetup` binds the camera to it (or
to its **Camera Target**, for example a child at shoulder height) and assigns the main camera to `PlayerController`.

To invert the vertical axis, set a negative **Gain** on the vertical axis of the input axis controller.

### Camera zoom

`CameraZoom`, on the player root, changes the distance of the `CinemachineOrbitalFollow` with the mouse wheel. The
wheel is ignored while a UI is open, and while another system claims it (`MouseWheelFocus`), such as the build
preview, which rotates with the wheel.

| Field | Default | Meaning |
|---|---|---|
| Zoom Action | | Mouse wheel (Vector2). Empty reads the mouse directly. |
| Zoom In Action, Zoom Out Action | Player/ZoomIn, Player/ZoomOut | Optional buttons zooming one step: d-pad up and down on a gamepad |
| Min Distance, Max Distance | 2, 8 | Closest and furthest distance |
| Step | 0.75 | Distance changed by one notch of the wheel |
| Invert | off | Reverses the wheel direction |
| Scroll Threshold | 0.1 | Minimum wheel movement |
| Response | 10 | How fast the distance follows the wheel |
| Aim Distance | 0 | Distance while aiming, clamped between Min and Max Distance. 0 uses Min Distance. |
| Aim Response | 14 | How fast the distance changes when aiming starts or stops |

While aiming with a ranged weapon, `AimCameraZoom` moves the camera to the aim distance, and back to the zoom distance
on release. The wheel is ignored while aiming, so the zoom distance is kept. Scripts can set `CameraZoom.Aiming`
themselves for other aims.

`AimCameraZoom` goes on the CinemachineCamera that follows the player. It also narrows the field of view while aiming:

| Field | Default | Meaning |
|---|---|---|
| Aim Field Of View | 45 | Field of view while aiming, in degrees |
| Zoom Speed | 10 | How fast the field of view changes |

### Mouse cursor

`PlayerCursorLock` locks and hides the cursor for the local player once spawned. The cursor is freed while a UI is
open, after pressing **Release Key** (Escape) and when the window loses focus. A click in the game locks it again.
**Free While Dead** frees it during death, for a death screen with buttons.

`CameraUILock`, on the Cinemachine camera, stops the orbit while a UI is open, so moving the mouse over the inventory
does not turn the camera. *Create Game UI* adds it automatically.

While a UI is open, `PlayerInputReader` also ignores aim and interact. Movement, jump, sprint and crouch keep working.
