# Crafting

The Crafting module turns items into recipes, adds crafting stations (workbench, forge, cooking fire...) and crafting
by hand from the inventory. Crafts are queued and run by the server: ingredients are taken when a craft is queued and
given back when it is cancelled.

The module is optional: add `UST_DISABLE_CRAFTING` to the Scripting Define Symbols to remove it, see
[Modules](../architecture/modules.md#disabling-an-optional-module).

## Setup

1. In the [Setup Window](../getting-started/setup-window.md), **Crafting > Add** adds the crafting panel, and hand crafting and the recipe book to the
   player prefab. By hand:
    - *Tools > UST > Create Game UI* adds the crafting panel right after the container panel of the inventory window,
      with the same look. For a UI built before, run *Tools > UST > Advanced > UI > Add Crafting Panel*.
    - *Tools > UST > Setup Player Prefab* adds `PlayerCrafter` (crafting by hand) and `RecipeBook` (recipe unlocks)
      next to `PlayerInventory`.
2. Create the station types you need: *Create > UST > Crafting > Station Type* (Workbench, Forge, Campfire...).
3. Open the station prefab, right click its root in the Hierarchy > **UST > Make Crafting Station**. It adds a
   `NetworkObject`, an output `InventoryContainer` in **Take Only** mode and `CraftingStation`. Assign the
   **Station Type**. For a station that needs fire, right click it again > **UST > Add Fuel Burner**, see
   [Fire and fuel](#fire-and-fuel).
4. Register the station prefab in your Network Prefabs list, or place it in the scene.
5. Give items a **Crafting > Recipe** trait, see below.

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
`RecipeBook` to the player prefab with *Tools > UST > Setup Player Prefab*, or alone with
*Tools > UST > Advanced > Crafting > Add Recipe Book To Selected Player*: without it, every recipe is known. Recipes
with **Always** never need it.

| Unlock | The player knows the recipe |
|---|---|
| Always | From the start. The default, so existing recipes do not change. |
| Any Ingredient Found | Once they have held one of its ingredients, like Valheim |
| All Ingredients Found | Once they have held each of its ingredients |
| Manual | Only when your code unlocks it: a recipe scroll, a quest, a skill |

The player gets a *New recipe* message (**Unlocked Format** on the `RecipeBook`, `{0}` being the item), except for the
recipes restored from the save when joining. Known recipes and found items are saved with the player; a recipe added
to the game later is unlocked at the next login if the player already found its ingredients.

An item counts as found once it is in the inventory or the hotbar of the player, however it got there: picked up,
crafted, bought, given by code.

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
| Can Repair | on | Players can repair here the worn items this station makes, see [Repair](#repair) |
| Repair Cost | 0.25 | Part of the recipe paid to repair an item from broken to new. 0: repairs are free. |
| Repaired Format, Repair Missing Format | {0} item(s) repaired, Not enough materials to repair {0} | Messages to the player. {0}: the number of items, or the item that could not be paid. |

The output is **Take Only**: players can take items out but never put their own in, so they cannot block the station.

When the output has no room for the next result, the queue waits and the panel shows *Output full*. Empty it by
dragging the items, Ctrl + click, or **Take all**. Stack limits are respected: a craft only finishes when all of its
items fit.

The **Take all** button of the inventory window also empties the station output, see
[Sort and Take All](ui/windows.md#sort-and-take-all).

Several players can use the same station: jobs run one after the other, and each player can cancel only their own.
Each job has a unique id given by the server, so a cancel always targets the job that was clicked, even if the queue
moved meanwhile. Restored jobs, which have no owner, show their cancel button to every player in range.

## Fire and fuel

A campfire, a furnace or a forge needs fuel: right click the station in the Hierarchy > **UST > Add Fuel Burner**.
It adds a `FuelBurner`, fills **Lit Behaviours** with the Lights and the `HeatSource` of the object, and **Lit
Objects** with its particle systems: they are on only while the fire burns. Keep the particles on child objects: an
object cannot turn itself off.

Give the fuel items a **Crafting > Burnable** trait: wood, coal, resin. **Burn Seconds** (60) is the fire time one item
adds.

| Field | Default | Meaning |
|---|---|---|
| Accepted Fuel | empty | Items this burner takes. Empty: any item with a Burnable trait. |
| Max Fuel Seconds | 1800 | Most fire time it holds |
| Starting Fuel Seconds | 0 | Fire time when it first appears |
| Burn While Idle | on | On: burns as long as it is lit, like a campfire that warms and lights. Off: only burns while the station crafts, like a furnace. |
| Required To Craft | on | The station only crafts while the fire burns. Off: the fire is only for heat and light. |
| Use Range | 3 | Distance to add fuel or put the fire out, checked on the server |
| Lit Objects | | Shown while the fire burns: flames, embers, smoke |
| Lit Behaviours | | Enabled while the fire burns: a Light, a HeatSource, an AudioSource |
| Prompt Format | Add fuel ({0}) | Interaction text of a burner without station. {0}: the fire time left. |
| No Fuel Message | No fuel in the inventory. | Shown when the player has no accepted fuel |
| Full Message | The fire is full. | Shown when the burner holds Max Fuel Seconds |

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

## Repair

Items with durability wear out (see [Items](inventory/index.md)). A station repairs the items it makes: those whose
recipe needs its station type. Hand crafting (`PlayerCrafter`, same fields) repairs the items made by hand, anywhere.
An item without a recipe cannot be repaired.

**Repair** in the crafting panel repairs every worn item of the player that can be repaired there: inventory, hotbar,
and every other container of the player, such as the equipment. The line next to it shows how many items are worn
and the total cost, green when the player has enough, red otherwise.

The cost of one item is **Repair Cost** times each ingredient of its recipe, in proportion to the wear, rounded up.
With a Repair Cost of 0.25 and a recipe of 4 Iron and 2 Wood:

| Durability left | Cost |
|---|---|
| 50 % | 1 Iron, 1 Wood (4 × 0.25 × 0.5 = 0.5, rounded up) |
| Nearly broken | 1 Iron, 1 Wood |
| Nearly broken, with a Repair Cost of 1 | 4 Iron, 2 Wood |

The server repairs the items one by one and takes the materials of each, until one cannot be paid: the player sees
how many were repaired, and which item was missing materials. The station must be in range and not behind a ward,
as for crafting. Repairs are instant.

```csharp
station.RequestRepairAll();                                          // the local player
string message = station.ServerRepairAll(clientId, out bool refused); // server side
bool worn = station.NeedsRepair(item, stack);
station.GetRepairCost(item, stack, cost);                            // fills a List<Ingredient>
```

## Crafting window

`CraftingWindow` sits on the inventory window object and shows its panel:

- next to the inventory, with the recipes made by hand, when the inventory opens without a container
  (**Hand Crafting With Inventory**);
- when the player interacts with a station: the inventory opens with the station output in the container panel, and
  the station recipes in the crafting panel.

Recipes the player cannot afford are dimmed. The selected recipe shows each ingredient as owned / needed, green or red,
for the chosen quantity. **-** and **+** change the quantity, by 10 with Shift, **Max** picks the most the player can
afford. The queue shows each job with its progress and a cancel button.

- The output of a station sits **under** the crafting panel, in one column (**Stack Output Under**), so the row of
  panels stays narrow. A chest alone opens as before.
- The queue shows its first 4 lines (**Max Visible Queue Rows**) and counts the others (*+3 more in the queue*).
- Ordering the same item again joins the last line of the queue when it is yours: repeated clicks make one line
  (*Stone Axe 0/5*) instead of filling the queue. The queue holds **Max Jobs** lines, 8 by default.
- A **Repair** line shows the worn items the player can repair here and their cost, see [Repair](#repair).
- At a station with a fire, a line shows the fire time left, **Add fuel** (one item, ten with Shift) and **Put out**
  or **Light**.

*Create Game UI* or *Tools > UST > Advanced > UI > Add Crafting Panel* builds it and fills every reference. The texts can be
translated or restyled from the inspector.

| Section | Field | Default | Meaning |
|---|---|---|---|
| | Inventory Window | | The inventory window the panel opens with |
| | Panel, Title | | Object shown and hidden, and its title: the station name, or Hand Title |
| | Hand Crafting With Inventory | On | Shows the hand recipes when the inventory opens without a container |
| | Hand Title | Crafting | Title of the panel for hand crafting |
| | Stack Output Under | On | Puts the output of a station under the crafting panel when both sit side by side in the same row |
| | Close Distance Margin | 0.5 | Extra distance, on top of the station Use Range, before an opened station closes |
| Recipes | Recipe List, Recipe Row Prefab | | Parent of the recipe rows, and the `CraftingRecipeRow` created for each recipe |
| Recipes | No Recipe Text | | Shown when no recipe is available here |
| Selected recipe | Details | | Object shown while a recipe is selected |
| Selected recipe | Selected Icon, Selected Name, Ingredients Text, Quantity Text | | Texts and icon of the selected recipe |
| Selected recipe | Less Button, More Button, Max Button, Craft Button | | Quantity buttons and craft button |
| Selected recipe | Enough Color, Missing Color | green, red | Ingredient lines the player can or cannot pay |
| Queue | Queue List, Queue Row Prefab | | Parent of the queued jobs, and the `CraftingQueueRow` created for each one |
| Queue | Status Text | | Shows Empty Queue Text, Output Full Text, Paused Text, the crafts not shown, or nothing |
| Queue | Empty Queue Text | Nothing in progress | Shown when the queue is empty |
| Queue | Output Full Text | Output full: take the items out | Shown when a finished job waits for room in the station output |
| Queue | Take All Button | | Moves the whole station output to the player inventory. Hidden when the inventory window has its own Take All button. |
| Queue | Max Visible Queue Rows | 4 | Queue lines shown at most |
| Queue | More Queued Format | +{0} more in the queue | {0}: the crafts not shown |
| Fuel | Fuel Row | | Shown for stations with a `FuelBurner` |
| Fuel | Fuel Text, Fuel Format, No Fire Text | , Fire {0}, Out | Fire time left. {0}: the time, or No Fire Text |
| Fuel | Add Fuel Button | | Adds one fuel item, ten with Shift |
| Fuel | Fire Button, Fire Button Label, Light Text, Put Out Text | , , Light, Put out | Lights the fire or puts it out |
| Fuel | Paused Text | No fire: add fuel to craft | Shown while the current craft waits for fire |
| Repair | Repair Row | | Shown for stations and hand crafting with Can Repair on |
| Repair | Repair Button | | Repairs every worn item of the player made here, as long as the materials last |
| Repair | Repair Text, Repair Format | , {0} worn: {1} | {0}: the number of worn items, {1}: the total cost |
| Repair | Free Repair Text, Nothing To Repair Text | free, Nothing to repair | |

Crafting panels built before repairs have no Repair line: rebuild the panel with *Add Crafting Panel*, Validate Scene
points to it.

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
- Stations built with the Building module are saved with their piece, see [Building](building/pieces.md#chests-workbenches-and-other-functional-pieces).
