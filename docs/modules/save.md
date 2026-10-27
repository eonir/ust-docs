# Save

Saves run on the server only. Clients never write their own save, so they cannot give themselves items.

## Setup

- In the scene: `SaveManager` and `WorldItemsSaver`.
- On the player root: `PlayerSaveIdentity`.

## What is saved

| Data | Component |
|---|---|
| Player position | `PlayerSaveIdentity` through `PlayerLife` |
| Vitals | `PlayerVitals` |
| Inventory and hotbar | `PlayerInventory` |
| Dropped items, loot bags, items placed in the scene | `WorldItemsSaver` |

Files are JSON, in `persistentDataPath/Saves/<slot>/world.json` and `players/<player id>.json`. Use **Open Save
Folder** on the SaveManager inspector to find them.

## When it saves

- every **Autosave Interval** seconds,
- when a client disconnects (its own file),
- when the server shuts down and when the application quits,
- when you call `SaveAll()`.

The world is loaded when the server starts. Call `SetSlot` before starting the server to use another slot.

## Player identity

The server recognizes a returning player by an id the client sends when it joins. By default, `LocalPlayerIdentity`
generates a random id once and keeps it in PlayerPrefs. It is simple but not secure, and a player who changes
computer starts again. See [Use Steam ids for saves](../guides/steam-identity.md).

## Saving your own data

Implement `IPlayerSaveable` on a player component, or `IWorldSaveable` on a scene component:

```csharp
public class PlayerLevel : NetworkBehaviour, IPlayerSaveable
{
    [Serializable] private class Data { public int level; }

    private int currentLevel;

    public string SaveKey => "level";

    public string CaptureState() => JsonUtility.ToJson(new Data { level = currentLevel });

    public void RestoreState(string json) => currentLevel = JsonUtility.FromJson<Data>(json).level;
}
```

!!! warning
    Never change a `SaveKey` once your game is released, or existing saves lose that data.
