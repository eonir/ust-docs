# Modules

Modules have assembly definitions with explicit dependencies, listed below. Shared interfaces in **UST.Core** let
systems use optional features without directly referencing their implementation: equipment can provide effects
through `IEffectReceiver`, for example.

Some modules also depend directly on other assemblies, such as Crafting on Inventory and UI. Disabling an optional
module with its supported define is different from deleting a shared dependency. Before removing an assembly, check
the modules that reference it and the scene and prefab components that use it.

## Assemblies

| Assembly | Depends on | Role |
|---|---|---|
| `UST.Core` | - | Interfaces shared by every module, spawn points |
| `UST.Player` | Core | Controller, input, camera zoom, interaction |
| `UST.Vitals` | Core | Vitals, death, respawn |
| `UST.Inventory` | Core | Items, containers, hotbar, world items, loot bags |
| `UST.Save` | Core | Save manager, player identity |
| `UST.Net` | Core | Network helpers, debug session menu |
| `UST.Weapons` | Core, Inventory, Player, Vitals | Melee and ranged weapons, ammo, aiming, PvP |
| `UST.Effects` | Core, Vitals, Inventory | Buffs and debuffs, effect zones, consumables, effects bar |
| `UST.Crafting` | Core, Inventory, UI | Recipes, crafting stations, hand crafting, crafting queue and window, fuel burners, recipe book |
| `UST.Weather` | Core | Day and night cycle, weather states and zones, synchronized and saved |
| `UST.Temperature` | Core, UI | Felt temperature, zones, heat sources, shelter, cold and heat stages |
| `UST.Equipment` | Core, Inventory, UI | Equipment slots and layout, armor, resistances, worn effects and models |
| `UST.AI` | Core, Inventory | Melee enemies, spawn director, demo island tool |
| `UST.Building` | Core, Inventory, UI | Build pieces, build tools, preview and snapping, build menu, saving built pieces |
| `UST.Gathering` | Core, Inventory, Player | Tools, resource nodes, object spawners, gathering feedback |
| `UST.Vehicle` | Core, Inventory, Player | Vehicles, seats, raycast wheel physics, fuel, repairs, trunk, vehicle HUD, saving vehicles |
| `UST.Sleep` | Core, Player, Weather | Beds, skipping the night when enough players sleep |
| `UST.Lobby` | Core, Net, Save, Multiplayer Services | Main menu, online sessions (Lobby, Relay), direct IP, loading screen, session menu |
| `UST.Chat` | Core, Net | Text chat over Netcode, chat box |
| `UST.Voice` | Core, Net, Vivox | Proximity voice chat, push to talk, list of speakers |
| `UST.Settings` | Core, Cinemachine, URP | Options menu, Keys tab, applying graphics and audio settings |
| `UST.Markers` | Core, Net | Pings, placed markers, far players, world markers, edge arrows, map and minimap |
| `UST.Shop` | Core, Inventory, UI | Traders, catalogs of buy and sell offers, stock, shop window |
| `UST.UI` | Core, Inventory, Vitals, Player | Item feed, screen messages |
| `UST.Editor` | all | Inspectors and tools, editor only |
| `UST.Crafting.Editor` | Crafting, Editor | Crafting menus and panel builder, editor only |
| `UST.Building.Editor` | Building, Editor | Building menus, build menu builder, demo pieces, editor only |
| `UST.Vehicle.Editor` | Vehicle, Editor | Vehicle menus, default vehicles, vehicle test scene, HUD builder, editor only |
| `UST.Sleep.Editor` | Sleep, Weather, Editor | Sleep menus and scene checks, editor only |
| `UST.Lobby.Editor`, `UST.Chat.Editor`, `UST.Voice.Editor` | their module, Editor | Menus, UI builders and scene checks, editor only |
| `UST.Demo` | all | Demo scene scripts |

## How modules communicate

