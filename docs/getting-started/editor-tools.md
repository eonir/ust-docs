# Editor Tools

## Menus

| Menu | What it does |
|---|---|
| *Tools > UST > Create Player Animator Controller* | Creates an Animator Controller with every parameter the scripts use, the IK pass enabled and the Locomotion and Crouch states |
| *Tools > UST > Create Default Vitals* | Creates Health, Hunger, Thirst, Stamina and a profile using them, with sensible default values |
| *Tools > UST > Validate Network Prefabs* | Lists the network prefabs missing from the NetworkManager and adds them in one click |
| *Tools > UST > UI > Create Game UI* | Creates the whole game UI, see [UI](../modules/ui.md) |
| *Tools > UST > UI > Save Layout As Default* | Saves the placement of the current UI as the default one, after a confirmation |
| *Tools > UST > UI > Reset Default Layout* | Goes back to the original layout, after a confirmation |

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

UI sprites placed in `Assets/UST/UI/Sprites` are imported as sprites automatically, with a 9-slice border taken from a
`_b12` suffix in their name.

## Asset Store Publishing Tools

If you build your own package on top of UST, Unity's *Asset Store Publishing Tools* validator reports common issues
early: Mixamo content, JPG textures, models with a wrong orientation, prefabs not at the origin, missing
documentation. Run it regularly rather than only before submitting.
