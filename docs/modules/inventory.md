# Inventory

The Inventory module holds the items: their definitions, the slot containers, the hotbar, the item in the hand, the
items lying on the ground, the loot bag dropped on death and the starting kits. The server owns every container:
clients only send requests, so items cannot be duplicated.

## Items

### Item definition

Create one with *Create > UST > Inventory > Item Definition*.

| Field | Meaning |
|---|---|
| Display Name, Icon, Description | Shown in the UI |
| Categories | Tags used to sort, filter and choose what drops on death |
| Max Stack | Maximum quantity in one slot |
| Weight | Weight of one unit, used when the weight limit is enabled |
| Size | Reserved for a future grid inventory. Slot inventories ignore it. |
| Has Durability, Max Durability | The item wears out. Items with durability never stack. |
| World Prefab | The prefab spawned when the item is dropped |
| Traits | What the item does, see below |

The item id used by saves is the asset GUID: renaming or moving the asset does not break saves.

### Item database

`ItemDatabase` (*Create > UST > Inventory > Item Database*) lists every item of the game. Right click it and choose
**Collect All Items In Project** to fill it.

!!! warning "One database for everyone"
    Items travel over the network as their index in the database, so every peer must use the same database. Saves
    use the item id instead, so reordering or growing the database never breaks a save.

### Categories

`ItemCategory` (*Create > UST > Inventory > Item Category*) is a tag with a name, an icon and a color: Weapon,
Resource, Ammo, Tool, Food... An item can have several. Categories are used by the death drop and will be used by UI
filters, chests and crafting recipes.

A category is only a label: it never changes what the item does. That is the job of traits.

### Traits

A trait is a capability: shown in the hand, swung, used as a tool, eaten... Add them with the **Add Trait** button at
the bottom of the item inspector. Each trait can be added once per item.

| Trait | Module | Role |
|---|---|---|
| Visual > Held Visual | Inventory | Model shown in the hand |
| Combat > Melee Swing | Inventory | Lets the item be swung with the attack input |
| Combat > Melee Weapon | Weapons | Damage dealt to characters |
| Gathering > Tool | Gathering | Tool type, tier, yield and wear for resource nodes |

An axe that both chops trees and fights has **Melee Swing**, **Tool** and **Melee Weapon**. A sword only has
**Melee Swing** and **Melee Weapon**.

Create your own traits: see [Create an item trait](../guides/create-item-trait.md).

## Containers

`InventoryContainer` is a networked list of slots. The same component is used for the player inventory, the hotbar,
loot bags and any chest you make.

| Field | Default | Meaning |
|---|---|---|
| Database | | The item database |
| Slot Count | 20 | Number of slots |
| Access Range | 3 | Max distance for a player to use a world container. Player containers are owner only. |
| Drop Distance | 1 | Distance in front of the container where dropped items appear |

### Moving items

`RequestMove(fromSlot, target, toSlot, amount)` asks the server to move items, inside a container or between two
containers. The server applies these rules:

| Target slot | Result |
|---|---|
| Empty | The items move. `amount` = 0 moves the whole stack, a smaller number splits it. |
| Same item, same durability | The items merge up to the max stack. The rest stays in place. |
| Different item | The two stacks swap. Only whole stacks can swap. |

The server also checks that the client is allowed to use both containers and that the weight limit allows the move.

### Server side helpers

| Method | Result |
|---|---|
| `Add(item, count)` | Fills existing stacks, then empty slots. Returns what did not fit. |
| `Remove(item, count)` | Removes the items, or nothing if there are not enough. |
| `RemoveAt(slot, count)` | Removes up to `count` from a slot. |
| `SetSlot(slot, stack)` | Overwrites a slot. |
| `ServerReduceDurability(slot, amount)` | Wears an item. A broken item disappears. |
| `Clear()` | Empties the container. |

Reading (`Get`, `GetDefinition`, `CountOf`, `TotalWeight`, `IsEmpty`) works on every peer. `SlotChanged` is raised
on every peer when a slot changes.

## Player inventory

On the player root:

| Component | Role |
|---|---|
| `InventoryContainer` x2 | One for the inventory (20 slots), one for the hotbar (8 slots, up to 10 number keys) |
| `PlayerInventory` | Links both containers, weight limit, selected hotbar slot |
| `HotbarInput` | Mouse wheel and number keys |
| `PlayerHeldItem` | Model of the selected item in the hand |
| `PlayerItemUser` | Swings the held item, see [Combat](combat.md) |
| `PlayerDeathDrop` | Optional: drops the items in a loot bag on death |
| `PlayerStartingKit` | Optional: starting and respawn kits |
| `InventoryDebugHUD` | Debug only |

### Adding and removing

