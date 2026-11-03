# Validate Scene

*UST Tools > Validation > Validate Scene* checks the open scene and the player prefab, and lists every setup problem
it finds, module by module, with a button to fix it when the fix is safe.

*Build Demo Scene* sets a **new** scene up correctly. Validate Scene checks **any** scene, at any time: a scene set up
by hand, a scene where a module was added later, or a demo scene that was changed since. Most "it does nothing"
problems are setup problems: a zone missing from a list, a mask that does not see a layer, a panel missing from the UI.

## When to run it

- After setting up your own scene, or adding UST to an existing project.
- After adding or removing a module, a player component, a spawner or a weather zone.
- After changing layers, or the masks of the player.
- Before a build, and before testing with a second player.
- When something does nothing and you do not know why.

## What it looks at

| Source | Where it comes from |
|---|---|
| The scene | Every open scene, inactive objects included |
| The player | The **Player Prefab** of the NetworkManager of the scene. Without a NetworkManager or a Player Prefab, the player checks are skipped |
| The project | Item definitions, item databases, build piece databases, and prefabs, for the checks that need them (recipes, stations) |

It changes nothing until you click a fix.

## The window

| Element | What it does |
|---|---|
| **Refresh** | Runs every check again. The list also refreshes after each fix, but not after a change made by hand |
| **Fix All (n)** | Runs every fix of the list at once, after a confirmation. Fixes that only open a window (*Open Item Database*) are left out. Hidden infos are left out too |
| **Show Infos** | Shows or hides the suggestions |
| Counters | Number of errors, warnings and infos |
| **Select** | Selects and pings the object concerned: a scene object, the player prefab, an item, a piece prefab |
| Fix button | Fixes this line. Its label says what it does |

Fixes change the scene and prefabs, so they are disabled in Play Mode. Some fixes run a UST Tools menu, which can show
its own dialog (for example *Put Pieces On Building Layer* reports how many prefabs it moved). Fixes on the player
open the player prefab, change it and save it, as the *To Selected Player* menus do. Scene changes can be undone with
Ctrl+Z; prefab changes cannot, use your version control.

### Levels

| Level | Meaning | Example |
|---|---|---|
| Error | A feature is broken | A client cannot spawn a prefab, no button can be clicked, an enemy cannot move |
| Warning | It works, but probably not as you intended | A weather zone that does nothing, a recipe that can never be crafted |
| Info | A suggestion | A module present in the scene but not on the player |

A scene can ship with infos left: they are choices. Errors and warnings should be read.

## Checks

A module with nothing in the scene, on the player and in the project is skipped. A module disabled with its
`UST_DISABLE_...` define is not compiled, so it is not checked either.

### Core

