# Player

The Player module moves the character, reads the input, drives the camera zoom and lets the player interact
with the world. It does not depend on any other gameplay module: stamina, death or inventory plug into it through
small interfaces, and everything keeps working without them.

## Prefab setup

The root of the player prefab holds:

| Component | Role |
|---|---|
| `NetworkObject` | Required by Netcode. |
| `NetworkTransform` | **Authority Mode: Owner**. Disable scale sync and rotation X and Z: the character only turns around Y. |
| `CharacterController` | Collision capsule. Set **Center Y** to half the height so the bottom of the capsule sits at the feet. |
| `PlayerController` | Movement, crouch, jump, stamina use. |
| `PlayerInputReader` | Reads the Input System actions. |
| `CameraZoom` | Camera distance with the mouse wheel, and closer while aiming. |
| `PlayerInteractor` | Pick up and interact. |
| `PlayerNetworkSetup` | Owner only components and camera binding. |
| `PlayerCursorLock` | Locks and hides the mouse cursor while playing. |

The child holding the `Animator` also holds:

| Component | Role |
|---|---|
| `PlayerFootIK` | Keeps the feet on steps and slopes. Needs **IK Pass** on the animator layer. |
| `NetworkAnimator` | **Authority Mode: Owner**. Without it, remote players slide without animation. |

!!! tip "Keep the root at 0, 0, 0"
    The Asset Store validator asks for prefabs at the origin. The character still spawns slightly above the ground
    thanks to **Spawn Height Offset** on `PlayerLife`.

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

