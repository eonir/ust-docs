# Inventory

The Inventory module holds the items: their definitions, the slot containers, the hotbar, the item in the hand, the
items lying on the ground, the loot bag dropped on death and the starting kits. The server owns every container:
clients only send requests, so items cannot be duplicated.

This module is described on several pages:

- [Containers and hotbar](containers.md)
- [Item in the hand](held-item.md)
- [Items in the world](world-items.md)

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

`ItemDatabase` (*Create > UST > Inventory > Item Database*) lists every item of the game. Items created or
duplicated from the **Item Database window** are added to it automatically. To fill it from existing assets, right
click it and choose **Collect All Items In Project**.

### Item Database window

*Tools > UST > Item Database* shows every item of the project:

- search by name, filter by category, or show only the items with issues,
- **+ New Item**, **Duplicate** (the copy gets its own id) and **Delete** (to the system trash, after a confirmation),
- the item inspector on the right, traits included,
- a warning icon in the list and the details at the top of the item: not in the database, no icon, no display name,
  weapon or tool without Held Visual, no world prefab, same display name as another item, and the warnings of each
  trait (a recipe ingredient missing from the database, for example),
- a right click on an item to add it to the database or remove it.

The window remembers the selected database. The status bar counts the items, those in the database and those with
issues.

!!! warning "One database for everyone"
    Items travel over the network as their index in the database, so every peer must use the same database. Saves
    use the item id instead, so reordering or growing the database never breaks a save.

### Categories

`ItemCategory` (*Create > UST > Inventory > Item Category*) is a tag with a name, an icon and a color: Weapon,
Resource, Ammo, Tool, Food... An item can have several. Categories are used by the death drop and will be used by UI
filters and chests.

A category is only a label: it never changes what the item does. That is the job of traits.

### Traits

A trait is a capability: shown in the hand, swung, used as a tool, eaten... Add them with the **Add Trait** button at
the bottom of the item inspector. Each trait can be added once per item.

| Trait | Module | Role |
|---|---|---|
| Visual > Held Visual | Inventory | Model shown in the hand |
| Combat > Melee Swing | Inventory | Lets the item be swung with the attack input |
| Combat > Melee Weapon | Weapons | Damage dealt to characters |
| Combat > Ranged Weapon | Weapons | Shots, ammo and aiming, see [Combat](../combat/ranged.md#ranged-weapons) |
| Gathering > Tool | Gathering | Tool type, tier, yield and wear for resource nodes |
| Effects > Consumable | Effects | Instant vital changes and effects when used, see [Effects](../effects.md#consumables) |
| Equipment > Equippable | Equipment | Wearable in a slot: armor, resistances, effects, model, see [Equipment](../equipment.md) |
| Building > Structure Damage | Building | Damage dealt to built pieces per hit, see [Building](../building/rules.md#piece-health) |
| Building > Build Tool | Building | Makes the item a build tool, see [Building](../building/index.md#build-tool) |
| Crafting > Recipe | Crafting | Ingredients, output count, station and craft time, see [Crafting](../crafting.md#recipe-trait) |

An axe that both chops trees and fights has **Melee Swing**, **Tool** and **Melee Weapon**. A sword only has
**Melee Swing** and **Melee Weapon**.

Create your own traits: see [Create an item trait](../../guides/create-item-trait.md).
