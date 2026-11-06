# Input and gamepad

Part of [Player](index.md).

## Input

`PlayerInputReader` replaces a global input manager: each player reads its own input, which is what multiplayer
needs. Assign actions from your input actions asset. The template uses `Assets/UST/UST_InputActions`, which never
replaces the `InputSystem_Actions` asset of your project:

| Field | Typical action | Used for |
|---|---|---|
| Move Action | Player/Move | Movement |
| Look Action | Player/Look | Body rotation, idle camera detection |
| Use Action | Player/Interact | Interaction |
| Jump Action | Player/Jump | Jump |
| Run Action | Player/Sprint | Sprint |
| Crouch Action | Player/Crouch | Crouch toggle |
| Aim Action | your own action | Body aligned with the camera |

Missing actions are simply ignored. The reader enables its actions itself, so no `PlayerInput` component is needed.

Your own scripts can read the input of the local player from the reader, instead of reading the devices:

| Member | Value |
|---|---|
| `move`, `look` | Vector2 of the Move and Look actions. `lookMove` is true while Look is not zero |
| `useButtonDown`, `useButtonHeld`, `useButtonUp` | Use action. Down and Held are false while a UI is open |
| `jumpButtonDown`, `jumpButtonHeld`, `jumpButtonUp` | Jump action |
| `RunDown`, `runButtonHeld`, `RunUp` | Run action |
| `crouchButtonDown`, `crouchButtonHeld` | Crouch action |
| `AimClickDown`, `AimHeld`, `AimUp` | Aim action. Down and Held are false while a UI is open |

*Down* is true on the frame the button is pressed, *Up* on the frame it is released, *Held* while it is down.

### Gamepad

Every action of `UST_InputActions` has a keyboard and mouse binding and a gamepad binding. Players change them in the Keys tab of the [options](../settings.md#changing-the-keys). Run *Tools > UST > Advanced > Input > Assign Input Actions* once: it fills the empty input fields of the player prefab and of the scene with these actions,
and adds the gamepad cursor to the game UI. Fields you already set are kept. *Create Game UI* and *Add Lobby To
Scene* run it on the scene too.

| Gamepad | In game | While building |
|---|---|---|
| Left stick, right stick | Move, look | |
| A, B | Jump, crouch | |
| X | Reload | Repair |
| Y | Interact (get in a vehicle, sleep, open...) | |
| RT, LT | Attack or use, aim | Build, hold for free placement |
| LB, RB | Previous and next hotbar slot | |
| D-pad up, down | Camera zoom | Snap mode, build menu |
| D-pad left | Ping, hold to place a marker ([Markers](../markers/index.md)) | Rotate the piece |
| D-pad right | Map ([Markers](../markers/index.md#map-and-minimap)) | Rotate the piece |
| L3, R3 | Sprint | Remove the piece |
| View, Start | Inventory | Session menu (Lobby module) |

**Menus**: when a window opens while the player uses a gamepad, a cursor appears, moved by the left stick. A clicks and
drags, X right-clicks, the right stick scrolls and B closes the window. The character does not move meanwhile. Every
window made for the mouse works this way without changes, your own too. See [Gamepad cursor](../ui/windows.md#gamepad-cursor).

**Camera**: the gamepad binding of Look has the `USTStickDelta(speed=9000, curve=1.5)` processor, which turns the stick
into a mouse-like movement. Raise **speed** for a faster camera; **curve** above 1 makes small movements finer. The mouse
sensitivity and the inverted look of the [options](../settings.md) are applied by `CameraUILock` to the gains of the
Cinemachine axis controller: the gains set in the inspector are the 1x sensitivity. It also makes the look independent
of the frame rate: a mouse moves the camera as much at 30 or 240 FPS, with the feel of its **Reference Frame Rate**
(60). Without it, Cinemachine scales a mouse movement by the frame time, and the camera slows down as the FPS rise.

**Hints**: the interaction prompt, the build hints and the vehicle controls show the buttons of the device in use,
and switch as soon as the player touches the other one.

Change any binding in `UST_InputActions`: everything follows, hints included. Your own scripts can use the same
helpers:

```csharp
if (GameInput.Pressed(myAction, Key.F)) { }              // the action, or the key when no action is assigned
bool held = GameInput.Held(myAction);
string button = GameInput.BindingText(myAction, "F");    // "F" or "Y", for the device in use
GameInput.DeviceChanged += gamepad => RefreshHints();
```

All of them ignore the keys while the player types in the chat.

!!! warning "Legacy input"
    The project must use the Input System only. `Input.GetAxis` and other legacy calls throw an exception in that
    mode.
