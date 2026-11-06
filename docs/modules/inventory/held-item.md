# Item in the hand

Part of [Inventory](index.md).

## Item in the hand

`PlayerHeldItem` shows the model of the selected hotbar item on every peer. Only the selection is synchronized: the
model itself is a local object, so holding an item costs no network object.

The models of the last items held stay on the character, hidden, and are shown again when their item comes back:
once each item was held, changing items creates and destroys nothing. **Kept Models** (8) sets how many are kept;
past it, the oldest is destroyed. 0 destroys each model when it leaves the hand. A model shown again gets the placement
of its Held Visual trait back. Call `ClearKept()` after changing the Held Visual of items at runtime.

1. Create a **model prefab**: only the mesh, no `NetworkObject`.
2. On the item: **Add Trait > Visual > Held Visual**, assign the model prefab and adjust the offsets.

| Held Visual field | Meaning |
|---|---|
| Prefab | The model prefab |
| Position Offset, Rotation Offset, Scale | Placement in the hand |
| Hold Type | Value of the `HoldType` Animator parameter while held: 0 empty hands, 1 one-handed, 2 rifle, 3 pistol. Add your own values. See [Player > Animator](../player/animator.md#animator). |
| Left Hand Grip | When the left hand goes to the Grip point: **Always**, **While Aiming** (pistols held with one hand at rest), **Never** |
| Override Grip, Grip Position, Grip Rotation | Grip placement saved with **Save Grip** in the Held Item Tuning window, used instead of the Grip point of the model |
| Weapon Pose | Who places the item: the animation (**Follow Hand**) or the rig (**While Aiming**, **Always**), see [Weapon pose](#weapon-pose) |
| Aim Pose Position, Aim Pose Rotation | Placement of the item while aiming, from the chest, towards the crosshair. Default position (0.15, -0.05, 0.35). Set by **Capture Aim Pose** or **Save As Aim Pose** |
| Rest Pose Position, Rest Pose Rotation | Placement of the item at rest, from the chest, along the character, used with **Always**. Default position (0.15, -0.25, 0.3). Set by **Capture Rest Pose** or **Save As Rest Pose** |
| Aim Yaw Offset, Aim Pitch Offset | Correction of the upper body aim with this item, for aim poses that do not point straight ahead |

The fields marked as tuned in Play Mode are written by the Held Item Tuning window: you rarely type them by hand.

| PlayerHeldItem field | Meaning |
|---|---|
| Inventory | The `PlayerInventory` whose selected hotbar slot is shown |
| Hand Socket | Where the model is attached. Empty: the right hand bone of the humanoid Animator. |
| Animator | Found automatically if empty |
| Hold Type Parameter | Animator int set to the **Hold Type** of the held item, 0 with empty hands. Default `HoldType`. Empty: no parameter is set, for an Animator without upper body layer |
| Kept Models | 8. Models of the last items held kept hidden on the character, see above |

The colliders of the model are disabled, so it never blocks raycasts or pushes the character. Rigs exported in
centimetres, with a scale of 100 on their bones, are compensated automatically.

### Placing the item in the hand

Everything is tuned in Play Mode, with the character animated, in one window: *Tools > UST > Held Item Tuning* (or
**Open Held Item Tuning** on `PlayerHeldItem`).

1. Press Play and select the item in the hotbar. **Freeze Animation** stops the character on its current pose, so the
   hand does not move while you place things.
2. **Placement in the hand**: click **Move** or **Rotate**, adjust the model in the Scene view, then **Save Placement**.
3. **Left hand**: choose when the left hand grips, then **Move** or **Rotate** the grip until the hand looks right,
   and **Save Grip**. Without a grip on the model, **Create Grip** adds one. **Use Model Grip** goes back to the Grip
   point of the model prefab.
4. **Aim**: tick **Preview Aim** to play the aim pose without holding the aim input (tick it before freezing the
   animation, so the pose has time to blend in), then correct it with **Yaw Offset** and **Pitch Offset**.
5. **Weapon pose** (ranged and two-handed weapons): see [Weapon pose](#weapon-pose).

Every change is written to the Held Visual trait of the item and kept after Play Mode. The grip is saved in the space
of the model, so two items can share a model with different grips.

### Points of the model

Add an empty child to the model prefab with a `HeldItemPoint` component:

| Type | Used by | Gizmo |
|---|---|---|
| **Muzzle** | Ranged weapons: where projectiles start. Its blue Z axis must point out of the barrel. | Red, with a line in the shot direction |
| **Grip** | `PlayerItemRig`: where the left hand goes. The hand takes its rotation. | Cyan |

Children named `Muzzle` and `Grip` work too, when there is no `HeldItemPoint`.

### Item rig

The left hand and the upper body aim use the **Animation Rigging** package (`com.unity.animation.rigging`).
`PlayerItemRig` drives the rig:

- the left hand holds the Grip point of the item (rifles, two-handed tools, crossbows), following **Left Hand Grip**,
- while aiming, the spine, chest and head turn towards the point under the crosshair, up, down and sideways, but
  never roll. The upper body keeps facing the aim whatever the legs do, strafes included. If the aim pose has a
  shoulder stance (a rifle held across the chest), restore it with the **Aim Yaw Offset** of the item. The weapon
  itself is then turned to the crosshair by `PlayerRangedWeapon` (**Weapon Alignment** at 1). The aim is
  synchronized, so every player sees the others aim.

**Setup**: open the player prefab, right click it (or its model) in the Hierarchy > **UST > Setup Item Rig**. The
tool adds a `RigBuilder` and `PlayerItemRig` on the Animator object, and a child **Item Rig** holding:

| Object | Constraint |
|---|---|
| Aim Target | Point the upper body aims at, placed by `PlayerItemRig` |
| Weapon Pivot > Weapon Pose | Where the item goes with a Weapon Pose, in front of the chest |
| Right Hand IK | Two Bone IK on the right arm, with **Right Hand Target** and **Right Elbow Hint**, used by the Weapon Pose |
| Aim Spine, Aim Chest, Aim Head | Multi-Aim constraints, weights 0.3, 0.5 and 0.8 |
| Left Hand IK | Two Bone IK on the left arm, with **Left Hand Rest** (target when no grip) and **Left Elbow Hint** |

The aim axes are measured on your character, so any humanoid works. The two elbow hints are placed under the chest bone,
so they turn with the upper body: move them to change where the elbows point. Running the tool again rebuilds the rig
and keeps the hints where you put them. Every reference of `PlayerItemRig` is filled by the tool: set them by hand only
for a rig built yourself.

| PlayerItemRig field | Meaning |
|---|---|
| Held Item, Rig Builder | The `PlayerHeldItem` of the player and the `RigBuilder` of the character. Filled by *Setup Item Rig* |
| Left Hand, Left Hand Rest | Two Bone IK of the left arm, and its target when the item has no grip |
| Right Hand, Right Hand Target | Two Bone IK of the right arm and its target, used by the Weapon Pose |
| Weapon Pivot, Weapon Pose | The pivot in front of the chest, turned towards the aim, and its child where the item goes |
| Aim Bones, Aim Target | The Multi-Aim constraints of the spine, chest and head, and the point they aim at. Their weights are kept as set in the constraints |
| Grip Name | Name of the grip child, when the model has no `HeldItemPoint` of type Grip |
| Release Tag | Animator state tag that frees the left hand, on any layer. Default **Reload**: give this tag to your reload states. *Create Player Animator Controller* already does. |
| Release Parameter | Optional Animator bool that frees the hand while true |
| Blend Speed | How fast the hand and the aim blend in and out |

### Weapon pose

By default the animation moves the right hand and the item follows it (**Follow Hand**). For ranged and two-handed
weapons, the **Weapon Pose** of the Held Visual trait turns this around: the item is placed in front of the chest and
both arms bend to hold it. Move or rotate the item and the hands follow, the left hand on its grip.

| Weapon Pose | Item placed by |
|---|---|
| Follow Hand | The animation, through the right hand |
| While Aiming | The rig while aiming, towards the point under the crosshair. The animation at rest. |
| Always | The rig, with an aim pose while aiming and a rest pose otherwise |

To set it up in the Held Item Tuning window, step 4:

1. Tick **Preview Aim**, then **Capture Aim Pose**: the current animated placement becomes the aim pose, and the
   mode switches to While Aiming. For a rest pose, untick Preview Aim and **Capture Rest Pose** (mode Always).
2. Tick **Edit Pose**, then **Move** or **Rotate** the Weapon Pose in the Scene view: the item, both hands and arms
   follow. **Save As Aim Pose** (with Preview Aim) or **Save As Rest Pose**.

The reload states (tag **Reload**) free both hands, so the reload animation plays as authored.

The left hand targets the grip itself, so it never lags one frame behind the item. The aim is read from any
`IAimState` of the player, implemented by `PlayerRangedWeapon`: two-handed tools keep their grip even without the
Weapons module.

!!! warning
    Never use the world prefab as the held visual: it contains a `NetworkObject` and an error is logged.

!!! tip
    Put the model prefab inside the world prefab as a child. Changing the model then updates both.
