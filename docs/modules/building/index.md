# Building

The Building module lets players build with a tool in the hand, such as a hammer: foundations, walls, floors, and any
prefab you turn into a piece, a chest or a workbench included. The owner sees a preview, green where the piece fits and
red where it does not, snapped to the pieces around. The server checks the place and the cost again, takes the
materials and spawns the piece. Built pieces are saved with the world.

The module is optional: add `UST_DISABLE_BUILDING` to the Scripting Define Symbols to remove it, see
[Modules](../../architecture/modules.md#disabling-an-optional-module).

This module is described on several pages:

- [Build pieces](pieces.md)
- [Rules, health and support](rules.md)
- [Ward](ward.md)

## Setup

In the [Setup Window](../../getting-started/setup-window.md), **Building > Add** does steps 1, 2 and 5. By hand:

1. *Tools > UST > Create Game UI* adds the build menu and the build HUD. For a UI built before, run
   *Tools > UST > Advanced > UI > Add Build Menu*.
2. *Tools > UST > Setup Player Prefab*, or *Tools > UST > Advanced > Building > Add Building To Selected Player* on
   the player prefab. It adds `PlayerBuilder`, uses the attack action to build, creates the transparent preview
   material, and adds `BuildRangeDisplay` and `SupportTint`.
3. *Tools > UST > Advanced > Building > Create Demo Pieces* creates a foundation, a floor, a wall, a half wall, a pillar and a
   ward, their database and a **Hammer** item added to the Item Database. They cost wood when an item with "wood" in its name
   exists.
4. Run *Tools > UST > Validation > Validate Network Prefabs*: every piece prefab must be a network prefab.
5. Run *Tools > UST > Advanced > Building > Add Building To Scene*: it adds the **Building Rules** and the saver of built pieces.
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

*Create Game UI* or *Tools > UST > Advanced > UI > Add Build Menu* builds both. Every text can be translated or restyled from the
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
| Controls | {0} build   {1} menu   {2} rotate | Shown after the snap mode. {0} the build key, {1} the menu key, {2} the rotation keys (*Wheel* with a mouse) |
| Snap Format | Snap: {0} ({1} to switch, hold {2} for free) | {0} the snap mode, {1} its key, {2} the free placement key |
| Remove Format | {1} remove {0} | Shown when the aimed piece can be removed. {0} its name, {1} the remove key |
| Repair Format | {0} repair | Shown when the aimed piece is damaged and can be repaired. {0} the repair key |
| Health Format | {0}  {1}/{2} | Health of the aimed piece: {0} name, {1} health, {2} max health |
| Support Format | Support {0}% | Support of the preview or of the aimed piece, in percent |
| Refresh Interval | 0.1 | Seconds between two refreshes of the texts while nothing changes. A new selection, target, problem or snap mode refreshes at once |

Every key shown comes from the actions of `PlayerBuilder`, for the device in use and the keys the player chose in
*Options > Keys*: on a gamepad the line reads *RT build   D-pad down menu   D-pad left/D-pad right rotate*.

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
| Rotation Step | 22.5 | Degrees per wheel notch, or per press of a rotate action |
| Rotate Left Action, Rotate Right Action | Player/RotateLeft, Player/RotateRight | Optional. Turn the preview by one step. Gamepad: d-pad left and right. |
| Repair Key, Repair Action | R, Player/BuildRepair | Repairs the aimed piece, inside wards you are part of. The action replaces the key when assigned. Gamepad: X. |
| Repair Amount | 1 | Health given back per repair, as a part of the max health. 1 repairs fully, 0.25 needs four presses. |
| Repair Cooldown | 0.5 | Seconds between two repairs |
| Placement Layers | Everything | Layers the preview lands on, and that can block a piece |
| Snap Radius | 0.75 | Distance from the aim under which a snap point of a built piece is used |
| Snap Mode Key, Snap Mode Action | T, Player/BuildSnapMode | Switches between Corners and Middle. Gamepad: d-pad up. |
| Free Placement Key, Free Placement Action | Left Shift, Player/BuildFreePlacement | Held: no snapping. Gamepad: left trigger. |
| Server Range Tolerance | 1.5 | Extra distance accepted by the server, for latency |
| Preview Material | | Transparent material tinted with the colors below |
| Valid Color / Invalid Color | green / red | Valid Color is used for pieces without a Piece Support |
| Support Colors | red to blue | Color of a valid preview by support, for pieces with a Piece Support. Left: just above Collapse Below. Right: on the ground. |

## Build tool

Give an item the **Building > Build Tool** trait.

| Field | Default | Meaning |
|---|---|---|
| Pieces | | Build Piece Database of the tool |
| Categories | | Build Category assets offered. Empty: all. A hammer for structures and a cooking tool for kitchens can share a database. |
| Range | 8 | Max distance to build or remove |
| Wear Per Piece | 1 | Durability used per piece built, for items with durability. 0 never wears. |

## Removing a piece

Middle click on a built piece in range, outside of any ward you are not part of. On the server:

1. every `IRemovalHandler` of the piece runs: a built workbench gives back the ingredients of its queued crafts;
2. the content of each `InventoryContainer` of the piece is dropped: a chest spills its items. Items without a world
   prefab are lost;
3. **Refund Ratio** of the cost goes to the player, and what does not fit is dropped;
4. the piece is despawned.

Pieces placed in the scene by you are never removed by players.

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