| Interface | Implemented by | Used by |
|---|---|---|
| `IStaminaSource` | `VitalStaminaSource` | `PlayerController` spends stamina without knowing the Vitals module |
| `IMovementState` | `PlayerController` | `SprintDecayModifier` makes vitals decay faster while running |
| `IMovementSpeedModifiers` | `PlayerController` | The Speed modifier of effects slows down or speeds up the character |
| `IAimFacing` | `PlayerController` | `PlayerItemUser` and `PlayerRangedWeapon` turn the character towards the aim |
| `IInteractable` | `WorldItem`, `LootBag` | `PlayerInteractor` |
| `IDeathSource` | `PlayerLife` | `PlayerDeathDrop`, `PlayerStartingKit`, `PlayerEffects` clears effects on death |
| `IPlayerTeleporter` | `PlayerLife` | `PlayerSaveIdentity` restores the saved position |
| `IRespawnPointOverride` | `Sleeper` | `PlayerLife` respawns the player at its bed instead of a spawn point |
| `ISavedPositionOverride` | `VehicleOccupant`, `Sleeper` | `PlayerSaveIdentity` saves a seated player next to the vehicle and a sleeping player next to the bed |
| `ITimeSkipListener`, `TimeSkip` | `PlayerVitals`, `PlayerEffects`, `CraftingQueue`, `FuelBurner`, `Trader`, `ObjectSpawner`, `WeatherDirector`, `EnemySpawnDirector` | `DayNightCycle.ServerSkipToMorning` advances them by the skipped time when the night is slept through |
| `IPlayerSaveable` | `PlayerVitals`, `PlayerInventory`, `PlayerEffects`, `PlayerCrafter`, `RecipeBook`, `Sleeper` | `PlayerSaveIdentity` |
| `IWorldSaveable` | `WorldItemsSaver` | `SaveManager` |
| `IObjectSaveable` | `StorageContainer`, `CraftingStation`, `FuelBurner` | `BuildPiecesSaver` saves them with each built piece |
| `IRemovalHandler` | `CraftingStation` | `PlayerBuilder` calls it before a built piece is removed |
| `DamageType`, `Resistance`, `IDamageModifier` | Weapons, `EnemyDefinition`, `PieceHealth`, `PlayerDamageReceiver` | Every damage receiver applies resistances by type, then the modifiers of its object (armor) |
| `EffectAsset`, `IEffectReceiver` | `EffectDefinition`, `PlayerEffects` | Equipment keeps effects active while worn, without depending on the Effects module |
| `WorldWeather` | `WeatherDirector` | `PlayerTemperature` adds the temperature of the weather; your code reads the wind |
| `IWorldClock`, `WorldClock` | `DayNightCycle` | `AmbientTemperature` cools the nights, `EnemySpawnDirector` spawns more at night |
| `IInsulationSource` | `PlayerEquipment` | `PlayerTemperature` reads the cold and heat insulation of the clothes |
| `IStatProvider`, `IResistanceSource`, `StatLine` | `PlayerVitals`, `PlayerInventory`, `PlayerEquipment`, `PlayerDamageReceiver` | The character stats panel lists the lines of every module |
| `IDamageable` | `PieceHealth`, `PlayerDamageReceiver`, `EnemyHealth` | Enemies, traps and your own code damage anything without knowing its module |
| `StationType`, `IStation`, `Stations` | `CraftingStationType`, `CraftingStation` | `PlayerBuilder` checks a station is near a piece that needs one |
| `IProtectionZone`, `ProtectionZones` | `BuildWard` | `InventoryContainer`, `CraftingStation`, `PlayerBuilder` check who may act in a protected area |
| `IPlayerDataSource` | `PlayerSaveIdentity` | `PlayerStartingKit` knows if the player is new |
| `IPlayerIdentityProvider` | `LocalPlayerIdentity` or yours | `SaveManager` |
| `IItemHitReceiver` | `ResourceNode`, `PlayerDamageReceiver` | `PlayerItemUser` calls it on the server after a validated hit |
| `IItemUseHandler` (Inventory) | `PlayerConsumer`, `PlayerEquipment` (inventory only, `IInventoryOnlyUseHandler`) | `HotbarInput` and `InventoryWindow` use an item before selecting or moving it |
| `GameSettings`, `OptionsCatalog`, `SettingKeys` | Every module | Player settings kept on the machine in `settings.json`; each module registers its options and follows its keys |
| `LocalStore` | `GameSettings`, `PlayerNames`, `LocalPlayerIdentity` | Values of this machine in readable JSON files, instead of PlayerPrefs |
| `InputRebinding` | `OptionsMenu` | Applies the saved key bindings to the input actions and lists them in the Keys tab |
| `GameInput` | Every module | Actions with a keyboard fallback, the device in use, binding texts for hints |
| `UIInputFocus` | Your UI, `ChatView` | `PlayerItemUser` ignores attacks while a UI is open; while typing (`PushTyping`), movement and every shortcut are ignored |
| `PlayerNames` (Net) | Set by the lobby | Chat, voice and session menu show the names of the players |
| `SpawnerManagedObject` | Added by `ObjectSpawner` | `WorldItemsSaver` and `ResourceNodesSaver` skip objects a spawner saves itself |
| `LocalMessages` | `GatherFeedback`, your systems | `ScreenMessageView` |

Every link is optional. Without `VitalStaminaSource`, stamina is unlimited. Without the Save module, the starting kit
is given on every join. Without `PlayerLife`, nobody dies.

## Disabling an optional module

Optional modules in `Assets/UST/Modules` are compiled only when their define is **not** set. Add the define in
*Project Settings > Player > Scripting Define Symbols* to remove a module from the build:

| Module | Define |
|---|---|
| Effects | `UST_DISABLE_EFFECTS` |
| Equipment | `UST_DISABLE_EQUIPMENT` |
| Weapons | `UST_DISABLE_WEAPONS` |
| Crafting | `UST_DISABLE_CRAFTING` |
| Building | `UST_DISABLE_BUILDING` |
| Gathering | `UST_DISABLE_GATHERING` |
| Shop | `UST_DISABLE_SHOP` |
| AI | `UST_DISABLE_AI` |
| Weather | `UST_DISABLE_WEATHER` |
| Temperature | `UST_DISABLE_TEMPERATURE` |
| Vehicle | `UST_DISABLE_VEHICLE` |
| Sleep | `UST_DISABLE_SLEEP` (also removed with `UST_DISABLE_WEATHER`) |
| Lobby | `UST_DISABLE_LOBBY` (also removed without the Multiplayer Services package) |
| Chat | `UST_DISABLE_CHAT` |
| Voice | `UST_DISABLE_VOICE` (also removed without the Vivox package) |
| Settings | `UST_DISABLE_SETTINGS` |
| Markers | `UST_DISABLE_MARKERS` |

!!! warning "Shared assets"
    Items, vitals, effects, categories and kits are shared ScriptableObjects. Never modify them from code at runtime: the
    change would apply to every player and would be lost in a build. Runtime values always live in components.
