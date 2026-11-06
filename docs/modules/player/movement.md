# Movement

Part of [Player](index.md).

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
