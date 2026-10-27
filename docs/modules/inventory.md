# Inventory

## Assets

| Asset | Menu | Role |
|---|---|---|
| `ItemDefinition` | *Create > UST > Inventory > Item Definition* | Name, icon, stack size, weight, durability, world prefab, categories, traits |
| `ItemDatabase` | *Create > UST > Inventory > Item Database* | Every item. Right click it > **Collect All Items In Project** |
| `ItemCategory` | *Create > UST > Inventory > Item Category* | Tag used for sorting and filtering |
| `StartingKit` | *Create > UST > Inventory > Starting Kit* | Items given on first join or on respawn |

!!! warning "One database"
    Items travel over the network as their index in the database, so every peer must use the same database.
    Saves use the item id instead, so reordering the database never breaks saves.

## Player setup

On the player root:

| Component | Settings |
|---|---|
| `InventoryContainer` (inventory) | Database, slot count (20) |
| `InventoryContainer` (hotbar) | Database, slot count (8, up to 10 number keys) |
| `PlayerInventory` | Both containers, optional weight limit |
| `HotbarInput` | Mouse wheel and number keys |
| `PlayerHeldItem` | Shows the selected item in the hand |
| `PlayerItemUser` | Swings the held item with the attack input |
| `PlayerDeathDrop` | Optional, drops items in a loot bag on death |
| `PlayerStartingKit` | Optional, starting and respawn kits |
| `InventoryDebugHUD` | Debug only |

## Categories and traits

- A **category** is a tag: it is used to sort, filter and choose what drops on death.
- A **trait** is a behaviour: held visual, tool, weapon, food... Add traits with the **Add Trait** button of the item
  inspector.

Never derive a behaviour from a category: an axe tagged *Tool* only must still be able to hit.

## Traits included

| Trait | Module | Role |
|---|---|---|
| Visual > Held Visual | Inventory | Model shown in the hand, with position, rotation and scale offsets |
| Combat > Melee Swing | Inventory | Range, radius, cooldown, impact delay, stamina cost, animation trigger |
| Combat > Melee Weapon | Weapons | Damage dealt to characters |
| Gathering > Tool | Gathering | Tool type, tier, yield, damage to nodes, wear |

An axe that both chops and fights has **Melee Swing**, **Tool** and **Melee Weapon**.

## Item in the hand

`PlayerHeldItem` shows the model of the selected hotbar item on every peer. The model is local, only the selection is
synchronized. It finds the right hand bone of a humanoid Animator, or uses the **Hand Socket** you assign. It
compensates the scale of rigs exported in centimetres.

!!! warning
    The held visual must be the **model prefab**, never the world prefab: it must not contain a `NetworkObject`.

## Weight

Enable **Use Weight** on `PlayerInventory`. The limit covers the inventory and the hotbar together. Picking up
stops when the limit is reached, and partial pick ups leave the rest on the ground. Change the limit at runtime
with `SetMaxWeight` on the server.

## Items in the world

A world prefab has a `NetworkObject`, a `WorldItem` and a collider on its root, and its model as a child. To place
an item in a scene, drop its world prefab and set **Placed Item** and **Placed Count**. Players pick it up with the
Interact input.

## Death drop

`PlayerDeathDrop` moves items into a single loot bag on death:

| Mode | Drops |
|---|---|
| Drop All | Everything |
| Drop None | Nothing |
| Drop Only | Items of the listed categories |
| Drop All Except | Everything but the listed categories |

**Owner Only Duration** keeps the bag for its owner for a while (0: anyone can loot right away). **Bag Lifetime**
despawns the bag (0: never).

## Scripting

```csharp
int wood = playerInventory.CountOf(woodItem);
int notAdded = playerInventory.ServerAdd(woodItem, 10);   // server only
bool removed = playerInventory.ServerRemove(woodItem, 5); // server only

container.RequestMove(fromSlot, otherContainer, toSlot);  // any client, validated by the server
container.RequestDrop(slot, 1);

if (item.TryGetTrait(out HeldVisualTrait visual)) { }
```