Every action of `UST_InputActions` has a keyboard and mouse binding and a gamepad binding. Players change them in the Keys tab of the [options](settings.md#changing-the-keys). Run *UST Tools > Advanced > Input > Assign Input Actions* once: it fills the empty input fields of the player prefab and of the scene with these actions,
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
| D-pad left | Ping, hold to place a marker ([Markers](markers.md)) | Rotate the piece |
| D-pad right | Map ([Markers](markers.md#map-and-minimap)) | Rotate the piece |
| L3, R3 | Sprint | Remove the piece |
| View, Start | Inventory | Session menu (Lobby module) |

**Menus**: when a window opens while the player uses a gamepad, a cursor appears, moved by the left stick. A clicks and
drags, X right-clicks, the right stick scrolls and B closes the window. The character does not move meanwhile. Every
window made for the mouse works this way without changes, your own too. See [Gamepad cursor](ui.md#gamepad-cursor).

**Camera**: the gamepad binding of Look has the `USTStickDelta(speed=9000, curve=1.5)` processor, which turns the stick
into a mouse-like movement. Raise **speed** for a faster camera; **curve** above 1 makes small movements finer. The mouse
sensitivity and the inverted look of the [options](settings.md) are applied by `CameraUILock` to the gains of the
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

## Movement

### Movement settings

`PlayerMovementSettings` is a ScriptableObject created with *Create > UST > Player > Movement Settings*. Assign it to
**Movement Settings** on `PlayerController`. Several characters can share one asset, or each can have its own.

| Field | Default | Meaning |
|---|---|---|
| Walking Speed | 1.4 | m/s |
| Running Speed | 2.6 | m/s while sprinting |
| Rotation Speed | 10 | How fast the body turns towards the movement direction |
| Align Player Rotation Speed | 15 | How fast the body turns towards the camera while aiming, strafing or attacking |
| Strafe Max Speed | 1.6 | Max speed sideways while facing the camera. Walking and running are capped to it. |
| Backward Max Speed | 1.2 | Max speed backwards while facing the camera |
| Jump Force | 5 | Initial vertical speed of a jump |
| Jump Cooldown | 0.2 | Seconds before the next jump |

!!! warning
    This asset is shared. Never change it from code at runtime: the change would apply to every character using it
    and would be lost in a build. Equipment or buffs should apply their own multipliers.

### Controller settings

| Section | Field | Default | Meaning |
|---|---|---|---|
| | Animator | | Animator of the character model, driven by the controller |
| | Player Camera | | Camera the movement and the aim are relative to: the main camera of the scene, where the CinemachineBrain is. Required |
| | Input Reader, Movement Settings | | The `PlayerInputReader` of the player and the [movement settings](#movement-settings) asset |
| | Gravity | | Downward acceleration while airborne |
| Move | Locomotion Smooth Time | 0.2 | Smoothing of the animator Velocity, MoveX and MoveY parameters |
| Move | Strafe Movement | On | The character faces the camera while moving and walks sideways and backwards. Off: it turns towards the movement. |
| Aim | Face Aim Point | On | While aiming or attacking, faces the point under the crosshair rather than the camera direction |
| Aim | Aim Layers, Aim Distance | Everything, 100 | Where the crosshair point is searched |
| Aim | Aim Yaw Offset | 0 | Extra degrees for aim animations that do not point straight ahead |
| Crouch | Crouch Height Ratio | 0.62 | Crouched capsule height, as a fraction of the standing height |
| Crouch | Headroom Layers | Everything | Layers that prevent standing up |
| Ground | Ground Stick Distance | 0.5 | How far below the feet the ground is still followed, so stairs do not read as falls |
| Ground | Ground Stick Slope | 1.2 | Steepest slope followed, as rise over run |
| Ground | Min Ground Stick Speed | 1.5 | Minimum speed the character is pressed onto the ground |
| Stamina | Sprint Stamina Per Second | 10 | Stamina spent while sprinting |
| Stamina | Sprint Stamina To Start | 10 | Stamina needed to start a sprint, so the sprint does not flicker around zero |
| Stamina | Jump Stamina Cost | 10 | Stamina spent per jump |

**Crouch** is a toggle. Standing up is refused while something is above the head: the capsule between the crouched
and the standing height is checked against **Headroom Layers**.

**Stamina** is optional. The controller looks for an `IStaminaSource` on the same object, such as
`VitalStaminaSource` from the Vitals module. Without one, stamina is unlimited.

### Locking movement

`canMove`, `canRotate`, `canRun` and `canJump` are public flags you can switch from your own code, for a cutscene,
a sitting animation or a stun. They show in the inspector and can be ticked in Play Mode to test.

`isCrouching`, `isRunning` and `isJumping` show the current state, in the inspector too. Read them, or `IsRunning`,
`IsAiming` and `IsFeetPlanted`, from your scripts; do not set them, the controller does.

## Animator

*UST Tools > Advanced > Player > Create Animator Controller* creates a controller with every parameter UST uses, the IK pass
enabled, two empty states, **Locomotion** and **Crouch**, and the **UpperBody** layer described below.
*right click in the Project window > UST > Add Upper Body Layer* adds the missing parameters and the layer to an existing
controller.

| Parameter | Type | Set by | Value |
|---|---|---|---|
| `Velocity` | Float | PlayerController | 0 idle, 0.5 walk, 1 run |
| `MoveX`, `MoveY` | Float | PlayerController | Movement relative to the body, from -1 to 1: 0.5 walking, 1 running |
| `HoldType` | Int | PlayerHeldItem | Hold type of the held item, 0 with empty hands |
| `Aiming` | Bool | PlayerRangedWeapon | Aiming, or weapon raised after a hip shot |
| `Fire`, `Reload` | Trigger | PlayerRangedWeapon | Shot fired, reload started (names set per weapon) |
| `Grounded` | Bool | PlayerController | On the ground |
| `Crouching` | Bool | PlayerController | Crouched |
| `Jump` | Trigger | PlayerController | Jump started |
| `Attack` | Trigger | PlayerItemUser | Swing started (name set per item) |
| `Dead` | Bool | PlayerLife | Character dead |

Suggested states:

- **Locomotion**: 2D Freeform Directional blend tree on `MoveX` and `MoveY`: idle at (0, 0), walks at 0.5 and runs
  at 1 in each direction (forward on Y, backward on -Y, sides on X). Without strafe animations, use a 1D blend tree on
  `Velocity` and turn **Strafe Movement** off.
- **Crouch**: crouched blend tree on `Velocity`, entered with `Crouching = true`.
- **Jump** then **Fall** while `Grounded = false`, back to Locomotion or Crouch when `Grounded = true`.

!!! warning "Root motion"
    Disable **Apply Root Motion** on the Animator. The controller moves the character itself and root motion would
    fight it.

### Upper body and hold types

The **UpperBody** layer plays the animations of the held item over the locomotion: the legs keep walking while the
arms hold a rifle. It uses an Avatar Mask created next to the controller, with the body, head and arms.

Each hold type has its states, linked by `HoldType`, `Aiming`, `Reload` and `Attack`:

| Hold type | States |
|---|---|
| 0 empty hands | **Empty**: the layer does nothing, the locomotion animates the whole body |
| 1 one-handed | **Hold One Handed**, **Attack One Handed** |
| 2 rifle | **Hold Rifle**, **Aim Rifle**, **Reload Rifle** |
| 3 pistol | **Hold Pistol**, **Aim Pistol**, **Reload Pistol** |

**Hold** states are 1D blend trees on `Velocity`: the hold animation at rest on the first line, the hold animation
while running on the second. One animation on both lines is fine. Set the **Hold Type** of each item in its Held
Visual trait.

!!! tip "One pose is enough"
    A single static pose per weapon type works: the legs come from the locomotion, and the aim parts align the weapon
    with the crosshair.

### Foot IK

`PlayerFootIK` keeps the feet on steps and slopes the animation does not know about. It goes on the object that
carries the **Animator**, not on the player root: Unity only calls the IK there. The animator layer needs **IK Pass**
on (*Create Animator Controller* enables it).

- **Up a step**: a foot that would sink into the step is lifted onto it.
- **Down a step**: a leg cannot stretch a whole tread lower, so the hips drop, but only for the foot the animation is
  putting down, and only as far as that foot needs. On flat ground the hips never move, so the walk does not bob.
- **Slopes**: the soles tilt to match the ground.
- In the air, when jumping or rolling, the effect fades out.

| Section | Field | Default | Meaning |
|---|---|---|---|
| Ground | Ground Layers | Everything | What counts as ground. Leave out the player layer and trigger volumes |
| Ground | Foot Bone Height | 0.056 | Height of the foot bone above the sole when planted, in metres. Measure it on the idle pose of your character |
| Ground | Probe Above | 0.6 | How far above the animated foot the ground probe starts |
| Ground | Probe Below | 0.9 | How far below the animated foot the probe reaches |
| Stepping down | Reach Down | On | Also lowers the feet onto ground below them, with the hips. Off: feet are only lifted |
| Stepping down | Max Hip Drop | 0.32 | Deepest the hips drop to reach a lower step, in metres |
| Stepping down | Plant Height | 0.10 | Below this height above the ground, the animation counts as putting the foot down. The swinging foot reaches about 0.21 walking and 0.36 running |
| Stepping down | Hip Response | 6 | How fast the hips follow. Low enough that a single step does not snap them |
| Correction | Dead Zone | 0.02 | Corrections smaller than this are ignored, so flat ground never jitters |
| Correction | Max Foot Lift | 0.35 | Largest lift accepted on one foot. Beyond, the animation is left alone rather than stretching the leg |
| Correction | Weight Response | 12 | How fast the effect fades in and out |
| Correction | Align To Slope | On | Tilts the soles to match the slope |

!!! tip "Tuning for another character"
    Start with **Foot Bone Height**: with the character idle on flat ground, it is the height of the foot bone above
    the ground. If the feet float or sink on flat ground, it is wrong. Then raise **Plant Height** if the hips drop
    late on stairs, or lower it if they bob while walking.

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

## Multiplayer setup

`PlayerNetworkSetup` disables the components listed in **Owner Only** on every copy of the player that is not
controlled locally. When the component is added, the list is filled with `PlayerController`, `PlayerInputReader`
and `CameraZoom`. Remote copies are then moved by their `NetworkTransform` and animated by their `NetworkAnimator`.

## Interaction

`PlayerInteractor` casts a ray from the center of the screen. The first `IInteractable` within **Range** (2.5 m,
measured from the chest of the character) becomes `Current`, and pressing the Use input calls its `Interact`.

- Colliders of the player itself are ignored.
- A solid collider that is not interactable blocks the ray. Triggers do not.
- Only the owner runs it.

`WorldItem` and `LootBag` implement `IInteractable`. Make your own:

```csharp
using UnityEngine;
using UST.Core;

public class Door : MonoBehaviour, IInteractable
{
    public string InteractionPrompt => "Open";

    public void Interact(GameObject interactor)
    {
        // Runs on the interacting client: send an RPC if the server must validate it.
    }
}
```

`InteractionPrompt` is the text a UI can show, such as "E - Open".
