# Editor Tools

## Menus

| Menu | What it does |
|---|---|
| *UST Tools > Item Database* | Window to create, find and edit every item, see [Inventory](../modules/inventory.md#item-database-window) |
| *UST Tools > Held Item Tuning* | Play Mode window to place the item in the hand, set the left hand grip and correct the aim, see [Item in the hand](../modules/inventory.md#placing-the-item-in-the-hand) |
| *UST Tools > Player > Create Animator Controller* | Creates an Animator Controller with every parameter the scripts use, the IK pass enabled, the Locomotion and Crouch states and the UpperBody layer |
| *UST Tools > Player > Add Upper Body Layer To Selected Controller* | Adds the missing parameters, an upper body Avatar Mask and the UpperBody layer to the selected controller |
| *UST Tools > Player > Setup Item Rig On Selected Character* | Creates the Animation Rigging rig for the left hand and the upper body aim, see [Item rig](../modules/inventory.md#item-rig) |
| *UST Tools > Vitals > Create Default Vitals* | Creates Health, Hunger, Thirst, Stamina and a profile using them, with sensible default values |
| *UST Tools > Crafting > Make Selected A Crafting Station* | Adds a NetworkObject, an output container and `CraftingStation` to the selected object |
| *UST Tools > Crafting > Add Hand Crafting To Selected Player* | Adds `PlayerCrafter` to the player, for the recipes without a station |
| *UST Tools > UI > Create Game UI* | Creates the whole game UI, with the crafting panel when the Crafting module is present, see [UI](../modules/ui.md) |
| *UST Tools > UI > Add Crafting Panel* | Adds or rebuilds the crafting panel next to the container panel of an existing inventory window, see [Crafting](../modules/crafting.md) |
| *UST Tools > UI > Save Layout As Default* | Saves the placement of the current UI as the default one, after a confirmation |
| *UST Tools > UI > Reset Default Layout* | Goes back to the original layout, after a confirmation |
| *UST Tools > Validation > Validate Network Prefabs* | Lists the network prefabs missing from the NetworkManager and adds them in one click |
| *UST Tools > Validation > Check Model Orientation* | Lists the meshes of the models in `Assets/UST` that the Asset Store validator reports as badly oriented |

## Create Default Vitals

Asks for a folder, then creates four vitals and a profile using them, ready to assign to `PlayerVitals`:

| Asset | Settings |
|---|---|
| Vital_Health | Server, red, regenerates 1 every 2 s after 5 s, if hunger and thirst are at 30 % or more |
| Vital_Hunger | Server, orange, -1 every 10 s, drains 1 health every 2 s when empty |
| Vital_Thirst | Server, blue, -1 every 7 s, drains 1 health every 2 s when empty |
| Vital_Stamina | Owner, yellow, regenerates 2 every 0.1 s after 1 s |
| VitalsProfile | The four vitals, enabled |

It never overwrites anything: assets with the same name get a number.

## Validate Network Prefabs

A client can only spawn a network prefab registered in the NetworkManager. A missing prefab works for the host but
fails on clients with *NetworkPrefab could not be found*.

The validator follows every object reference of:

- the components of the open scenes (spawners, savers...),
- the player prefab and its components (the loot bag...),
- every item definition of the project (world prefabs),

then every network prefab it finds, in case it references others. Every prefab with a `NetworkObject` on its root
that is not registered is listed. The Console shows which object references it: click the message to select it.
Empty or broken entries of the lists, such as deleted prefabs, are reported too.

Run it after adding items, spawners or prefabs, and restart the Multiplayer Play Mode players afterwards.

## Inspectors

| Component | Extra in the inspector |
|---|---|
| Item Definition | **Add Trait** menu, listing every trait of the project, and a foldout per trait |
| Inventory Container | In Play Mode, the live content of each slot and the total weight |
| World Item | In Play Mode, the live stack |
| Save Manager | Save path, **Open Save Folder**, **Delete This Slot**, **Save Now**. With **Clear Slot In Editor**, the slot is deleted on every Play and Stop. |
| Effect Definition | **Add Modifier** menu, listing every modifier of the project |
| Effect Database | Context menu **Collect All Effects In Project** |
| Player Effects | In Play Mode, the active effects with **Remove**, and **Apply** to give any effect |
| Player Held Item | The held item in Play Mode, and **Open Held Item Tuning** |

UI sprites placed in `Assets/UST/UI/Sprites` are imported as sprites automatically, with a 9-slice border taken from a
`_b12` suffix in their name.

## Asset Store Publishing Tools

If you build your own package on top of UST, Unity's *Asset Store Publishing Tools* validator reports common issues
early: Mixamo content, JPG textures, models with a wrong orientation, prefabs not at the origin, missing
documentation. Run it regularly rather than only before submitting.

### Check Model Orientation

The validator reports a model when one of its meshes has a rotation inside the imported model, even if the object
looks right in the scene. *UST Tools > Validation > Check Model Orientation* applies the same rule and logs each
mesh with its path inside the model and its rotation. Click a message to select the model.

| Logged rotation | Cause | Fix |
|---|---|---|
| (270, 0, 0) on every mesh | Blender's axis conversion, written as a rotation | Enable **Bake Axis Conversion** in the model import settings, or export from Blender with **Apply Transform** |
| A mesh under a bone (eyes, hair, props) | The mesh is parented to a bone instead of being skinned | In Blender, unparent it, weight it to the bone with a vertex group of the same name, then parent it with **Armature Deform** |

After a re-export of a humanoid character, apply **Humanoid** again in the Rig tab, or the character collapses.

### Static variables

The validator lists every script with static fields or events. This check does not detect whether they are reset.
UST resets all of them when Play Mode starts, so the template works with *Enter Play Mode Options* without domain
reload, which is the default from Unity 6.6. Do the same in your own scripts:

```csharp
[RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.SubsystemRegistration)]
private static void ResetStatics() => Instance = null;
```
