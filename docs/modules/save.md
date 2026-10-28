# Save

The Save module writes the world and the players to JSON files. It runs on the server only: clients never write
their own save, so they cannot give themselves items by editing a file. The world belongs to the host, and so do
the characters played on it.

## Setup

| Where | Component |
|---|---|
| Scene | `SaveManager` |
| Scene | `WorldItemsSaver` (items on the ground, loot bags, items placed in the scene) |
| Scene | `ResourceNodesSaver` (resource nodes placed in the scene), with the Gathering module |
| Player root | `PlayerSaveIdentity` |

Object spawners save themselves, nothing to add for them.

## Save Manager

| Field | Default | Meaning |
|---|---|---|
| Slot Name | Default | Folder of the save. Use one slot per world. |
| Autosave Interval | 300 | Seconds between two automatic saves. 0 disables autosave. |
| Load On Server Start | on | Loads the slot when the server starts |
| Identity Provider | empty | Component that identifies players, see below. Empty: local id. |

The inspector also shows:

- the **path** of the slot, selectable to copy it,
- **Open Save Folder**, which opens it in the file explorer,
- **Delete This Slot**, after a confirmation, disabled in Play Mode,
- **Save Now**, enabled in Play Mode on the host.

## What is saved

| Data | Saved by |
|---|---|
| Player position and rotation | `PlayerSaveIdentity`, restored through `PlayerLife` |
| Vitals | `PlayerVitals` |
| Inventory and hotbar | `PlayerInventory` |
| Dropped items and loot bags | `WorldItemsSaver` |
| Items placed in the scene: picked up or partially picked up | `WorldItemsSaver` |
| Resource nodes placed in the scene: health, depletion, regrow time | `ResourceNodesSaver` |
| Objects of each spawner and their pending respawns | `ObjectSpawner` |

## Files

```
persistentDataPath/Saves/<slot>/
├── world.json
└── players/
    ├── <player id>.json
    └── ...
```

Files are readable JSON with a version number, so old saves can be migrated when the format changes. Each file is
first written to a temporary file then copied over the old one, so a crash during a save cannot corrupt it.

## When it saves

- every **Autosave Interval** seconds,
- when a client disconnects: its own file only,
- when the server shuts down and when the application quits,
- when you call `SaveAll()` or press **Save Now**.

The world is loaded one frame after the server starts, so objects placed in the scene are spawned before they are
restored. To use another slot, call `SetSlot("MyWorld")` before starting the server. `SlotExists("MyWorld")` tells
whether a slot has a save, for a "Continue" button.

`Saved` and `Loaded` events are raised on the server.

### Items placed in the scene

The world file remembers which items placed in the scene were picked up. When it is loaded, only those are removed:
an item added to the scene after the save was made, for example in an update of your game, appears with its placed
content.

!!! note "Saves made before this change"
    They do not hold the list of picked up items, so a scene item picked up before the update comes back once.

### Ammo

Rounds loaded in a weapon are part of the item stack, so they are kept in the inventory, on the ground, in loot bags
and in saves.

## Player identity

The server must recognize a returning player to give back their character. The network client id changes on every
connection, so the client sends a stable id when it joins, and the server loads the matching file.

`IPlayerIdentityProvider` decides that id:

| Provider | Id | Notes |
|---|---|---|
| `LocalPlayerIdentity` (default) | Random id created once, kept in PlayerPrefs | Works with no setup. A player who changes computer or clears PlayerPrefs starts again. Each Multiplayer Play Mode virtual player gets its own id. |
| Your own | Steam id, Unity Authentication id... | See [Use Steam ids for saves](../guides/steam-identity.md) |

!!! warning "Trust"
    The id is sent by the client, so a modified client could claim somebody else's id. That is acceptable between
    friends. For public servers, verify the id on the server with the platform authentication.

Two connected players with the same id is detected: the second one is neither loaded nor saved, and a warning is
logged.

## Saving your own data

### Player data

Implement `IPlayerSaveable` on a component of the player object:

```csharp
using System;
using Unity.Netcode;
using UnityEngine;
using UST.Core;

public class PlayerLevel : NetworkBehaviour, IPlayerSaveable
{
    [Serializable]
    private class Data
    {
        public int level;
    }

    private int currentLevel;

    public string SaveKey => "level";

    public string CaptureState() => JsonUtility.ToJson(new Data { level = currentLevel });

    public void RestoreState(string json) => currentLevel = JsonUtility.FromJson<Data>(json).level;
}
```

Both methods are called on the server. `RestoreState` is called after the player spawned, when its save was found.

### World data

Implement `IWorldSaveable` on a component of the scene. It works the same way, with the whole world file.

!!! warning "Save keys"
    Never change a `SaveKey` once your game is released: existing saves would lose that data. Use `JsonUtility`
    friendly data: classes and structs with public fields and arrays, no dictionaries.

## Known limits

- Loading happens when the server starts. There is no "load while playing".
- A player saved while dead comes back with 0 health and dies again on load.
- In the editor, saving when you stop Play Mode depends on the order Unity shuts things down. Autosave and **Save
  Now** are reliable.
