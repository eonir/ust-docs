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
| Align Player Rotation Speed | 15 | How fast the body turns towards the camera while aiming |
| Jump Force | 5 | Initial vertical speed of a jump |
| Jump Cooldown | 0.2 | Seconds before the next jump |

!!! warning
    This asset is shared. Never change it from code at runtime: the change would apply to every character using it
    and would be lost in a build. Equipment or buffs should apply their own multipliers.

### Controller settings

| Section | Field | Default | Meaning |
|---|---|---|---|
| | Gravity | | Downward acceleration while airborne |
| Move | Locomotion Smooth Time | 0.2 | Smoothing of the animator Velocity parameter |
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
enabled and two empty states, **Locomotion** and **Crouch**.

| Parameter | Type | Set by | Value |
|---|---|---|---|
| `Velocity` | Float | PlayerController | 0 idle, 0.5 walk, 1 run |
| `Grounded` | Bool | PlayerController | On the ground |
| `Crouching` | Bool | PlayerController | Crouched |
| `Jump` | Trigger | PlayerController | Jump started |
| `Attack` | Trigger | PlayerItemUser | Swing started (name set per item) |
| `Dead` | Bool | PlayerLife | Character dead |

Suggested states:

- **Locomotion**: 1D blend tree on `Velocity` with idle at 0, walk at 0.5, run at 1.
- **Crouch**: crouched blend tree on `Velocity`, entered with `Crouching = true`.
- **Jump** then **Fall** while `Grounded = false`, back to Locomotion or Crouch when `Grounded = true`.

!!! warning "Root motion"
    Disable **Apply Root Motion** on the Animator. The controller moves the character itself and root motion would
    fight it.

## Camera

The scene holds one `CinemachineCamera` with:

- `CinemachineOrbitalFollow` for the third person orbit,
- `CinemachineInputAxisController` with the Look action on each axis,
- `CinemachineImpulseListener` so `CameraJuice` shakes are felt.

Leave its **Tracking Target** empty. When the local player spawns, `PlayerNetworkSetup` binds the camera to it (or
to its **Camera Target**, for example a child at shoulder height) and assigns the main camera to `PlayerController`.

To invert the vertical axis, set a negative **Gain** on the vertical axis of the input axis controller.

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
