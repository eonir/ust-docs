# Modules

Every module is an assembly definition. Modules talk to each other through small interfaces in **UST.Core**, never
directly, so you can remove one without breaking the others.

## Assemblies

| Assembly | Depends on | Role |
|---|---|---|
| `UST.Core` | - | Interfaces shared by every module, spawn points |
| `UST.Player` | Core | Controller, input, camera feedback, interaction |
| `UST.Vitals` | Core | Vitals, death, respawn |
| `UST.Inventory` | Core | Items, containers, hotbar, world items, loot bags |
| `UST.Save` | Core | Save manager, player identity |
| `UST.Net` | Core | Network helpers, debug session menu |
| `UST.Weapons` | Core, Inventory, Player, Vitals | Melee weapon damage, PvP |
| `UST.Gathering` | Core, Inventory, Player | Tools, resource nodes, object spawners, gathering feedback |
| `UST.UI` | Core, Inventory, Vitals, Player | Item feed, screen messages |
| `UST.Editor` | all | Inspectors and tools, editor only |
| `UST.Demo` | all | Demo scene scripts |

## How modules communicate

| Interface | Implemented by | Used by |
|---|---|---|
| `IStaminaSource` | `VitalStaminaSource` | `PlayerController` spends stamina without knowing the Vitals module |
| `IMovementState` | `PlayerController` | `SprintDecayModifier` makes vitals decay faster while running |
| `IInteractable` | `WorldItem`, `LootBag` | `PlayerInteractor` |
| `IDeathSource` | `PlayerLife` | `PlayerDeathDrop`, `PlayerStartingKit` |
| `IPlayerTeleporter` | `PlayerLife` | `PlayerSaveIdentity` restores the saved position |
| `IPlayerSaveable` | `PlayerVitals`, `PlayerInventory` | `PlayerSaveIdentity` |
| `IWorldSaveable` | `WorldItemsSaver` | `SaveManager` |
| `IPlayerDataSource` | `PlayerSaveIdentity` | `PlayerStartingKit` knows if the player is new |
| `IPlayerIdentityProvider` | `LocalPlayerIdentity` or yours | `SaveManager` |
| `IItemHitReceiver` | `ResourceNode`, `PlayerDamageReceiver` | `PlayerItemUser` calls it on the server after a validated hit |
| `UIInputFocus` | Your UI | `PlayerItemUser` ignores attacks while a UI is open |
| `SpawnerManagedObject` | Added by `ObjectSpawner` | `WorldItemsSaver` and `ResourceNodesSaver` skip objects a spawner saves itself |
| `LocalMessages` | `GatherFeedback`, your systems | `ScreenMessageView` |

Every link is optional. Without `VitalStaminaSource`, stamina is unlimited. Without the Save module, the starting kit
is given on every join. Without `PlayerLife`, nobody dies.

## Disabling an optional module

Optional modules in `Assets/UST/Modules` are compiled only when their define is **not** set. Add the define in
*Project Settings > Player > Scripting Define Symbols* to remove a module from the build:

| Module | Define |
|---|---|
| Equipment | `UST_DISABLE_EQUIPMENT` |
| Weapons | `UST_DISABLE_WEAPONS` |
| Crafting | `UST_DISABLE_CRAFTING` |
| Building | `UST_DISABLE_BUILDING` |
| Gathering | `UST_DISABLE_GATHERING` |
| Shop | `UST_DISABLE_SHOP` |
| AI | `UST_DISABLE_AI` |
| Weather | `UST_DISABLE_WEATHER` |
| Vehicle | `UST_DISABLE_VEHICLE` |

!!! warning "Shared assets"
    Items, vitals, categories and kits are shared ScriptableObjects. Never modify them from code at runtime: the
    change would apply to every player and would be lost in a build. Runtime values always live in components.
