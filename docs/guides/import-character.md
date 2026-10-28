# Use your own character

Any humanoid character works with UST. This guide lists the import settings that avoid the usual problems: a character
facing backwards, feet sliding, a body twisted while aiming.

## Character model

1. In the model import settings, **Rig** tab: **Animation Type: Humanoid**, **Avatar Definition: Create From This
   Model**, then **Apply**.
2. Click **Configure** and check that the legs, arms and spine are mapped to the right bones, not to twist or roll
   bones. If the model was not exported in a clean T-pose, use **Pose > Enforce T-Pose**.
3. Put the model as the child of the player prefab that holds the `Animator`, and assign its **Avatar**.
4. Run *Tools > UST > Validation > Check Model Orientation*: no mesh of the model should be listed.

!!! tip "Re-apply Humanoid after each export"
    After a new export of the model, apply **Humanoid** again in the Rig tab, or the character collapses on itself.

### A model facing backwards

Rotate the child holding the model by 180 on Y in the player prefab. The root of the prefab stays at 0, 0, 0: it
defines the forward direction for movement, aiming and the network.

You can also rotate the model in your 3D software. In Blender, rotate the armature by 180 on Z, select the armature
and all its meshes, then **Apply > All Transforms**. Then, on every animation file using **Copy From Other Avatar**
with this model, switch to **Create From This Model**: the animations keep their own orientation and the Humanoid
retargeting does the rest.

## Animations

For each animation, in the import settings:

| Setting | Value | Why |
|---|---|---|
| Rig > Animation Type | Humanoid | Generic animations ignore the Avatar Mask and do not retarget |
| Rig > Avatar Definition | Create From This Model, or Copy From Other Avatar with an avatar of the same orientation | A wrong avatar plays the animation backwards |
| Animation > Loop Time, Loop Pose | On | For locomotion and hold animations |
| Root Transform Rotation | Bake Into Pose, **Based Upon: Original** | **Body Orientation** turns the whole character with the torso of the animation |
| Root Transform Position (Y) and (XZ) | Bake Into Pose | The controller moves the character, not the animation |

The **loop match** indicators next to each setting should be green.

On the `Animator`, disable **Apply Root Motion**.

!!! tip "Animations from the same pack"
    A character and animations made for the same rig retarget without surprise. Mixing packs works, but each animation
    may hold the hands a little differently: the left hand IK and the weapon alignment make up for it.

## Common problems

| Problem | Cause | Fix |
|---|---|---|
| Feet rub or cross while walking | Retargeting of the legs | **Configure > Muscles & Settings > Feet Spacing**, around 0.1. Try the **Foot IK** box of the Animator state. |
| Tiny steps | The walk animation is blended with the idle | Blend tree thresholds: idle 0, walk 0.5, run 1, **Automate Thresholds** off |
| Feet slide on the ground | Animation pace and movement speed differ | Adjust the speeds of the Movement Settings, or the **Speed** of the motion in the blend tree |
| Zigzag while running | Root rotation of the animation | Root Transform Rotation: **Bake Into Pose**, **Based Upon: Original** |
| Walks backwards | Avatar of the animation turned differently from the model | **Create From This Model** on the animation file |
| Arms in T-pose with an item | A state of the UpperBody layer has no animation | Fill the state, or check the **Hold Type** of the item |
| The weapon bounces while walking | Hips and spine come from the locomotion | Add **Body** to the upper body Avatar Mask |