```csharp
int wood = playerInventory.CountOf(woodItem);              // inventory + hotbar, any peer
int notAdded = playerInventory.ServerAdd(woodItem, 10);    // server only
bool removed = playerInventory.ServerRemove(woodItem, 5);  // server only
```

`ServerAdd` fills existing stacks first (hotbar, then inventory), then empty slots (hotbar, then inventory). It
respects the weight limit and returns what did not fit.

### Items received

`LocalItemsReceived(item, count)` is raised on the owner every time `ServerAdd` adds something: picking up,
looting, gathering, debug give and later crafting. The [item feed](ui.md) uses it. Starting kits and save loading do
not raise it, so the feed is not flooded when a player joins.

### Weight

Enable **Use Weight** on `PlayerInventory`.

| Field | Default | Meaning |
|---|---|---|
| Use Weight | off | Enables the limit |
| Max Weight | 50 | Limit for the inventory and the hotbar together |

- Picking up stops at the limit: a partial pick up leaves the rest on the ground.
- Moving between the inventory and the hotbar never counts, the carried weight does not change.
- `SetMaxWeight(value)` changes the limit at runtime on the server, for a backpack or a strength bonus.

### Hotbar

`HotbarInput` selects the hotbar slot:

| Field | Default | Meaning |
|---|---|---|
| Scroll Action | | Optional. Without it, the mouse wheel is read directly. |
| Invert Scroll | off | Reverses the wheel direction |
| Number Keys | on | Keys 1 to 0 select the first 10 slots. Keys are physical, so the top row works on any layout. |
| Scroll Threshold | 0.1 | Minimum wheel movement |

The selection is synchronized: other players see which item you hold. Read it with `SelectedIndex`,
`SelectedStack` and `SelectedItem`.

## Item in the hand

`PlayerHeldItem` shows the model of the selected hotbar item on every peer. Only the selection is synchronized: the
model itself is a local object, so holding an item costs no network object.

1. Create a **model prefab**: only the mesh, no `NetworkObject`.
2. On the item: **Add Trait > Visual > Held Visual**, assign the model prefab and adjust the offsets.

| Held Visual field | Meaning |
|---|---|
| Prefab | The model prefab |
| Position Offset, Rotation Offset, Scale | Placement in the hand |

| PlayerHeldItem field | Meaning |
|---|---|
| Hand Socket | Where the model is attached. Empty: the right hand bone of the humanoid Animator. |
| Animator | Found automatically if empty |

The colliders of the model are disabled, so it never blocks raycasts or pushes the character. Rigs exported in
centimetres, with a scale of 100 on their bones, are compensated automatically.

!!! warning
    Never use the world prefab as the held visual: it contains a `NetworkObject` and an error is logged.

!!! tip
    Put the model prefab inside the world prefab as a child. Changing the model then updates both.

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

Players pick items up with the Interact input. The server checks the distance and adds what fits: the rest stays on
the ground.

!!! note "Two ranges"
    `PlayerInteractor.Range` is what the player can target, on the client. `WorldItem.Pickup Range` is what the server
    accepts, against cheating. Keep the server range larger, 0.5 to 1 m more, to absorb latency and the difference of
    measurement.

### Dropping

`RequestDrop(slot, amount)` on a container drops items in front of it. The item needs a world prefab.

### Items placed in the scene

Drop the world prefab in the scene and set **Placed Item** and **Placed Count**. Once picked up, it never comes back,
and the save remembers it. For items that come back, use an [object spawner](gathering.md#object-spawners).

## Death drop

`PlayerDeathDrop` moves items into a single loot bag when the player dies.

| Field | Default | Meaning |
|---|---|---|
| Mode | Drop All | See below |
| Categories | | Categories used by Drop Only and Drop All Except |
| Drop Inventory, Drop Hotbar | on | Containers concerned |
| Bag Prefab | | The loot bag prefab |
| Owner Only Duration | 60 | Seconds during which only the dead player can loot the bag. 0: anyone right away. |
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

The bag is created with exactly as many slots as needed, and despawns when empty or at the end of its lifetime.
Interacting with it takes everything that fits. A container UI will later let players pick items one by one.

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

`InventoryDebugHUD` draws the hotbar at the bottom of the screen.

| Key | Action |
|---|---|
| Tab | Open or close the inventory |
| Click a slot, then another | Move the stack |
| Shift + click | Move half the stack |
| G | Drop one item of the selected hotbar slot |

A **Give** panel lists every item of the database, in the editor and development builds only. The HUD is drawn with
IMGUI and is not meant to ship.

## Inspecting at runtime

In Play Mode, the inspectors of `InventoryContainer` and `WorldItem` show a **Runtime** section with the live
content of each slot. Select the Player 2 in its Multiplayer Play Mode window to see what a client received.
