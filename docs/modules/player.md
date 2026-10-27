# Player

## Player prefab

The root of the player prefab holds:

| Component | Role |
|---|---|
| `NetworkObject` | |
| `NetworkTransform` | **Authority Mode: Owner**. Disable scale sync and rotation X and Z. |
| `CharacterController` | Center Y = half the height, so the capsule bottom sits at the feet |
| `PlayerController` | Movement, crouch, jump, stamina use |
| `PlayerInputReader` | Input System actions |
| `CameraJuice` + `CinemachineImpulseSource` | Camera feedback |
| `PlayerInteractor` | Pick up and interact |
| `PlayerNetworkSetup` | Owner only setup |

The child holding the `Animator` also holds `PlayerFootIK` and a `NetworkAnimator` with **Authority Mode: Owner**.

## Input

`PlayerInputReader` reads `InputActionReference` fields: Move, Look, Use, Jump, Run, Crouch and Aim. Assign the
actions of your input actions asset. Missing actions are ignored.

## Movement settings

`PlayerMovementSettings` is a ScriptableObject holding walking speed, running speed, rotation speeds, jump force and
jump cooldown. Create one with *Create > UST > Player > Movement Settings* and assign it to the controller.

## Animator

*Tools > UST > Create Player Animator Controller* creates a controller with every parameter the scripts use:

| Parameter | Type | Value |
|---|---|---|
| `Velocity` | Float | 0 idle, 0.5 walk, 1 run |
| `Grounded` | Bool | |
| `Crouching` | Bool | |
| `Dead` | Bool | Set by `PlayerLife` if it exists |
| `Jump` | Trigger | |

Disable **Apply Root Motion**: the controller moves the character itself.

## Camera

The scene holds one `CinemachineCamera` with `CinemachineOrbitalFollow`, a `CinemachineInputAxisController` and a
`CinemachineImpulseListener`. Leave its tracking target empty: `PlayerNetworkSetup` binds it to the local player
when it spawns. To invert the vertical axis, set a negative **Gain** on the vertical axis of the input axis
controller.

## Interaction

`PlayerInteractor` casts a ray from the screen center and calls `Interact` on the first `IInteractable` within
`range` when the Use input is pressed. Make your own interactable objects by implementing `IInteractable`:

```csharp
public class Door : MonoBehaviour, IInteractable
{
    public string InteractionPrompt => "Open";

    public void Interact(GameObject interactor)
    {
        // Called on the client: send an RPC if the server must validate it.
    }
}
```
