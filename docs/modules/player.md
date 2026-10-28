# Player

The Player module moves the character, reads the input, drives the camera feedback and lets the player interact
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
| `CameraJuice` + `CinemachineImpulseSource` | Camera feedback: speed field of view, landing shake, hit shake. |
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
a sitting animation or a stun.

## Animator

*Tools > UST > Create Player Animator Controller* creates a controller with every parameter UST uses, the IK pass
enabled, two empty states, **Locomotion** and **Crouch**, and the **UpperBody** layer described below.
*Tools > UST > Add Upper Body Layer To Selected Controller* adds the missing parameters and the layer to an existing
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

## Camera

The scene holds one `CinemachineCamera` with:

- `CinemachineOrbitalFollow` for the third person orbit,
- `CinemachineInputAxisController` with the Look action on each axis,
- `CinemachineImpulseListener` so `CameraJuice` shakes are felt.

Leave its **Tracking Target** empty. When the local player spawns, `PlayerNetworkSetup` binds the camera to it (or
to its **Camera Target**, for example a child at shoulder height) and assigns the main camera to `PlayerController`.

To invert the vertical axis, set a negative **Gain** on the vertical axis of the input axis controller.

### Mouse cursor

`PlayerCursorLock` locks and hides the cursor for the local player once spawned. The cursor is freed while a UI is
open, after pressing **Release Key** (Escape) and when the window loses focus. A click in the game locks it again.
**Free While Dead** frees it during death, for a death screen with buttons.

`CameraUILock`, on the Cinemachine camera, stops the orbit while a UI is open, so moving the mouse over the inventory
does not turn the camera. *Create Game UI* adds it automatically.

While a UI is open, `PlayerInputReader` also ignores aim and interact. Movement, jump, sprint and crouch keep working.

## Multiplayer setup

`PlayerNetworkSetup` disables the components listed in **Owner Only** on every copy of the player that is not
controlled locally. When the component is added, the list is filled with `PlayerController`, `PlayerInputReader` and
`CameraJuice`. Remote copies are then moved by their `NetworkTransform` and animated by their `NetworkAnimator`.

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
