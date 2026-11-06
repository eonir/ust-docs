# Player

The Player module moves the character, reads the input, drives the camera zoom and lets the player interact
with the world. It does not depend on any other gameplay module: stamina, death or inventory plug into it through
small interfaces, and everything keeps working without them.

This module is described on several pages:

- [Input and gamepad](input.md)
- [Movement](movement.md)
- [Animator and Foot IK](animator.md)
- [Camera](camera.md)

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
