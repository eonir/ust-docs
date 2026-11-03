# Crafting

The Crafting module turns items into recipes, adds crafting stations (workbench, forge, cooking fire...) and crafting
by hand from the inventory. Crafts are queued and run by the server: ingredients are taken when a craft is queued and
given back when it is cancelled.

The module is optional: add `UST_DISABLE_CRAFTING` to the Scripting Define Symbols to remove it, see
[Modules](../architecture/modules.md#disabling-an-optional-module).

## Setup

1. *UST Tools > UI > Create Game UI* adds the crafting panel right after the container panel of the inventory
   window, with the same look. For a UI built before, run **UST Tools > UI > Add Crafting Panel**.
2. For crafting by hand, select the player prefab and run **UST Tools > Crafting > Add Hand Crafting To Selected
   Player**. It adds `PlayerCrafter` next to `PlayerInventory`.
3. Create the station types you need: *Create > UST > Crafting > Station Type* (Workbench, Forge...).
4. Open the station prefab, select its root and run **UST Tools > Crafting > Make Selected A Crafting Station**. It
   adds a `NetworkObject`, an output `InventoryContainer` in **Take Only** mode and `CraftingStation`. Assign the
   **Station Type**.
5. Register the station prefab in your Network Prefabs list, or place it in the scene.
6. Give items a **Crafting > Recipe** trait, see below.

## Recipe trait

An item has one recipe. A variant made from other ingredients is another item: *Wooden Arrow* and *Flint Arrow* are two
items with their own recipe.

| Field | Default | Meaning |
|---|---|---|
| Ingredients | | Items and counts taken for one craft, from the inventory and the hotbar |
| Output Count | 1 | Items made by one craft, e.g. **20** for a bundle of arrows |
| Station | none | Station type needed. Empty: crafted by hand. |
| Craft Time | 0 | Seconds per craft. **0** is instant. |

The Item Database window warns about a recipe without ingredients, an empty ingredient, an item used in its own recipe
and an ingredient missing from the database.

## Crafting station

| Field | Default | Meaning |
|---|---|---|
| Station Type | | Recipes of this type are made here |
| Database | | Optional when the station has an output: the output's database is used |
| Output | | Container where finished items wait. Empty: they go straight to the player who asked, and what does not fit is dropped at their feet. |
| Use Range | 3 | Distance to queue crafts. The window closes past it. |
| Cancel When Away | off | On: the crafts of a player who walks away are cancelled and their ingredients given back. Off: crafts carry on without the player. |
| Max Jobs | 8 | Jobs in the queue at once |
| Max Count Per Job | 99 | Most crafts one request can ask for |

The output is **Take Only**: players can take items out but never put their own in, so they cannot block the station.

When the output has no room for the next result, the queue waits and the panel shows *Output full*. Empty it by
dragging the items, Ctrl + click, or **Take all**. Stack limits are respected: a craft only finishes when all of its
items fit.

Several players can use the same station: jobs run one after the other, and each player can cancel only their own.

## Crafting window

`CraftingWindow` sits on the inventory window object and shows its panel:

- next to the inventory, with the recipes made by hand, when the inventory opens without a container
  (**Hand Crafting With Inventory**);
- when the player interacts with a station: the inventory opens with the station output in the container panel, and
  the station recipes in the crafting panel.

Recipes the player cannot afford are dimmed. The selected recipe shows each ingredient as owned / needed, green or red,
for the chosen quantity. **-** and **+** change the quantity, by 10 with Shift, **Max** picks the most the player can
afford. The queue shows each job with its progress and a cancel button.

*Create Game UI* or *UST Tools > UI > Add Crafting Panel* builds it and fills every reference. The texts can be
translated or restyled from the inspector.

| Section | Field | Default | Meaning |
|---|---|---|---|
| | Inventory Window | | The inventory window the panel opens with |
| | Panel, Title | | Object shown and hidden, and its title: the station name, or Hand Title |
| | Hand Crafting With Inventory | On | Shows the hand recipes when the inventory opens without a container |
| | Hand Title | Crafting | Title of the panel for hand crafting |
| | Close Distance Margin | 0.5 | Extra distance, on top of the station Use Range, before an opened station closes |
| Recipes | Recipe List, Recipe Row Prefab | | Parent of the recipe rows, and the `CraftingRecipeRow` created for each recipe |
| Recipes | No Recipe Text | | Shown when no recipe is available here |
| Selected recipe | Details | | Object shown while a recipe is selected |
| Selected recipe | Selected Icon, Selected Name, Ingredients Text, Quantity Text | | Texts and icon of the selected recipe |
| Selected recipe | Less Button, More Button, Max Button, Craft Button | | Quantity buttons and craft button |
| Selected recipe | Enough Color, Missing Color | green, red | Ingredient lines the player can or cannot pay |
| Queue | Queue List, Queue Row Prefab | | Parent of the queued jobs, and the `CraftingQueueRow` created for each one |
| Queue | Status Text | | Shows Empty Queue Text, Output Full Text, or nothing |
| Queue | Empty Queue Text | Nothing in progress | Shown when the queue is empty |
| Queue | Output Full Text | Output full: take the items out | Shown when a finished job waits for room in the station output |
| Queue | Take All Button | | Moves the whole station output to the player inventory |

`CraftingRecipeRow`, on the recipe row prefab: **Button**, **Icon**, **Label**, an optional **Selection** object shown on
the selected recipe, and an optional **Canvas Group** dimmed to **Unavailable Alpha** (0.45) when the player cannot
afford it.

`CraftingQueueRow`, on the queue row prefab: **Icon**, **Label**, **Progress Fill** (a filled image showing the progress
of the current job) and **Cancel Button**.

## Scripting

```csharp
// Queue 5 crafts at a station, paid by the local player.
station.RequestCraft(arrow, 5);

// Server side, for a quest reward or an NPC.
string refusal = station.ServerQueue(clientId, arrow, 5);
if (refusal != null) Debug.Log(refusal);
```

`CraftingQueue` is the base of `CraftingStation` and `PlayerCrafter`. Derive from it to make a queue that delivers
elsewhere: override `CanUse` and `ServerDeliver`.

## Save

- Crafts made by hand are saved with the player, and resume when they come back.
- Stations placed in the scene save their queue and their output with the world, with the time already spent on the
  current craft. No saver to add: each station saves itself.
- A restored station job has no owner anymore: the first player in range who cancels it gets the ingredients back.
  With **Cancel When Away**, restored jobs are never cancelled for distance.
- Stations built with the Building module are saved with their piece, see [Building](building.md#chests-workbenches-and-other-functional-pieces).