| Check | Level | Why | Fix |
|---|---|---|---|
| No NetworkManager | Error | Nothing can start; the other network checks are skipped | - |
| NetworkManager without Player Prefab | Error | No player spawns; the player checks are skipped | - |
| Unregistered network prefabs | Error | The host spawns them, clients fail with *NetworkPrefab could not be found*. Same search as [Validate Network Prefabs](editor-tools.md#validate-network-prefabs) | **Register in** the first prefab list of the NetworkManager |
| Broken prefab list entries | Warning | Empty entries or deleted prefabs in the lists | - |
| No Save Manager | Info | The world and the players are not saved. Fine for a test scene | - |
| Several Save Managers | Error | They save over each other | - |
| UI without EventSystem | Error | No button and no slot can be clicked | **Add EventSystem**, with the Input System UI module |
| No game UI | Warning | No inventory window in the scene | **Create Game UI** |
| No stats panel | Info | The player has stats (`IStatProvider`), but the UI cannot show them | **Add Stats Panel** |
| UI texts catching clicks | Info | A TMP text with **Raycast Target** on and nothing clickable on it or above it. Placed over a button, it swallows its clicks, as the ammo counter over the Stats button. Texts inside a button or a slot are not listed | **Turn Raycast Target off** on all of them |
| Items in no Item Database | Error | They cannot be given, saved or sent over the network | **Open Item Database** |
| Items with trait issues | Warning | A trait reports a problem, such as a recipe without ingredient or a hammer without pieces. The Item Database window shows each message | **Open Item Database** |

### Building

| Check | Level | Why | Fix |
|---|---|---|---|
| Player builds, no Building Rules | Warning | Built pieces are not saved, and the default rules apply | **Add Building To Scene** |
| Player builds, no build menu | Warning | The right click opens nothing | **Add Build Menu** |
| Player builds, no Build Tool item | Warning | No item opens the build menu | - |
| No Building layer | Warning | Pieces share a layer with the terrain and trees: roofs, snapping and weapon masks cannot tell them apart | **Create Layer And Apply** |
| Pieces not on the Building layer | Warning | A piece or one of its children is on another layer | **Put On Building Layer** |
| Player mask without the Building layer | Error | `PlayerBuilder` *Placement Layers*: cannot place on pieces. `PlayerInteractor` *Layers*: cannot use stations and chests built as pieces. `PlayerItemUser` *Hit Layers*: tools do not hit pieces. `PlayerRangedWeapon` *Hit Layers*: bullets go through pieces | **Add Layer To Mask**, on the player prefab |
| Roof Layers set to Everything | Info | On `WeatherExposure` and `PlayerTemperature`: a tree above the player counts as a roof | **Use Building Layer Only**. Keep Everything if your roofs are not build pieces |
| Piece without id | Error | It cannot be saved. Opening the prefab once gives it one | - |
| Pieces sharing an id | Error | A duplicated prefab keeps the id of the original: saves mix them up | - |
| Pieces without collider | Warning | They cannot be aimed at, snapped to, repaired or removed | - |
| Pieces without category | Info | A hammer limited to some categories never offers them | - |
| Station nobody can place | Warning | A piece needs a station type that is not in the scene, and no build piece is that station: it can never be built | - |

### Crafting

| Check | Level | Why | Fix |
|---|---|---|---|
| No crafting panel | Warning | The player has hand crafting or the project has stations, but no station and no hand crafting can be opened | **Add Crafting Panel** |
| Recipes without station, no hand crafting | Warning | Those recipes can never be crafted | **Add Hand Crafting**, on the player prefab |
| Recipes needing a missing station | Warning | The station type is in neither the scene nor any prefab of the project | - |
| Station without NetworkObject | Error | Its queue and output never work | Run *Make Selected A Crafting Station* on it |
| Station without Station Type | Warning | No recipe uses it | - |

### Equipment

| Check | Level | Why | Fix |
|---|---|---|---|
| Equippable items, no equipment on the player | Info | Items with an Equippable trait cannot be worn | **Add Equipment**, on the player prefab |
| Equipment, no equipment panel | Warning | The worn items cannot be seen or changed | **Add Equipment Panel** |

### Temperature

| Check | Level | Why | Fix |
|---|---|---|---|
| Player feels temperature, no Ambient Temperature | Warning | The temperature stays at the default of the player | **Add Temperature To Scene** |
| Several Ambient Temperatures | Warning | Only one is used | - |
| Scene has temperature, player does not feel it | Info | Heat sources, temperature zones or an ambient temperature, but no `PlayerTemperature` | **Add Temperature To Player** |

### Weather

| Check | Level | Why | Fix |
|---|---|---|---|
| Several Day Night Cycles | Error | They fight over the sun and the time | - |
| Day Night Cycle without NetworkObject | Error | The time is not synchronized | - |
| Zones without director | Warning | Weather zones do nothing without a Weather Director | **Add Weather To Scene** |
| Several Weather Directors | Error | Keep one | - |
| Director without NetworkObject | Error | The weather is not synchronized | - |
| Empty table | Error | A director or zone with no weather never shows any | - |
| Only weights of 0 | Error | No weather is ever picked | - |
| Empty table entries | Warning | Entries without a Weather State | - |
| Zones missing from the director | Warning | A zone only works once it is in the **Zones** list of the director | **Add To Director** |
| No Weather Exposure on the player | Info | The weather never gives its effect (Wet...) and the stats panel does not show it | **Add Weather Exposure**, on the player prefab |

### AI

| Check | Level | Why | Fix |
|---|---|---|---|
| No baked NavMesh | Error | Enemies cannot move, and the spawner finds no point to spawn | **Bake NavMesh**, on every NavMesh Surface of the scene |
| Spawner without enemy, or empty entry | Warning | Nothing spawns | - |
| Enemy without NetworkObject | Error | It cannot be spawned | - |
| Enemy without Enemy AI | Warning | It stands still | - |
| Enemy without Enemy Definition | Info | It uses the fallback stats | Select it, then *UST Tools > AI > Create Definition For Selected Enemy* |

## Adding your own checks

Any class implementing `ISceneCheck` (namespace `UST.Editor.Validation`) in an Editor assembly is found and run.
Nothing registers it: it appears in the window on the next compilation, in a group named by `Module`. The assembly
needs a reference to `UST.Editor`, and to the assemblies of the types it checks.

```csharp
using UST.Editor.Validation;

public class MyModuleCheck : ISceneCheck
{
    public string Module => "My Module";

    public void Run(ValidationContext context)
    {
        if (context.OnPlayer<MyComponent>() == null)
        {
            context.Add(IssueLevel.Warning, "The player has no MyComponent: it never does X.", context.PlayerPrefab,
                "Add MyComponent", () => ValidationContext.EditPlayer(context.PlayerPrefab, root => root.AddComponent<MyComponent>()));
        }

        if (context.InScene<MyManager>().Length > 1)
            context.Add(IssueLevel.Error, "Several MyManagers in the scene: keep one.");
    }
}
```

An exception thrown by a check is caught: the window shows *The check failed* in its group and the Console has the
stack trace. The other checks still run.

### ValidationContext

| Member | What it gives |
|---|---|
| `NetworkManager` | The NetworkManager of the open scenes, or null |
| `PlayerPrefab` | Its Player Prefab, or null |
| `Add(level, message, context, fixLabel, fix)` | Adds a line. `context` is the object **Select** pings; `fix` is an `Action`, `fixLabel` its button text. Without `fix`, there is no button |
| `InScene<T>()` | Every object of this type in the open scenes, inactive ones included |
| `SceneHas<T>()` | True when the scenes have at least one |
| `OnPlayer<T>()` | The first component of this type on the player prefab or its children. Works with interfaces |
| `Assets<T>()` | Every asset of this type in the project |
| `PrefabsWith<T>()` | Every prefab with this component on its root |
| `Menu(path)` | A fix that runs a menu |
| `MenuOnPlayer(path)` | A fix that runs a *To Selected Player* menu on the player prefab contents, then saves the prefab |
| `EditPlayer(prefab, edit)` | Opens the prefab contents, runs `edit` on its root, saves the prefab |
| `NameOf(object)` | The hierarchy path of a scene object, or the name of an asset, for messages |

Write messages that say what goes wrong in game, not only what is missing: "the player cannot place pieces on other
pieces" helps more than "mask without layer 8". Use Info for choices the user may make on purpose.

Lines whose fix label starts with *Open* are treated as links and left out of **Fix All**.
