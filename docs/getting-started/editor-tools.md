# Editor Tools

## Menus

| Menu | What it does |
|---|---|
| *Tools > UST > Create Player Animator Controller* | Creates an Animator Controller with every parameter the scripts use, the IK pass enabled, the Locomotion and Crouch states and the UpperBody layer |
| *Tools > UST > Add Upper Body Layer To Selected Controller* | Adds the missing parameters, an upper body Avatar Mask and the UpperBody layer to the selected controller |
| *Tools > UST > Create Default Vitals* | Creates Health, Hunger, Thirst, Stamina and a profile using them, with sensible default values |
| *Tools > UST > Validate Network Prefabs* | Lists the network prefabs missing from the NetworkManager and adds them in one click |
| *Tools > UST > UI > Create Game UI* | Creates the whole game UI, see [UI](../modules/ui.md) |
| *Tools > UST > UI > Save Layout As Default* | Saves the placement of the current UI as the default one, after a confirmation |
| *Tools > UST > UI > Reset Default Layout* | Goes back to the original layout, after a confirmation |
| *Tools > UST > Validation > Check Model Orientation* | Lists the meshes of the models in `Assets/UST` that the Asset Store validator reports as badly oriented |

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
| Save Manager | Save path, **Open Save Folder**, **Delete This Slot**, **Save Now** |
| Player Held Item | In Play Mode, **Select Held Model**, **Save Placement To Item**, **Select Grip** and **Save Grip To Item**, see [Item in the hand](../modules/inventory.md#item-in-the-hand) |

UI sprites placed in `Assets/UST/UI/Sprites` are imported as sprites automatically, with a 9-slice border taken from a
`_b12` suffix in their name.

## Asset Store Publishing Tools

If you build your own package on top of UST, Unity's *Asset Store Publishing Tools* validator reports common issues
early: Mixamo content, JPG textures, models with a wrong orientation, prefabs not at the origin, missing
documentation. Run it regularly rather than only before submitting.

### Check Model Orientation

The validator reports a model when one of its meshes has a rotation inside the imported model, even if the object
looks right in the scene. *Tools > UST > Validation > Check Model Orientation* applies the same rule and logs each
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
