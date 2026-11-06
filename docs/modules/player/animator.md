# Animator and Foot IK

Part of [Player](index.md).

## Animator

*Tools > UST > Advanced > Player > Create Animator Controller* creates a controller with every parameter UST uses, the IK pass
enabled, two empty states, **Locomotion** and **Crouch**, and the **UpperBody** layer described below.
Right click a controller in the Project window > **UST > Add Upper Body Layer** adds the missing parameters and the layer to an existing
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
