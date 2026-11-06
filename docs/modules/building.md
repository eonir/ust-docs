# Building

The Building module lets players build with a tool in the hand, such as a hammer: foundations, walls, floors, and any
prefab you turn into a piece, a chest or a workbench included. The owner sees a preview, green where the piece fits and
red where it does not, snapped to the pieces around. The server checks the place and the cost again, takes the
materials and spawns the piece. Built pieces are saved with the world.

The module is optional: add `UST_DISABLE_BUILDING` to the Scripting Define Symbols to remove it, see
[Modules](../architecture/modules.md#disabling-an-optional-module).

## Setup

1. *UST Tools > Create Game UI* adds the build menu and the build HUD. For a UI built before, run
   *UST Tools > Advanced > UI > Add Build Menu*.
2. Select the player prefab and run *UST Tools > Advanced > Building > Add Building To Selected Player*. It adds `PlayerBuilder`,
   uses the attack action to build, creates the transparent preview material, and adds `BuildRangeDisplay` and
   `SupportTint`.
3. *UST Tools > Advanced > Building > Create Demo Pieces* creates a foundation, a floor, a wall, a half wall, a pillar and a
   ward, their database and a **Hammer** item added to the Item Database. They cost wood when an item with "wood" in its name
   exists.
4. Run *UST Tools > Validation > Validate Network Prefabs*: every piece prefab must be a network prefab.
5. Run *UST Tools > Advanced > Building > Add Building To Scene*: it adds the **Building Rules** and the saver of built pieces.
6. Give yourself the Hammer, hold it and build.

## Controls

| Input | Action |
|---|---|
| Attack (left click) | Build the selected piece |
| Right click | Open or close the build menu |
| Mouse wheel | Rotate by **Rotation Step**. The camera does not zoom while a piece is previewed |
| T | Switch the snap mode: **Corners** or **Middle** |
| Hold Left Shift | Free placement: no snapping, the piece goes exactly where you aim |
| Middle click | Remove the built piece under the crosshair |
| R | Repair the damaged piece under the crosshair |

The HUD shows the selected piece, its cost as owned / needed, why it cannot be built here (*Too far*, *Something is
in the way*, *Missing materials*, *Nothing to build on*) and the controls.

### Build menu and HUD

*Create Game UI* or *UST Tools > Advanced > UI > Add Build Menu* builds both. Every text can be translated or restyled from the
inspector.

`BuildMenuWindow`, the menu opened with the right click:

| Field | Default | Meaning |
|---|---|---|
| Window | | Object shown and hidden |
| Tab List, Tab Prefab | | Parent of the category tabs, and the button created for each one |
| Piece List, Piece Button Prefab | | Parent of the piece buttons of the open tab, and the `BuildPieceButton` created for each piece |
| Piece Name, Piece Description, Piece Cost | | Texts of the selected piece. The cost lists each item as owned / needed |
| Enough Color, Missing Color | green, red | Color of a cost line the player can or cannot pay |
| Active Tab Color, Tab Color | orange, dark grey | Color of the open tab and of the others |
| Close Key | Escape | Closes the menu, as the right click does |
| Other Tab Name | Other | Tab of the pieces without a category, shown last |

`BuildPieceButton`, on the piece button prefab: **Button**, **Icon**, **Label**, an optional **Selection** object shown
on the selected piece, and an optional **Canvas Group** dimmed to **Unavailable Alpha** (0.45) when the player cannot
afford the piece.

`BuildHUDView`, the line above the hotbar while a build tool is held:

| Field | Default | Meaning |
|---|---|---|
| Root | | Object shown while building |
| Piece Text, Cost Text, Hint Text | | Selected piece, its cost, and the controls or the reason it cannot be built |
| Enough Color, Missing Color, Problem Color | green, red, red | Cost lines the player can or cannot pay, and the reason a piece cannot be built |
| Controls | LMB build   RMB menu   Wheel rotate | Fixed controls, shown after the snap mode |
| Snap Format | Snap: {0} ({1} to switch, hold {2} for free) | {0} the snap mode, {1} its key, {2} the free placement key |
| Remove Format | MMB remove {0} | Shown when the aimed piece can be removed. {0} its name |
| Repair Format | {0} repair | Shown when the aimed piece is damaged and can be repaired. {0} the repair key |
| Health Format | {0}  {1}/{2} | Health of the aimed piece: {0} name, {1} health, {2} max health |
| Support Format | Support {0}% | Support of the preview or of the aimed piece, in percent |
| Refresh Interval | 0.1 | Seconds between two refreshes of the texts while nothing changes. A new selection, target, problem or snap mode refreshes at once |

The snap, free placement and repair keys shown come from `PlayerBuilder`: change a key there and the HUD follows.
**Controls** is plain text: update it if you change the rotation input or the mouse buttons.

## Build piece

Add `BuildPiece` to the root of a network prefab.

| Field | Default | Meaning |
|---|---|---|
| Id | | Stable id used by saves, the GUID of the prefab. Set automatically. |
| Display Name, Icon, Description | | Shown in the build menu |
| Category | | Build Category asset: the tab of the build menu. Empty: an *Other* tab, shown last. |
| Cost | | Items taken from the inventory and the hotbar |
| Refund Ratio | 1 | Part of the cost given back when the piece is removed |
| Removable | on | Off for pieces players can never take down |
| Required Station | | Station type needed nearby, e.g. a Workbench. Empty: anywhere. See [Stations](#stations). |
| Snap Prefix | Snap | Children whose name starts with it are snap points |
| Needs Support | on | Must land on something or snap to a piece. Off: can float, for lights or signs. The server checks it too: a piece that touches nothing is refused. |
| Placement Offset | 0 | Added when the piece is not snapped, in its own space. A negative Y sinks a foundation into the ground. |
| Snap Priority | 0 | Snap points of pieces with a higher priority win near the aim. The demo pillar has 1, so a pillar joins the pillar under a floor rather than the floor. |

**Snap points** are empty children at the corners of the piece. Select a piece to see them as yellow spheres. The demo
pieces show the pattern: a flat piece has its 8 corners, a wall the 4 corners of its middle plane so it stands on the
edges of floors, and a pillar the centers of its two ends so it stands on their corners.

Two snap modes, switched in game:

- **Corners**: the snap points themselves.
- **Middle**: the middles of the edges between them, found automatically (two snap points that differ along a single
  axis). A wall centered on the middle of a foundation edge, a half wall on a pillar...

A snap point of a built piece is used when it is within **Snap Radius** of the aimed point, or of the aim ray itself:
aim past the edge of a floor, in the air, to place the next floor beside it, even upstairs on pillars. Every pair of
snap points gives a position. The one whose center is closest to where you aim, pushed out of the aimed face, and that
nothing blocks, wins. Snap points of the piece under the crosshair count first, however far from where you aim: aim at
the middle of the top of a wall to stack the next wall on it. Then come those of pieces with a higher **Snap
Priority**. Hold Shift to place freely.

Pieces may clip into each other by a quarter of their size on each axis, so a pillar can pass through the corner of a
floor. Anything else solid, and a real overlap with another piece, blocks the placement.

The space a piece takes is read from its colliders (box, sphere, capsule, mesh), or its meshes without colliders.
Anything solid inside that box blocks the piece, except the terrain and triggers. Pieces may touch their neighbours.

### Layers

Building does not depend on a layer: pieces are found by their `BuildPiece` component, and the preview has no
collider. You can put pieces on their own layer, for example for the camera Deoccluder. *UST Tools > Advanced > Building > Put Pieces On Building Layer* creates a **Building** layer when missing and moves every piece of every Build Piece Database to it,
children included. *Create Demo Pieces* does the same for the demo pieces. Then check these masks:

| Mask | Why |
|---|---|
| **Placement Layers** of `PlayerBuilder` | Must include the layer, or nothing can be built on or snapped to the pieces |
| Interact mask of `PlayerInteractor` | Must include it to use built wards, chests and workbenches |
| **Hit Layers** of `PlayerItemUser` and of weapons | Only if pieces should be hit |
| Cinemachine Deoccluder | In **Collide Against**, the camera is pushed by walls. In **Transparent Layers**, it goes through. |

!!! warning "Pieces made of several models"
    The layer must be set on **every object of the piece** that has a collider, not only on the root. A piece built
    from several models, or with its collider on a child like the demo pieces (*Model*), keeps the old layer on those
    children otherwise: the Deoccluder or the placement mask then ignore part of the piece. When changing the layer of
    the root, answer **Yes, change children**.

## Building rules

`BuildingRules`, one per scene, sets the rules of the session. Without it the defaults apply.

| Field | Default | Meaning |
|---|---|---|
| Pieces Take Damage | on | Built pieces have health and can be destroyed |
| Player Damage | on | Players can damage built pieces, for raids |
| Enemy Damage | on | Enemies and any non player attacker can damage built pieces |
| Ward Blocks Player Damage | off | Players cannot damage pieces inside a ward they are not part of |
| Default Hit Damage | 5 | Damage of a hit by an item without a Structure Damage trait, when the hit carries no damage |
| Projectile Damage Multiplier | 0.25 | Multiplies projectile damage on pieces: weapon damage is made for characters |
| Destroyed Refund Ratio | 0.5 | Part of the cost dropped on the ground when a piece is destroyed or collapses |
| Structural Support | on | Pieces with a Piece Support need support, and collapse without it. Off: everything holds. |
| Support Check Radius | 30 | Distance around a built or removed piece where support is computed again |

Typical setups: PvE with **Player Damage** off; PvP raids with everything on; a peaceful builder with **Pieces Take
Damage** off.

## Piece health

Add `PieceHealth` to a build piece to make it destructible. The demo pieces have one.

| Field | Default | Meaning |
|---|---|---|
| Max Health | 200 | |
| Damage Multiplier | 1 | Multiplies all damage taken: 0.5 for stone, 2 for thatch |
| Resistances | | By damage type, on top of the multiplier: stone takes little from Slash, a lot from Blunt. See [Damage types](combat.md#damage-types). |

### How much damage a hit does

Every hit on a piece goes through three steps:

```
base damage
  1. the item has the Building > Structure Damage trait  ->  damage of the trait
  2. else, a projectile (bullet, bolt)                    ->  damage of the weapon
  3. else, a melee hit                                    ->  Default Hit Damage (Building Rules)

x Projectile Damage Multiplier (Building Rules)   projectiles only
x Damage Multiplier (Piece Health)                 the material of the piece
x Resistance to the damage type (Piece Health)     the type of the weapon or the attack
```

| Setting | Where | Scope |
|---|---|---|
| **Structure Damage** trait | On the item, *Add Trait > Building > Structure Damage* | That item only: a sledgehammer at 60, a sword at 5 |
| **Default Hit Damage** | Building Rules | Every melee item without the trait |
| **Projectile Damage Multiplier** | Building Rules | Every projectile, with or without the trait |
| **Damage Multiplier** | Piece Health | That piece: 0.5 for stone, 2 for thatch |

Example, with the defaults: a pistol dealing 25 to characters deals 25 x 0.25 = 6.25 to a wooden wall, and
3.1 to a stone wall at 0.5. A sledgehammer with Structure Damage 60 deals 60 to the wooden wall.

!!! tip "Give your tools the trait"
    The trait is how you make raiding tools: axes and hammers that break walls, weapons that barely scratch them.
    Without it, every melee item deals the same Default Hit Damage to pieces.

Damage also comes from anything else through `IDamageable.ServerTakeDamage(amount, source, attackerClientId)` in
UST.Core: enemies, traps, fire. The amount is used as the base damage, then multiplied by the Damage Multiplier of the
piece. An enemy hitting with `ItemHit` counts as an enemy, since its attacker is not a player. Melee and projectile hits
need the **Hit Layers** of the weapon to see the pieces.

At 0 health the piece is destroyed: its removal handlers run, its containers spill, and **Destroyed Refund Ratio** of
its cost drops on the ground. Health is saved with the piece, and the build HUD shows the health of the aimed piece.
Pieces placed in the scene never take damage. With a build tool in the hand, **R** repairs the aimed piece, for free,
like in Valheim. `ServerRepair(amount)` gives health back from your own code.

## Structural support

Add `PieceSupport` to a build piece to make it need support, like in Valheim. The demo pieces have one.

- A piece touching the ground has **full support** (100 %). The ground is anything solid that is not a build piece and
  does not move: terrain, rocks, scene buildings. Pieces placed in the scene by you also count as ground.
- Any other piece gets the best support of the pieces it touches, **minus a loss**: small when held from below
  (**Vertical Loss**), larger when held from the side (**Horizontal Loss**), and in between for diagonals.
- Under **Collapse Below**, the piece breaks: it drops **Destroyed Refund Ratio** of its cost and the content of its
  containers, like a destroyed piece.

| Field | Default | Meaning |
|---|---|---|
| Horizontal Loss | 0.2 | Support lost when held from the side. 0.2: four floors out from a pillar. |
| Vertical Loss | 0.125 | Support lost when held from below. 0.125: about seven levels high. |
| Collapse Below | 0.1 | The piece breaks under this support |

The loss is the one of the piece being held, so its material sets how far it reaches: a stone floor at 0.1 spans
twice as far as wood. Pieces without `PieceSupport` (furniture, decorations) never collapse and hold nothing.

In game, with the build tool:

- the preview takes the color of its support on the **Support Colors** gradient of `PlayerBuilder`: red just above
  the collapse limit, then orange, yellow, green, and blue on the ground. It turns to **Invalid Color** with *Not
  enough support*;
- the HUD shows the support of the preview and of the aimed piece.
- `SupportTint`, on the player, tints the built pieces with the same colors, so the weak point of a base shows before
  a pillar comes down.

| Field | Default | Meaning |
|---|---|---|
| Mode | Nearby | **None**, **Aimed Piece** only, or every piece within **Radius** |
| Radius | 30 | |
| Strength | 0.6 | How much of the support color replaces the color of the piece |
| Refresh Interval | 0.25 | Seconds between two updates of the tint |

The tint uses a property block on `_BaseColor` (URP) and `_Color` (Built-in), and is removed as soon as the build tool
leaves the hand. A shader without these properties is not tinted.

Removing or destroying a piece computes the support around it again on the next frame. Pieces left without support
collapse, which computes their neighbours again: an overhang falls apart piece by piece, over a few frames. Support is
computed by the server only, and computed again for every piece after loading a save.

## Stations

Each piece decides: a piece with a **Required Station** (field of `BuildPiece`) can only be built within the **Build
Range** of a station of that type (20 by default, on `CraftingStation`), and a piece without one can be built
anywhere. Wood pieces can need a Workbench and stone pieces a Forge, for example. The preview says *Needs a Workbench nearby*. Station types are the Crafting station
types, so a workbench both crafts and allows building. A built workbench works too: build it first, it should need no
station itself.

Your own stations can count: implement `IStation` and register it in `Stations`, in UST.Core.

## Range display

`BuildRangeDisplay`, on the player, shows while a build tool is in the hand:

- the area of every active ward nearby, blue when you are part of it, red otherwise;
- the build range of the stations the selected piece needs, in yellow. Nothing for a piece without a Required
  Station;
- the future area of a ward you are about to place, to see at once if it overlaps another one.

| Field | Default | Meaning |
|---|---|---|
| Mode | Ring | **None**, **Ring** on the ground, **Dome**, or **Ring And Dome** |
| Max Distance | 80 | Wards and stations further away are not shown |
| Own Ward Color, Foreign Ward Color, Station Color | blue, red, yellow | |
| Ring Material | M_RangeRing | Tinted by the colors |
| Ring Width | 0.08 | |
| Segments | 96 | Points of the ring |
| Ground Layers | Everything | The ring follows these layers: terrain, floors, foundations |
| Dome Material | M_RangeDome | Uses the **UST/Range Dome** shader: faint inside, brighter on the edge |
| Dome Alpha | 0.35 | |

The ring costs almost nothing and follows the ground. The dome is a large transparent surface: it covers the whole
screen when you stand inside, so it costs more on low end hardware and hides a bit of the view. The UST/Range Dome
shader works in the Built-in pipeline and URP; in HDRP, give the dome your own transparent material.

## Build categories

*Create > UST > Building > Build Category*: one asset per tab of the build menu.

| Field | Meaning |
|---|---|
| Display Name | Name of the tab. Empty: the asset name. |
| Icon | For your own build menu |
| Order | Tabs are sorted by it, then by name |

Assets rather than text: a typo cannot hide a piece, and renaming a category changes nothing else. *Create Demo
Pieces* creates **Structure** and **Utility**. The Item Database warns about a build tool whose categories match no
piece of its database, or with an empty entry.

## Build Piece Database

*Create > UST > Building > Build Piece Database*. Lists the pieces of a tool, and travels over the network as indices:
every peer must use the same one. Right click its header > **Collect All Pieces In Project** to fill it.

## Build tool

Give an item the **Building > Build Tool** trait.

| Field | Default | Meaning |
|---|---|---|
| Pieces | | Build Piece Database of the tool |
| Categories | | Build Category assets offered. Empty: all. A hammer for structures and a cooking tool for kitchens can share a database. |
| Range | 8 | Max distance to build or remove |
| Wear Per Piece | 1 | Durability used per piece built, for items with durability. 0 never wears. |

## Player Builder

| Field | Default | Meaning |
|---|---|---|
| Place Action | | Builds. Set to the attack action by the setup tool. |
| Remove Action | | Optional. Without it, middle click. |
| Menu Action | | Optional. Without it, right click. |
| Wheel Rotates | On | Rotates the preview with the mouse wheel. While a piece is previewed, the wheel is claimed: the camera zoom and the hotbar ignore it |
| Rotate Action | | Optional mouse wheel action (Vector2). Without it, the mouse is read directly |
| Invert Wheel | Off | Reverses the rotation direction |
| Wheel Threshold | 0.1 | Minimum wheel movement |
| Rotation Step | 22.5 | Degrees per wheel notch |
| Repair Key | R | Repairs the aimed piece, inside wards you are part of |
| Repair Amount | 1 | Health given back per repair, as a part of the max health. 1 repairs fully, 0.25 needs four presses. |
| Repair Cooldown | 0.5 | Seconds between two repairs |
| Placement Layers | Everything | Layers the preview lands on, and that can block a piece |
| Snap Radius | 0.75 | Distance from the aim under which a snap point of a built piece is used |
| Snap Mode Key | T | Switches between Corners and Middle |
| Free Placement Key | Left Shift | Held: no snapping |
| Server Range Tolerance | 1.5 | Extra distance accepted by the server, for latency |
| Preview Material | | Transparent material tinted with the colors below |
| Valid Color / Invalid Color | green / red | Valid Color is used for pieces without a Piece Support |
| Support Colors | red to blue | Color of a valid preview by support, for pieces with a Piece Support. Left: just above Collapse Below. Right: on the ground. |

## Removing a piece

Middle click on a built piece in range, outside of any ward you are not part of. On the server:

1. every `IRemovalHandler` of the piece runs: a built workbench gives back the ingredients of its queued crafts;
2. the content of each `InventoryContainer` of the piece is dropped: a chest spills its items. Items without a world
   prefab are lost;
3. **Refund Ratio** of the cost goes to the player, and what does not fit is dropped;
4. the piece is despawned.

Pieces placed in the scene by you are never removed by players.

## Ward

Without a ward, anyone can take down any built piece and open any built chest, as in Valheim. A **ward** is a piece
that protects the area around it. Add `BuildWard` to a build piece; the demo **Ward** is a post with a glowing orb.

While the ward is on, inside its **Radius**, only its owner and the players they let in can:

- build, and take pieces down,
- open storages and move items in or out of any world container,
- use crafting stations, and take their output.

Loot bags stay open to everyone. Player inventories are never affected.

| Who | Interact with the ward |
|---|---|
| Owner, the player who built it | Turns it on or off |
| Another player, ward off | Joins the ward, or leaves it |
| Another player, ward on | "This area is protected" message |

So the owner turns the ward off, friends interact with it to join, and the owner turns it on again. A ward cannot be
built where its area would overlap the active ward of another player, and the preview says *Too close to another
ward*. Building inside a ward you are not part of shows *Protected by a ward*.

| Field | Default | Meaning |
|---|---|---|
| Radius | 15 | Size of the protected sphere |
| Active Visual | | Optional object shown while the ward is on |
| Turn On Prompt, Turn Off Prompt | Turn on ward, Turn off ward | Interaction text for the owner, or anyone while the ward is off |
| Join Prompt, Leave Prompt | Join ward, Leave ward | Interaction text for a player who is not, or is, part of the ward |
| Protected Prompt | Ward: turn it off to join | Shown to other players while the ward is on |

The owner and the joined players are remembered by their save id (`IPlayerDataSource.PlayerKey`), so the ward keeps
them across sessions. Without the Save module, the session client id is used and is lost on restart.

Your own systems can respect wards: `ProtectionZones.IsAllowed(clientId, position)` in UST.Core, and implement
`IProtectionZone` for other kinds of protected areas (a safe zone around a spawn, a claim flag...).

## Chests, workbenches and other functional pieces

Any prefab becomes buildable: add `BuildPiece` to a storage or a crafting station prefab and list it in the database.

- A built **chest** (`StorageContainer`) saves its content with the piece.
- A built **workbench** (`CraftingStation`) saves its queue and its output with the piece, and gives the ingredients
  of its queue back when it is removed.

Your own components save with the piece by implementing `IObjectSaveable`, and react to removal with
`IRemovalHandler`, both in UST.Core.

## Save

`BuildPiecesSaver`, in the scene, saves every piece built at runtime: which piece, its position and rotation, and the
state of its `IObjectSaveable` components. Give it the databases of all your build tools: a piece missing from them
is skipped with a warning when loading.

## Scripting

```csharp
// Server side: a piece was built, for a quest or a log.
PlayerBuilder.ServerPieceBuilt += (piece, clientId) => Debug.Log(piece.DisplayName + " built by " + clientId);

// Server side: take a piece down, refunding nobody.
PlayerBuilder.ServerRemove(piece, null);
```

## Not in this version

- Physics on collapse: broken pieces disappear, they do not fall.
