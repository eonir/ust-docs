# Editor Tools

## Menus

The **UST Tools** menu keeps what you use often. Actions on one object are in its right click menu, and the rest is
under **Advanced**.

| Menu | What it does |
|---|---|
| *UST Tools > Setup Window* | The entry point, see [Setup Window](setup-window.md): each module of the project with what it has in the open scene and on the player prefab, a button to add what is missing, the demo, the player setup and the main tools. Opens once on the first import. |
| *UST Tools > Create Full Survival Demo* | Sets up a playable demo of every module in the open scene, see [Build the demo scene](../guides/demo-scene.md) |
| *UST Tools > Setup Player Prefab* | Adds to `Player.prefab` the components of every module in the project: building, hand crafting, recipe book, equipment, temperature, weather exposure, bed respawn, vehicles. Each one only when missing. |
| *UST Tools > Create Game UI* | Creates the whole game UI, with the stats panel, and the equipment panel, the crafting panel and the build menu when their modules are present, see [UI](../modules/ui.md) |
| *UST Tools > Item Database* | Window to create, find and edit every item, see [Inventory](../modules/inventory.md#item-database-window) |
| *UST Tools > Held Item Tuning* | Play Mode window to place the item in the hand, set the left hand grip and correct the aim, see [Item in the hand](../modules/inventory.md#placing-the-item-in-the-hand) |
| *UST Tools > Validation > Validate Scene* | Lists every setup problem of the open scene and the player prefab, module by module, with a fix button when possible, see [Validate Scene](validate-scene.md) |
| *UST Tools > Validation > Validate Network Prefabs* | Lists the network prefabs missing from the NetworkManager and adds them in one click |
| *UST Tools > Validation > Check Model Orientation* | Lists the meshes of the models in `Assets/UST` that the Asset Store validator reports as badly oriented |

### Right click menus

| Right click | Menu | What it does |
|---|---|---|
| An object in the Hierarchy | *UST > Make Crafting Station* | Adds a NetworkObject, an output container and `CraftingStation`, see [Crafting](../modules/crafting.md) |
| An object in the Hierarchy | *UST > Add Fuel Burner* | Adds a `FuelBurner` (campfire, furnace, brazier) and links the lights, heat source and particles shown while it burns, see [Fire and fuel](../modules/crafting.md#fire-and-fuel) |
| An object in the Hierarchy | *UST > Add World Marker* | Shows the object on screen with an icon and its distance, see [Markers](../modules/markers.md) |
| A character in the Hierarchy | *UST > Setup Item Rig* | Creates the Animation Rigging rig for the left hand and the upper body aim, see [Item rig](../modules/inventory.md#item-rig) |
| Anywhere in the Hierarchy | *UST > Bed*, *UST > Trader*, *UST > Weather Zone* | Places a bed, a trader with the first Shop Catalog of the project, or a weather zone registered on the director |
| A model, in the Hierarchy or the Project window | *UST > Create Enemy From Model*, *UST > Create Vehicle From Model* | Makes a network enemy or vehicle prefab from the model |
| An enemy prefab | *UST > Create Enemy Definition* | Creates an Enemy Definition and assigns it |
| An Animator Controller in the Project window | *UST > Add Upper Body Layer* | Adds the missing parameters, an upper body Avatar Mask and the UpperBody layer |

### Advanced

*UST Tools > Advanced* holds one submenu per module, with each step the Setup window and the demo run, for when you
want only one of them: add a module to the scene, add one component to the selected player, rebuild one panel of the
UI, bake the map, create the default vitals, vehicles or demo pieces, the demo island, the vehicle test scene, the
Animator Controller of the player, the default UI layout, or assign the input actions again.

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

## Validate Scene

*UST Tools > Validation > Validate Scene* lists every setup problem of the open scene and the player prefab, module by
module, with a fix button when possible. Every check, its level and its fix are described in
[Validate Scene](validate-scene.md), with how to add your own checks.

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
| Player Equipment | In Play Mode, the item worn in each slot, the total armor and the total resistances by damage type |
| Player Held Item | The held item in Play Mode, and **Open Held Item Tuning** |
| Day Night Cycle | **Time Of Day** slider, in edit mode to preview the light at any hour, in Play Mode to jump the world to that hour on the server. Also the day number. See [Weather > Testing an hour](../modules/weather.md#testing-an-hour) |
| Weather Director | In Play Mode, the weather shown on this screen, the weather of the world and of each zone with the time left, and a button per weather to force it (host or server). See [Weather > Testing a weather](../modules/weather.md#testing-a-weather) |
| Weather Exposure | In Play Mode, whether the player is under a roof, and the weather of the region they stand in with its time left and its force buttons |

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
