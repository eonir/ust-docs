# Setup Window

*Tools > UST > Setup Window* is the entry point of the template. It shows what each module of the project has in the
open scene and on the player prefab, and adds what is missing in one click. It opens by itself the first time the
template is imported.

Everything it does is also in the menus, so you can work without it. It only reads and runs those menus: closing it
changes nothing.

## Start

| Button | What it does |
|---|---|
| **Create Full Survival Demo** | Turns the open scene into a playable demo of every module, with the demo content: island, campfire, workbench, armor, bed, trader, map. See [Build the demo scene](../guides/demo-scene.md). |
| **Validate Scene** | Lists what is still wrong in the scene and the player prefab, with a fix button when possible. See [Validate Scene](validate-scene.md). |
| **Create Game UI** | Creates the whole game UI. |
| **Add Every Module To Scene** | Adds to the open scene what every module needs, and sets up the player prefab, **without** the demo content. Use it on your own scene. |

Save the scene before **Create Full Survival Demo** or **Add Every Module To Scene**: the map picture is saved next to
it, and is skipped otherwise.

## Player prefab

The line shows the components of the modules that `Assets/UST/Player/Prefabs/Player.prefab` misses, for example
*Missing: Recipe book, Bed respawn*.

| Button | What it does |
|---|---|
| **Setup Player Prefab** | Adds every missing component, each with its default setup, and saves the prefab. Same as *Tools > UST > Setup Player Prefab*. |
| **Select Player Prefab** | Selects the prefab in the Project window. |

| Module | Component added to the player |
|---|---|
| Building | `PlayerBuilder` |
| Crafting | `PlayerCrafter` (hand crafting), `RecipeBook` (recipe unlocks) |
| Equipment | `PlayerEquipment`, its container and the death drop of worn items |
| Weather | `WeatherExposure` |
| Temperature | `PlayerTemperature` |
| Sleep | `Sleeper` (the bed respawn point is saved) |
| Vehicle | `VehicleOccupant` |

A component already on the prefab is never changed. A module removed from the project is left out.

## Modules

One box per module present in the project. Each one lists what the module adds:

- **Green**: in place.
- **Orange, *missing***: not in the open scene yet.
- ***(player)***: a component of the player prefab, see above.

**Add** adds everything the module misses, in the scene and on the player, then the box turns green and the button
reads **Done**. Some steps show their own dialog, for example when the scene has no NetworkManager yet.

| Module | In the scene | On the player |
|---|---|---|
| Game UI | Game UI, stats panel, player nameplates | |
| Building | Building rules and saver, build menu | Building |
| Crafting | Crafting panel | Hand crafting, recipe book |
| Equipment | Equipment panel | Equipment |
| Weather | Day and night cycle, weather director | Weather exposure |
| Temperature | Ambient temperature | Temperature |
| Sleep | Sleep director | Bed respawn |
| AI | Enemy spawner | |
| Vehicle | Vehicles saver, vehicle HUD | Vehicles |
| Lobby | Lobby | |
| Chat | Chat | |
| Voice | Voice | |
| Settings | Options menu | |
| Markers | Markers, map and minimap | |
| Shop | Shop window | |

The window refreshes when the scene changes and when it gets the focus.

## Tools

Shortcuts to **Item Database**, **Held Item Tuning**, **Validate Network Prefabs** and **Assign Input Actions**.

## Right click menus

What works on one object is in its right click menu, under **UST**, in the Hierarchy or the Project window:

| Right click | Menu | What it does |
|---|---|---|
| An object in the Hierarchy | *Make Crafting Station* | A workbench, a forge, a campfire that cooks: NetworkObject, output container, `CraftingStation` |
| An object in the Hierarchy | *Add Fuel Burner* | A fire that burns fuel, with its lights, heat and particles on only while it burns |
| An object in the Hierarchy | *Add World Marker* | Shows the object on screen with an icon and its distance |
| A character in the Hierarchy | *Setup Item Rig* | The rig that holds items in the hands |
| Anywhere in the Hierarchy | *Bed*, *Trader*, *Weather Zone* | Places a bed, a trader, or a zone with its own weather |
| A model, in the Hierarchy or the Project window | *Create Enemy From Model*, *Create Vehicle From Model* | A network enemy or vehicle prefab |
| An enemy prefab | *Create Enemy Definition* | Its stats asset |
| An Animator Controller | *Add Upper Body Layer* | The layer used to hold items while moving |
| An Animator Controller | *Add Use Item State* | The state that plays the use animation of consumables |

The same items are in the *GameObject > UST* menu at the top of the editor.

## Advanced

*Tools > UST > Advanced* has one submenu per module, with each step the Setup window and the demo run: add one module to
the scene, add one component to the selected player, rebuild one panel of the UI, bake the map, create the default
vitals, vehicles or build pieces, the demo island, the vehicle test scene, the Animator Controller of the player, the
default UI layout, or assign the input actions again. Use them to redo a single step, for example to rebuild the
crafting panel after an update. The full list is in [Editor Tools](editor-tools.md).

## For your own modules

The modules and their steps are listed in `SetupCatalog` (`Assets/UST/Editor/Setup/SetupCatalog.cs`). Each step names
a component type, found by name so a missing module is skipped, and the menu that adds it. Add an entry for your own
module and the window, *Setup Player Prefab*, *Add Every Module To Scene* and the demo pick it up:

```csharp
new SetupCatalog.Module
{
    name = "Fishing", description = "Rods, fish, water spots", page = "modules/fishing",
    scene = new[] { Scene("Fish spawner", "MyGame.FishSpawner, MyGame", "Tools/UST/Advanced/Fishing/Add Fishing To Scene") },
    player = new[] { Player("Fishing", "MyGame.PlayerFisher, MyGame", "Tools/UST/Advanced/Fishing/Add Fishing To Selected Player") }
}
```

A player step's menu works on `Selection.activeGameObject`: *Setup Player Prefab* selects the root of the player
prefab before running it.
