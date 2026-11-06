# Build the demo scene

*UST Tools > Create Full Survival Demo* turns the open scene into a playable demo of every module in the project. It is
the quickest way to see the systems working together, and a reference of how they are wired.

Each step only runs when its result is missing, so the tool can run again after adding a module, and never overwrites
what you set up by hand. Steps of a module removed with its `UST_DISABLE_...` symbol are skipped. Some steps show their
own dialog; a report lists what was done at the end.

## Steps

| Step | What it does | Skipped when |
|---|---|---|
| Game UI | Places the `UST UI` prefab, with its settings, then adds the stats panel, the nameplates, the equipment panel, the crafting panel and the build menu it misses | The scene has an inventory window and the panels |
| Player prefab | Runs *Setup Player Prefab*: the components of every module in the project | The player already has the component |
| Demo island | Builds the 300 m test island with trees, sea and NavMesh, see [Demo island](../modules/ai.md#demo-island) | An object named *Demo Island* exists |
| Building in scene | Adds the Building Rules and the saver of built pieces | The scene has Building Rules |
| Demo pieces | Creates the demo pieces, their database and the Hammer | The demo database exists |
| Campfire piece | A buildable campfire with a light, a Heat Source (25 degrees, 6 m) and a Fuel Burner starting with 10 minutes of fire, costing wood, added to the demo pieces. A campfire made by an older version gets its Fuel Burner. | The campfire exists, burns fuel and is listed |
| Temperature in scene | Adds the Ambient Temperature | The scene has one |
| Day and night | Adds the Day Night Cycle, using the directional light as the sun | The scene has one |
| Weather | Adds the Weather Director with Clear, Rain, Storm and Fog | The scene has one |
| Snow on the summit | A weather zone with Snow on the highest point of the island | The director already has zones |
| Demo workbench | A buildable workbench (5 wood, no station needed) with a take only output, added to the demo pieces. The wooden demo pieces then need a workbench nearby. | It exists, and the pieces already have a station |
| Recipes | Hand recipes for the hammer (3 wood) and the axe (4 wood) | The item already has a recipe |
| Demo armor | **Fur Coat** (chest, armor 4, cold insulation 12, Cold 0.7) and **Wooden Helmet** (head, armor 3), made from wood at the workbench, added to the Item Database | They exist |
| Starting kit | Axe, hammer and 10 wood | The player already has a starting kit: yours is kept |
| Enemy prefab | Makes an enemy from the model of the player, with its Enemy Definition | The project has an enemy prefab |
| Enemy spawner | Adds a spawn director with every enemy prefab | The scene has a director |
| Wood burns | A Burnable trait on the wood item: 60 s of fire per log | It has one |
| Other modules | Sleep, vehicles saver and HUD, lobby, chat, voice, options menu, markers and shop window, as the Setup window adds them | Each one is in the scene |
| Bed | A bed 5 m from the spawn point | The scene has a bed |
| Trader | A trader 5 m from the spawn point with the **Demo Trader** catalog: Wooden Helmet for 8 wood (stock 3), Fur Coat for 12 wood, buys axes for 2 wood | The scene has a trader |
| Map | The map area fitted to the island, its picture baked, the map window and the minimap | The scene has a map area, or the scene is not saved yet |

Last, **Validate Network Prefabs** opens to register every new network prefab: the pieces, the campfire, the enemy.
Then run *UST Tools > Validation > [Validate Scene](../getting-started/validate-scene.md)* to check that
nothing is missing.

## Then

1. Save the scene and press Play.
2. Chop trees for wood. Build a workbench with the hammer: right click opens the build menu.
3. Build a foundation, walls, a campfire and a ward around the workbench.
4. Craft a Fur Coat at the workbench and wear it: the stats panel shows the armor and the cold resistance.
5. Wait for the night, or climb to the snowy summit: it gets colder, the campfire and the coat matter, and enemies come
   in greater numbers.

## Known gaps

- The enemy uses the model and the animations of the player: replace it with a real one, see [AI](../modules/ai.md).
- Only the hammer, the axe and the demo armor get recipes: add **Crafting > Recipe** traits to the other items.
- The demo armor has no model on the character, only stats.
- Your own workbench prefab is left untouched: the demo uses its own.
