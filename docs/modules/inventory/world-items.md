# Items in the world

Part of [Inventory](index.md).

## Items in the world

### World prefab

A world prefab is what lies on the ground:

- root: `NetworkObject`, `WorldItem`, a collider,
- child: the model prefab.

Add it to the **Network Prefabs** list of the NetworkManager and assign it to **World Prefab** on the item.

| WorldItem field | Default | Meaning |
|---|---|---|
| Database | | The item database |
| Placed Item, Placed Count | | Content of an object placed in the scene or spawned by an `ObjectSpawner`. Ignored for dropped items. |
| Pickup Range | 3 | Max distance the server accepts for a pick up |
| Prompt Format, Stack Prompt Format | Pick up {0}, Pick up {0} x{1} | Interaction text. {0} is the item name, {1} the quantity. |
| Snap To Ground | On | Dropped items are placed on the ground below the drop point instead of floating |
| Ground Layers | Everything | Layers the drop raycast hits |
| Max Snap Distance | 20 | How far below the drop point the ground is searched |
| Random Yaw | On | Dropped items get a random rotation around the vertical axis |

Players pick items up with the Interact input. The server checks the distance and adds what fits: the rest stays on
the ground.

!!! note "Two ranges"
    `PlayerInteractor.Range` is what the player can target, on the client. `WorldItem.Pickup Range` is what the server
    accepts, against cheating. Keep the server range larger, 0.5 to 1 m more, to absorb latency and the difference of
    measurement.

### Dropping

`RequestDrop(slot, amount)` on a container drops items in front of it. The item needs a world prefab.

The dropped item is placed on the ground, its collider resting on it. To make items fall and roll instead, add a
`Rigidbody` to the world prefab: ground snapping is then skipped.

### Items placed in the scene

Drop the world prefab in the scene and set **Placed Item** and **Placed Count**. Once picked up, it never comes back,
and the save remembers it. For items that come back, use an [object spawner](../gathering.md#object-spawners).

## Death drop

`PlayerDeathDrop` moves items into a single loot bag when the player dies.

| Field | Default | Meaning |
|---|---|---|
| Mode | Drop All | See below |
| Categories | | Categories used by Drop Only and Drop All Except |
| Drop Inventory, Drop Hotbar | on | Containers concerned |
| Bag Prefab | | The loot bag prefab |
| Owner Only Duration | 60 | Seconds during which only the dead player can loot the bag. 0: anyone right away. The owner is also matched by their save id, so they keep access after reconnecting. |
| Bag Lifetime | 600 | Seconds before the bag despawns. 0: never. |
| Bag Offset | 0, 0.5, 0 | Position of the bag relative to the player |

| Mode | Drops |
|---|---|
| Drop All | Everything |
| Drop None | Nothing |
| Drop Only | Items of the listed categories |
| Drop All Except | Everything except the listed categories |

### Loot bag prefab

Root: `NetworkObject`, `InventoryContainer` (same database as the player, access range 3), `LootBag`, a collider and
a model. Add it to the Network Prefabs list.

| LootBag field | Default | Meaning |
|---|---|---|
| Container | | The `InventoryContainer` of the bag |
| Prompt | Loot | Interaction text |
| Empty Check Interval | 1 | Seconds between two checks on the server: an empty bag despawns at the next one |

The bag is created with exactly as many slots as needed, and despawns when empty or at the end of its lifetime.
Interacting with it opens it in the [inventory window](../ui/windows.md#opened-containers), where items are taken one by one.
Without an inventory window, interacting takes everything that fits.

!!! note
    A bag restored from a save is no longer locked for its owner: player ids are not kept with the bag.

## Starting kits

`StartingKit` (*Create > UST > Inventory > Starting Kit*) is a list of entries:

| Entry field | Meaning |
|---|---|
| Item, Count | What to give |
| Target | Hotbar or Inventory |
| Slot | Preferred slot. -1: the first free one. A taken slot or a count above one stack overflows to free slots. |

`PlayerStartingKit`, on the player root:

| Field | Meaning |
|---|---|
| Starting Kit | Given once, to players without a save |
| Respawn Kit | Given on every respawn. Empty: nothing. |

With the Save module, a returning player never gets the starting kit again. Without it, the kit is given on every
join. To test the starting kit again, delete the player file from the save folder.

!!! tip
    The respawn kit adds to what the player still has. Keep it consistent with your death drop rules.

## Debug HUD

The real UI is described in [UI](../ui/index.md). `InventoryDebugHUD` is an IMGUI overlay for testing: remove it when you use
the inventory window, both use Tab.

| Key | Action |
|---|---|
| Tab | Open or close the inventory |
| Click a slot, then another | Move the stack |
| Shift + click | Move half the stack |
| G | Drop one item of the selected hotbar slot |

| Field | Default | Meaning |
|---|---|---|
| Inventory | | The `PlayerInventory` shown |
| Toggle Key | Tab | Opens and closes the overlay |
| Drop Key | G | Drops one item of the selected hotbar slot |
| Slot Size, Columns | 64, 5 | Size of a slot in pixels and slots per row |

A **Give** panel lists every item of the database, in the editor and development builds only. The HUD is drawn with
IMGUI and is not meant to ship.

## Inspecting at runtime

In Play Mode, the inspectors of `InventoryContainer` and `WorldItem` show a **Runtime** section with the live
content of each slot. Select the Player 2 in its Multiplayer Play Mode window to see what a client received.
