# Containers and hotbar

Part of [Inventory](index.md).

## Containers

`InventoryContainer` is a networked list of slots. The same component is used for the player inventory, the hotbar,
loot bags and any chest you make.

| Field | Default | Meaning |
|---|---|---|
| Database | | The item database |
| Slot Count | 20 | Number of slots |
| Access Range | 3 | Max distance for a player to use a world container. Player containers are owner only. The host player follows the same rules as everyone, and wards can restrict world containers, see [Building](../building/ward.md#ward). |
| Drop Distance | 1 | Distance in front of the container where dropped items appear |
| Take Only | off | Players can only take items out, as in the output of a crafting station. Server code can still add items. |

### Moving items

`RequestMove(fromSlot, target, toSlot, amount)` asks the server to move items, inside a container or between two
containers. The server applies these rules:

| Target slot | Result |
|---|---|
| Empty | The items move. `amount` = 0 moves the whole stack, a smaller number splits it. |
| Same item, same durability | The items merge up to the max stack. The rest stays in place. |
| Different item | The two stacks swap. Only whole stacks can swap. |

The server also checks that the client is allowed to use both containers and that the weight limit allows the move.

| Request | Result |
|---|---|
| `RequestDrop(slot, amount)` | Drops items in front of the container. 0 drops the whole stack. |
| `RequestSort()` | Merges the stacks that can stack and orders them by Item Database index, the most durable first, without gaps. Containers with a `SlotFilter`, such as the equipment, are left as they are. |
| `RequestTakeAll()` | Moves everything in a world container to the local player, as `ServerAdd` does: hotbar stacks first, then the inventory, within the weight limit. What does not fit stays. |

### Server side helpers

| Method | Result |
|---|---|
| `Add(item, count)` | Fills existing stacks, then empty slots. Returns what did not fit. |
| `Remove(item, count)` | Removes the items, or nothing if there are not enough. |
| `RemoveAt(slot, count)` | Removes up to `count` from a slot. |
| `SetSlot(slot, stack)` | Overwrites a slot. |
| `ServerReduceDurability(slot, amount)` | Wears an item. A broken item disappears. |
| `Clear()` | Empties the container. |
| `ServerSort(clientId)`, `ServerTakeAll(clientId)` | Same as the requests above, for a given player, with the same access checks. |

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
| `PlayerItemUser` | Swings the held item, see [Combat](../combat/index.md) |
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
looting, gathering, debug give and later crafting. The [item feed](../ui/index.md) uses it. Starting kits and save loading do
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

`HotbarInput` selects the hotbar slot with the number keys, like Valheim. By default the mouse wheel zooms the
camera instead, see [Player > Camera zoom](../player/camera.md#camera-zoom).

| Field | Default | Meaning |
|---|---|---|
| Scroll Selects | off | Lets the mouse wheel change the selected slot. Leave it off when the wheel zooms the camera. |
| Scroll Action | | Optional. Without it, the mouse wheel is read directly. |
| Invert Scroll | off | Reverses the wheel direction |
| Number Keys | on | Keys 1 to 0 select the first 10 slots. Keys are physical, so the top row works on any layout. |
| Scroll Threshold | 0.1 | Minimum wheel movement |
| Previous Action, Next Action | Player/Previous, Player/Next | Select the previous or next slot: LB and RB on a gamepad |

A key on an item that an `IItemUseHandler` of the player accepts uses it instead of selecting it, and the held item
stays in hand. With the Effects module, a key on a consumable eats or drinks it, see
[Effects > Consumables](../effects.md#consumables). Handlers that also implement `IInventoryOnlyUseHandler` are skipped by the hotbar keys: armor in the hotbar is selected by its key and equipped only with a right click in the inventory.

The selection is synchronized: other players see which item you hold. Read it with `SelectedIndex`,
`SelectedStack` and `SelectedItem`.
