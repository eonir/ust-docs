# Windows

Part of [UI](index.md).

## Inventory window

`InventoryWindow` opens and closes with **Tab** (or the assigned **Toggle Action**) and closes with **Escape**.

| Field | Meaning |
|---|---|
| Window | Object shown while open |
| Inventory Grid | Grid of the player inventory |
| Hotbar Grid | Grid of the hotbar taking part in drag and drop: the gameplay hotbar |
| Container Panel, Container Grid, Container Title | Side panel of an opened container |
| Close Distance Margin | An opened container closes when the player walks away further than its access range plus this margin |
| Toggle Action, Toggle Key, Close Key | Input |

While the window is open:

- the cursor is free,
- the camera does not orbit (`CameraUILock`),
- attacking, aiming and interacting are ignored,
- moving, jumping, sprinting, crouching and the hotbar keys still work.

### Slot interactions

| Action | Result |
|---|---|
| Drag to another slot | Move, merge or swap |
| Right drag, or Shift + drag | Move half the stack |
| Drop outside any UI | Drop the items in the world |
| Shift + click | Send the stack to the other side: the opened container, otherwise between inventory and hotbar |
| Right click | Use the item, for example a consumable, from the inventory or the hotbar. While a split stack follows the cursor, a right click cancels it instead. |

Every action is a request validated by the server, see [Inventory](../inventory/containers.md#moving-items).

### Opened containers

Interacting with a loot bag opens it in the window, so players take items one by one. Any container can be opened
from your own code:

```csharp
container.OpenLocally();   // shows it in the inventory window of the local player
```

Without an inventory window in the scene, a loot bag falls back to taking everything at once.

### Grids

`ContainerGridView` creates one slot per slot of its container inside a `GridLayoutGroup`, so the panel follows the
size of the container.

| Field | Meaning |
|---|---|
| Slot Prefab | The slot prefab |
| Slot Parent | Parent of the slots, this object if empty |
| Columns | Fixed column count. 0 keeps the GridLayoutGroup setting. -1 picks it from the number of slots, up to Max Auto Columns. |
| Interactable | Accepts drag and drop and clicks |

`-1` suits loot bags, whose size changes every time.

## Character stats panel

A **Stats** button in the corner of the inventory panel opens the character stats, alone at the left edge of the
screen, so it never pushes the other panels. Move it freely: it is outside any layout group. *Create Game UI* adds it; for a UI built before, run *Tools > UST > Advanced > UI > Add Stats Panel*.

The panel lists what the game has, by section:

| Section | Lines | From |
|---|---|---|
| Vitals | Every vital, current / max | `PlayerVitals` |
| Defense | Armor, then each damage type resisted or weak: *Cold 50 % resist*, *Fire 30 % weak*, *Poison Immune* | `PlayerEquipment`, and the resistances of every `IResistanceSource` multiplied by type (natural resistances of `PlayerDamageReceiver`, worn items) |
| Movement | Speed from equipment, when not 100 % | `PlayerEquipment` |
| Inventory | Weight, when the weight limit is used | `PlayerInventory` |
| Environment | Weather where the player stands, felt temperature and stage: *Snow*, *4 °C Cold* | `WeatherExposure`, `PlayerTemperature` |

| Field | Default | Meaning |
|---|---|---|
| Open By Default | on | Shows the panel when the inventory opens, until the player hides it with the **Stats** button |
| Refresh Interval | 0.5 | Seconds between two updates while open |
| Section Order | Vitals, Environment, Defense, Movement, Inventory | Display order of the sections. Sections not listed come after, by name. Removing an entry does not hide its section |
| Panel, Toggle Button, Body | | The panel shown and hidden, the **Stats** button of the inventory that toggles it, and the text that lists the stats. Filled by *Add Stats Panel* |
| Section Color, Resist Color, Weak Color | gold, green, red | Section titles, resistances above 0 % and weaknesses below 0 % |
| Value Column | 60 | Position of the values, in percent of the width |

Add your own lines from any component of the player: implement `IStatProvider` (UST.Core) and add `StatLine`s with a
section, a label, a value and an order. A level module adds a "Level" section, a skill module its skills.

The sections are plain names, given by the modules:

| Section | Filled by |
|---|---|
| Vitals | Health, hunger, thirst, stamina… |
| Environment | `WeatherExposure` (weather where the player stands), `PlayerTemperature` (felt temperature and stage) |
| Defense | Armor and resistances: `PlayerEquipment`, `PlayerDamageReceiver` |
| Movement | Speed: `PlayerEquipment` |
| Inventory | Weight: `PlayerInventory` |

The names are constants of `StatSections` (UST.Core). A section of your own just needs a new name; add it to
Section Order to place it.

```csharp
public class PlayerLevel : MonoBehaviour, IStatProvider
{
    public int level = 1;

    public void CollectStats(List<StatLine> lines)
    {
        lines.Add(new StatLine("Level", "Level", level.ToString()));
    }
}
```

## Gamepad cursor

`GamepadCursor`, on the **UST UI** canvas, shows a cursor moved by the left stick whenever a window is open
(`UIInputFocus`) and the player uses a gamepad. It drives a virtual mouse, so every window made for the mouse works
with the gamepad: A clicks and drags, X right-clicks, the right stick scrolls. The character does not move meanwhile.
*Create Game UI* and *Tools > UST > Advanced > Input > Assign Input Actions* add it.

| Field | Default | Meaning |
|---|---|---|
| Cursor Sprite, Cursor Color, Cursor Size | dot, gold, 22 | Look of the cursor. A sprite is pointed by its top left corner. |
| Speed | 1100 | Pixels per second at full deflection, on a 1080p screen |
| Scroll Speed | 45 | Scrolling with the right stick |

Your own window only has to call `UIInputFocus.Push()` when it opens and `Pop()` when it closes.
