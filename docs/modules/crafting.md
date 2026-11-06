# Crafting

The Crafting module turns items into recipes, adds crafting stations (workbench, forge, cooking fire...) and crafting
by hand from the inventory. Crafts are queued and run by the server: ingredients are taken when a craft is queued and
given back when it is cancelled.

The module is optional: add `UST_DISABLE_CRAFTING` to the Scripting Define Symbols to remove it, see
[Modules](../architecture/modules.md#disabling-an-optional-module).

## Setup

1. *Tools > UST > Create Game UI* adds the crafting panel right after the container panel of the inventory
   window, with the same look. For a UI built before, run **Tools > UST > Advanced > UI > Add Crafting Panel**.
2. For crafting by hand, select the player prefab and run **Tools > UST > Advanced > Crafting > Add Hand Crafting To Selected Player**. It adds `PlayerCrafter` next to `PlayerInventory`.
3. Create the station types you need: *Create > UST > Crafting > Station Type* (Workbench, Forge...).
4. Open the station prefab, select its root and run **right click in the Hierarchy > UST > Make Crafting Station**. It
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
| Unlock | Always | When a player knows the recipe, see below |

The Item Database window warns about a recipe without ingredients, an empty ingredient, an item used in its own recipe
an ingredient missing from the database, and repeated ingredient lines.

Repeated lines for the same item are added together when checking affordability and taking ingredients. For example,
two lines of 5 Wood require 10 Wood for one craft. A rejected request takes no ingredients; the server checks the
combined cost before removing items.

## Recipe unlock

Recipes a player does not know are hidden from them in the crafting window, and the server refuses them. Add a
`RecipeBook` to the player prefab with **Tools > UST > Advanced > Crafting > Add Recipe Book To Selected Player**: without it, every
recipe is known.

| Unlock | The player knows the recipe |
|---|---|
| Always | From the start. The default, so existing recipes do not change. |
| Any Ingredient Found | Once they have held one of its ingredients, like Valheim |
| All Ingredients Found | Once they have held each of its ingredients |
| Manual | Only when your code unlocks it: a recipe scroll, a quest, a skill |

The player gets a *New recipe* message. Known recipes and found items are saved with the player; a recipe added to the
game later is unlocked at the next login if the player already found its ingredients.

```csharp
// Server: a quest reward, a scroll read, a skill learned.
player.GetComponent<RecipeBook>().ServerUnlock(cookedMeat);

bool known = RecipeBook.LocalKnows(cookedMeat);   // local player
RecipeBook.Unlocked += item => ShowPopup(item);   // local player, new recipe
```

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
Each job has a unique id given by the server, so a cancel always targets the job that was clicked, even if the queue
moved meanwhile. Restored jobs, which have no owner, show their cancel button to every player in range.

## Fire and fuel

A campfire, a furnace or a forge needs fuel: add a `FuelBurner` to the station with **right click in the Hierarchy > UST > Add Fuel Burner**. It also fills **Lit Behaviours** with the Lights and the `HeatSource` of the object, and **Lit Objects**
with its particle systems: they are on only while the fire burns.

Give the fuel items a **Crafting > Burnable** trait: wood, coal, resin. **Burn Seconds** (60) is the fire time one item
adds.

| Field | Default | Meaning |
|---|---|---|
| Accepted Fuel | empty | Items this burner takes. Empty: any item with a Burnable trait. |
| Max Fuel Seconds | 1800 | Most fire time it holds |
| Starting Fuel Seconds | 0 | Fire time when it first appears |
| Burn While Idle | on | On: burns as long as it is lit, like a campfire that warms and lights. Off: only burns while the station crafts, like a furnace. |
| Required To Craft | on | The station only crafts while the fire burns. Off: the fire is only for heat and light. |
| Use Range | 3 | Distance to add fuel or put the fire out |

The crafting window shows the fire time left with **Add fuel** (one item, ten with Shift) and **Put out** / **Light**.
Adding fuel lights the fire again. When the fire goes out, the current craft is put on hold and carries on where it
stopped once fuel is added; the panel shows *No fire: add fuel to craft*.

A `FuelBurner` also works alone, without a station: a brazier or a campfire only for heat. The Interact input then adds
one fuel item. A skipped night ([Sleep](sleep.md)) burns the fuel, and a station only crafts for as long as it lasted.
The fuel and the lit state are shared by every player and saved, for burners placed in the scene and for built ones.

For a UI built before, rebuild the crafting panel (*Tools > UST > Advanced > UI > Add Crafting Panel*, **Rebuild**) to get the fuel
line.

```csharp
burner.RequestAddFuel(1);           // local player, from their inventory
burner.ServerAddFuelSeconds(600f);  // server: free fire time, for a quest or a debug tool
bool canCraft = burner.HasFire;
```

## Crafting window

The output of a station sits under the crafting panel, in one column (**Stack Output Under**, on). The queue shows its
first 4 lines (**Max Visible Queue Rows**) and counts the others. Ordering the same item again joins the last line of
the queue when it is yours, so repeated clicks make one line (*Stone Axe 0/5*) instead of filling the queue.

`CraftingWindow` sits on the inventory window object and shows its panel:

- next to the inventory, with the recipes made by hand, when the inventory opens without a container
  (**Hand Crafting With Inventory**);
- when the player interacts with a station: the inventory opens with the station output in the container panel, and
  the station recipes in the crafting panel.

Recipes the player cannot afford are dimmed. The selected recipe shows each ingredient as owned / needed, green or red,
for the chosen quantity. **-** and **+** change the quantity, by 10 with Shift, **Max** picks the most the player can
afford. The queue shows each job with its progress and a cancel button.

*Create Game UI* or *Tools > UST > Advanced > UI > Add Crafting Panel* builds it and fills every reference. The texts can be
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
